---
title: "[Algorithm] Quick Select: K 번째 원소 찾기"
date: 2024-04-16 09:50:00 +0900
categories: [Problem Solving, Algorithm Snippet]
tags: [algorithm, python, selection]
math: true
---

## 1. Selection Algorithm

<span class="ulr">**정렬되지 않은 리스트나 배열에서 k 번째 수를 찾는 문제**</span>를 <span class="shl">**selection 문제**</span>라고 한다.

selection 문제를 해결하는 방법은 크게 세 가지로 나뉜다.

1. <span class="shlp">**오름차순 정렬 후 index로 접근**</span> ($O(n\log n)$)
2. <span class="shlp">**heap 사용**</span> ($O(n\log k)$)
3. <span class="shlp">**selection 알고리즘**</span> (ex. quick select)

효율적인 정렬 알고리즘의 시간 복잡도가 $O(n\log n)$ 이므로, **selection 문제는 $O(n\log n)$ 내에 해결**할 수 있다.

<br>

## 2. Quick Select

**quick sort와 유사하게 partitioning**을 이용하지만, <span class="shl">**binary search 처럼 k 번째 요소가 속한 부분만을 확인**</span>한다는 점에서 다르다.

**partitioning**이란, <span class="ulr">임의의 값인 **pivot을 기준**</span>으로 pivot 보다 <span class="ulr">작은 수는 왼쪽</span>에, <span class="ulr">큰 수는 오른쪽</span>에 위치하도록 요소를 배치하는 방법이다.

partitioning 결과로 나뉜 두 부분 중, **k 번째 요소가 속한 부분에 대해서만** 재귀적으로 quick select를 수행하면 된다.

| `pivot_index`의 크기 | 동작 |
| --- | --- |
| `pivot_index > k` | left part에 대해 재귀 수행 |
| `pivot_index < k` | right part에 대해 재귀 수행 |
| `pivot_index == k` | `k` 번째 원소 발견! 🎉 |

<br>

## 3. Code

```python
def partition(left, right):
    # pivot보다 작은 원소들 이후의 첫 번째 index를 가리키게 되므로,
    # 마지막에 pivot과 swap 되면 pivot의 index를 나타냄
    p = left

    # 배열의 맨 오른쪽 값을 pivot으로 설정
    pivot = arr[right]

    for i in range(left, right):
        if arr[i] <= pivot:
            arr[i], arr[p] = arr[p], arr[i]
            p += 1

    arr[p], arr[right] = arr[right], arr[p]

    return p

def quick_select(left, right):
    pivot_index = partition(left, right)

    if pivot_index > k:     # -- left part
        return quick_select(left, pivot_index - 1)
    elif pivot_index < k:   # -- right part
        return quick_select(pivot_index + 1, right)
    else:                   # -- find kth element
        return arr[pivot_index]

quick_select(0, len(nums) - 1)
```

<br>

## 4. Time Complexity

> quick sort와 마찬가지로 <span class="shl">**pivot 값에 따라**</span> 알고리즘의 성능이 결정된다.
{: .prompt-tip}

- **best case**: $O(n)$
    
    > 선택된 pivot 값이 k 번째 수인 경우
    > 
- **worst case**: $O(n^2)$
    
    > 선택된 pivot 값이 배열 내 가장 작거나 큰 값인 경우
    > 
- **average**: $O(n)$
    
    > pivot 값을 기준으로 배열이 1/2로 나누어지는 경우, $n+\frac{n}{2}+\frac{n}{4}+\dots+\frac{n}{2^k}=2n$
    > 

<br>

## 5. Related Problem

