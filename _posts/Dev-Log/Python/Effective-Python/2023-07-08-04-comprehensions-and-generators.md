---
title: "[Effective Python] 04. Comprehensions and Generators"
date: 2023-07-08 22:50:00 +0900
categories: [Dev-Log, Python]
tags: [python, effective python, book, comprehensions, generators]
math: true
---

> 본문은 "파이썬 코딩의 기술 (Effective Python, 2판)"을 읽고 정리한 내용입니다.

<br>

## Better Way 27: `map`과 `filter` 대신 컴프리헨션을 사용하라

리스트, 딕셔너리, 집합 등을 다룰 때 `map`과 `filter`를 사용하기보다는 컴프리헨션을 사용하는 것을 권장한다.

- `map`, `filter` 사용 버전
    
  ```python
  a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

  alt_list = list(map(lambda x: x**2, filter(lambda x: x % 2 == 0, a)))
  alt_dict = dict(map(lambda x: (x, x**2),
                      filter(lambda x: x % 2 == 0, a)))
  alt_set = set(map(lambda x: x**3,
                      filter(lambda x: x % 3 == 0, a)))
  ```
    
- 리스트 / 딕셔너리 / 집합 컴프리헨션 사용 버전
    
  ```python
  even_squares_list = [x**2 for x in a if x % 2 == 0]
  even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
  threes_cubed_set = {x**3 for x in a if x % 3 == 0}
  ```
    

<br>

## Better Way 28: 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라

- 컴프리헨션에 들어가는 **하위 식이 세 개 이상 되지 않도록 제한**하는 것을 권장한다.
    - ex) (조건문 두 개), (루프 두 개), (조건문 한 개 + 루프 한 개)
- 컴프리헨션이 더 복잡해지는 경우, 일반 **`if` 문과 `for` 문을 사용**하고 **도우미 함수**를 작성하라.

### 컴프리헨션의 특징

- 동일 레벨의 두 루프는 왼쪽에서부터 순서대로 실행된다.
    
  ```python
  matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
  flat = [x for row in matrix for x in row]
  ```
    
- 컴프리헨션은 여러 `if` 조건을 허용하며, 여러 조건을 같은 수준의 루프에 사용하면 암시적으로 `and` 식을 의미한다.
    
  ```python
  b = [x for x in a if x > 4 if x % 2 == 0]
  c = [x for x in a if x > 4 and x % 2 == 0] # -- b와 동일
  ```
        

<br>

