---
title: "[Python] 숫자의 각 자릿수 구하기"
date: 2023-11-14 19:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, math]
math: true
---

주어진 정수 `num`에 대해 각 자릿수를 구하는 함수는 다음과 같다.

```python
def get_number(num):
    i = 1
    while i <= num:
        print(num // i % 10)
        i *= 10
    return
```

실행 결과는 다음과 같다.

```python
get_number(2345)
5
4
3
2
```
