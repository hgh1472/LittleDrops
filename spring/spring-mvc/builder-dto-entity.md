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

# @Builder를 이용한 DTO와 ENTITY 변환

JSON방식으로 Request를 받으면, 주로 DTO를 통해 받게 된다.

DTO를 사용할 때 생성자보다는 `Builder` 패턴을 사용하게 된다.

그 이유로는 크게 **가독성**과 **불변성**이 존재한다.

먼저 생성자를 통해 DTO를 만드는 예시를 들어보자.

**생성자를 통한 DTO 생성**

```java
TodoListInfo todoListInfo = todoService.readTodoListInfo(code);
ReadTodoListDto readTodoListDto = new ReadTodoListDto(todoListInfo.getCode(), todoListInfo.getTodoList(), todoListInfo.getWriter());
```

생성자를 이용할 경우 인자를 생성자 변수의 순서에 맞춰 넣어줘야 한다. 만일 동일한 타입일 경우 컴파일러에서 오류를 잡을 수 없기 때문에 오류를 발견하기 어렵다.

**Builder를 이용한 DTO 생성**

```java
public static ReadTodoListDto of(TodoListInfo todoListInfo) {
        return ReadTodoListDto.builder()
                .code(todoListInfo.getCode())
                .todoDtoList(todoListInfo.getTodoList()
                        .stream()
                        .map(todo -> TodoDto.of(todo))
                        .toList())
                .writer(LoginOutputDto.of(todoListInfo.getWriter()))
                .build();
    }
```

생성자로 만드는 것보다 코드가 길다.

하지만 `Builder` 패턴을 이용하면 어떤 변수에 어떤 데이터가 들어가는지 쉽게 판단할 수 있다.

또한 `Builder` 패턴을 사용하면 파라미터 하나하나 생성자를 만들 필요가 없다. 필요한 항목만 해당 코드를 설정해줄 수 있다.

#### @NoArgsConstructor

**JPA의 기본 스펙은 Entity에 기본 생성자가 필수로 있어야 한다.**

`Builder`란 생성자의 매개변수가 많을 때 좀 더 편리하게 객체를 만들 수 있게 도와주는데 전체 생성자가 필요하다.

그래서 `@Builder` , `@NoArgsConstructor` 애노테이션만 사용한다면 전체 생성자가 없기 때문에 오류가 발생한다. 따라서 Entity에 `@Builder` 를 사용하고 싶다면, `@NoArgsConstructor` 와 `@AllArgsConstructor` 를 사용해야 한다.

***

#### 참고

[기본 생성자에 관해 질문드립니다. - 인프런](https://www.inflearn.com/questions/105043/%EA%B8%B0%EB%B3%B8-%EC%83%9D%EC%84%B1%EC%9E%90%EC%97%90-%EA%B4%80%ED%95%B4-%EC%A7%88%EB%AC%B8%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4)

[\[Spring\] @Builder를 사용할 때 고려해야 할 생성자 문제 알아보기](https://devlog-wjdrbs96.tistory.com/419)

[\[Java\] DTO와 빌더패턴](https://yeoooo.github.io/java/BuilderPattern/)