> LeetCode의 [**973. K Closest Points to Origin**](https://leetcode.com/problems/k-closest-points-to-origin/) 문제를 풀어보자!
{: .prompt-info}

### [방법 #1] Max Heap 구성하기

max heap의 길이를 최대 `k`로 유지한다.

```python
import heapq

class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        """
        heapq
        - TC: O(nlogk)
        - SC: O(k)
        """

        def get_distance(point):
            x, y = point
            return x * x + y * y
        
        q = []
        for point in points:
            heapq.heappush(q, (-get_distance(point), point))  # as max heap

            if len(q) > k:
                heapq.heappop(q)
        
        return [point for _, point in q]
```

<br>

### [방법 #2] Quick Select 응용하기 (<span class="blue">Randomized Quick Select</span>)

1. `partition(left, right)`에서 `pivot`을 선택할 때 random 하게 선택하는 <span class="shl">**randomized quick select**</span>를 사용하였다.
    
    - `pivot`을 맨 오른쪽(`right`) 원소로 설정한 코드와 비교했을 때, 실제로 제출 시 실행 시간이 1/10로 단축되었다.

        > quick select도 quick sort와 마찬가지로, `pivot`을 최댓값 혹은 최솟값으로 설정하면 $O(n^2)$이기 때문이다.

    - randomized quick select를 구현하기 위해서는 **(1) `randint(left, right)`로 `random`을 구하고**, **(2) 이를 `right`와 swap**만 해주면 된다. 이렇게 되면 선택된 **random pivot이 맨 오른쪽(`right`)에 위치**하게 된다.

2. 이 문제에서는 "값"이 아닌 "거리"를 기준으로 하고 있으므로, 단순히 pivot의 값을 비교하는 것이 아닌 <span class="shl">**`i`가 가리키는 원소의 distance를 비교**</span>해야 함에 주의한다.

    > `points[right]`가 아닌 `get_distance(points[right])`를, `points[i]`가 아닌 `get_distance(points[i])`와 비교해야 한다.
    {: .prompt-danger}

3. 적당한 `pivot_index`를 찾은 순간 실행을 종료하기 위해 더 직관적인 <span class="shl">**iterative 형식**</span>으로 작성했다.

    > recursive 형식뿐만 아니라 iterative 형식으로도 작성 가능하다!


```python
class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        """
        quick select
        - TC: O(n)
        - SC: O(1)
        """

        def get_distance(point):
            x, y = point
            return x * x + y * y

        def partition(left, right):
            p = left

            # pivot를 random하게 선택하고, right와 swap
            # 즉, pivot을 right 위치로 이동시킴
            random = randint(left, right)
            points[random], points[right] = points[right], points[random]

            # pivot의 distance 구하기
            pivot_distance = get_distance(points[right])

            for i in range(left, right):
                # 현재 보고 있는 point의 distance가 pivot distance 이하이면 p와 swap
                if get_distance(points[i]) <= pivot_distance:
                    points[i], points[p] = points[p], points[i]
                    p += 1
            
            # 마지막으로 p와 right swap
            # - p를 기준으로, points[p]의 distance보다 작은 distance인 point는 왼쪽에,
            # - points[p]의 distance보다 큰 distance인 point은 오른쪽에 위치
            points[p], points[right] = points[right], points[p]

            return p

        # iterative quick select
        left, right, pivot_index = 0, len(points) - 1, len(points)
        while pivot_index != k:
            pivot_index = partition(left, right)
            if pivot_index < k:
                left = pivot_index + 1
            else:
                right = pivot_index - 1
        
        return points[:k]
```

<br>

## References

- <https://www.geeksforgeeks.org/quickselect-algorithm/>
- <https://www.daleseo.com/quick-select/>
- <https://devraphy.tistory.com/372>
- <https://dad-rock.tistory.com/351>
- <https://leetcode.com/problems/k-closest-points-to-origin/solutions/1647773/c-python-simple-solutions-w-explanation-sort-heap-randomized-quickselect-o-n>
- <https://stackoverflow.com/questions/66625693/quick-select-with-random-pick-index-or-with-median-of-medians>
- <https://stackoverflow.com/questions/29980907/why-choose-a-random-pivot-in-quicksort>
