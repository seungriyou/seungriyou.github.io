---
title: "[Better Way #13] 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라"
date: 2023-06-13 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

## 별표 식 `*`

- 리스트를 서로 겹치지 않게 여러 조각으로 나누는 경우, **별표 식(starred expression)**을 통해 모든 값을 담는 언패킹을 할 수 있다.
    
    > 기본적인 언패킹 시에는 **언패킹할 시퀀스의 길이를 미리 알고 있어야 한다**는 한계점을 개선한다.
    {: .prompt-info} 
    
  ```python
  car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
  car_ages_descending = sorted(car_ages, reverse=True)

  oldest, second_oldest, *others = car_ages_descending
  print(oldest, second_oldest, others)
  # 20 19 [15, 9, 8, 7, 6, 4, 1, 0]

  oldest, *others, youngest = car_ages_descending
  print(oldest, youngest, others)
  # 20 0 [19, 15, 9, 8, 7, 6, 4, 1]

  *others, second_youngest, youngest = car_ages_descending
  print(youngest, second_youngest, others)
  # 0 1 [20, 19, 15, 9, 8, 7, 6, 4]
  ```
    
- 별표 식은 **아무 위치**에서나 사용할 수 있다.
- 언패킹 대입에서 별표 식을 사용하려면 **필수인 부분이 적어도 하나**는 있어야 한다.
- 한 수준의 언패킹 패턴에 **별표 식을 두 개 이상 쓸 수 없다**. (여러 계층으로 이뤄진 경우 제외)
- 별표 식은 항상 **`list` 인스턴스**가 되며, 언패킹하는 시퀀스에 남는 원소가 없으면 **빈 리스트**가 된다.
    - 원소가 최소 N개 들어있다는 사실을 미리 아는 시퀀스를 처리할 때 유용하다.
        
      ```python
      short_list = [1, 2]
      first, second, *rest = short_list
      print(first, second, rest)
      # 1 2 []
      ```
        
    - 한편, 결과 데이터가 모두 메모리에 들어갈 수 없는 경우, 메모리가 부족하여 프로그램이 멈출 수 있다.
- **이터레이터**의 경우에도 별표 식을 이용하여 값을 깔끔하게 처리할 수 있다.
    
  ```python
  def generate_csv():
      yield ("날짜", "제조사", "모델", "연식", "가격")
      for _ in range(200):
          yield (...)

  it = generate_csv() # 이터레이터 선언
  header, *rows = it
  print('CSV 헤더:', header)
  print('행 수:', len(rows))

  # CSV 헤더: ('날짜', '제조사', '모델', '연식', '가격')
  # 행 수: 200
  ```
