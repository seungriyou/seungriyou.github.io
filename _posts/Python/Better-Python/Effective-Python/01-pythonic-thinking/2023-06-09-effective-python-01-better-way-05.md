---
title: "[Better Way #05] 복잡한 식을 쓰는 대신 도우미 함수를 작성하라"
date: 2023-06-09 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

- **복잡한 식**은 도우미 함수로 옮기고, 특정 로직을 **반복 적용**하려면(단지 두세 번에 불과할지라도) 도우미 함수를 작성하는 것을 권장한다.
- 복잡하지만 한 줄 짜리인 식에 `bool` 연산자 **`or`나 `and`**를 사용하는 것보다 **`if`/`else` 식**을 사용하는 편이 더 가독성이 좋다.
- 코드를 줄여쓰는 것보다 가독성을 좋게 하는 것이 더 가치있다.
- ex) query string을 parsing 하는 예제
  
  ```python
  from urllib.parse import parse_qs
  
  my_values = parse_qs("빨강=5&파랑=0&초록=", keep_blank_values=True)
  print(repr(my_values)) # {'빨강': ['5'], '파랑': ['0'], '초록': ['']}
  ```
  
  ```python
  # query string을 parsing하는 동작이 반복적으로 필요할 경우, 도우미 함수를 작성
  def get_first_int(values, key, default=0):
      found = values.get(key, [""])
      if found[0]:
          return int(found[0])
      return default
  
  red = get_first_int(my_values, "빨강")
  green = get_first_int(my_values, "초록")
  opacity = get_first_int(my_values, "투명도")
  print(f"빨강: {red!r}") # 빨강: 5
  print(f"초록: {green!r}") # 초록: 0
  print(f"투명도: {opacity!r}") # 투명도: 0
  ```
