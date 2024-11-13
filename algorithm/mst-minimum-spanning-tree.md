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

# MST(Minimum Spanning Tree)

> **최소신장트리(MST)**란 가중그래프의 총 간선 무게가 최소인 신장트리를 말한다.

* **신장트리** : (자유)트리인 신장 부그래프
* **신장 부그래프** : 그래프 G의 모든 정점들을 포함하는 부그래프

## 최소신장트리 속성

### 싸이클 속성

* **T** : 가중그래프 G의 **최소신장트리**
* **e** : **T**에 존재하지 않는 **G**의 간선
* **C** : **e**를 **T**에 추가하여 형성된 싸이클

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

**C**의 모든 간선 **f**에 대해, `weight(f) <= weight(e)`가 성립한다.

### 분할 속성

* **G**의 정점들을 두 개의 부분집합 **U**와 **V**로 분할
* **e** : 분할을 가로지르는 최소 무게의 간선

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

간선 **e**를 포함하는 **G**의 최소신장트리가 반드시 존재한다.

## 최소신장트리 알고리즘

### Prim-Jarnik 알고리즘

* **탐욕 알고리즘**
* 단순 연결 무방향 가중그래프
* 임의의 정점 s를 택하여, s로부터 시작하여 정점들을 **배낭**에 넣어가며 배낭 안에서 최소신장트리인 **T**를 키워나간다. => s는 T의 루트가 된다.
* 각 정점 v에 라벨 d(v)를 정의 => d(v)는 배낭 안의 정점과 배낭 밖의 정점을 연결하는 간선의 무게

<figure><img src="../.gitbook/assets/image (2) (1).png" alt="" width="377"><figcaption></figcaption></figure>

* 배낭 밖의 정점들을 우선순위 큐에 저장
  * 키 : 거리
  * 원소 : 정점
* Q.replaceKey(e,k) : 원소 e의 키를 k로 변경하고 Q 내의 e의 위치를 갱신한다.
* 시간복잡도 : $$O(mlogn)$$
  * 우선순위 큐를구  binary heap으로 구현 시, $$O(mlogm)$$

#### Prim-Jarnik 알고리즘 수행 예시

<figure><img src="../.gitbook/assets/image (5) (1).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (6) (1).png" alt="" width="563"><figcaption></figcaption></figure>

### Kruskal 알고리즘

* 탐욕 알고리즘
* 알고리즘을 위한 초기 작업
  * 모든 정점을 각각의 독자적인 배낭에 넣는다.
  * 배낭밖의 간선들을 우선순위 큐에 저장
    * 키 : 무게
    * 원소 : 간선
* 반복의 각 회전에서
  * 두 개의 다른 배낭 속에 양끝점을 가진 최소 무게의 간선을 최소신장트리 T에 포함하고 두 배낭을 하나로 합친다.
* 반복이 완료되면 최소신장트리 T를 포함하는 한 개의 배낭만 남는다.

<figure><img src="../.gitbook/assets/image (4) (1).png" alt="" width="411"><figcaption></figcaption></figure>

* 반복의 회전마다 간선 (u, v)를 최소신장트리 T에 추가
* 시간복잡도 : $$O(mlogn)$$

#### Kruskal 알고리즘 수행 예시

<figure><img src="../.gitbook/assets/image (7).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (8).png" alt="" width="563"><figcaption></figcaption></figure>
