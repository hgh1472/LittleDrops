# 상속 (Inheritance)

### 상속을 하는 이유

어느 객체는 다른 객체의 하위 개념이 될 수 있다.

EX) 자동차

* 전기차
* 가솔린차

전기차와 가솔린차는 `자동차`라는 개념에 더 구체적인 개념이다. 즉, 전기차와 가솔린차는 `자동차` 에 속한다. 그래서 자동차라는 개념에 속하는 공통 기능이 존재할 것이다. 이런 경우에 상속을 이용한다.

### 상속 관계

상속을 이용하면 기존 클래스의 필드와 메서드를 상속받은 클래스에서 재사용할 수 있다.

따라서 기존 클래스의 속성과 기능을 그대로 사용한다. 상속을 하기 위해서는 `extends` 키워드를 사용하고 1개만 가능하다.

코드로 예시를 들어보자.

**Car.java**

```java
public class Car {
    public void move() {
        System.out.println("움직입니다.");
    }
}

```

**ElectricCar.java**

```java
public class ElectricCar extends Car {
    public void charge() {
        System.out.println("차가 충전중입니다.");
    }
}
```

**GasolineCar.java**

```java
public class GasolineCar extends Car {
    public void fill() {
        System.out.println("주유중입니다.");
    }
}
```

`GasolineCar` 와 `ElectricCar` 는 `Car` 를 extends하므로 move 메서드를 사용할 수 있다.

상속했기 때문에 부모의 기능을 자식이 물려받는 것이다. 반대로 부모인 `Car` 는 자식 클래스에 접근할 수 없다. 부모 클래스의 코드에서는 자식에 대한 정보를 알 수 있는 부분이 아예 없다.

### 메모리 구조

`new ElectricCar()` 를 호출하면 `ElectricCar` 뿐만 아니라 상속관계에 있는 `Car` 까지 포함해서 인스턴스를 생성한다. 참조값은 하나이지만 실제로 그 내부에는 `Car` 와 `ElectricCar` 두 가지 클래스 정보가 존재한다. 상속관계를 사용하면 위부에서는 하나의 인스턴스를 생성하는 것 같지만, 내부에서는 부모와 자식 모두 생성되고 공간도 구분된다.

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

**electricCar.charge()** 호출

electricCar가 charge()를 호출하게 되면 참조값을 확인하여 `x001.charge()` 를 호출한다. 이 때 부모인 `Car` 를 통해서 `charge()` 를 찾을지, `ElectricCar` 를 통해서 `charge()` 를 찾을지 선택해야 한다.

<figure><img src=".gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

이때는 **호출하는 변수의 타입(클래스)을 기준으로 선택**한다.

**electricCar.move()** 호출

`electricCar.move()` 를 호출하면 먼저 `x001` 로 이동한다. 내부에는 `Car`, `ElectricCar` 두 개가 존재하는데 호출하는 변수인 `ElectricCar` 타입을 먼저 확인한다.

하지만 , `ElectricCar` 에는 `move()` 메서드가 없으므로 부모 타입인 `Car` 에서 찾는다.

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

만약 부모에서도 해당 기능이 존재하지 않는다면, 계속해서 상위 부모로 올라간다. 아예 없는 경우에는 컴파일 오류가 발생한다.

### 메서드 오버라이딩

`Car.move()` 기능이 존재한다. 그런데 `ElectricCar` 의 경우, `move()` 메서드의 출력을 변경하고 싶다고 가정해보자.

위처럼 부모에게서 상속받은 기능을 자식 클래스가 재정의 하는 것을 \*\*메서드 오버라이딩(Overriding)\*\*이라고 한다.

**ElectricCar.java**

```java
public class ElectricCar extends Car {
		@Override
		public void move() {
				System.out.println("전기차가 빠르게 이동합니다.");
		}
		
		public void charge() {
				System.out.println("충전합니다.");
		}
}
```

`ElectricCar` 는 부모 클래스인 `Car` 의 `move()` 기능을 사용하지 않고 자신의 `move()` 를 사용하고 싶은 상황이다.

이렇게 부모의 기능을 자식이 재정의 하는 것을 메서드 오버라이딩이라고 한다.

`ElectricCar` 의 `move()` 를 호출하면 `Car` 의 `move()` 가 아닌, `ElectricCar` 의 `move()` 가 호출된다.

#### 오버라이딩과 메모리 구조

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

1. `electricCar.move()` 호출
2. 호출한 `electricCar` 의 타입은 `EletricCar` ⇒ 인스턴스 내부에 `ElectricCar` 타입부터 시작

**오버로딩(Overloading)과 오버라이딩(Overriding)**

**오버로딩**은 메서드 이름이 같고, 매개변수가 다른 메서드를 여러 개 정의하는 것을 말한다. 같은 이름의 메서드를 여러 개 정의한 상황이다.

