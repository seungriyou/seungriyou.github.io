---
title: "[Better Way #07] range 보다는 enumerate를 사용하라"
date: 2023-06-09 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch01 pythonic thinking]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 01. Pythonic Thinking"**을 읽고 정리한 내용입니다.

<br>

## `enumerate` 내장 함수

- 이터레이터를 지연 계산 제너레이터(lazy generator)로 감싼다.
- `(루프 인덱스, 이터레이터의 다음 값)` 쌍을 넘겨준다(`yield`).
- 두 번째 파라미터로 어디부터 수를 세기 시작할지 지정할 수 있다. (default: `0`)
    
  ```python
  flavor_list = ["바닐라", "초콜릿", "피칸", "딸기"]
  for i, flavor in enumerate(flavor_list, 1):
      print(f"{i}: {flavor}")
    
  # 1: 바닐라
  # 2: 초콜릿
  # 3: 피칸
  # 4: 딸기
  ```
    