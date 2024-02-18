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

# Dijkstra

**다익스트라 알고리즘**은 Dynamic Programming을 활용한 `최단경로 탐색 알고리즘`이다.  다익스트라 알고리즘은 특정한 하나의 정점에서 다른 모든 정점으로 가는 최단 경로를 알려준다. 하지만 **이때 음의 간선을 포함할 수 없다.**

다익스트라 알고리즘이 Dynamic Programming 문제인 이유는 **최단 경로는 여러 개의 최단 경로로 이루어져있기 때문**이다. 즉, 작은 문제가 큰 문제의 부분 집합에 속해있다.

### 동작 과정

1. 출발 노드를 설정
2. 출발 노드를 기준으로 각 노드의 최소 비용을 저장
3. 방문하지 않는 노드 중 가장 비용이 적은 노드 선택
4. 해당 노드를 거쳐서 특정 노드로 가는 경우를 고려하여 최소 비용 갱신
5. 3\~4번을 반복한다.

```python
def dijkstra(start):
    q = []
    distance = [sys.maxsize] * (n + 1)
    distance[start] = 0
    visited = [False] * (n + 1)
    heapq.heappush(0, (0, start))
    while q:
        dist, node = heapq.heappop(q)
        if visited[node]:
            continue
        for i in range(1, n+1):
            if cost[node][i] == sys.maxsize:
                continue
            if distance[i] > distance[node] + cost[node][i]:
                distance[i] = distance[node] + cost[node][i]
                visited[node] = False
                heapq.heappush(q, (distance[i], i))
```

> heapq에 튜플이 삽입될 경우엔, 튜플의 첫 번째 요소가 정렬의 기준이 된다.
