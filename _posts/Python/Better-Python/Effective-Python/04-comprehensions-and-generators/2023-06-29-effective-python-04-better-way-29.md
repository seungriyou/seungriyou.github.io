---
title: "[Better Way #29] 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라"
date: 2023-06-29 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

- **대입식(`:=`)**을 통해 **컴프리헨션이나 제너레이터** 식의 **조건 부분**에서 사용한 값을 다른 위치에서 재사용할 수 있다.
    
  ```python
  def get_batches(count, size):
      return count // size
  
  stock = {
      "못": 125,
      "나사못": 35,
      "나비너트": 8,
      "와셔": 24
  }
  
  order = ["나사못", "나비너트", "클립"]
  ```
  ```python
  # -- 딕셔너리 컴프리헨션
  found_dict = {name: batches for name in order
                if (batches := get_batches(stock.get(name, 0), 8))}
  print(found_dict)
  # {'나사못': 4, '나비너트': 1}
  ```
  ```python
  # -- 제너레이터
  found_it = ((name, batches) for name in order
              if (batches := get_batches(stock.get(name, 0), 8)))
  print(next(found_it))
  print(next(found_it))
  # ('나사못', 4)
  # ('나비너트', 1)
  ```
    
- 조건 부분에서 사용하지 않고 **값 부분**에서 사용하면 다음의 **문제가 발생**할 수 있다.
    - 해당 변수를 읽을 수 없는 참조 오류가 발생할 수 있다. (`if`절과 `for`문의 레벨이 같음)
    - 그 값에 대한 조건 부분이 없다면 루프 밖 영역으로 루프 변수가 누출된다. (cf. 클로저)
        
      > 루프 변수는 누출되지 않는 편이 좋으며, 컴프리헨션의 루프 변수는 원래 누출되지 않는다.
      > 
      
      ```python
      # (1) 컴프리헨션의 루프 변수: 누출 X
      half = [count // 2 for count in stock.values()]
      print(half)
      print(count)  # -- 루프 변수가 누출되지 않기 때문에 오류 발생
      # [62, 17, 4, 12]
      # NameError: name 'count' is not defined
      ```      
      ```python
      # (2) 컴프리헨션의 값 부분에서 := 연산자를 사용하는 경우: 누출 O
      half = [(last := count // 2) for count in stock.values()]
      print(half)
      print(last)   # -- 클로저이므로 접근 가능
      # [62, 17, 4, 12]
      # 12
      ```