---
title: "[Python] 정렬 기준 커스터마이징 하기"
date: 2023-10-05 19:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, sort, functools]
math: true
---

LeetCode의 [**179. Largest Number**](https://leetcode.com/problems/largest-number/) 문제를 풀면서 파이썬에서 정렬 기준을 커스터마이징 하는 방법을 알아보았다.

<br>

## [방법 1] `functools.cmp_to_key()` 사용하기
`functools.cmp_to_key()`는 다음과 같이 `sort()`, `sorted()` 함수의 `key` 파라미터로 전달할 수 있다.

```python
from functools import cmp_to_key

array.sort(key=cmp_to_key(compare))
```

이때, `compare`는 두 개의 인자 `x`, `y`를 입력 받은 후, 정의된 비교 동작을 통해 `-1`, `0`, `1` 중 하나의 값을 리턴함으로써 두 인자의 순서를 결정한다.

| Return Value | Description |
| --- | --- |
| `-1` | `x`가 `y`보다 먼저 |
| `0` | 두 인자가 동일함 |
| `1` | `y`가 `x`보다 먼저 |

```python
def compare(x, y):
    if int(x + y) > int(y + x):
        return 1
    elif int(x + y) < int(y + x):
        return -1
    else:
        return 0
```

<br>

따라서 문제를 다음과 같이 해결할 수 있다.

```python
from functools import cmp_to_key

# # 굳이 int() 변환할 필요 없음 (순차적으로 unicode 값을 비교하므로)
def compare(x, y):
    t1 = x + y
    t2 = y + x
    if t1 > t2:
        return -1   # -- x < y 순서
    elif t1 < t2:
        return 1    # -- y < x 순서
    else:
        return 0

 
def solution(numbers):
    nums = [str(n) for n in numbers]
    nums.sort(key=cmp_to_key(compare))
    return ''.join(nums).lstrip('0') or '0'
```

<br>

## [방법 2] 정렬 알고리즘 직접 구현하기

여러 정렬 알고리즘 중 하나를 직접 구현하는 방법으로, 나는 `O(nlogn)` time complexity를 가지는 merge sort를 구현했다.

```python
def merge_sort(arr):
    if len(arr) < 2:
        return arr
    
    mid = len(arr) // 2
    l_arr = merge_sort(arr[:mid])
    r_arr = merge_sort(arr[mid:])
    
    l = r = 0
    merged_arr = []
    while l < len(l_arr) and r < len(r_arr):
        # 굳이 int() 변환할 필요 없음 (순차적으로 unicode 값을 비교하므로)
        if l_arr[l] + r_arr[r] > r_arr[r] + l_arr[l]:
            merged_arr.append(l_arr[l])
            l += 1
        else:
            merged_arr.append(r_arr[r])
            r += 1
    
    merged_arr += l_arr[l:]
    merged_arr += r_arr[r:]
    
    return merged_arr

def solution(numbers):
    nums = [str(n) for n in numbers]
    return ''.join(merge_sort(nums)).lstrip('0') or '0'
```
