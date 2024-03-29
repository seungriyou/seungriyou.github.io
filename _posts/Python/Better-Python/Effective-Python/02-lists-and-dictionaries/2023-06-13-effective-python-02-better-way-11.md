---
title: "[Better Way #11] 시퀀스를 슬라이싱하는 방법을 익혀라"
date: 2023-06-13 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

## 시퀀스 슬라이싱

- 맨 앞부터 슬라이싱할 때는 0을 생략한다.
- 끝까지 슬라이싱할 때는 끝 인덱스를 생략한다.
- 끝에서부터 원소를 찾고 싶다면 음수 인덱스를 사용한다.
- 리스트의 **인덱스 범위를 넘어가는** 시작과 끝 인덱스는 **조용히 무시**된다. (하지만 같은 인덱스에 직접 접근하면 예외가 발생한다.)
    
    > 따라서 시퀀스의 시작이나 끝에서 길이를 제한하는 슬라이스(`a[:20]`이나 `a[-20:]`)를 쉽게 표현할 수 있다.
    > 
- 리스트를 슬라이싱한 결과는 **완전히 새로운 리스트**이다.
- 원래 시퀀스에서 슬라이스가 가리키는 부분을 대입 연산자 오른쪽에 있는 시퀀스로 **대입**할 수 있으며, 이때 슬라이스와 대입되는 리스트의 **길이가 같을 필요가 없다**.
    
    > 따라서 결과 리스트의 길이는 원본 리스트의 길이와 달라질 수 있다.
    > 
- 슬라이싱에서 시작과 끝 인덱스를 모두 생략(`[:]`)하면 **원래 리스트를 복사한 새 리스트**를 얻게 된다.
- 시작과 끝 인덱스가 없는 슬라이스(`[:]`)에 대입하면, **슬라이스가 참조하는 리스트의 내용**을 대입하는 리스트(연산자 오른쪽)의 복사본으로 덮어쓴다.
    
  ```python
  a = ['a', 'b', 47, 11, 22, 14, 'h']
  b = a
  print("이전 a:", a)
  print("이전 b:", b)
  a[:] = [101, 102, 103]
  assert a is b # 같은 리스트 객체임
  print("이후 a:", a) # 새로운 내용이 들어있음
  print("이후 b:", b) # 같은 리스트 객체이기 떄문에 a와 내용이 같음

  # 이전 a: ['a', 'b', 47, 11, 22, 14, 'h']
  # 이전 b: ['a', 'b', 47, 11, 22, 14, 'h']
  # 이후 a: [101, 102, 103]
  # 이후 b: [101, 102, 103]
  ```
    
- 어떠한 파이썬 클래스에도 `__getitem__`과 `__setitem__` 특별 메서드를 통해 슬라이싱을 추가할 수 있다.