**오버라이딩**은 하위 클래스에서 상위 클래스의 메서드를 재정의하는 것을 의미한다. 따라서 **상속 관계에서 사용**한다. 부모의 기능을 자식이 다시 재정의하는 것이다. 즉, 자식의 새로운 기능이 부모의 기존 기능을 넘어타서 기존 기능을 새로운 기능으로 덮어버린다고 이해하자.

**오버라이딩 조건**

* 메서드 이름은 같아야 한다.
* 메서드 매개변수 타입, 순서, 개수가 같아야 한다.
* 접근 제어자는 상위 클래스의 메서드보다 더 제한적이어서는 안된다.
* 예외는 상위 클래스의 메서드보다 더 많은 예외를 `throws` 로 선언할 수 없다.
* static, final, private 키워드가 붙은 메서드는 오버라이딩 할 수 없다.
  * static은 **클래스 레벨에서 작동**하므로 인스턴스 레벨에서 사용하는 오버라이딩이 의미가 없다.
  * final 메서드는 재정의를 금지한다.
  * private 메서드는 해당 클래스에서만 접근 가능하기 때문에 하위 클래스에서 보이지 않는다.
* 생성자 오버라이딩은 할 수 없다.

### Super - 부모 참조

부모와 자식의 필드명이 같거나 메서드가 오버라이딩 되어 있으면, 자식에서 부모의 필드나 메서드를 호출할 수 없다. 이때 `super` 키워드를 사용하면 부모를 참조할 수 있다.

```java
public class Parent {
		public String value = "parent";
		
		public void hello() {
				System.out.println("Parent.hello");
		}
}

public class Child extends Parent {
		public String value = "child";
		
		@Override
		public void hello() {
				System.out.println("Child.hello");
				
		public void call() {
				System.out.println("this value = " + this.value);
				System.out.println("super value = " + super.value);
				
				this.hello();
				super.hello();
		}
}
```

필드 이름과 메서드의 이름이 같지만 `super` 를 사용해서 부모 클래스에 있는 기능을 사용할 수 있다.

<figure><img src=".gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

### super - 생성자

상속관계의 인스턴스를 생성하면 위에서 보다시피 메모리 내부에는 자식과 부모 클래스가 각각 만들어진다. `Child` 를 만들면 `Parent` 까지 함께 만들어지는 것이다. 따라서 각각의 생성자가 모두 호출된다.

**상속관계를 사용하면 자식 클래스의 생성자에서 부모 클래스의 생성자를 반드시 호출해야 한다.**

```java
public class classA {
		public classA() {
				System.out.println("ClassA 생성자");
		}
}

public class ClassB extends ClassA {
		public ClassB(int a) {
				super(); // 기본 생성자 생략 가능
				System.out.println("ClassB 생성자 a = " + a);
		}
		
		public ClassB(int a, int b) {
				super(); // 기본 생성자 생략 가능
				System.out.println("ClassB 생성자 a = " + a + " b = " + b);
		}
}
```

`ClassB` 는 `ClassA` 를 상속받았다. 상속을 받으면 생성자의 첫줄에 `super(...)` 를 사용하여 부모 클래스의 생성자를 호출해야 한다.

부모 클래스의 생성자가 기본 생성자인 경우, `super()` 를 생략할 수 있다. 만약 기본 생성자가 없는 경우, `super(...)` 를 생략할 수 없다.

## 추상 클래스

`Animal` 과 `Animal` 을 상속한 `Cat`, `Dog`, `Caw` 클래스가 존재한다고 해보자. 개, 고양이, 소가 실제 존재하는 것은 당연하지만, 동물이라는 추상적인 개념이 실제로 존재하는 것은 이상하다. `Animal` 클래스는 다형성을 위해 필요한 것이지 직접 인스턴스를 생성해서 사용할 일은 없다.

```java
public class Animal {
		public void sound() {
				System.out.println("동물 울음 소리");
		}
}

public class Dog extends Animal {
		@Override
		public void sound() {
				System.out.println("멍멍");
		}
}

public class Cat extends Animal {
		@Override
		public void sound() {
				System.out.println("냐옹");
		}
}

public class Caw extends Animal {
		@Override
		public void sound() {
				System.out.println("음매");
		}
}
```

하지만 `Animal` 도 클래스이기 때문에 인스턴스를 생성하고 사용하는데 아무런 제약이 없다. 실수로 `new Animal()` 을 사용해서 `Animal` 의 인스턴스를 생성할 수도 있다.

만약 `Animal` 을 상속받은 `Pig` 클래스를 만들었다고 해보자. 그런데 개발자 실수로 `sound()` 메서드를 오버라이딩 하는 것을 빠트렸다. 그러면 코드상 아무 문제가 없지만 실제 프로그램을 실행하면 `Animal` 의 `sound()` 메서드가 실행된다.

