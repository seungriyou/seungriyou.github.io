---
title: "[Algorithm] Undirected & Directed Graph의 Cycle Detection"
date: 2024-03-18 19:50:00 +0900
categories: [Problem Solving, Algorithm Snippet]
tags: [algorithm, python, graph, cycle detection, union find, topological sort, bfs, dfs]
math: true
image: 
  path: /assets/img/posts/Problem-Solving/Algorithm/thumbnail.png
---

주어진 그래프 내에 **사이클이 존재하는지 여부**를 판단하는 방법은 해당 그래프가 **무방향 그래프(undirected graph)인지 방향 그래프(directed graph)인지에 따라** 다르다.

그래프에 종류에 따라 cycle detection을 수행하는 방법에 대해 코드로 알아보자!

<br>

## 1. Undirected Graph의 Cycle Detection
> 예제 문제: [**[LeetCode] 684. Redundant Connection**](https://leetcode.com/problems/redundant-connection/)
{: .prompt-info}

### 1-1. Disjoint Set (Union-Find)
<span class="shl">**서로소 집합**</span>을 이용하면 **무방향 그래프** 내에 사이클이 존재하는지 여부를 판단할 수 있다.

<br>

전체 동작은 다음과 같다.

1. 각 간선을 확인하며 두 노드의 루트 노드를 확인한다.
    1. 루트 노드가 서로 다르다면 두 노드에 대하여 `union` 연산을 수행한다.
    2. **루트 노드가 서로 같다면, 두 노드를 `union` 할 때 사이클이 발생**할 것이다.

2. 그래프에 포함되어 있는 모든 간선에 대하여 1번 과정을 반복한다.

```python
class Solution:
    def findRedundantConnection(self, edges: List[List[int]]) -> List[int]:
        # 특정 원소가 속한 집합 찾기
        def find_parent(parent, x):
            if parent[x] != x:
                parent[x] = find_parent(parent, parent[x])
            return parent[x]

        # 두 원소가 속한 집합 합치기
        def union_parent(parent, a, b):
            parent_a, parent_b = find_parent(parent, a), find_parent(parent, b)
            if parent_a < parent_b:
                parent[parent_b] = parent_a
            else:
                parent[parent_a] = parent_b

        # node 개수 == edge 개수 (node: 1 ~ n번)
        parent = [i for i in range(len(edges) + 1)]

        for a, b in edges:
            if find_parent(parent, a) == find_parent(parent, b):
                # ** cycle detected! **
                # n vertices and n edges, there can be only one cycle
                return [a, b]
            else:
                union_parent(parent, a, b)
```

<br>

### 1-2. DFS
<span class="shl">**DFS**</span>를 통해서도 **무방향 그래프** 내에 사이클이 존재하는지 여부를 판단할 수 있다.

```python
class Solution:
    def findRedundantConnection(self, edges: List[List[int]]) -> List[int]:
        graph = [[] for _ in range(len(edges) + 1)]

        def is_cyclic(u, v):
            if u == v:
                return True
            
            for nu in graph[u]:
                if nu not in visited:
                    visited.add(nu)
                    if is_cyclic(nu, v):
                        return True
            
            return False
        
        for u, v in edges:
            visited = set()

            if is_cyclic(u, v):
                # ** cycle detected! **
                # n vertices and n edges, there can be only one cycle
                return [u, v]
            
            graph[u].append(v)
            graph[v].append(u)
```

<br>

## 2. Directed Graph의 Cycle Detection
> 예제 문제: [**[LeetCode] 207. Course Schedule**](https://leetcode.com/problems/course-schedule/)
{: .prompt-info}

### 2-1. Topological Sort (BFS)
<span class="shl">**위상 정렬**</span>을 이용하여 모든 노드를 방향성에 어긋나지 않도록 순서대로 나열할 수 있으므로, **방향 그래프** 내에 사이클이 존재하는지 여부를 판단할 수 있다.

<br>

전체 동작은 다음과 같다.

1. 진입 차수가 0인 노드를 **큐**에 넣는다.

2. 큐가 빌 때까지 다음의 과정을 반복한다.
    1. 큐에서 원소를 꺼내 해당 노드에서 출발하는 간선을 그래프에서 제거한다.
    2. 새롭게 진입차수가 0이 된 노드를 큐에 넣는다.

3. 위의 동작을 반복하다가, **모든 원소를 방문하기 전에 큐가 빈다**면 사이클이 존재한다고 판단할 수 있다.

    > 사이클이 존재하는 경우, 사이클에 포함되어 있는 원소 중에서 어떠한 원소도 큐에 들어가지 못하기 때문이다. 따라서 기본적으로 위상 정렬 문제에서는 사이클이 발생하지 않는다고 명시하는 경우가 더 많다.

> 즉, 위상 정렬을 수행하며 **방문한 노드의 개수가 전체 노드의 개수와 같은지** 여부를 확인하면 된다.
{: .prompt-tip}

```python
from collections import deque

# numCourses: 노드의 개수
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        graph = [[] for _ in range(numCourses)]
        indegree = [0] * numCourses

        for a, b in prerequisites:
            graph[b].append(a)  # b -> a
            indegree[a] += 1

        def topo_sort():
            cnt = 0
            # indegree[i] == 0인 i부터 시작
            q = deque(i for i, ind in enumerate(indegree) if ind == 0)

            while q:
                pos = q.popleft()
                cnt += 1

                for npos in graph[pos]:
                    # npos의 indegree 감소
                    indegree[npos] -= 1
                    # npos의 indegree가 0이 되면 q에 넣기
                    if indegree[npos] == 0:
                        q.append(npos)

            return cnt

        cnt = topo_sort()

        return cnt == numCourses
```

<br>

### 2-2. 3-State DFS
> ref: <https://www.geeksforgeeks.org/detect-cycle-in-a-graph/>

<span class="shl">**3-state DFS**</span>를 이용하면 **방향 그래프** 내에 사이클이 존재하는지 여부를 판단할 수 있다.

- <span class="shlp">**일반적인 DFS**</span>는 **2-state**로, `visited: list[bool]`를 관리하며 각 노드가 visited 인지 unvisited 인지 두 가지 상태로 나눈다.

    | `visited[i]` | Description | Meaning |
    | ----- | ----------- | --- |
    | `False`   | unvisited   | 방문 가능 |
    | `True`  |  visited  | 방문 후보에서 제외 |

- 하지만 <span class="shlp">**cycle detection을 수행하는 DFS**</span>는 **3-state**를 사용하기 위해 `visited: list[int]`를 관리해야 하며, 각 state는 다음과 같다.

    > 해당 노드가 current path에 포함되는지 여부를 나타내는 state가 추가 되었다.

    | `visited[i]` | Description | Meaning |
    | --- | --- | --- |
    | `0` | unvisited |  방문 가능 |
    | `-1` | being visited this time (in current path) | 다시 마주친다면 <span class="red">**cycle 발생**</span> |
    | `1` | visited | 방문 후보에서 제외 |

<br>

#### [1] `visited` 리스트를 이용한 코드

```python
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        graph = [[] for _ in range(numCourses)]
        visited = [0] * numCourses

        for a, b in prerequisites:
            graph[b].append(a)  # b -> a


        def is_cyclic(pos):
            # <1> pos가 현재 보고 있는 경로에 포함된다면 cyclic
            if visited[pos] == -1:
                return True
            
            # <2> pos를 이전에 이미 방문했다면 방문 X
            if visited[pos] == 1:
                return False
            
            # <3> pos 방문하기
            visited[pos] = -1   # 현재 pos를 보고있다고 표시
            for npos in graph[pos]:
                if is_cyclic(npos):
                    return True

            visited[pos] = 1    # 현재 pos를 방문했다고 표시

            return False


        # 각 노드에서 출발
        for i in range(numCourses):
            if is_cyclic(i):
                return False

        return True
```

<br>

#### [2] `visited` & `current_path` 집합을 이용한 코드

[1]번 코드에서 `visited[i] == -1`인 상황을 `current_path`에 기록하고, `visited[i] == 1`인 상황을 `visited`에 기록하며 찾을 수 있도록 변경한 코드이다. 이렇게 풀면 더 직관적인 네이밍을 사용할 수 있게 된다.

```python
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        graph = [[] for _ in range(numCourses)]
        visited = set()
        current_path = set()

        for a, b in prerequisites:
            graph[b].append(a)  # b -> a
        

        def is_cyclic(pos):
            # <1> pos가 현재 보고 있는 경로에 포함된다면 cyclic
            if pos in current_path:
                return True
            
            # <2> pos를 이전에 이미 방문했다면 방문 X
            if pos in visited:
                return False
            
            # <3> pos 방문하기
            current_path.add(pos)       # 현재 pos를 보고있다고 표시
            for npos in graph[pos]:
                if is_cyclic(npos):
                    return True

            current_path.remove(pos)    # 현재 pos를 보고있다고 표시한 것 취소
            visited.add(pos)            # 현재 pos를 방문했다고 표시

            return False


        # 각 노드에서 출발
        for i in range(numCourses):
            if is_cyclic(i):
                return False
        
        return True
```
