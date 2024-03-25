---
title: "[Python] Bitwise & 연산을 이용하여 여러 Dictionary 간 교집합 구하기"
date: 2023-12-11 19:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, bitwise, dictionary]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

LeetCode의 [**383. Ransom Note**](https://leetcode.com/problems/ransom-note/)를 풀다가 여러 dictionary 간의 intersection을 구하는 방법을 찾아보게 되었다.

<br>

[**Bitwise Operators in Python**](https://realpython.com/python-bitwise-operators/#built-in-data-types) 문서에 따르면, 파이썬의 bitwise operators는 다음의 built-in data type들에 대해 정의되어 있다고 한다.

- `int`
- `bool`
- [`set`](https://realpython.com/python-sets/) and [`frozenset`](https://realpython.com/python-sets/#frozen-sets)
- [`dict`](https://realpython.com/python-dicts/) (<span class="hl">since Python 3.9</span>)

<br>

따라서 다음과 같이 bitwise `&` 연산자를 이용하여 dictionary 간 intersection을 구할 수 있다.

```python
print(f"{cnt1=}")
print(f"{cnt2=}")
print(f"{cnt1&cnt2=}")
```

```python
cnt1=Counter({'a': 2})
cnt2=Counter({'a': 1, 'b': 1})
cnt1&cnt2=Counter({'a': 1})
```
