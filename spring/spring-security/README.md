---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
---

# Spring Security

> 이 내용은 Spring Security Architecture 공식 문서입니다. Servlet을 기반으로하는 어플리케이션의 스프링 시큐리티 아키텍쳐에 대한 내용을 다루고 있습니다.

## Architecture

### Review of Filters

스프링 시큐리티의 Servlet은 Servlet Filter 레벨을 기반으로 한다. 아래 이미지는 HTTP request에 대한 핸들러의 일반적인 구조를 나타낸다.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

클라이언트는 어플리케이션의 request를 보내고 컨테이너는 Filter와 Servlet을 포함하는 FilterChain을 생성한다. 이 때 request URI 경로에 기반해 HttpServletRequest를 처리한다. 스프링 MVC 어플리케이션에서 Servlet은 DispatcherServlet의 구현체이다.

하나의 Servlet은 기껏해야 1개의 HttpServletRequest와 HttpServletResponse를 다룬다. 그러나 2개 이상의 Filter를 사용하여 다음 작업을 수행할 수 있다.

* 다음 Filter가 호출되지 않도록 한다. 이 경우 Filter는 HttpServletResponse를 작성한다.
* 다음 Filter와 Servlet에서 사용되는 HttpServletRequest 또는 HttpServletResponse를 수정한다.