## Better Way 29: 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라

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
  
  # -- 딕셔너리 컴프리헨션
  found_dict = {name: batches for name in order
                if (batches := get_batches(stock.get(name, 0), 8))}
  print(found_dict)
  # {'나사못': 4, '나비너트': 1}
  
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
      print(count) # -- 루프 변수가 누출되지 않기 때문에 오류 발생
      # [62, 17, 4, 12]
      # NameError: name 'count' is not defined
      
      # (2) 컴프리헨션의 값 부분에서 := 연산자를 사용하는 경우: 누출 O
      half = [(last := count // 2) for count in stock.values()]
      print(half)
      print(last) # -- 클로저이므로 접근 가능
      # [62, 17, 4, 12]
      # 12
      ```
        

<br>

## Better Way 30: 리스트를 반환하기보다는 제너레이터를 사용하라

```python
# -- 리스트를 반환하는 함수
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == " ":
            result.append(index + 1)
    return result

# -- 이터레이터를 반환하는 함수 (제너레이터)
def index_words_it(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == " ":
            yield index + 1
```

### 리스트를 반환하는 함수의 문제점

1. 코드에 잡음이 많고 핵심을 알아보기 어렵다.
2. 반환하기 전에 리스트에 모든 결과를 다 저장해야 하므로, 입력이 매우 크면 프로그램이 메모리를 소진할 수 있다.

### <span class="hl">이터레이터의 특징</span>

- `next` 내장 함수를 호출할 때마다 이터레이터는 제너레이터 함수를 다음 `yield` 식까지 진행시킨다.
- 쉽게 리스트로 변환할 수 있다. (ex. `list(it)`)
- 리스트에 모든 값을 저장할 필요가 없으므로 사용하는 메모리 크기를 어느정도 제한할 수 있게 된다.
    
    > → 작업 메모리에 모든 입력과 출력을 저장할 필요가 없으므로 입력이 아주 커도 출력 시퀀스를 만들 수 있다.
    > 
- 이터레이터에 상태가 있으므로, 호출하는 쪽에서 재사용이 불가능하다.

<br>

## Better Way 31: 인자에 대해 이터레이션할 때는 방어적이 돼라

```python
def normalize(numbers): # -- numbers가 이터레이터인 경우,
    total = sum(numbers) # -- (1) 이터레이터 동작 (→ 원소 소진)
    result = []
    for value in numbers: # -- (2) 이터레이터 동작 (→ 비어있음)
        percent = 100 * value / total
        result.append(percent)
    return result
```

입력 인자를 **여러 번 이터레이션 하는 함수/메서드**의 경우, 입력 받은 인자가 **이터레이터**면 함수가 이상하게 동작하거나 결과가 없을 수 있다.

- **이터레이터는 단 한 번만 결과를 생성**하기 때문에, 모든 원소를 다 소진해서 `StopIteration` 예외가 발생한 이터레이터나 제너레이터를 다시 이터레이션하면 아무 결과도 얻을 수 없다.
- 하지만 이미 소진된 이터레이터에 대해 이터레이션을 추가로 수행해도 아무런 오류가 발생하지 않기 때문에, 출력이 없는 이터레이터와 이미 소진돼버린 이터레이터를 구분할 수 없다.

### 이터레이터 프로토콜

```python
visits = [15, 35, 80] # -- 리스트 또한 이터러블 컨테이너
percentages = normalize(visits)
print(percentages)
assert sum(percentages) == 100.0
# [11.538461538461538, 26.923076923076923, 61.53846153846154]
```

```python
class ReadVisits: # -- 이터레이터 프로토콜을 따르는 이터러블 컨테이너
    def __init__(self, data_path):
        self.data_path = data_path
    
    def __iter__(self): # -- 제너레이터로 구현
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
    > ⇒ `for` 루프는 반환받은 이터레이터 객체가 데이터를 소진(`StopIteration` 예외)할 때까지 반복적으로 이터레이터 객체에 대해 `next` 내장 함수를 호출한다.
    > 
- 사용자 정의 클래스에서 **`__iter__` 메서드를 제너레이터로 구현**하기만 하면 쉽게 **이터러블 컨테이너 타입**을 정의할 수 있다.
- `ReadVisits` 뿐만 아니라 리스트 또한 이터레이터 프로토콜을 따르는 이터러블 컨테이너이다.

### 이터러블 컨테이너가 아닌 이터레이터인지 감지하는 두 가지 방법
> 이터러블 컨테이너가 아닌 <span class="hl">이터레이터는 반복적으로 이터레이션이 불가능</span>하므로 거부해야 한다.
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
        

<br>

## Better Way 32: 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라

### 제너레이터 식 `()`

```python
roots = ((x, x**0.5) for x in it)
print(next(roots))
```

- **리스트 컴프리헨션의 문제점**: 입력이 커지면 메모리를 상당히 많이 사용하고 프로그램이 중단될 수 있다.
- **제너레이터 식**이란 리스트 컴프리헨션과 제너레이터를 일반화한 것으로, `()` 사이에 리스트 컴프리헨션과 비슷한 구문을 넣어 제너레이터 식을 만들 수 있다.
- 제너레이터 식에 들어있는 식으로부터 원소를 하나씩 만들어내는 **이터레이터가 생성**된다.
    - 한 번에 원소를 하나씩 반환하므로 메모리 문제를 피할 수 있다.
    - `next` 내장 함수를 이용할 수 있다.
- **두 제너레이터 식을 합성**할 수 있다.
    - 이터레이터를 전진시킬 때마다 내부의 이터레이터도 전진한다.
    - 가능한 메모리를 효율적으로 사용하면서 동작한다.
- 아주 **큰 입력 스트림**에 대해 **여러 기능을 합성**해 적용해야 할 때 유용하다.
- 제너레이터가 반환하는 이터레이터에는 상태가 있으므로 **이터레이터를 단 한 번만 사용**해야 한다.

<br>

## Better Way 33: `yield from`을 사용해 여러 제너레이터를 합성하라

### `yield from` 식

- 파이썬 인터프리터가 사용자 대신 `for` 루프를 내포시키고 `yield` 식을 처리하도록 한다.
- 여러 제너레이터를 모아서 하나의 제너레이터로 합성할 때 사용할 수 있다.
- 직접 제너레이터를 `for` 문으로 처리할 때보다 개선된 성능을 보인다.

```python
# 다음의 두 함수는 동일한 제너레이터를 반환한다.

def animate(): # -- for 루프 사용
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta

def animate_composed(): # -- yield from 사용
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)
```

<br>

## Better Way 34: `send`로 제너레이터에 데이터를 주입하지 말라

### `send` 메서드

- 파이썬 제너레이터가 지원하는 메서드로, `yield` 식을 양방향 채널로 만들어주어 데이터를 제너레이터에 주입할 수 있도록 한다.
- 제너레이터는 **`send`로 주입된 값**을 **`yield` 식이 반환하는 값**을 통해 받으며, 이 값을 변수에 저장해 활용할 수 있다.
    - 제너레이터가 재개(resume)될 때 `yield`가 `send`에 전달된 파라미터 값을 반환한다.
    - 따라서 최초로 `send`를 호출할 때 인자로 전달할 수 있는 유일한 값은 `None` 뿐이다. (아직 `yield` 식에 도달하지 못했으므로)
- `send`와 제너레이터를 합성하는 `yield from`을 함께 사용하면 각각의 내포된 제너레이터의 출력의 처음에 `None`이 등장하게 된다.

### `send` 메서드를 사용하지 않고 제너레이터에 데이터를 주입하는 방법

- **함수에 이터레이터를 전달하는 방법**
    
  ```python
  def wave_cascading(amplitude_it, steps):
      step_size = 2 * math.pi / steps
      for step in range(steps):
          radians = step * step_size
          fraction = math.sin(radians)
          amplitude = next(amplitude_it)  # -- 다음 입력 받기
          output = amplitude * fraction
          yield output
  ```
    
- **합성에 사용할 여러 제너레이터 함수에 같은 이터레이터를 넘기는 방법**
    - **이터레이터는 상태**가 있으므로, 내포된 각각의 제너레이터는 앞에 있는 제너레이터가 처리를 끝낸 시점부터 데이터를 가져와 처리한다.
    - 아무데서나 이터레이터를 가져올 수 있으며, 이터레이터가 완전히 동적인 경우에도 잘 작동한다.
    - 하지만 **제너레이터가 항상 thread-safe 한 것은 아니므로**, 스레드 경계를 넘나들면서 제너레이터를 사용해야 한다면 `async` 함수가 더 나은 해법일 수 있다.
    
  ```python
  def complex_wave_cascading(amplitude_it):
      # -- 동일한 이터레이터를 여러 제너레이터 함수에 넘김
      yield from wave_cascading(amplitude_it, 3)
      yield from wave_cascading(amplitude_it, 4)
      yield from wave_cascading(amplitude_it, 5)
  
  def run_cascading():
      amplitudes = [7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
      it = complex_wave_cascading(iter(amplitudes))
      for amplitude in amplitudes:
          output = next(it)
          transmit(output)
  
  run_cascading()
  ```
    

<br>

## Better Way 35: 제너레이터 안에서 `throw`로 상태를 변화시키지 말라

### `throw` 메서드

- `throw` 메서드를 통해 제너레이터가 마지막으로 실행한 `yield` 식의 위치에서 예외를 다시 발생시킬 수 있다.
- `throw`를 사용하는 방법은 가독성을 해친다.

```python
def timer(period):
    current = period
    while current:
        current -= 1
        try:
            yield current
        except Reset:
            current = period

def run():
    it = timer(4)
    while True:
        try:
            if check_for_reset():
                current = it.throw(Reset())
            else:
                current = next(it)
        except StopIteration:
            break
        else:
            announce(current)

run()
```

### `throw` 메서드를 사용하지 않고 예외적인 동작을 제공하는 방법

- **이터러블 컨테이너 객체(`__iter__` 메서드 구현)**를 사용해 **상태가 있는 클로저를 정의**하는 것이다. (예외적인 경우에 상태 전이)
- 제너레이터와 예외를 함께 사용해야 한다면, 비동기 기능을 사용할 때 더 좋게 구현할 수 있는 경우도 많다.

```python
class Timer:
    def __init__(self, period):
        self.current = period
        self.period = period
    
    def reset(self):
        self.current = self.period
    
    def __iter__(self):
        while self.current:
            self.current -= 1
            yield self.current

def run():
    timer = Timer(4)
    for current in timer:
        if check_for_reset():
            timer.reset()
        announce(current)

run()
```

<br>

## Better Way 36: 이터레이터나 제너레이터를 다룰 때는 `itertools`를 사용하라

```python
import itertools

help(itertools)
```

### 여러 이터레이터 연결하기

```python
# -- chain: 여러 이터레이터를 하나의 순차적인 이터레이터로 합친다.
it = itertools.chain([1, 2, 3], [4, 5, 6])
print(list(it))
# [1, 2, 3, 4, 5, 6]

# -- repeat: 한 값을 반복하는 이터레이터를 반환한다.
#            두 번쨰 인자로 최대 횟수를 지정할 수 있다.
it = itertools.repeat("안녕", 3)
print(list(it))
# ['안녕', '안녕', '안녕']

# -- cycle: 원소들을 계속해서 반복하여 내놓는 이터레이터를 반환한다.
it = itertools.cycle([1, 2])
result = [next(it) for _ in range(10)]
print(result)
# [1, 2, 1, 2, 1, 2, 1, 2, 1, 2]

# -- tee: 한 이터레이터를 병렬적으로 두 번째 인자로 지정된 개수만큼의 이터레이터로 만든다.
#         반환된 이터레이터들을 같은 속도로 소비하지 않으면 메모리 사용량이 늘어난다.
it1, it2, it3 = itertools.tee(["하나", "둘"], 3)
print(list(it1))
print(list(it2))
print(list(it3))
# ['하나', '둘']
# ['하나', '둘']
# ['하나', '둘']

# -- zip_longest: 여러 이터레이터 중 짧은 쪽 이터레이터의 원소를 다 사용한 경우, 
#                 fillvalue로 지정한 값을 넣어준다.
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

### 이터레이터에서 원소 거르기

```python
# -- islice: 이터레이터를 복사하지 않으면서 원소 인덱스를 이용해 슬라이싱하고 싶을 때 사용한다.
#            (끝)만 지정하거나, (시작, 끝)을 지정하거나, (시작, 끝, 증가값)을 지정할 수 있다.
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

first_five = itertools.islice(values, 5)
print("앞에서 다섯 개:", list(first_five))
middle_odds = itertools.islice(values, 2, 8, 2)
print("중간의 홀수들:", list(middle_odds))
# 앞에서 다섯 개: [1, 2, 3, 4, 5]
# 중간의 홀수들: [3, 5, 7]

# -- takewhile: 이터레이터에서 주어진 술어가 True를 반환하는 동안 원소를 돌려준다. 
#               (즉, 술어가 False를 반환하는 첫 원소가 나타날 때까지)
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.takewhile(less_than_seven, values)
print(list(it))
# [1, 2, 3, 4, 5, 6]

# -- dropwhile: takewhile의 반대로, 술어가 True를 반환하는 동안 원소를 건너뛴다.
#               (즉, 술어가 False를 반환하는 첫 원소가 나타날 때까지)
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.dropwhile(less_than_seven, values)
print(list(it))
# [7, 8, 9, 10]

# -- filterfalse: filter 내장 함수의 반대로, 주어진 이터레이터에서 술어가 False를 반환하는 모든 원소를 돌려준다.
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = lambda x: x % 2 == 0
filter_result = filter(evens, values)
print("Filter:", list(filter_result))
filter_false_result = itertools.filterfalse(evens, values)
print("Filter false:", list(filter_false_result))
# Filter: [2, 4, 6, 8, 10]
# Filter false: [1, 3, 5, 7, 9]
```

### 이터레이터에서 원소의 조합 만들어내기

```python
# -- accumulate: 파라미터를 두 개 받는 함수를 반복 적용하면서 이터레이터 원소를 값 하나로 줄여준다.
#                functools 내장 모듈에 있는 reduce 함수의 결과와 같다.
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

# -- product: 하나 이상의 이터레이터에 들어있는 원소들의 데카르트 곱을 반환한다.
#             깊은 리스트 컴프리헨션 대신에 사용하면 편리하다.
single = itertools.product([1, 2], repeat=2)
print("리스트 한 개:", list(single))
# 리스트 한 개: [(1, 1), (1, 2), (2, 1), (2, 2)]

multiple = itertools.product([1, 2], ["a", "b"])
print("리스트 두 개:", list(multiple))
# 리스트 두 개: [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

# -- permutations: 이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 순열을 돌려준다.
it = itertools.permutations([1, 2, 3, 4], 2)
print(list(it))
# [(1, 2), (1, 3), (1, 4), (2, 1), (2, 3), (2, 4), 
# (3, 1), (3, 2), (3, 4), (4, 1), (4, 2), (4, 3)]

# -- combinations: 이터레이터가 내놓는 원소들로부터 만들어낸 길이 N인 조합을 돌려준다.
it = itertools.combinations([1, 2, 3, 4], 2)
print(list(it))
# [(1, 2), (1, 3), (1, 4), (2, 3), (2, 4), (3, 4)]

# -- combinations_with_replacement: combinations와 같지만 원소의 반복을 허용한다. (중복 조합)
it = itertools.combinations_with_replacement([1, 2, 3, 4], 2)
print(list(it))
# [(1, 1), (1, 2), (1, 3), (1, 4), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (4, 4)]
```