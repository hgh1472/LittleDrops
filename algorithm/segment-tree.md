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

# Segment Tree

세그먼트 트리는 특정 구간의 부분합을 구하기 위해 사용되고 이진 트리의 형태를 띄고 있다.

세그먼트 트리는 $$O(logn)$$의 시간복잡도로 부분합을 구할 수 있다.

먼저 단순한 방법으로 구간의 숫자를 전부 더해 부분합을 구하는 경우, $$O(n)$$ 시간복잡도를 가질 것이다. 또는 각 원소까지의 합을 나타내는 배열을 이용한다면, 배열을 입력받은 후$$O(1)$$ 에도 가능할 것이다.

하지만, 원소의 값이 바뀐다면 어떻게 해야할까? 특정 원소값이 바뀐다면, 결국 합을 나타내는 배열을 사용하더라도 $$O(n)$$ 이 소요된다.

따라서 $$O(logn)$$ 이 소요되는 세그먼트 트리를 사용한다.

세그먼트 트리는 다음과 같이 이루어져있다.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

각 노드 안의 숫자는 합의 범위를 뜻한다.

즉 루트노드는 0\~5번 원소의 합, 각 리프 노드는 각 원소 자체의 값을 뜻한다.

그렇다면 이 구조를 통해 어떻게 생성하고 합을 구하고 값을 변경할 수 있을까?

#### 생성

간단하게 구현하기 위해 배열(tree)을 이용한다. 원소의 개수가 N개일 때, 4N만큼의 크기의 배열을 생성한다. 넉넉한 크기의 배열을 생성하는 것이다.

이 때 `tree` 의 인덱스의 시작을 1이라고 설정하는 것이 편하다. 특정 노드에서 `인덱스 * 2`는 왼쪽 자식 노드, `인덱스 * 2 + 1`은 오른쪽 자식 노드를 가르키게 되기 때문이다.

생성은 재귀를 이용한다. 루트에서부터 시작해서 범위가 1인 리프 노드에 도달하게 되면 원소값을 저장한다. 범위가 2 이상이라면 왼쪽 자식의 노드값과 오른쪽 자식의 노드값을 더한다.

```java
// start와 end는 원소가 존재하는 범위를 인자로 넣어준다.
// index는 루트의 인덱스를 넣어준다.
public static long init(int start, int end, int index) {
    if (start == end) {
        tree[index] = arr[start];
        return tree[index];
    }
    int mid = (start + end) / 2;
    tree[index] = init(start, mid, index * 2) + init(mid+1, end, index * 2 + 1);
    return tree[index];
}
```

#### 구간합 구하기

우리가 구하고자 하는 구간이 2\~5 라고 해보자.

그렇다면 각 노드의 구간이 있을 때 3가지 경우로 나눌 수 있다.

* 노드의 구간이 원하는 구간 밖에 있을 때 ⇒ 0
* 노드의 구간 내에 원하는 구간에 존재할 때 ⇒ 왼쪽 자식과 오른쪽 자식으로 나누어 조사한다.
* 노드의 구간이 원하는 구간 내에 존재할 때 ⇒ 그 값을 더한다.

위의 3가지 케이스를 코드로 구현하면 다음과 같다.

```java
// start, end는 현재 메소드에서 탐색하는 구간 => 초기는 1, n
// left, right는 부분합을 알고싶은 구간
// index는 탐색하는 노드의 위치 => 루트에서부터 시작
public static long getSum(int start, int end, int left, int right, int index) {
    // 노드의 구간이 원하는 구간 밖에 존재
    if (end < left || right < start)
        return 0;
    // 노드의 구간이 원하는 구간 내에 존재
    if (left <= start && end <= right)
        return tree[index];
    // 노드의 구간 내에 원하는 구간에 존재할 때 => 왼쪽 오른쪽으로 나누어 조사
    int mid = (start + end) / 2;
    return getSum(start, mid, left, right, index * 2) + getSum(mid + 1, end, left, right, index*2+1);
}
```

예를 들어, 위의 이미지에서 우리가 원하는 구간이 \[4, 5]라고 해보자.

* \[0, 5]
  * \[3, 5]
    * \[3, 4]
      * `[4]`
    * `[5]`

\[0, 5]에서부터 타고 내려가서 결국 \[4]와 \[5]를 더해서 \[4, 5] 구간합을 반환하게 된다.

#### 값 변경

세그먼트 트리에서 특정 원소의 값을 변경하게 된다면, 왼쪽 자식 노드와 오른쪽 자식 노드 중에 해당 원소 값을 포함한 부분합의 노드일 경우 값을 변경해주면 된다.

```java
// start, end는 메소드에서 노드의 부분합 범위
// index는 메소드에서 탐색하는 현재 노드
// what은 값이 변경된 원소 번호
// value는 변경된 값이 아닌 (현재값 - 변경될 값)을 나타낸다.
public static void update(int start, int end, int index, int what, Long value) {
		// 해당 원소가 구간내에 포함되어 있지 않다면 패스
    if (what < start || what > end)
        return;
    tree[index] += value;
    if (start == end)
        return;
    int mid = (start + end) / 2;
    update(start, mid, index * 2, what, value);
    update(mid+1, end, index*2+1, what, value);
}
```

**참고**

[https://velog.io/@kimdukbae/자료구조-세그먼트-트리-Segment-Tree](https://velog.io/@kimdukbae/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%EC%84%B8%EA%B7%B8%EB%A8%BC%ED%8A%B8-%ED%8A%B8%EB%A6%AC-Segment-Tree)
