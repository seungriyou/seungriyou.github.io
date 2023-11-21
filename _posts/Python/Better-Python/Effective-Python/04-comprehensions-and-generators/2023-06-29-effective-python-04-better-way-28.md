---
title: "[Better Way #28] 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라"
date: 2023-06-29 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

## 컴프리헨션 사용 시 권장 사항

- 컴프리헨션에 들어가는 **하위 식이 세 개 이상 되지 않도록 제한**하는 것을 권장한다.

    > ex) (조건문 두 개), (루프 두 개), (조건문 한 개 + 루프 한 개)
    
- 컴프리헨션이 더 복잡해지는 경우, 일반 **`if` 문과 `for` 문을 사용**하고 **도우미 함수**를 작성하라.

<br>

## 컴프리헨션의 특징

- 동일 레벨의 두 루프는 **왼쪽에서부터** 순서대로 실행된다.
    
  ```python
  matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
  flat = [x for row in matrix for x in row]
  ```
    
- 컴프리헨션은 여러 `if` 조건을 허용하며, 여러 조건을 **같은 수준**의 루프에 사용하면 **암시적으로 `and` 식**을 의미한다.
    
  ```python
  b = [x for x in a if x > 4 if x % 2 == 0]
  c = [x for x in a if x > 4 and x % 2 == 0] # -- b와 동일
  ```
