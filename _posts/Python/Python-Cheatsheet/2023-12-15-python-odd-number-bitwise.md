---
title: "[Python] Bitwise & 연산을 이용하여 홀수 판단 하기"
date: 2023-12-15 15:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, math, bitwise]
math: true
---

홀수를 판단하는 기본적인 방법은 2로 나눈 나머지가 1인지 확인하는 것이다.

```python
if c % 2 == 1:
```

이때, 다음과 같이 bitwise `&` 연산을 사용하면 성능을 optimize 할 수 있다.

```python
if c & 1:
```
