---
title: "[Algorithm] Floyd’s Cycle Detection: Linked List에 Cycle이 존재하는지 판단하기"
date: 2023-12-15 11:50:00 +0900
categories: [Problem Solving, Algorithm Snippet]
tags: [algorithm, python, linked list, cycle detection]
math: true
---

## Floyd’s Cycle Detection이란?

Floyd’s Cycle Detection 알고리즘은 속도가 다른 두 pointer를 이용하여 <span class="hl">$O(1)$의 space complexity</span>로 <span class="hl">linked list에 cycle 혹은 loop가 존재하는지 여부</span>를 검사하는 알고리즘이다.

이때, <span class="hl">fast pointer는 slow pointer의 두 배 빠른 속도</span>로 이동하므로 Hare-Tortoise 알고리즘으로 불리기도 한다.

```python
fast = fast.next.next  # -- 두 칸 이동
slow = slow.next       # -- 한 칸 이동
```

<br>

> cycle detection을 위해서는 hash table을 사용할 수도 있으나 space complexity가 $O(N)$ 이 된다. 따라서 $O(1)$ space complexity constraint 조건이 주어진다면 Floyd’s Cycle Detection을 사용하자.
>
> ```python
> # hash table을 사용하는 방법 (O(N) space)
> 
> class ListNode:
>     def __init__(self, x):
>         self.val = x
>         self.next = None
> 
> class Solution:
>     def hasCycle(self, head: Optional[ListNode]) -> bool:
>         p = head
>         visited = set()
> 
>         while p is not None:
>             if p in visited:
>                 return True
>             visited.add(p)
>             p = p.next
> 
>         return False
> ```
{: .prompt-info}

<br>

### Main Idea

해당 linked list에 loop가 없다면 fast pointer는 `NULL`, 즉, tail에 도달하게 된다.

하지만 해당 linked list에 loop가 존재한다면, 언젠가는 <span class="hl">fast pointer와 slow pointer는 만나게 된다</span>.

```python
if fast == slow:
    # fast pointer meets slow pointer!
```

<br>

### Complexity

| --- | --- |
| **Time Complexity** | $O(N)$ |
| **Space Complexity** | $O(1)$ (only for two pointers) |

<br>

## Floyd’s Cycle Detection 적용 예시

### [1] Cycle 존재 여부 판단

1. slow pointer는 한 칸씩, fast pointer는 두 칸씩 이동시킨다.
2. 두 pointer가 만나게 되면 cycle이 존재하는 것으로 판단하고, fast pointer가 `NULL`에 도달하면 cycle이 존재하지 않는 것으로 판단한다.

```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = fast = head

        while fast is not None and fast.next is not None:
            fast = fast.next.next
            slow = slow.next
            if fast == slow:
                # print("cycle detected!")
                return True

        return False
```

<br>

### [2] Cycle 길이 계산

1. cycle을 찾은 후(즉, slow pointer와 fast pointer가 만난 후), fast pointer를 저장한다.
2. slow pointer를 다시 한 칸씩 이동시키며 fast pointer를 다시 만날 때까지 길이를 1씩 증가시킨다.

```python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = fast = head

        while fast is not None and fast.next is not None:
            fast = fast.next.next
            slow = slow.next
            if fast == slow:
                # print("cycle detected!")
                print(self.calculateCycleLength(fast))
                return True

        return False
    
    def calculateCycleLength(self, fast: Optional[ListNode]) -> int:
        pos = fast
        cycle_length = 0
        
        while True:
            pos = pos.next
            cycle_length += 1
            if pos == fast:
                break
        
        return cycle_length
```

<br>

### [3] Cycle 시작점 찾기

![floyd's cycle detection](/assets/img/posts/Problem-Solving/Algorithm/23-12-15-01.png)
_ref: <https://yuminlee2.medium.com/floyds-cycle-detection-algorithm-b27ed50c607f>_

1. cycle을 찾은 후(즉, slow pointer와 fast pointer가 만난 후), fast pointer는 그 자리에 그대로 두고 slow pointer는 시작점으로 옮긴다.
2. slow pointer(`head`에서 시작)와 fast pointer(`meeting point`에서 시작)를 동시에 한 칸씩 이동시킨다. 그러다가 두 pointer가 만나는 지점이 cycle의 시작 node가 된다.
    
    > slow pointer는 `a`만큼, fast pointer는 `c`만큼 이동하고 만나게 되는데, 사실 `a` == `c`이므로 만나게 되는 지점이 cycle의 시작 지점이 된다. 그 이유는 cycle을 찾을 때 fast pointer가 slow pointer의 2배 속도로 이동했기 때문이다.
    >

```python
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        slow = fast = head

        while fast is not None and fast.next is not None:
            fast = fast.next.next
            slow = slow.next

            if fast == slow:
                # slow를 head로 옮기기
                slow = head
                # slow와 fast가 다시 만날 때까지 한 칸씩 이동
                while slow != fast:
                    slow = slow.next
                    fast = fast.next

                return slow

        return None
```

<br>

## Related Problems

- <https://leetcode.com/problems/linked-list-cycle/>
- <https://leetcode.com/problems/linked-list-cycle-ii/>
- <https://leetcode.com/problems/find-the-duplicate-number/>

<br>

## References

- <https://www.geeksforgeeks.org/floyds-cycle-finding-algorithm/>
- <https://yuminlee2.medium.com/floyds-cycle-detection-algorithm-b27ed50c607f>
- <https://medium.com/swlh/floyds-cycle-detection-algorithm-32881d8eaee1>
