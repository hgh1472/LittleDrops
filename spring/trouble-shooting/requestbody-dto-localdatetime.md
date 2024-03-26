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

# @RequestBody DTO LocalDateTime 파싱 에러

Controller에서 @RequestBody를 통해 DTO를 받던 중 에러가 발생했다.

```java
Resolved [org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize value of type `java.time.LocalDateTime` from String "2024-03-26 13:00": Failed to deserialize java.time.LocalDateTime: (java.time.format.DateTimeParseException) Text '2024-03-26 13:00' could not be parsed at index 10]
```

실제 받으려했던 DTO 대신 설명을 위해 다른 DTO를 예로 들어보자.

**TestDto.java**

```java
@Data
@Builder
@AllArgsConstructor
public class TestDto {
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
    private LocalDateTime startTime;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
    private LocalDateTime endTime;
}
```

이 DTO는 LocalDateTime을 필드로 가지고 있다. 위의 DTO를 Controller에서 @RequestBody를 통해서 JSON 데이터를 입력받으려고 하면 에러가 발생한다.

왜 에러가 발생할까?

**@RequestBody는 기본 생성자를 필요로 한다.**

컨트롤러에서 @RequestBody로 지정한 객체는 HttpMessageConverter를 이용하여 JSON에서 자바 객체로 변환된다. JSON 요청은 MappingJackson2HttpMessageConverter를 사용하게 된다.

자바 객체에서 JSON으로 변환되는 것을 직렬화(Serialize), JSON에서 자바로 변환되는 것을 역직렬화(deserialize)라 한다.

@RequestBody로 요청받은 **DTO는 @JsonProperty나 @JsonAutoDetect를 통해 명시**하거나 **기본 생성자를 통해 자동으로 변환**할 수 있게끔 해야한다.

기본적으로 Jackson은 JSON 필드의 이름을 Java객체의 getter 및 setter 메서드와 일치시켜 JSON 객체 필드를 Java 객체의 필드에 매핑시킨다.

```java
getName -> name
```

즉, 동일한 이름의 필드를 직접 찾는 것이 아닌 getter와 setter를 찾아 매핑하게 된다. 따라서 DTO에 getter와 setter 둘 다 존재하지 않는다면 `UnrecognizedPropertyException` 이 발생한다.

DTO에 setter없이 값이 어떻게 주입될까?

Jackson에서 값을 주입할 때 setter를 사용하는 것이 아닌 **reflection을 이용해서 값을 주입**한다. 따라서 setter를 넣어주지 않고 getter만 추가해도 문제가 발생하지 않는다.

***

#### 참고

[@RequestBody로 지정한 DTO에 기본생성자가 필요한 이유](https://yongc.tistory.com/74)