이 부분에 제약을 걸기 위해서 추상 클래스와 추상 메서드가 사용된다.

**추상 클래스**

동물과 같이 부모 클래스는 제공하지만, 실제 생성하면 안되는 클래스를 추상 클래스라 한다.

추상 클래스는 이름 그대로 추상적인 개념을 제공하고 실체인 인스턴스는 존재하지 않는다. 대신 상속을 목적으로 사용되고 부모 클래스 역할을 담당한다.

```java
abstract class AbstractAnimal {}
```

* 추상 클래스는 선언할 때 `abstract` 키워드를 붙인다.
* 추상 클래스는 기존 클래스와 완전히 같다. 다만 `new AbstractAnimal()` 와 같이 직접 인스턴스를 생성하지 못하는 제약이 추가된 것이다.

**추상 메서드**

부모 클래스를 상속받는 자식 클래스가 반드시 오버라이딩 해야 하는 메서드를 부모 클래스에서 정의할 수 있다. 이를 추상 메서드라 한다. 추상 메서드는 실체가 존재하지 않고 메서드 바디가 없다.

```java
public abstract void sound();
```

* 추상 메서드는 앞에 `abstract` 키워드를 붙인다.
* **추상 메서드가 하나라도 있는 클래스는 추상 클래스로 선언해야 한다.**
* **추상 메서드는 상속받는 자식 클래스가 반드시 오버라이딩해서 사용해야 한다.**

추상 클래스는 나머지가 모두 일반적인 클래스와 동일하다. 추상 클래스는 단지 제약이 추가된 클래스일 뿐이다.

<figure><img src=".gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

**순수 추상 클래스**

추상 클래스의 모든 메서드가 추상 메서드이면 **순수 추상 클래스**라고 한다.

이 때 코드를 실행할 바디 부분이 전혀 없다. 따라서 순수 추상 클래스는 실행 로직을 전혀 가지고 있지 않다. 단지 다형성을 위한 부모 타입으로써 껍데기 역할만 제공한다.

* 인스턴스 생성 X
* 상속 시 자식 클래스는 모든 메서드 오버라이딩
* 다형성을 위해 사용

이는 인터페이스와 유사하게 느껴진다. 예를 들어 USB 인터페이스는 규격이 존재한다. 이 규격에 맞추어 제품을 개발해야 연결이 가능하다. 순수 추상 클래스가 USB 인터페이스라면, USB 인터페이스에 맞추어 마우스, 키보드 등 장치들을 구현할 수 있다.

이런 순수 추상 클래스의 개념을 더 편리하게 사용하도록 인터페이스가 존재한다.

### 인터페이스

인터페이스는 `interface` 키워드를 사용한다.

```java
public interface InterfaceAnimal {
		void sound(); // public abstract 키워드 생략 가능
		void move();
}
```

* 인터페이스는 순수 추상 클래스에서 약간의 편의 기능이 추가된다.
* 인터페이스의 메서드는 모두 `public`, `abstract` 이다.
* 메서드에 `public abstract` 를 생략할 수 있다.
* 다중 상속을 지원한다.
* 인터페이스에서 멤버 변수는 `public`, `static`, `final` 이 모두 포함되었다고 간주한다.

```java
public interface InterfaceAnimal {
		int MY_PI = 3.14; // public static final int MY_PI = 3.14;
} 
```

### 상속 vs 구현

부모 클래스의 기능을 자식 클래스 상속 받을 때 **상속**받는다고 표현하지만, 부모 인터페이스의 기능을 자식이 상속받을 때는 인터페이스를 **구현**한다고 표현한다.

상속은 부모의 기능을 물려받는 것이 목적이다. 하지만 인터페이스는 물려받을 수 있는 기능이 없고, 인터페이스에서 정의한 모든 메서드를 자식이 오버라이딩 해서 기능을 구현해야 한다.

인터페이스는 메서드 이름만 있는 설계도이고, 이 설계도가 실제 어떻게 작동하는지는 하위 클래스에서 모두 구현해야 한다.

자바 입장에서는 상속과 구현이 똑같다. 우리가 사용할 때 표현하는 단어가 다른 것이다.

**인터페이스를 사용하는 이유**

인터페이스를 사용해야 하는 이유로 편리함 외에도 다른 이유가 있다.

* 제약
  * 순수 추상 클래스의 경우 미래에 누군가 실행 가능한 메서드를 끼워넣을 수 있다. 이렇게 되면 추가된 기능을 자식 클래스에서 구현하지 않을 수도 있고, 더는 순수 초상 클래스가 아니게 된다. 따라서 이러한 문제를 차단할 수 있다.
* 다중 구현
  * 클래스 상속은 1개만 받을 수 있지만, 인터페이스는 다중 상속이 가능하다.

> 자바 8에 등장한 `default` 메서드를 사용하면 인터페이스도 메서드를 구현할 수 있지만 제외하자.
