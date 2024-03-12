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

# Spring Security + JWT 토큰을 통한 로그인

### JWT 토큰을 통한 로그인

JWT(Json Web Token)은 일반적으로 클라이언트와 서버 통신 시 권한 인가(Authorization)을 위해 사용하는 토큰이다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Security + JWT 기본 동작 원리</p></figcaption></figure>

1. 클라이언트에서 ID/PW를 통해 로그인 요청
2. 서버에서 DB에 해당 ID/PW를 가진 Member가 있다면, Access Token과 Refresh Token을 발급해준다.
3. 클라이언트는 발급받은 Access Token을 헤더에 담아서 서버가 허용한 API를 사용할 수 있게 된다.

Access Token과 Refresh Token은 웹, 앱 애플리케이션에서 **인증 및 권한 부여**를 관리하기 위해 사용되는 토큰이다.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Access Token + Refresh Token 재발급 원리</p></figcaption></figure>

#### Access Token

> 인증된 사용자가 특정 리소스에 접근할 때 사용되는 토큰

* 유효 기간이 지나면 만료
* 만료된 경우, 새로운 Access Token을 얻기 위해 Refresh Token 사용

#### Refresh Token

> Access Token의 갱신을 위해 사용되는 토큰

* 일반적으로 Access Token과 함께 발급
* Access Token이 만료되면 Refresh Token을 사용하여 새로운 Access Token 발급
* 사용자가 지속적으로 인증 상태를 유지할 수 있도록 유지(매번 로그인할 필요 X)

***

실제 토큰을 생성하고 인증하는 과정을 코드를 통해서 알아보자.

### JwtTokenDto

```java
@Builder
@Data
@AllArgsConstructor
public class JwtTokenDto {
    private String grantType; // JWT에 대한 인증 타입
    private String accessToken;
    private String refreshToken;
}
```

단순하고 직관적이며 널리 사용되는 `Bearer` 인증 방식을 사용한다.

이 인증 방식은 Access Token을 HTTP 요청의 Authorization 헤더에 포함하여 전송한다.

### 암호키 설정

`application.properties` 에 **jwt.secret** 키를 저장한다.

해당 키는 토큰의 암호화, 복호화에 사용된다. HS256 알고리즘을 사용하기 위해 32글자 이상을 사용한다.

```properties
jwt.secret=c3ByaW5nLWJvb3Qtc2VjdXJpdHktand0LXR1dG9yaWFsLWppd29vbi1zcHJpbmctYm9vdC1zZWN1cml0eS1qd3QtdHV0b3JpYWwK
```

### JwtTokenProvider

Spring Security와 JWT 토큰을 사용하여 인증과 권한 부여를 처리하는 클래스이다.

이 클래스에서 JWT 토큰의 생성, 복호화, 검증 기능이 있다.

```java

@Slf4j
@Component
public class JwtTokenProvider {
    private final Key key;

    public JwtTokenProvider(@Value("${jwt.secret}") String secretKey) {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        this.key = Keys.hmacShaKeyFor(keyBytes);
    }

    /**
     * 인증 객체(Authentication)를 기반으로 Access Token과 Refresh Token 생성
     * Access Token: 인증된 사용자의 권한 정보와 만료 시간을 담고 있음
     * Refresh Token: Access Token의 갱신을 위해 사용됨
     * @param authentication
     * @return
     */
    public JwtTokenDto generateToken(Authentication authentication) {
        //권한 가져오기
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));
        long now = (new Date()).getTime();

        // Access Token 생성
        Date accessTokenExpiresIn = new Date(now + 86400000);
        String accessToken = Jwts.builder()
                .setSubject(authentication.getName())
                .claim("auth", authorities)
                .setExpiration(accessTokenExpiresIn)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        // Refresh Token 생성
        String refreshToken = Jwts.builder()
                .setExpiration(new Date(now + 86400000))
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return JwtTokenDto.builder()
                .grantType("Bearer")
                .accessToken(accessToken)
                .refreshToken(refreshToken)
                .build();
    }

    /**
     * 주어진 Access Token을 복호화하여 사용자의 인증 정보(Authentication) 생성
     * 토큰의 Claims에서 권한 정보를 추출하고, User 객체를 생성하여 Authentication 객체로 반환
     * Collection<? extends GrantedAuthority>로 리턴받는 이유
     * 권한 정보를 다양한 타입의 객체로 처리할 수 있고, 더 큰 유연성과 확장성을 가질 수 있음
     * @param accessToken
     * Authentication 객체 생성 과정
     * 1. 토큰의 클레임에서 권한 정보를 가져온다. "auth" 클레임은 토큰에 저장된 권한 정보를 나타냄
     * 2. 가져온 권한 정보를 SimpleGrantedAuthority 객체로 변환하여 컬렉션에 추가
     * 3. UserDetails 객체를 생성하요 주체와 권한 정보, 기타 필요한 정보를 설정
     * 4. UsernamepasswordAuthenticationToken 객체를 생성하여 주체와 권한 정보를 포함한 인증(Authentication) 객체를 생성
     * @return
     * 
     */
    public Authentication getAuthentication(String accessToken) {
        // Jwt 토큰 복호화
        Claims claims = parseClaims(accessToken);

        if (claims.get("auth") == null) {
            throw new RuntimeException("권한 정보가 없는 토큰입니다.");
        }

        // 클레임에서 권한 정보 가졍괴
        Collection<? extends GrantedAuthority> authorities = Arrays.stream(claims.get("auth").toString().split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

        // UserDetails 객체를 만들어서 Authentication return
        // UserDetails: interface, User : UserDetails를 구현한 클래스
        UserDetails principal = new User(claims.getSubject(), "", authorities);
        return new UsernamePasswordAuthenticationToken(principal, "", authorities);
    }

    /**
     * 주어진 토큰을 검증하여 유효성을 확인
     * Jwts.parserBuilder를 사용하여 토큰의 서명 키를 설정하고, 예외 처리를 통해 토큰의 유효성 여부를 판단
     * @param token
     * @return
     */
    // 토큰 정보를 검증하는 메서드
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                    .setSigningKey(key)
                    .build()
                    .parseClaimsJws(token);
            return true;
        } catch (SecurityException | MalformedJwtException e) {
            log.info("Invalid JWT token", e);
        } catch (ExpiredJwtException e) {
            log.info("Expired JWT Token", e);
        } catch (UnsupportedJwtException e) {
            log.info("Unsupported JWT Token", e);
        } catch (IllegalArgumentException e) {
            log.info("JWT claims string is empty", e);
        }
        return false;
    }

    /**
     * Claims : 토큰에서 사용할 정보의 조각
     * 주어진 Access Token을 복호화하고, 만료된 토큰의 경우에도 Claims 반환
     * parseClaimsJws() 메서드가 JWT 토큰의 검증과 파싱을 모두 수행
     * @param accessToken
     * @return
     */
    private Claims parseClaims(String accessToken) {
        try {
            return Jwts.parserBuilder()
                    .setSigningKey(key)
                    .build()
                    .parseClaimsJws(accessToken)
                    .getBody();
        } catch (ExpiredJwtException e) {
            return e.getClaims();
        }
    }
}
```



