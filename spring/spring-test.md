---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: false
  pagination:
    visible: false
---

# Spring Test

DB에 접근하는 코드를 테스트하고 싶을 때는 어떻게 해야할까?

단순하게 서버를 실행시키고, 실제 데이터를 입력해볼 수 있다. 하지만 매번 이 작업을 하는 것은 상당히 번거로운 일이다. 테스트 코드를 활용해보자.

테스트 코드는 `src/test` 에 있으므로, `src/test` 에서의 `application.properties` 파일을 설정해야 한다. 그러므로 이 안에서 DB url이나 username 등 설정을 해야 한다.

```
spring.datasource.url=
spring.datasource.username=
```

테스트 코드를 작성해보자.

```java
@SpringBootTest // @SpringBootApplication을 찾아서 설정으로 사용한다.
class RepositoryTest {
    @Autowired
    ItemRepository itemRepository;
    
    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);

        //when
        Item savedItem = itemRepository.save(item);

        //then
        Item findItem = itemRepository.findById(item.getId()).get();
        assertThat(findItem).isEqualTo(savedItem);
    }
}
```

위와 같은 설정의 테스트는 문제점이 존재한다. 실제 실행할 서버의 데이터베이스와 테스트의 데이터베이스가 동일하다. 따라서 이전에 서버를 실행하면서 저장된 데이터들이 보관되어 있다. **즉, 데이터베이스를 분리시켜야 한다.**

### 데이터베이스 분리

이를 해결하기 위해서 테스트 환경과 실제 환경을 분리시켜야 한다. 이 때 떠올릴 수 있는 가장 간단한 방법은 테스트 전용 DB가 있으면 된다. 즉 새로운 테스트 전용 DB를 만들고 테스트 코드는 이 DB를 통해서 실행하도록 하면 된다.

```
# 테스트 DB 환경 설정
spring.datasource.url=
spring.datasource.username=
```

이 때 테스트 데이터베이스에도 테스트할 테이블을 생성해야 한다. 그리고 `src/test/application.properties` 를 테스트 환경에 맞게 다시 설정한다.

이렇게 한 후에 테스트를 실행하면 완벽할까? 테스트를 여러 개 해야되는 상황을 떠올려보자. `save()` 테스트가 실행되면 코드 내의 삽입된 Item은 그대로 존재할 것이다. 이 때 다른 테스트가 있다면, 이 테스트에 `save()` 에서 생성된 Item은 이 테스트에 영향을 주게 된다.

만약에 유니크 키에 해당하는 데이터가 삽입되지 않는 것을 확인하는 테스트가 있다고 해보자.

<pre class="language-java"><code class="lang-java"><strong>
</strong><strong>// ItemA 생성 후 삽입
</strong>
// 같은 유니크키를 가지는 ItemB 삽입 불가 확인
</code></pre>

이 코드는 마지막 로직에서 삽입이 안되는지 확인한다. 그런데 만약 `save()` 에서 ItemA와 같은 유니크키를 가지는 데이터가 이미 삽입되었다면, ItemA 삽입 단게에서 오류가 발생한다. 결과적으로 앞서 실행된 테스트로 인해 해당 테스트가 영향을 받았다. 이 문제를 해결하기 위해서는 각각의 테스트는 독립적으로 동작해야 한다.

테스트에 있어서 중요한 원칙이 있다.

* **테스트는 다른 테스트와 격리해야 한다.**
* **테스트는 반복해서 실행할 수 있어야 한다.**

이를 해결하기 위해서 테스트가 끝날 때마다 추가된 데이터에 `DELETE SQL` 문을 사용하는 것도 방법이 될 수 있지만, 이 방법이 좋은 해결책일 것 같지는 않다. 만약 중간에 실패한다면, `DELETE SQL` 을 호출하지 못할 수도 있다.

### 롤백

**트랜잭션**에 대해서 생각해보자. 트랜잭션은 커밋과 롤백이 존재했다. 우리는 **롤백**을 이용한다.

테스트가 끝나고 트랜잭션을 롤백한다면 데이터는 제거된다. 중간에 실패했다하더라도, 트랜잭션을 커밋하지 않았기 때문에 DB에 해당 데이터는 반영되지 않는다. 따라서 각각의 테스트를 트랜잭션 단위로 생각하고, 해당 테스트 종료 전 롤백을 수행한다.

각 테스트에 적용하기 위해 `@BeforeEach`, `@AfterEach` 를 사용한다.

```java
@SpringBootTest // @SpringBootApplication을 찾아서 설정으로 사용한다.
class RepositoryTest {
    @Autowired
    ItemRepository itemRepository;
    
    @Autowired
    PlatformTransactionManager transactionManager;
    TransactionStatus status;
    
    @BeforeEach
    void beforeEach() {
        // 트랜잭션 시작
        status = transactionManager.getTransaction(new DefaultTransactionDefinition());
    }
    
    @AfterEach
    void afterEach() {
        transactionManager.rollback(status);
    }
    
    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);

        //when
        Item savedItem = itemRepository.save(item);

        //then
        Item findItem = itemRepository.findById(item.getId()).get();
        assertThat(findItem).isEqualTo(savedItem);
    }
}
```

