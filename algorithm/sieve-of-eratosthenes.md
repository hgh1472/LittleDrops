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

# Sieve of Eratosthenes

에라토스테네스의 체는 소수를 구하는 알고리즘이다.

이 때 특정 숫자가 소수인지 판별하는 것이 아니라 1부터  특정 숫자 N 까지의  숫자 중, 어떤 수가 소수인지 판별할 수 있다.

예를 들어 N이 100이면, 1부터 100까지의 모든 소수를 알아낼 수 있다.

<figure><img src="https://upload.wikimedia.org/wikipedia/commons/b/b9/Sieve_of_Eratosthenes_animation.gif" alt=""><figcaption></figcaption></figure>

위의 그림을 예시로 들어보자.

1. 원하는 범위(N)만큼 boolean 배열 isNotPrime을 생성한다. (**소수 => False**)
2. 제일 작은 소수인 2부터 N까지 다음 과정을 수행한다.
   1. 만약 isNotPrime\[i]가 True라면, 다음 숫자로 넘어간다.
   2. 숫자 i의 2배수부터  i 배수에 해당 숫자의 isNotPrime값을 True로 바꿔준다.

위 과정을 수행한 후, isNotPrime 값이 False인 숫자가 소수이다.

