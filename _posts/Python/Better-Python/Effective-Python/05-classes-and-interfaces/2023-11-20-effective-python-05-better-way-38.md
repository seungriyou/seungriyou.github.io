---
title: "[Better Way #38] 간단한 인터페이스의 경우 클래스 대신 함수를 받아라"
date: 2023-11-20 12:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>


## 훅 (Hook)

- 파이썬에서 함수는 **first-class citizen**으로 취급되므로, 함수나 메서드를 다른 함수에 넘기거나 변수 등으로 참조할 수 있다.
- 파이썬 내장 API가 실행되는 과정에서 <span class="hl">**전달된 함수를 실행**하는 경우</span>, 이러한 함수를 <span class="hl">**훅(hook)**</span>이라 한다.
    
  ```python
  names = ["소크라테스", "아르키메데스", "플라톤", "아리스토텔레스"]
  names.sort(key=len) # -- 리스트를 문자열의 길이를 기준으로 정렬
  print(names)
  # ['플라톤', '소크라테스', '아르키메데스', '아리스토텔레스']
  ```
    
<br>

### [예시] `defaultdict`의 인자로 `__call__` 메서드를 정의한 클래스의 인스턴스 전달

- `defaultdict`의 경우, 딕셔너리 안에 없는 키에 접근할 경우에 호출되는 **인자가 없는 함수**를 전달할 수 있다. 이 함수는 **존재하지 않는 키에 해당하는 값이 될 객체를 반환**해야 한다.
- 다음과 같은 `log_missing` 함수를 이용하면 <span class="hl">**정해진 동작(= 값 삽입)**</span>과 <span class="hl">**부수 효과(= 로그 남기기)**</span>를 분리할 수 있다.
    
  ```python
  from collections import defaultdict
  
  def log_missing():
      print("키 추가됨")
      return 0
  
  current = {"초록": 12, "파랑": 3}
  increments = [
      ("빨강", 5),
      ("파랑", 17),
      ("주황", 9)
  ]
  result = defaultdict(log_missing, current) # -- hook 전달
  print("이전:", dict(result))
  for key, amount in increments:
      result[key] += amount
  print("이후:", dict(result))
  
  # 이전: {'초록': 12, '파랑': 3}
  # 키 추가됨
  # 키 추가됨
  # 이후: {'초록': 12, '파랑': 20, '빨강': 5, '주황': 9}
  ```
    
- **존재하지 않는 키에 접근한 총 횟수를 카운트**하려는 경우, 다음과 같은 방법으로 접근할 수 있다.
  ```python
  class BetterCountMissing:
      def __init__(self):
          self.added = 0
          
      def __call__(self):
          self.added += 1
          return 0

  counter = BetterCountMissing()
  result = defaultdict(counter, current)  # -- __call__에 의존
  for key, amount in increments:
      result[key] += amount
  assert counter.added == 2
  ```

  1. 추적하고 싶은 **상태를 저장하는 작은 클래스**를 정의한다.
  2. `defaultdict`에 전달할 디폴트 값 훅을 1번에서 정의한 클래스의 **`__call__` 특별 메서드**에 정의한다. 해당 메서드는 **상태를 저장하는 클로저 역할**을 하게 된다.
      
      > `__call__`을 사용하면 객체를 함수처럼 호출할 수 있으며(= **callable**), `callable` 내장 함수를 호출하면 `True`가 반환된다.
      > 
    
<br>

> **`__call__` 메서드**
>
> - `__call__` 메서드를 정의함으로써 **함수가 인자로 쓰일 수 있는 부분**에 **해당 클래스의 인스턴스**를 사용할 수 있다. 이를 통해 외부에서는 `__call__` 내부에서 어떤 일이 벌어지는지에 대해 전혀 알 필요가 없게 된다.
> - 상태를 유지하기 위한 함수가 필요한 경우, **상태가 있는 클로저를 정의**하는 대신 **`__call__` 메서드가 있는 클래스를 정의**할지 고려해보자.
{: .prompt-info}