`transactionManager` 는 `PlatformTransactionManager` 를 주입받아서 사용한다. 스프링 부트는 자동으로 적절한 트랜잭션 매니저를 스프링 빈으로 등록한다. `@BeforeEach` 를 통해 각각 테스트 케이스 실행 전에 트랜잭션을 실행한다. `@AfterEach` 를 통해 테이트 케이스 완료 후 트랜잭션을 롤백한다.

### @Transactionnal

위에서 우리는 `@BeforeEach` , `@AfterEach` 를 사용했다. 이 과정마저도 스프링은 `@Transactional` 애노테이션 하나로 깔끔하게 해결할 수 있다.

```java
@Transactional
@SpringBootTest // @SpringBootApplication을 찾아서 설정으로 사용한다.
class RepositoryTest {
    @Autowired
    ItemRepository itemRepository;
    
    /*
    @Autowired
    PlatformTransactionManager transactionManager;
    TransactionStatus status;
    
    @BeforeEach
    void beforeEach() {
        // 트랜잭션 시작
        status = transactionManager.getTransaction(new DefaultTransactionDefinition());
    }
    
    @AfterEach
    void afterEach() {
        transactionManager.rollback(status);
    }
    */
    
    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);

        //when
        Item savedItem = itemRepository.save(item);

        //then
        Item findItem = itemRepository.findById(item.getId()).get();
        assertThat(findItem).isEqualTo(savedItem);
    }
}
```

스프링이 제공하는 `@Transactional` 애노테이션은 주로 우리가 서비스에서 사용하면 로직이 성공적으로 수행 시 커밋한다. 그런데 이 애노테이션을 테스트에서 사용하면 스프링은 트랜잭션을 실행하고 테스트가 끝나면 트랜잭션을 자동으로 롤백시킨다. 이때 애노테이션은 메서드에 붙여도 된다.

### 임베디드 모드 DB

위 상황을 살펴보면 테스트용 DB는 만들고 바로 지워지므로 DB 내 데이터는 지속되지 않는다. 단순히 테스트 검증용도이기 때문에 모두 삭제해도 상관이 없다.

몇몇 데이터베이스는 임베디드 모드를 제공한다. 임베디드 모드는 DB를 애플리케이션에 내장해서 함께 실행한다고 보면 된다. 애플리케이션이 종료되면 임베디드 모드로 동작하는 데이터베이스도 함께 종료되고, 데이터도 모두 사라진다.

`src/test/application.properties` 에서 데이터베이스에 접근하는 설정 정보를 없애고 실행시키면, 스프링 부트는 임베디드 모드로 접근하는 `DataSource` 를 만들어서 제공한다.

이 때 임베디드 모드를 사용하면, ITEM 테이블에 대해서 모르고 있는 상태가 된다. 테스트를 실행하기전 테이블 생성 SQL을 날려도 되지만 스프링 부트는 이에 대한 기능을 제공한다.

`src/test` 에 `schema.sql` 을 만들고 테이블 생성 SQL을 작성한다.

```sql
drop table if exists item CASCADE;
create table item
(
 id bigint generated by default as identity,
 item_name varchar(10),
 price integer,
 quantity integer,
 primary key (id)
);
```

이로써 우리는 테스트를 위해 서버를 직접 실행시켜 확인하지 않아도 된다. 그리고 실제 서버 DB와 테스트 DB를 분리하였기 때문에 서로에게 영향을 주지 않고, 롤백을 통해 테스트간에도 서로 영향을 주지 않는다.

결국 테스트는 다른 테스트와 격리되어 있고, 반복해서 실행할 수 있게 되었다.

### TDD(Test-Driven-Development)

TDD는 동작하는 코드를 작성하기 전에 테스트를 먼저 작성하고, 그 테스트를 통과하는 코드를 작성함으로써 테스트된 동작하는 코드를 개발하는 방법이다.

TDD는 세 가지 단계가 한 사이클로 이루어진다.

* 테스트 작성(빨간 불)
* 실행 가능하게 코드 작성(초록 불)
* 리팩토링

먼저 컴파일조차 되지 않는 코드를 대상으로 먼저 테스트를 작성한다. 그 다음 테스트 코드가 컴파일되고 실행까지 되도록 코드를 작성한 후, 테스트가 통과되면 테스트의 보호아래 코드를 다듬는다.

예를 들어 계산기에 대한 테스트를 한다고 가정해보자. 아직 `Calculator` 객체와 메서드도 작성하지 않았다. 이 때 우리는 존재하지 않는 객체로 인해 컴파일조차 되지 않더라도 먼저 작성하고 컴파일부터 도도록 코드를 작성해나간다. 즉, 먼저 테스트를 실행시켰을 때 빨간 불이 나오도록 코드를 작성한다.

