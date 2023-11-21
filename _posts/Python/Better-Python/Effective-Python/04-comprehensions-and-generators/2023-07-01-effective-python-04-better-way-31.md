---
title: "[Better Way #31] 인자에 대해 이터레이션할 때는 방어적이 돼라"
date: 2023-07-01 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

## 이터레이터의 `StopIteration`

```python
def normalize(numbers):     # -- numbers가 이터레이터인 경우,
    total = sum(numbers)    # -- (1) 이터레이터 동작 (→ 원소 소진)
    result = []
    for value in numbers:   # -- (2) 이터레이터 동작 (→ 비어있음)
        percent = 100 * value / total
        result.append(percent)
    return result
```

- 입력 인자를 **여러 번 이터레이션 하는 함수/메서드**의 경우, 입력 받은 인자가 **이터레이터**면 함수가 이상하게 동작하거나 결과가 없을 수 있다.
- <span class="hl">**이터레이터는 단 한 번만 결과를 생성**</span>하기 때문에, 모든 원소를 다 소진해서 `StopIteration` 예외가 발생한 이터레이터나 제너레이터를 다시 이터레이션하면 아무 결과도 얻을 수 없다.
- 하지만 이미 소진된 이터레이터에 대해 이터레이션을 추가로 수행해도 아무런 오류가 발생하지 않기 때문에, 출력이 없는 이터레이터와 이미 소진돼버린 이터레이터를 구분할 수 없다.

<br>

## 이터레이터 프로토콜

```python
visits = [15, 35, 80]   # -- 리스트 또한 이터러블 컨테이너
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
# [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

```python
class ReadVisits:         # -- 이터레이터 프로토콜을 따르는 이터러블 컨테이너
    def __init__(self, data_path):
        self.data_path = data_path
    
    def __iter__(self):   # -- 제너레이터로 구현
        with open(self.data_path) as f:
            for line in f:
                yield int(line)
        
path = "my_numbers.txt"
visits = ReadVisits(path)
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
# [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

- 컨테이너와 이터레이터가 **`iter`, `next` 내장 함수**나 **`for` 루프** 등의 관련 식과 상호작용하는 절차를 정의한다.
    
    > **`for x in foo` → `iter(foo)` → `foo.__iter__` → 이터레이터 객체(`__next__` 정의) 반환**
    > 
    > 
    > ⇒ **`for` 루프**는 반환받은 이터레이터 객체가 **데이터를 소진(`StopIteration` 예외)할 때까지** 반복적으로 이터레이터 객체에 대해 **`next` 내장 함수**를 호출한다.
    {: .prompt-info}
- 사용자 정의 클래스에서 <span class="hl">**`__iter__` 메서드를 제너레이터로 구현**</span>하기만 하면 쉽게 **이터러블 컨테이너 타입**을 정의할 수 있다.
- `ReadVisits` 뿐만 아니라 리스트 또한 이터레이터 프로토콜을 따르는 이터러블 컨테이너이다.

<br>

## 이터러블 컨테이너가 아닌 이터레이터인지 감지하는 두 가지 방법
> <span class="hl">**이터러블 컨테이너가 아닌 이터레이터**는 반복적으로 이터레이션이 불가능</span>하므로 거부해야 한다.
{: .prompt-info}

- **`iter` 내장 함수에 넘겨서 반환되는 값이 원래 값과 같은지 확인한다.**
    
  > `iter(이터레이터)` 인 경우, 전달받은 이터레이터가 그대로 반환된다.
  > 
  
  ```python
  def normalize_defensive(numbers):
      if iter(numbers) is numbers: # -- 이터레이터라면,
          raise TypeError("컨테이너를 제공해야 합니다.")
      ...
  ```
    
- **`collections.abc.Iterator` 클래스를 `isinstance` 함수와 함께 사용한다.**
    
  ```python
  from collections.abc import Iterator
  
  def normalize_defensive(numbers):
      if isinstance(numbers, Iterator): # -- 이터레이터라면,
          raise TypeError("컨테이너를 제공해야 합니다.")
      ...
  ```
        