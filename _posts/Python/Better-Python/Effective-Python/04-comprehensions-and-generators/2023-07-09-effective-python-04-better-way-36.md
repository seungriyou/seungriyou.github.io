---
title: "[Better Way #36] 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라"
date: 2023-07-09 23:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

```python
import itertools

help(itertools)
```

<br>

## 여러 이터레이터 연결하기
- `chain`: 여러 이터레이터를 하나의 순차적인 이터레이터로 합친다.
  ```python
  it = itertools.chain([1, 2, 3], [4, 5, 6])
  print(list(it))
  # [1, 2, 3, 4, 5, 6]
  ```
- `repeat`: 한 값을 반복하는 이터레이터를 반환한다. 두 번쨰 인자로 최대 횟수를 지정할 수 있다.
  ```python
  it = itertools.repeat("안녕", 3)
  print(list(it))
  # ['안녕', '안녕', '안녕']
  ```
- `cycle`: 원소들을 계속해서 반복하여 내놓는 이터레이터를 반환한다.
  ```python
  it = itertools.cycle([1, 2])
  result = [next(it) for _ in range(10)]
  print(result)
  # [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]
  ```
- `tee`: 한 이터레이터를 병렬적으로 두 번째 인자로 지정된 개수만큼의 이터레이터로 만든다. 반환된 이터레이터들을 같은 속도로 소비하지 않으면 메모리 사용량이 늘어난다.
  ```python
  it1, it2, it3 = itertools.tee(["하나", "둘"], 3)
  print(list(it1))
  print(list(it2))
  print(list(it3))
  # ['하나', '둘']
  # ['하나', '둘']
  # ['하나', '둘']
  ```
- `zip_longest`: 여러 이터레이터 중 짧은 쪽 이터레이터의 원소를 다 사용한 경우, `fillvalue로` 지정한 값을 넣어준다.
  ```python
  keys = ["하나", "둘", "셋"]
  values = [1, 2]

  normal = list(zip(keys, values))
  print("zip:", normal)
  # zip: [('하나', 1), ('둘', 2)]

  it = itertools.zip_longest(keys, values, fillvalue="없음")
  longest = list(it)
  print("zip_longest:", longest)
  # zip_longest: [('하나', 1), ('둘', 2), ('셋', '없음')]
  ```

<br>

## 이터레이터에서 원소 거르기
- `islice`: 이터레이터를 복사하지 않으면서 원소 인덱스를 이용해 슬라이싱하고 싶을 때 사용한다. `(끝)`만 지정하거나, `(시작, 끝)`을 지정하거나, `(시작, 끝, 증가값)`을 지정할 수 있다.
  ```python
  values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

  first_five = itertools.islice(values, 5)
  print("앞에서 다섯 개:", list(first_five))
  middle_odds = itertools.islice(values, 2, 8, 2)
  print("중간의 홀수들:", list(middle_odds))
  # 앞에서 다섯 개: [1, 2, 3, 4, 5]
  # 중간의 홀수들: [3, 5, 7]
  ```
- `takewhile`: 이터레이터에서 주어진 술어가 True를 반환하는 동안 원소를 돌려준다. (즉, 술어가 False를 반환하는 첫 원소가 나타날 때까지)
  ```python
  values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  less_than_seven = lambda x: x < 7
  it = itertools.takewhile(less_than_seven, values)
  print(list(it))
  # [1, 2, 3, 4, 5, 6]
  ```
- `dropwhile`: takewhile의 반대로, 술어가 True를 반환하는 동안 원소를 건너뛴다. (즉, 술어가 False를 반환하는 첫 원소가 나타날 때까지)
  ```python
  values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  less_than_seven = lambda x: x < 7
  it = itertools.dropwhile(less_than_seven, values)
  print(list(it))
  # [7, 8, 9, 10]
  ```
- `filterfalse`: filter 내장 함수의 반대로, 주어진 이터레이터에서 술어가 False를 반환하는 모든 원소를 돌려준다.
  ```python
  values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  evens = lambda x: x % 2 == 0
  filter_result = filter(evens, values)
  print("Filter:", list(filter_result))
  filter_false_result = itertools.filterfalse(evens, values)
  print("Filter false:", list(filter_false_result))
  # Filter: [2, 4, 6, 8, 10]
  # Filter false: [1, 3, 5, 7, 9]
  ```

<br>

## 이터레이터에서 원소의 조합 만들어내기
- `accumulate`: 파라미터를 두 개 받는 함수를 반복 적용하면서 이터레이터 원소를 값 하나로 줄여준다. functools 내장 모듈에 있는 reduce 함수의 결과와 같다.
  ```python
  values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  sum_reduce = itertools.accumulate(values)   # -- 이항 함수를 넘기지 않으면 이터레이터 원소의 합계 계산
  print("합계:", list(sum_reduce))
  # 합계: [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]

  def sum_modulo_20(first, second):
      output = first + second
      return output % 20
  modulo_reduce = itertools.accumulate(values, sum_modulo_20)
  print("20으로 나눈 나머지의 합계:", list(modulo_reduce))
  # 20으로 나눈 나머지의 합계: [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]
  ```
- `product`: 하나 이상의 이터레이터에 들어있는 원소들의 데카르트 곱을 반환한다. 깊은 리스트 컴프리헨션 대신에 사용하면 편리하다.
  ```python
  single = itertools.product([1, 2], repeat=2)
  print("리스트 한 개:", list(single))
  # 리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]

  multiple = itertools.product([1, 2], ["a", "b"])
  print("리스트 두 개:", list(multiple))
  # 리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
  ```
- `permutations`: 이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 순열을 돌려준다.
  ```python
  it = itertools.permutations([1, 2, 3, 4], 2)
  print(list(it))
  # [(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), 
  # (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]
  ```
- `combinations`: 이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 조합을 돌려준다.
  ```python
  it = itertools.combinations([1, 2, 3, 4], 2)
  print(list(it))
  # [(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]
  ```
- `combinations_with_replacement`: combinations와 같지만 원소의 반복을 허용한다. (중복 조합)
  ```python
  it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
  print(list(it))
  # [(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]
  ```
