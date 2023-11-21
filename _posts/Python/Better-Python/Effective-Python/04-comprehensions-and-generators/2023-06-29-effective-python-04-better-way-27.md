---
title: "[Better Way #27] map과 filter 대신 컴프리헨션을 사용하라"
date: 2023-06-29 12:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

리스트, 딕셔너리, 집합 등을 다룰 때 `map`과 `filter`를 사용하기보다는 컴프리헨션을 사용하는 것을 권장한다.

- `map`, `filter` 사용 버전
    
  ```python
  a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

  alt_list = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, a)))
  alt_dict = dict(map(lambda x: (x, x**2),
                      filter(lambda x: x % 2 == 0, a)))
  alt_set = set(map(lambda x: x**3,
                      filter(lambda x: x % 3 == 0, a)))
  ```
    
- 리스트 / 딕셔너리 / 집합 컴프리헨션 사용 버전
    
  ```python
  even_squares_list = [x**2 for x in a if x % 2 == 0]
  even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
  threes_cubed_set = {x**3 for x in a if x % 3 == 0}
  ```
