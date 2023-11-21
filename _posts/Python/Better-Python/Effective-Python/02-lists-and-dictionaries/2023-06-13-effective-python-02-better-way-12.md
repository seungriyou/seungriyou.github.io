---
title: "[Better Way #12] 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라"
date: 2023-06-13 23:20:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>


- 증가값을 `-1`로 사용해 문자열을 슬라이싱하면 뒤집을 수 있다.
    
  ```python
  # 바이트 문자열
  x = b'mongoose'
  y = x[::-1]
  print(y) # b'esoognom'

  # 유니코드 문자열
  x = '안녕'
  y = x[::-1]
  print(y) # 녕안

  # UTF-8 문자열 (작동 X) (2바이트 이상으로 이뤄졌던 문자들은 코드가 깨짐)
  w = '안녕'
  x = w.encode('utf-8')
  y = x[::-1]
  z = y.decode('utf-8')
  # UnicodeDecodeError: 'utf-8' codec can't decode byte 0x95 in position 0: invalid start byte
  ```
    
- 슬라이싱 구문에 스트라이딩까지 들어가면 불분명하기 때문에, **시작값이나 끝값을 증가값과 함께 사용하지 않는 것을 권장**한다.
    1. 증가값을 사용해야 하는 경우, **양수값**으로 만들고 **시작과 끝 인덱스를 생략**한다.
    2. **스트라이딩을 먼저 수행**하고 그 결과를 변수에 대입한다.
        
        > 결과 슬라이스의 크기를 가능한 한 줄일 수 있어야 한다.
        > 
    3. 변수에 대입한 값을 **슬라이싱**한다.
        
        > 스트라이딩 후 슬라이싱을 하면 데이터를 한 번 더 얕게 복사하게 된다. 프로그램이 이 **두 단계 연산에 필요한 시간과 메모리를 감당**할 수 없다면 `itertools` 내장 모듈의 `islice` 메서드를 고려하자.
        > 
    
  ```python
  y = x[::2]    # (1) 스트라이딩
  z = y[1:-1]   # (2) 슬라이싱
  ```
    