그 다음 위의 과정을 통해 돌아가는 코드까지 작성했다면, 이제 해결하는 코드를 작성한다. 이 때 **가능한 빠르게 테스트가 통과하는 코드를 작성한다.** 이 때 여기서 ‘빠르게’라는 말은 성능적으로 빠르게라는 뜻이 아니라, 초록불을 띄우는 코드를 빨리 작성하라는 뜻이다.

```java
public class SampleTest {
	@Test
	void 샘플_테스트() {
		Sample sample = new Sample(true);
		assertThat(sample.myState()).isTrue();
	}
}
```

위 코드에서 Sample 클래스를 생성하고 `myState`의 반환값이 true인지 확인한다. 이 때, `Sample`클래스를 정의하지 않았다면 컴파일조차 되지 않을 것이다. 따라서 테스트를 실행하면 빨간불로 나오게 된다.

그러면 이제 이 테스트를 돌리기 위해서 `Sample` 클래스를 생성해야 한다.

```java
public class Sample {
	Sample(boolean bool) {
	}
	
	public boolean myState() {
		return false;
	}
}
```

이제 테스트 코드는 컴파일될 것이다. 테스트 메서드는 실행되지만 실패하게 된다. 보다시피 `myState`는 false를 반환하는데, 테스트 코드에서는 true가 기대값으로 설정되어있다. 이 때 TDD에 맞춰 최대한 빨리 빨간불을 초록불로 만들려면 어떻게 해야할까?

정말 단순하게 생각하면 반환값을 true로 만들면 된다.

```java
public class Sample {

	Sample(boolean bool) {
	}

	public boolean myState() {
		return true;
	}
}
```

이제 테스트가 성공하게 되어 초록불을 띄우게 된다. TDD의 두번째 단계까지 완료하게 된 셈이다. 이제 리팩토링을 해보자. 이 과정에서 테스트에 있는 데이터와 코드에 있는 데이터의 중복을 제거해야한다.

```java
assertThat(sample.myState()).isTrue();

public boolean myState() {
	return true;
}
```

테스트 코드에도 true가 존재하고, `Sample` 클래스의 `myState`에서도 true가 존재한다. 결국 중복으로 true가 등장하고 있다. 이를 리팩토링 해보자.

```java
public class Sample {
	private boolean state;
	
	Sample(boolean state) {
		this.state = state;
	}
	
	public boolean myState() {
		return state;
	}
}
```

이를 통해 `myState` 는 하드 코딩된 값이 아니라 객체 내부의 상태를 반환하게 된다. 그리고 생성자의 파라미터 이름을 알맞게 변경하였다.

하나의 클래스를 정의해가는 과정에서 컴파일이 되지 않는다는 것을 알면서도 실행하고, 컴파일만 될 뿐 테스트는 당연히 실패한다는 것을 알지만 테스트를 실행했다. 사실 당연한 과정이라 지루하고 그냥 건너뛰어도 될 것 같다는 생각이 든다. 게다가 예시의 코드는 상당히 단순한 클래스라 TDD가 그닥 필요하지 않다고 느껴진다.

하지만 지루함 속에는 내가 생각한대로 코드가 동작한다는 확신을 얻을 수 있다. `myState` 메서드는 생성자로 건네준 상태를 반환한다. 이 코드의 예시는 상당히 단순하지만, 코드가 길어졌을 때 로직 상 문제가 발생해서 돌려보고 값을 넣어보고 로그를 찍어보는 상황이 많다. 이 때 TDD에 대해 정의한 켄트백은 코드 변경에 대한 두려움을 지루함을 바꾸는 과정이라 말한다.

예시 상황처럼 이런 코드도 작은 단계로 세분화할 필요는 없지만, 필요한 경우 작은 단계로 나누어 진행할 줄 알아야 한다.

물론 언제나 TDD를 통한 개발이 언제나 좋은 것은 아니다. TDD 플로우를 통한 개발은 그만큼 테스트의 리소스가 소모된다. 주어진 기간이 짧은데, TDD 플로우를 적용하여 개발한다는 것은 바람직하지 않다. 결국 **테스트 코드를 작성하는 비용과 수작업으로 테스트하는 비용을 비교해야 한다.** 위의 Sample 클래스는 사실 바로 Sample 클래스에 대한 코드를 작성해나가는 것이 훨씬 효율적이다. 크게 복잡하지 않거나 소규모 프로젝트에서 테스트가 없기 때문에 퀄리티가 떨어지기는 쉽지않다.

순수하게 개발이라는 관점에서 TDD는 충분히 좋은 방법이지만, 모든 것을 고려한 실제 상황에서는 쉽게 도입하기는 어렵다고 생각한다.

***

#### 참고

[https://tech.kakaopay.com/post/implementing-tdd-in-practical-applications/](https://tech.kakaopay.com/post/implementing-tdd-in-practical-applications/)

[https://www.inflearn.com/course/스프링-db-2/dashboard](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-db-2/dashboard)
