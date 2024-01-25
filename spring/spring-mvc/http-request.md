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

# HTTP Request

## HTTP Request

HTTP 요청으로 오는 데이터는 주로 3가지로 분류할 수 있다.

* **GET - 쿼리 파라미터**
  * /url\*\*?username=hello\&age=20\*\*
  * 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 전달
  * 검색, 필터, 페이징에서 주로 사용
* **POST - HTML Form**
  * content-Type: application/x-www-form-urlencoded
  * 메시지 바디에 쿼리 파라미터 형식으로 전달
  * 회원 가입, 상품 주문, HTML Form 사용
* **HTTP message Body**
  * HTTP API에서 주로 사용
  * 주로 JSON 사용

사실 GET 쿼리 파라미터 전송 방식이든, POST HTML Form 전송 방식이든 둘 다 형식이 같으므로 구분없이 조회할 수 있다.

⇒ **요청 파라미터(request parameter) 조회**

### 요청 파라미터 조회

이러한 요청 파라미터 조회는 여러가지 방법을 사용할 수 있다.

* **HttpServlet**
* **@RequestParam**
* **@ModelAttribute**

#### HttpServlet

`HttpServlet` 은 request로부터 파라미터를 직접 꺼내야 한다.

```java
request.getParameter("username");
Integer.parseInt(request.getParameter("age"));
```

#### @RequestParam

스프링이 제공하는 `@RequestParam` 을 이용하여 요청 파라미터를 편리하게 사용할 수 있다.

```java
public String requestParamV2(
				@RequestParam("username") String memberName,
				@RequestParam("age") int memberAge) {}
```

`@RequestParam` 를 통해 파라미터 이름으로 바인딩 한다. **@RequestParam**의 `name(value)` 속성이 파라미터 이름으로 사용된다.

만약 HTTP 파라미터 이름이 변수 이름과 같으면 `(”username”)` 부분은 생략 가능하다.

**@RequestParam의 옵션**

* **required: 파라미터 필수 여부**
  * 기본값이 파라미터 필수(true)
  * 파라미터 이름만 있고 값이 없어도 통과된다.
* **defaultValue: 파라미터 기본값 설정**
  * 빈 문자의 경우에도 설정한 기본값이 적용

#### @ModelAttribute

요청 파라미터를 받아서 필요한 객체를 만들고 객체에 값을 넣어주어야 한다. 스프링은 이 과정을 `@ModelAttribute`를 통해 자동화해준다.

```java
public String modelAttrubute(@ModelAttribute HelloData helloData) {}
```

### HTTP message Body

요청 파라미터와 다르게 HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 @RequestParam, @ModelAttribute 사용 X (단, HTML Form 형식은 제외)

#### 단순 텍스트

* HttpServlet
  * `request.getInputStream()`
* InputStream, OutputStream → 스프링 MVC가 지원
  * `requestBodyString(InputStream inputStream, Writer responseWriter)`
    * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
    * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출
* HttpEntity → 스프링 MVC가 지원
  * `requestBodyStringV3(HttpEntity<String> httpEntity)`
  * HttpEntity: HTTP header, body 정보를 편리하게 조회
  * HttpEntity는 응답에도 사용 가능
* @RequestBody → 스프링 MVC가 지원
  * `requestBodyStringV4(@RequestBody String messageBody)`
  * HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.

#### JSON

만약 이전처럼 HttpServlet, @RequestBody를 통해 String으로 받는다면 `ObjectMapper`를 통해서 객체를 생성해야 한다.

즉, String으로 인자를 받고 문자로 된 JSON 데이터인 `messageBody`를 ObjectMapper를 통해 자바 객체로 변환한다.

스프링은 이 부분 또한 @ModelAttribute 처럼 한번에 객체로 변환시킬 수 있다.

```java
public String requestBodyJson(@RequestBody HelloData data) {}
```

물론 HttpEntity도 가능하다.

HttpEntity, @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
