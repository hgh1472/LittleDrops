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

# 순환 참조

TODO 리스트 토이 프로젝트를 해보다가 순환 참조 문제가 발생했다.

> **순환참조**란, **참조하는 대상이 서로 물려 있어서 참조할 수 없게 되는 현상**을 말한다.

**TodoListInfo.java**

```java
@Entity
@Getter
@Builder
@NoArgsConstructor @AllArgsConstructor
public class TodoListInfo {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long code;

    @OneToOne
    private Member writer;

    @OneToMany(mappedBy = "todoListInfo", cascade = CascadeType.ALL)
    private List<Todo> todoList = new ArrayList<>();
}
```

**Todo.java**

```java
@Entity
@Getter
@Builder
@NoArgsConstructor @AllArgsConstructor
public class Todo {
    @Column(name = "todo_id")
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "todo_list_info_code")
    private TodoListInfo todoListInfo;

    private String description;

    private boolean isCompleted;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
    private LocalDateTime startTime;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
    private LocalDateTime endTime;

}
```

위 두 객체를 보면 `TodoListInfo` 에서는 `List<Todo>` 를 통해, `Todo` 에서는 `TodoListInfo` 를 통해 서로 참조하고 있다.

만약 JSON 응답으로 이 관계를 그대로 보내게 되면 순환 참조 문제가 발생한다.

예를 들어, `TodoListInfo` 의 형식 그대로 응답으로 내보냈다고 해보자.

그러면 `TodoListInfo` 의 `List<Todo>` 에서 각각의 `Todo` 는 `TodoListInfo` 를 참조할 것이다. 이 방식이 계속 반복되게 된다.

이러한 순환 참조를 막기 위한 2가지 방법을 알아보자.

### @JsonIgnore

`Todo` 클래스의 `TodoListInfo` 필드에 @JsonIgnore 애노테이션을 추가해준다.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "todo_list_info_code")
@JsonIgnore
private TodoListInfo todoListInfo;
```

이 애노테이션을 사용 시 조회할 경우 해당 필드를 조회하지 않는다.

### DTO

해당 응답에 따른 DTO를 생성하여 필요한 데이터만 담아서 반환하면 순환 참조 문제를 방지할 수 있다.

**ReadTodoListDto.java**

```java
@Data
@Builder
public class ReadTodoListDto {
    private Long code;

    private List<TodoDto> todoDtoList = new ArrayList<>();

    private LoginOutputDto writer;

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
}
```

**TodoDto.java**

```java
@Data
@Builder
public class TodoDto {

    private String description;

    private boolean isCompleted;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
    private LocalDateTime startTime;

    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm", timezone = "Asia/Seoul")
    private LocalDateTime endTime;

    public Todo toEntity() {
        return Todo.builder()
                .description(description)
                .isCompleted(isCompleted)
                .startTime(startTime)
                .endTime(endTime)
                .build();
    }

    public static TodoDto of(Todo todo) {
        return TodoDto.builder()
                .description(todo.getDescription())
                .isCompleted(todo.isCompleted())
                .startTime(todo.getStartTime())
                .endTime(todo.getEndTime())
                .build();
    }
}
```

`TodoListInfo` 를 조회할 때, DTO를 이용하여 필요한 정보만 담아줌으로써 순환 참조를 방지한다.

`TodoDto` 에는 `TodoListInfo` 에 대한 참조가 빠져있다.
