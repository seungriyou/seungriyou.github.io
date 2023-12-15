---
title: "[Python] Sequence에서 홀수 번 등장하는 원소 찾기 (w/ Hash Table)"
date: 2023-12-15 16:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, hash table, sequence]
math: true
---

어떤 sequence에 대해 홀수 번 등장하는 원소를 구하거나, 각 원소의 등장 횟수가 홀수 번인지 짝수 번인지 판단해야 하는 경우가 있다.

<br>

가장 나이브하게 생각하면 `Counter`를 만들고, 각 item을 순회하면서 확인하면 된다.

다른 방법으로는 **hash table(Python의 경우, `set`)**을 사용하는 방법이 있다.

<br>

간단히 이야기하면 다음과 같다.

> **Hash Table을 이용하여 Sequence에서 홀수 번 등장하는 원소 찾기**
> 
> 1. 전체 원소를 순회하면서 hash table에 존재하는 원소이면 `remove` 하고, 존재하지 않는 원소이면 `add` 한다.
> 2. 최종 hash table에는 홀수 번 등장하는 원소만 존재한다.
{: .prompt-info}

<br>

따라서 hash table이 비어있는지 여부만 검사하면 홀수 번 등장하는 원소가 존재하는지 여부를 추가 로직 없이 빠르게 알 수 있고, 해당 원소가 무엇인지도 바로 알 수 있다.

<br>

예제로 LeetCode의 [**409. Longest Palindrome**](https://leetcode.com/problems/longest-palindrome/)을 풀어보면 다음과 같이 hash table을 적용해서 풀 수 있다.

```python
class Solution:
    def longestPalindrome(self, s: str) -> int:
        appears_odd = set()

        for c in s:
            if c in appears_odd:
                appears_odd.remove(c)
            else:
                appears_odd.add(c)
        
        return len(s) - len(appears_odd) + 1 if appears_odd else len(s)
```
