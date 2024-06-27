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

# Controller vs Service

Controller는 웹 MVC에서 컨트롤러 역할을 한다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Controller는 Model과 View 사이에서 데이터 흐름을 제어한다. 사용자가 접근한 URL에 따라 요청을 파악하고 URL에 적절한 메서드를 호출하여 Service에서 비즈니스 로직을 처리한다. 이 후 결과를 Model에 저장하여 View에게 전달하는 역할을 수행한다. -> Model과 View의 역할을 분리하는 요소이다.

1. 사용자의 요청을 Controller가 받는다.
2. Controller는 Service에서 비즈니스 로직을 처리한 후 결과를 Model에 담는다.
3. Model에 저장된 결과를 바탕으로 시각적 요소 출력을 담당하는 View를 제어하여 사용자에게 전달한다.

이를 Web에 적용하여 생각해보자.

1. User가 웹사이트에 접속한다.
2. Controller는 사용자가 요청한 웹 페이지를 보여주기 위해 Model을 호출한다.
3. Model은 비즈니스 로직을 통해 DB 및 파일과 같은 데이터를 제어한 후 결과를 반환한다. 이후 Controller는 Model에게 반환받은 결과를 View에 반영한다.
4. 데이터를 받아온 View가 사용자에게 웹 페이지를 출력하여 보여준다.

그렇다면 Controller와 Service의 로직 구분은 어떻게 해야할까?

극단적이지만 이렇게 생각하면 좋다.

`Controller 같은 웹 계층이 없어도 애플리케이션이 동작해야 한다.`

Controller는 웹 계층을 처리하기 위한 코드만 존재하는 것이 좋다. 예를 들어서 웹 계층이 없이 단순히 메인 메서드를 통해서 콘솔에서만 동작하는 애플리케이션을 추가로 개발해야 해도 대부분의 서비스 로직을 재사용할 수 있어야한다.

반대로 말하면 웹 계층을 위한 폼 데이터를 처리하고, 화면에 뿌릴 데이터를 모아서 넘겨주고 이런 웹 계층 관련 일들은 모두 Controller에서 담당해야 한다.
