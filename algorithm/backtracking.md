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

# BackTracking

백트래킹(BackTracking)은 이름의 뜻과 유사한 알고리즘이다.

가능한 모든 경로를 탐색하다가, 원하는 조건이 아니면 더 이상 탐색을 하지 않고 이전 단계로 돌아가는 알고리즘이다.



### 백트래킹과 DFS

백트래킹과 DFS 알고리즘은 유사한 부분이 있다.

트리가 하나 있다고 생각했을 때, DFS는 모든 노드를 다 탐색한다.

하지만 백트래킹은 탐색하면서 조건에 해당하지 않은 부분이 있다면, 이전의 탐색으로 돌아간다.



하나의 트리를 예시로 들어보자.



<figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt="" width="250"><figcaption></figcaption></figure>

루트에서부터 각 레벨별로 숫자 1개씩 뽑아야 하고, 가운데 숫자가 2가 들어가면 안된다는 조건이 있다고 가정해보자. (4-2-1 -> X)

백트래킹을 이용하면 2로 들어가는 순간 중간 숫자가 2가 되어서 2 이후로의 탐색을 하지 않고 6으로 넘어갈 것이다.

하지만 DFS는 모든 경우를 탐색한다.
