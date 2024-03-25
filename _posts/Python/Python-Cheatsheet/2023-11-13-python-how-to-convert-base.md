---
title: "[Python] 10진수를 n진수로 변환하는 함수"
date: 2023-11-13 19:50:00 +0900
categories: [Python, Python Cheatsheet]
tags: [cheatsheet, python, math, base]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

`n`이 16 이하인 값일 때, 10진수인 `num`을 `n`진수로 변환하는 함수는 다음과 같다.

```python
def n_base(n, num):
    DIGITS = '0123456789ABCDEF'
    if num == 0:
        return DIGITS[0]

    result = ''
    while num > 0:
        d, m = divmod(num, n)
        result += DIGITS[m]
        num = d
    return result[::-1]
```
