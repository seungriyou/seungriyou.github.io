---
title: "[Better Way #17] 내부 상태에서 원소가 없는 경우를 처리할 때는 setdefault 보다 defaultdict를 사용하라"
date: 2023-06-22 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch02 list and dictionary]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 02. List and Dictionary"**을 읽고 정리한 내용입니다.

<br>

- `collections` 내장 모듈에 있는 **`defaultdict` 클래스**는 **키가 없을 때 자동으로 디폴트 값을 저장**해 딕셔너리를 다룰 때 키가 없는 경우를 쉽게 처리할 수 있다.
    
  ```python
  from collections import defaultdict

  class Visits:
      def __init__(self):
          self.data = defaultdict(set)
      
      def add(self, country, city):
          self.data[country].add(city)

  visits = Visits()
  visits.add("영국", "버스")
  visits.add("영국", "런던")
  print(visits.data)
  # defaultdict(<class 'set'>, {'영국': {'런던', '버스'}})
  ```
    
- 임의의 키가 들어있는 딕셔너리가 어떻게 생성되었는지 모르는 경우, **딕셔너리의 원소에 접근하려면 우선 `get`을 사용**해야 한다.
- 하지만 `setdefault`가 더 짧은 코드를 만들어내는 몇 가지 경우에는 `setdefault`를 사용하는 것도 고려해볼 수 있다.