### JwtAuthenticationFilter

클라이언트 요청시 JWT 인증을 하기 위해 설치하는 **커스텀 필터** 로, **UsernamePasswordAuthenticationFilter** 이전에 실행한다. 클라이언트로부터 들어오는 요청에서 JWT 토큰을 처리하고, 유효한 토큰의 경우 해당 토큰의 인증 정보(Authentication)를 **SecurityContext**에 저장하여 인증된 요청을 처리할 수 있도록 한다.

**즉, JWT를 통해 username + password 인증을 수행한다.**

```java
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final JwtTokenProvider jwtTokenProvider;


    /**
     * 1. resolveToken() 메서드를 사용하여 요청 헤더에서 JWT 토큰을 추출
     * 2. JwtTokenProvider의 validateToken() 메서드로 JWT 토큰의 유효성 검증
     * 3. 토큰이 유효하면 JwtTokenProvider의 getAuthentication() 메서드로 인증 객체를 가져와서 SecurityContext에 저장 -> 요청을 처리하는 동안 인증 정보가 유지된다.
     * 4. chain.doFilter()를 호출하여 다음 필터로 요청을 전달.
     */
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 1. Request Header 에서 JWT 토큰 추출
        String token = resolveToken((HttpServletRequest) request);

        // 2. validateToken으로 토큰 유효성 검사
        if (token != null && jwtTokenProvider.validateToken(token)) {
            Authentication authentication = jwtTokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        filterChain.doFilter(request, response);
    }

    /**
     * 주어진 HttpServletRequest에서 토큰 정보를 추출하는 역할
     * "Authorization" 헤더에서 "Bearer" 접두사로 시작하는 토큰을 추출하여 반환
     */
    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}

```

### SecurityConfig

```java

/**
 * @Configuration 애노테이션이 @EnableWebSecurity에 포함되어 있다.
 */
@EnableWebSecurity
@Configuration
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtTokenProvider jwtTokenProvider;

    /**
     * PasswordEncoder interface의 구현체가 BCryptPasswordEncoder임을 수동 빈 등록을 통해서 명시한다.
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        return httpSecurity
                // Rest API이므로 basic auth 및 csrf 보안을 사용하지 않는다.
                .httpBasic().disable()
                .csrf().disable()
                // JWT를 사용하기 때문에 세션을 사용하지 않는다.
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeHttpRequests()
                .requestMatchers("/login").permitAll()
                .requestMatchers("/join").permitAll()
                .requestMatchers("/login/test").permitAll()
                // MEMBER 권한이 있어야 요청할 수 있음
                .requestMatchers("/members/{id}").hasAuthority(Authority.MEMBER.getAuthority())
                .and()
                // JWT 인증을 위하여 직접 구현한 필터를 UsernamePasswordAuthenticationFilter 전에 실행
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class)
                .build();
    }
}

```
