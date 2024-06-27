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

# Union-Find

`Union-Find` 는 **상호배타적으로 이루어진 집합**을 효율적으로 표현하기 위한 자료구조이다. 즉, 공통 원소가 존재하지 않는 부분 집합들로 이루어져 있을 때 사용한다.

**서로 다른 두 개의 집합을 병합**하는 `Union` 연산과 **집합의 원소가 어떤 집합에 속해있는지 판단**하는 `Find` 연산이 존재한다.

### Find

`Find` 연산은 한 원소가 어떤 집합의 속해있는지 찾는 연산이다.

하나의 집합에 있는 각 원소들을 같은 집합에 속해있음을 나타내야 한다.

<figure><img src="../.gitbook/assets/image (4) (1).png" alt="" width="217"><figcaption></figcaption></figure>

`Find` 연산은 원소가 속해있는 집합의 루트 노트를 찾는다.

위 경우에서 `Find(1)` = `Find(3)` = `Find(2)` = `Find(4)` = 1 로 다 같다.

> `Find(x)` : x가 속한 집합의 **루트 노드** 값을 찾는다.

```java
int find(int x) {
		if (set[x] == x)
				return x;
		else
				return find(set[x]);
}
```

`set[x]` 는 x가 속해있는 집합을 뜻한다. 만약 `set[x]` 가 x가 아닌 경우, 재귀적으로 집합의 루트 노드를 찾아간다.

위의 이미지에서 `set` 은 다음과 같다.

| 0 | 1 | 2 | 3 | 4 | 5 |
| - | - | - | - | - | - |
| 0 | 1 | 1 | 1 | 2 | 5 |

### Union

`Union` 연산은 두 원소가 속한 집합을 합친다.

<figure><img src="../.gitbook/assets/image (5) (1).png" alt="" width="187"><figcaption></figcaption></figure>

위 상황에서 `Union(5, 2)` 의 경우 5와와 2가 속한 각각의 집합을 하나의 집합으로 합친다.

> `Union(x, y)` : x와 y가 속한 두 집합을 합친다.

```java
void union(int x, int y) {
		int rootX = find(x);
		int rootY = find(y);
		
		set[rootX] = rootY;
}
```

위 이미지에서 `Union(5, 2)` 를 수행하면 다음과 같이 `set` 은 변경된다.

<figure><img src="../.gitbook/assets/image (6) (1).png" alt="" width="194"><figcaption></figcaption></figure>

| 0 | 1 | 2 | 3 | 4 | 5 |
| - | - | - | - | - | - |
| 1 | 1 | 1 | 1 | 2 | 0 |

### Union-Find 최적화

만약 트리 구조가 비대칭인 경우 어떻게 될까?

<figure><img src="../.gitbook/assets/image (7) (1).png" alt="" width="202"><figcaption></figcaption></figure>

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2c55e26f-a197-45df-a067-a452102c4060/0ff692fe-357a-4128-897b-c37ef1d10208/Untitled.png)

| 0 | 1 | 2 | 3 | 4 | 5 |
| - | - | - | - | - | - |
| 0 | 0 | 1 | 2 | 3 | 4 |

`Find(5)` 의 경우 `Find` 연산을 5회 반복해야 한다. 위 그림을 다음과 같이 변경한다면 훨씬 효율적이다.

<figure><img src="../.gitbook/assets/image (8) (1).png" alt="" width="199"><figcaption></figcaption></figure>

| 0 | 1 | 2 | 3 | 4 | 5 |
| - | - | - | - | - | - |
| 0 | 0 | 0 | 0 | 0 | 0 |

#### Find 연산 최적화

<figure><img src="../.gitbook/assets/image (9) (1).png" alt="" width="194"><figcaption></figcaption></figure>

이 상황에서 `Find(2)` 를 수행했을 때, `2-1-0` 으로 타고 올라가게 된다.

이때, `set[2]` 와 `set[1]` 의 값을 집합의 루트노드로 바꿔준다. (set\[1]은 이미 0)

<figure><img src="../.gitbook/assets/image (10).png" alt="" width="199"><figcaption></figcaption></figure>

| 0 | 1 | 2     | 3 | 4 | 5 |
| - | - | ----- | - | - | - |
| 0 | 0 | 1 → 0 | 1 | 0 | 0 |

`Find` 연산을 하면서 위 작업을 수행한다면, 다음 `Find` 연산을 수행할 때 비교적 시간복잡도가 낮아진다.

```java
int find(int x) {
		if (set[x] == x)
				return x;
		else {
				set[x] = find(set[x]);
				return set[x];
		}
}
```

#### Union 연산 최적화

두 집합을 합칠 때, 높이가 더 낮은 집합을 높이가 더 높은 집합 밑에 넣는다.

<figure><img src="../.gitbook/assets/image (11).png" alt="" width="185"><figcaption></figcaption></figure>

높이가 더 높은 집합(높이 N)을 높이가 더 낮은 집합(높이 M) 밑으로 합치는 경우, 시간복잡도가 `O(N + M - 1) = 4`가 된다.

<figure><img src="../.gitbook/assets/image (13).png" alt="" width="166"><figcaption></figcaption></figure>

하지 높이가 더 낮은 집합을 높이가 더 높은 집합에 합치는 경우, `O(N) = 3`으로 유지된다.

<figure><img src="../.gitbook/assets/image (12).png" alt="" width="180"><figcaption></figcaption></figure>

```java
void union(int x, int y) {
		int rootX = find(x);
		int rootY = find(y);
		
		if (rootX == rootY)
				return;
		
		/* 높이가 더 낮은 트리를 높이가 높은 트리 밑으로 넣는다. */
		if (rank[rootX] < rank[rootY])
				root[rootX] = rootY;
		else {
				root[rootY] = rootX;
				/* 만약 높이가 같다면 합친 후 rootX의 높이를 +1 한다. */
				if (rank[rootX] == rank[rootY])
						rank[rootX]++;
		}
}
```
