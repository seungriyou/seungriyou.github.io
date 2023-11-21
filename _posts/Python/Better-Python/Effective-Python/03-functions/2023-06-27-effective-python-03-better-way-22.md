---
title: "[Better Way #22] 가변 위치 인자를 사용해 시각적인 잡음을 줄여라"
date: 2023-06-27 20:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

## 위치 기반 인자 (positional argument) - `*args`

- **마지막 위치 인자 이름 앞에 `*`**를 붙이면 그 인자는 **가변 위치 기반 인자**가 되며, 그 이후의 모든 위치 기반 인자는 **선택 사항**이 된다.
    
  ```python
  def log(message, *values):
      if not values:
          print(message)
      else:
          values_str = ", ".join(str(x) for x in values)
          print(f"{message}: {values_str}")
          
  log("내 숫자는", 1, 2)
  log("안녕")

  # 내 숫자는: 1, 2
  # 안녕
  ```
    
- 가변 인자 함수에 이미 존재하는 시퀀스(ex. 리스트)를 사용하고 싶다면 **`*` 연산자**를 사용하면 된다. 이는 시퀀스의 원소들을 함수의 위치 인자로 넘길 것을 명령한다.

<br>

## 가변 위치 기반 인자의 문제점

1. 선택적인 위치 인자가 함수에 전달되기 전에 **항상 튜플로 변환**된다.
    - **제너레이터**를 인자로 넘기는 경우(`func(*generator)`), 튜플은 제너레이터가 만들어낸 모든 값을 포함하며 메모리를 아주 많이 소비할 수 있다.
    - 가변적인 부분에 들어가는 인자의 개수가 처리하기 좋을 정도로 충분이 적다는 사실을 알고 있는 경우에만 적합하다.
2. 함수에 **새로운 위치 인자를 추가**하면 해당 함수를 호출하는 모든 코드를 **변경**해야 한다.
    - `*args`를 받아들이는 함수를 확장할 때는 키워드 기반의 인자만 사용해야 한다.