Filter의 효과는 이를 통과하는 FilterChain에 있다.

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
		chain.doFilter(request, response);
}
```

하나의 Filter는 다음 Filter 구현체와 Servlet에만 영향을 줄 수 있다. 따라서 각각의 Filter가 사용되는 순서는 상당히 중요하다.

### DelegatingFilterProxy

스프링은 `DelegatingFlterProxy` 를 제공한다. 이는 Servlet 컨테이너와 스프링의 `Application Context` 사이를 연결해준다. Servlet 컨테이너는 Servlet 스펙이기 때문에 스프링에서 정의된 빈을 주입받아 사용할 수 없다.

이 때 DelegatingFilterProxy를 통해 스프링 컨테이너에서 존재하는 특정 Bean을 찾아 요청을 위임한다.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### FilterChainProxy

`FilterChainProxy` 는 스프링 시큐리티에 의해 제공되는 특별한 필터이다. SecurityFilterChain을 통해 많은 필터 인스턴스를 위임한다.FilterChainProxy도 Bean이므로 DelegatingFilterProxy로 wrapping된다.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### SecurityFilterChain

SecurityFilterChain은 현재 request에서 어떤 스프링 시큐리티 Filter 인스턴스가 사용되어야 할지 결정하기 위해 FilterChainProxy에 의해서 사용된다.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

`SecurityFilterChain` 안의 Security Filter들은 일반적으로 Bean이다. 그러나 DelegatingFilterProxy 대신에 `FilterChainProxy` 로 등록된다.

`FilterChainProxy` 는 Servlet 컨테이너나 DelegatingFilterProxy에 직접적으로 등록하는 것에 많은 이점을 제공한다.

1. 스프링 시큐리티 Servlet의 시작점을 제공한다. ⇒ trouble shooting에 용이
2. 보이지 않는 작업을 수행할 수 있다. (HttpFireWall 등)
3.  SecurityFilterChain이 invoke되는 시점을 결정하는 데에 유연성을 제공한다.

    URL을 기반으로 동작하는데, RequestMathcer interface를 사용함으로써 invocation을 결정할 수 있다.

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

위 그림에서 `FilterChainProxy` 가 어떤 SecurityFilterChain이 사용될지 결정한다. 부합하는 첫번째 `SecurityFilterChain` 만 적용된다. 만약 `/api/message/` 가 요청된다면, `/api/**` 패턴의 $$SecurityFilterChain_0$$이 매치된다. 따라서 비록$$SecurityFilterChain_n$$도 매치될지라도, $$SecurityFilterChain_ 0$$ 이 실행된다.

만약 `/essages/` URL이 요청된다면 $$SecurityFilterChain_0$$과는 매치되지 않는다. 따라서 `FilterChainProxy` 는 각각의 `SecurityFiterChain` 을 시도해보다가 결국 $$SecurityFilterChain_n$$이 적용된다.

$$SecurityFilterChain_0$$은 3개의 Filter 인스턴스로 구성되어있고, $$SecurityFilterChain_n$$은 4개의 필터 인스턴스로 구성되어 있다. 각각의 `SecurityFilterChain` 은 특별하고 독립적으로 구성될 수 있다. 사실 `SecurityFilterChain` 은 만약 어플리케이션이 Spring Security가 특정 request를 무시하길 원한다면, Filter 인스턴스를 가지지 않게 할 수도 있다.

### Security Filters

Security Filter는 SecurityFilterChain API와 함께 FilterChainProxy에 삽입된다. 이 Filter들은 인증, 인가 등 다양한 목적으로 사용될 수 있다. Filter는 원하는 순서로 실행되기 위해 특정한 순서로 실행된다. 예를 들어 **인증을 수행하는 Filter는 인가를 수행하는 Filter 전에 실행되어야 한다.** Spring Security의 Filter들의 순서를 아는 것은 일반적으로 필요한 것은 아니지만, 순서를 아는 것이 도움이 되는 때가 있다.

[https://github.com/spring-projects/spring-security/blob/6.3.1/config/src/main/java/org/springframework/security/config/annotation/web/builders/FilterOrderRegistration.java](https://github.com/spring-projects/spring-security/blob/6.3.1/config/src/main/java/org/springframework/security/config/annotation/web/builders/FilterOrderRegistration.java)

security configuration의 한가지 예를 들어보자.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
		@Bean
		public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
				http
						.csrf(Customizer.withDefaults())
						.authorizeHttpRequest(authorize -> authorize
								.anyRequest().authenticated()
						)
						.httpBasic(Customizer.withDefaults())
						.formLogin(Customizer.withDefaults());
				return http.build();
		}
}
```

위 Configuration은 다음의 Filter 순서를 가진다.

* **CsrfFilter** ← `HttpSecurity#csrf`
  * `CsrfFilter` 는 CSRF 공격을 막기 위해 실행된다.
* **UsernamePasswordAuthenticationFilter** ← `HttpSecurity#formLogin`
* **BasicAuthenticationFilter** ← `HttpSecurity#httpBasic`
  * request를 authenticate하기 위해 실행된다.
* **AuthorizationFilter** ← `HttpSecurity#authorizeHttpRequests`
  * `AuthorizationFilter` 는 request를 authorize 하기 위해 실행된다.

위의 나와있는 Filter 외에 다른 Filter 인스턴스가 있을 수 있다. 특정 request에 실행되는 Filter 리스트를 보고 싶다면, print 해서 볼 수 있다.

### Printing the Security Filters

가끔은 특정 request에 실행되는 Security Filter 리스트를 보는게 유용할 때도 있다. 예를 들어 추가한 Filter가 확실히 실행되는지 확인하고 싶다면 확인하면 된다.

Filter 리스트는 어플리케이션이 실행될 때 INFO 레벨에서 프린트된다. 따라서 다음과 같이 콘솔에서 확인할 수 있다.

```
2023-06-14T08:55:22.321-03:00  INFO 76975 --- [           main] o.s.s.web.DefaultSecurityFilterChain     : Will secure any request with [
org.springframework.security.web.session.DisableEncodeUrlFilter@404db674,
org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@50f097b5,
org.springframework.security.web.context.SecurityContextHolderFilter@6fc6deb7,
org.springframework.security.web.header.HeaderWriterFilter@6f76c2cc,
org.springframework.security.web.csrf.CsrfFilter@c29fe36,
org.springframework.security.web.authentication.logout.LogoutFilter@ef60710,
org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@7c2dfa2,
org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@4397a639,
org.springframework.security.web.authentication.ui.DefaultLogoutPageGeneratingFilter@7add838c,
org.springframework.security.web.authentication.www.BasicAuthenticationFilter@5cc9d3d0,
org.springframework.security.web.savedrequest.RequestCacheAwareFilter@7da39774,
org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@32b0876c,
org.springframework.security.web.authentication.AnonymousAuthenticationFilter@3662bdff,
org.springframework.security.web.access.ExceptionTranslationFilter@77681ce4,
org.springframework.security.web.access.intercept.AuthorizationFilter@169268a7]
```

또는 직접 logging을 이용해서 확인할 수도 있다.

### Adding a Custom Filter to the Filter Chain

대부분 기본 Security Filter들은 어플리케이션에 security를 제공하기에 충분하다. 하지만 Custom Filter를 security filter chain에 추가하고 싶을 때가 존재할 것이다.

예를 들어, tenant id header를 가져가서 현재 유저가 해당 tenant에 접근을 가지고 있는지 확인하는 Filter를 추가하고 싶다고 해보자. 우리는 현재 유저를 알 필요가 있기 때문에 어디에 Filter를 추가해야 할지 알 수 있다. 우리는 인증 Filter 뒤에다가 추가해야 한다.

```java
public class TenantFilter implements Filter {
		@Override
		public void doFilter(ServletRequest servletReqest, ServletResponse servletresponse, FilterChain filterChain) throws IOException, ServletException {
				HttpServletRequest request = (HttpServletRequest) servletRequest;
				HttpServletResponse = response = (HttpServletResponse) servletResponse;
				String tenantId = request.getHeader("X-Tenant-Id");
				boolean hasAccess = isUserAllowed(tenantId);
				if (hasAccess) {
						filterChain.doFilter(request, response);
						return;
				}
				throw new AccessDeniedException("Access denied");
		}
}
```

샘플 코드는 다음과 같이 동작한다.

1. request header로부터 tenant id를 얻는다.
2. 현재 유저가 tenant id에 접근 권한이 있는지 검사한다.
3. 접근 권한을 가지고 있다면 나머지 필터를 실행한다.
4. 접근 권한이 없다면 `AccessDeniedException` 을 던진다.

> `Filter` 를 implements 하는 대신, `OncePerRequestFilter` 를 extends 할 수도 있다. `OncePerRequestFilter` 는 filter의 base class이고, request 당 1번만 실행된다. 그리고 `HttpServletRequest` 와 `HttpServletResponse` 를 파라미터로 받는 `doFilterInternal` 메소드를 제공한다.

이제, Filter를 security filter chain에 적용해보자.

```java
@Bean
SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
		http
				// ...
				.addFilterBefore(new TenantFilter(), AuthorizationFilter.class);
		return http.build();
}
```

AuthorizationFilter 전에 TenantFilter를 추가하기 위해 `HttpSecurity#addFilterBefore` 를 사용한다.

이 필터를 `AythorizationFilter` 전에 추가함으로써 `TenantFilter` 가 authentication 필터 뒤에 실행하게 했다. 또한 특정 필터 뒤에 새로운 필터를 추가하기 위해 `HttpSecurity#addFilterAfter` 를 사용하거나, 특정 필터 포지션에 새로운 핉러를 추가하기 위해 `HttpSecurity#addFilterAt` 을 사용할 수 있다.

커스텀 필터를 스프링 빈으로 선언할 때 주의해야 한다. 필터에 `@Component` 어노테이션을 사용하거나 configuration에 빈으로 선언하거나 둘 중에 하나만 사용해야 한다. 스프링 부트는 자동적으로 이것을 내장된 컨테이너에 등록한다. 따라서 필터가 컨테이너에 의해 1번, 스프링 시큐리티에 의해서 1번 총 2번 invoke될 수 있다.

만약 DI의 이점을 얻기 위해 Spring Bean으로 필터를 등록하면서 중복 invocation을 피하려면, `FilterRegistrationBean` 으로 선언하고 `enabled` 을 false로 설정함으로써 스프링 부트가 필터를 컨테이너에 등록하지 않게 할 수 있다.

```java
@Bean
public FilterRegistrationBean<TenantFilter> tenantFilterRegistration(TenantFilter filter) {
		FilterRegistrationBean<TenantFilter> registration = new FilterRegistrationBean<>(filter);
		registration.setEnabled(false);
		return registration;
}
```

### Handling Security Exceptions

`ExceptionTranslationFilter` 는 `AccessDeniedException` 과 `AuthenticationException` 을 HTTP response 으로 변환할 수 있다. `ExceptionTranslationFilter` 는 Security Filter 중 하나로 FilterChainProxy에 삽입된다.

아래 이미지는 `ExceptionTranslationFilter` 와 다른 컴포넌트와의 관계를 보여준다.

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

1. `ExceptionTranslationFilter` 가 어플리케이션의 나머지 부분을 실행하기 위해 `FilterChain.doFilter(request, response)` 를 실행한다.
2. 만약 유저가 인증되지 않았거나 `AuthenticationException` 인 경우, Authentication을 시작한다.
   1. SecurityContextHolder가 초기화된다.
   2. `HttpServletRequest` 는 인증이 성공하면 original request를 replay하는데 사용할 수 있도록 저장된다.
   3. `AutenticationEntryPoint` 는 클라이언트로부터 credential을 요청하기 위해 사용된다. 예를 들어 로그인 페이지로 리다이렉트하거나, `WWW-Authenticate` 헤더를 보낼 수 있다.
3. `AccessDeniedException` 인 경우, 접근을 제한한다. `AccessDeniedHandler` 가 접근을 제한하기 위해 실행된다.

> 만약 어플리케이션이 `AccessDeniedException` 이나 `AuthenticationException` 을 일으키지 않는다면, `ExceptionTranslationFilter` 는 아무것도 하지 않는다.

`ExceptionTranslationFilter` 의 의사코드는 다음과 같이 나타낼 수 있다.

```java
try {
		filterChain.doFilter(request, response); // 1
} catch (AccessDeniedException | AuthenticationException ex) {
		if (!autenticated || ex instanceof AuthenticationException) {
				startAuthentication(); // 2
		} else {
				accessDenied(); // 3
		}
}
```

1. `FilterChain.doFilter(request, response)` 는 어플리케이션의 나머지 부분을 실행하는 것과 같다. 만약 어플리케이션의 나머지 부분이 `AuthenticationException` 또는 `AccessDeniedException` 을 발생시킨다면, 여기서 처리된다.
2. 만약 유저가 인증되지 않았거나 `AuthenticationException` 인 경우, Authentication을 시작한다.
3. 그렇지 않다면, 접근을 제한한다.

### Saving Requests Between Authentication

request가 authentication을 가지고 있지 않고 authentication을 요구하는 리소스일 때, 인증이 성공한 후 re-request하기 위해 인증된 리소스에 대한 요청을 저장할 필요가 있다. 스프링 시큐리티에서 이 것은 `RequestCache` 구현체를 사용하는 `HttpServletRequest` 를 저장함으로써 이뤄진다.

#### RequestCache

`HttpServletRequest` 는 `RequestCache` 에 저장된다. 유저가 성공적으로 인증되었을 때, `RequestCache` 는 원래의 request를 replay한다. `RequestCacheAwareFilter` 는 유저가 인증된 후 저장된 `HttpServletRequest` 를 얻기 위해 `RequestCache` 를 사용한다. 반면에 `ExceptionTranslationFilter` 는 유저를 로그인 endpoint로 리다이렉트 하기 전이나 `AuthenticationException` 을 발견한 후에 `HttpServletRequest` 를 저장하기 위해 `RequestCache` 를 사용한다.

기본적으로, `HttpSessionRequestCache` 가 사용된다. 아래 코드는 `RequestCache` 구현체를 커스텀하는 방법에 대해 설명한다. 저장된 request에 `continue` 라는 이름의 파라미터가 존재하는지 아닌지 `HttpSession` 을 체크한다.

```java
@Bean
DefaultSecurityFilterChain springSecurity(HttpSecurity http) throws Exception {
		HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
		requestCache.setMatchingRequestParameterName("continue");
		http
				// ...
				.requestCache((cache) -> cache
						.requestCache(requestCache)
				;
		return http.build();
}
```

#### Prevent the Request From Being Saved

유저의 인증되지 않은 request를 세션에 저장하고 싶지 않은 여러가지 이유가 있을 것이다. 저장 내용을 유저의 브라우저로 저장시키고 싶거나 데이터베이스에 저장시키고 싶을 수 있다. 또는 로그인 하기 전 유저가 방문하려한 페이지 대신에 항상 홈 페이지로 리다이렉션을 원하면 이 기능을 차단할 수 있다.

이를 위해서 `NullRequestCache` 구현체를 사용할 수 있다.

```java
@Bean
SecurityFilterChain springSecurity(HttpSecurity http) throws Exception {
		RequestCache nullRequestCache = new NullRequestCache();
		http
				// ..
				.requestCache((cache) -> cache
						.requestCache(nullRequestCache)
				);
		return http.build();
}
```

#### RequestCacheAwareFilter

`RequestCacheAwareFilter` 는 original request를 replay 하기 위해 `RequestCache` 를 사용한다.

### Logging

스프링 시큐리티는 관련된 모든 시큐리티 이벤트에 대해 DEBUG와 TRACE 레벨로 포괄적인 로깅을 제공한다. 이는 보안을 위해 스프링 시큐리티가 request가 거부된 이유를 response body에 담지 않기 때문에 유용할 수 있다. 만약 401 에러나 403 에러를 만난다면, 어떻게 진행되고 있는지 이해하는 것을 돕는 로그 메세지를 찾는 것이 좋다.

한 유저가 POST requst를 CSRF 토큰 없이 CSRF 프로텍션이 활성화된 리소스에 POST request를 시도하는 예를 들어보자. 로그없이 사용자는 왜 request가 거절됐는지에 대한 이유에 대한 설명 없이 403 에러를 마주한다. 그러나 만약 스프링 시큐리티에 대해 로깅을 적용한다면, 다음과 같은 메세지를 볼 수 있다.

```
2023-06-14T09:44:25.797-03:00 DEBUG 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Securing POST /hello
2023-06-14T09:44:25.797-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking DisableEncodeUrlFilter (1/15)
2023-06-14T09:44:25.798-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking WebAsyncManagerIntegrationFilter (2/15)
2023-06-14T09:44:25.800-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking SecurityContextHolderFilter (3/15)
2023-06-14T09:44:25.801-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking HeaderWriterFilter (4/15)
2023-06-14T09:44:25.802-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.security.web.FilterChainProxy        : Invoking CsrfFilter (5/15)
2023-06-14T09:44:25.814-03:00 DEBUG 76975 --- [nio-8080-exec-1] o.s.security.web.csrf.CsrfFilter         : Invalid CSRF token found for <http://localhost:8080/hello>
2023-06-14T09:44:25.814-03:00 DEBUG 76975 --- [nio-8080-exec-1] o.s.s.w.access.AccessDeniedHandlerImpl   : Responding with 403 status code
2023-06-14T09:44:25.814-03:00 TRACE 76975 --- [nio-8080-exec-1] o.s.s.w.header.writers.HstsHeaderWriter  : Not injecting HSTS header since it did not match request to [Is Secure]
```

CSRF 토큰이 없다는 것이 명확해지고 request가 거절된 이유를 알게된다.

어플리케이션이 모든 시큐리티 이벤트에 대해 로그를 남기도록 설정하기 위해, 다음 설정을 추가한다.

**application.proerties**

```
logging.level.org.springframework.security=TRACE
```

**logback.xml**

```
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- ... -->
    </appender>
    <!-- ... -->
    <logger name="org.springframework.security" level="trace" additivity="false">
        <appender-ref ref="Console" />
    </logger>
</configuration>
```
