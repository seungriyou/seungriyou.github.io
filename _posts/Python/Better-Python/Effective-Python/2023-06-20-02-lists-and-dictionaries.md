---
title: "[Effective Python] 02. Lists and Dictionaries"
date: 2023-06-20 10:50:00 +0900
categories: [Python, Better Python]
tags: [python, effective python, book, list, dictionary]     # TAG names should always be lowercase
math: true
---

> 본문은 "파이썬 코딩의 기술 (Effective Python, 2판)"을 읽고 정리한 내용입니다.

<br>

> **딕셔너리(`dict`) 타입**
> 
> - 일반적으로 해시 테이블(hash table)이나 연관 배열(associative array) 구조 안에 값을 저장한다.
> - 분할상환(armotization) 복잡도로 상수 시간($O(1)$)에 원소를 삽입하고 찾을 수 있다.
>     
>     > **분할상환 복잡도**: 각 연산이 발생하는 빈도와 시간을 함께 고려해 평균적으로 비용을 계산함으로써 산정한 복잡도
> 
>     - **데이터 삽입 연산**: 중간중간 테이블을 재배치하기 위한 시간이 오래 걸릴 수 있다.
>     - **데이터 읽기 연산**: 매우 빠르며, 일반적으로 데이터 삽입 연산보다 훨씬 많이 사용된다.
{: .prompt-info}

<br>

## Better Way 11: 시퀀스를 슬라이싱하는 방법을 익혀라

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

<br>

## Better Way 12: 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라

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
    2. 스트라이딩을 먼저 수행하고 그 결과를 변수에 대입한다.
        
        > 결과 슬라이스의 크기를 가능한 한 줄일 수 있어야 한다.
        > 
    3. 변수에 대입한 값을 슬라이싱한다.
        
        > 스트라이딩 후 슬라이싱을 하면 데이터를 한 번 더 얕게 복사하게 된다. 프로그램이 이 **두 단계 연산에 필요한 시간과 메모리를 감당**할 수 없다면 `itertools` 내장 모듈의 `islice` 메서드를 고려하자.
        > 
    
  ```python
  y = x[::2] # (1) 스트라이딩
  z = y[1:-1] # (2) 슬라이싱
  ```
    

<br>

## Better Way 13: 슬라이싱보다는 나머지를 모두 잡아내는 언패킹을 사용하라

- 리스트를 서로 겹치지 않게 여러 조각으로 나누는 경우, **별표 식(starred expression)**을 통해 모든 값을 담는 언패킹을 할 수 있다.
    
    > 기본적인 언패킹 시에는 **언패킹할 시퀀스의 길이를 미리 알고 있어야 한다**는 한계점을 개선한다.
    > 
    
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
    

<br>

## Better Way 14: 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라

> **`list` 내장 타입 `sort` 메서드**는 자연스러운 순서를 사용해 오름차순으로 정렬한다.
> 
> 
> → 거의 대부분의 내장 타입(문자열, 부동 소수점 수 등)에 대해 잘 작동한다.
> 

### `sort` 메서드의 `key` 파라미터

원소 타입에 자연스러운 순서가 정의되어 있지 않는 경우, `key` 파라미터를 통해 리스트의 각 원소 대신 **비교에 사용할**(= 비교 가능한, 자연스러운 순서가 정의된) **객체를 반환하는 도우미 함수**를 정의할 수 있다. (ex. 원소 애트리뷰트에 접근, 인덱스를 사용하여 값 얻기 등)

```python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight
    
    def __repr__(self):
        return f"Tool({self.name!r}, {self.weight})"

tools = [
    Tool("수준계", 3.5),
    Tool("해머", 1.25),
    Tool("스크류드라이버", 0.5),
    Tool("끌", 0.25)
]

print("미정렬:", repr(tools))
tools.sort(key=lambda x: x.name)
print("정렬:", repr(tools))
```

### `key` 함수를 통해 원소 값을 변형하는 경우

ex) 대소문자를 구분하지 않고 알파벳 순으로 문자열 정렬하기

```python
places = ["home", "work", "New York", "Paris"]
places.sort()
print("대소문자 구분:", places)
places.sort(key=lambda x: x.lower()) # 소문자로 변환 후 정렬
print("대소문자 무시:", places)
```

### 여러 정렬 기준을 사용해야 하는 경우

1. **`key` 함수의 리턴 타입으로 `tuple` 타입을 사용하는 방법**
    - 정렬 기준으로 사용할 애트리뷰트를 우선순위 순서대로 튜플에 넣는다.
    - 모든 비교 기준의 정렬 순서가 같아야 한다. (모두 오름차순 or 모두 내림차순)
    - `reverse` 파라미터를 넘기면 튜플에 들어있는 기준들의 정렬 순서가 같이 영향을 받게 된다.
    - 여러 정렬 기준의 정렬 순서가 달라야 하는 경우, 부호를 바꿀 수 있는 타입(ex. 숫자값)이라면 **단항 부호 반전 연산자 `-`**를 사용해 정렬 방향을 혼합할 수 있다.
    
    ```python
    power_tools = [
        Tool("드릴", 4),
        Tool("원형 톱", 5),
        Tool("착암기", 40),
        Tool("연마기", 4)
    ]
    
    power_tools.sort(key=lambda x: (-x.weight, x.name)) # (내림차순, 오름차순)
    print(power_tools)
    # [Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
    ```
    
2. **정렬 기준의 우선순위가 점점 높아지는 순서대로 `sort` 함수를 호출하는 방법**
    - 부호를 바꿀 수 없는 타입이라면 **각 정렬 기준마다 `reverse` 파라미터로 정렬 순서를 지정**하면서 `sort` 함수를 여러 번 사용해야 한다.
    - 이것이 가능한 이유는 파이썬이 **stable sort** 알고리즘을 제공하기 때문이다.
        
        > **stable sort** = `key` 함수가 반환하는 값이 서로 같은 경우, 리스트에 들어있던 원래 순서를 그대로 유지
        > 
    - 최종적으로 리스트에서 얻어내고 싶은 **정렬 기준 우선순위의 역순으로 정렬을 수행**해야 한다.
    - 꼭 필요할 때만 이 방법을 사용하는 것을 권장한다.
    
    ```python
    power_tools = [
        Tool("드릴", 4),
        Tool("원형 톱", 5),
        Tool("착암기", 40),
        Tool("연마기", 4)
    ]
    
    power_tools.sort(key=lambda x: x.name)
    power_tools.sort(key=lambda x: x.weight, reverse=True)
    print(power_tools)
    # [Tool('착암기', 40), Tool('원형 톱', 5), Tool('드릴', 4), Tool('연마기', 4)]
    ```
    

<br>

## Better Way 15: 딕셔너리 삽입 순서에 의존할 때는 조심하라

### 파이썬의 `dict` 인스턴스

- **파이썬 3.5 이전**: 딕셔너리에 대해 이터레이션 수행 시, 그 순서는 원소가 삽입된 순서와 일치하지 않았다.
- **파이썬 3.6 이후**: 딕셔너리가 삽입 순서를 보존하도록 동작이 개선되었다. 특히, 3.7+에서는 아예 파이썬 언어 명세에 해당 내용이 포함되었다.
- **파이썬 내 `dict` 타입의 활용 사례** (= 파이썬 3.6+부터 삽입 순서가 보존되는 동작)
    - 함수에 대한 키워드 인자(`**kwargs`)
    - 클래스의 인스턴스 딕셔너리 (`instance.__dict__`)

> **`collections.OrderedDict`**
> 
> - 파이썬 3.7+의 표준 `dict`의 동작과 비슷하기는 하지만, 성능 특성은 많이 다르다.
> - 키 삽입과 `popitem()` 호출을 매우 자주 처리해야 한다면, 표준 `dict` 보다 `OrderedDict`가 더 낫다.
{: .prompt-info}

### 덕 타이핑 (Duck Typing)

- 파이썬은 정적 타입 지정 언어가 아니므로, 대부분의 경우 엄격한 클래스 계층보다는 **객체의 동작이 객체의 실질적인 타입을 결정**하는 덕 타이핑에 의존한다.
- 덕 타이핑이란 동적 타입 지정의 일종으로, 객체가 실행 시점에 어떻게 행동하는지를 기준으로 객체의 타입을 판단하는 타입 지정 방식이다.
- `collections.abc` 모듈을 사용하여 파이썬에서 제공하는 클래스와 비슷하지만 다르게 동작하는 클래스를 새롭게 정의할 수 있다.

### 덕 타이핑을 고려하여 딕셔너리와 비슷한 클래스를 조심스럽게 다루는 방법

1. `dict` 인스턴스의 삽입 순서 보존에 의존하지 않고 코드를 작성하는 방법 (가장 보수적임)

    ```python
    def get_winner(ranks):
        for name, rank in ranks.items():
            if rank == 1:
                return name
    ```
    
2. 실행 시점에 명시적으로 `dict` 타입을 검사하는 방법
    
    ```python
    def get_winner(ranks):
        if not isinstance(ranks, dict):
            raise TypeError("dict 인스턴스가 필요합니다.")
        return next(iter(ranks)) # dict 인스턴스의 삽입 순서 보존에 의존
    ```
    
3. 타입 어노테이션(annotation)과 정적 분석(static analysis)을 사용해 `dict` 값을 강제하는 방법 (ex. `mypy`)
    
    ```python
    from typing import Dict
    
    def get_winner(ranks: Dict[str, int]) -> str:
        return next(iter(ranks))
    ```
    

<br>

## Better Way 16: `in`을 사용하고 딕셔너리 키가 없을 때 `KeyError`를 처리하기보다는 `get`을 사용하라

### 딕셔너리 키가 없는 경우를 처리하는 방법

1. **`if` 문과 `in`을 사용하는 방법**
    
    ```python
    if key in counters:
        count = counters[key]
    else:
        count = 0
    
    counters[key] = count + 1
    ```
    
2. **`KeyError` 예외를 활용하는 방법**
    
    ```python
    try:
        count = counters[key]
    except KeyError:
        count = 0
    
    counters[key] = count + 1
    ```
    
3. **`dict` 내장 타입의 `get` 메서드를 사용하는 방법** [→ 가장 코드가 짧고 깔끔한 방법 ✅]
    
    > **`get`의 두 번째 인자** = 키(첫 번째 인자)가 딕셔너리에 들어 있지 않을 때 돌려줄 디폴트 값
    > 
    
    ```python
    count = counters.get(key, 0)
    
    counters[key] = count + 1
    ```
    

### 딕셔너리 키가 없는 경우를 처리하는 방법: 복잡한 값(ex. `list`)이 저장되어 있는 경우

> **이중 대입문 `votes[key] = names = []`**
>
> → 참조를 통해 리스트 내용을 변경할 수 있으므로, `names.append()` 호출 후 다시 리스트를 딕셔너리에 대입할 필요가 없다.
{: .prompt-info}

1. **`if` 문과 `in`을 사용하는 방법**
    
    ```python
    if key in votes:
        names = votes[key]
    else:
        votes[key] = names = [] # 이중 대입문
    
    names.append(who)
    ```
    
2. **`KeyError` 예외를 활용하는 방법**
    
    ```python
    try:
        names = votes[key]
    except:
        votes[key] = names = []
    
    names.append(who)
    ```
    
3. **`dict` 내장 타입의 `get` 메서드를 사용하는 방법**
    
    ```python
    names = votes.get(key)
    if names is None:
        votes[key] = names = []
    
    names.append(who)
    ```
    
4. **`dict` 내장 타입의 `get` 메서드와 대입식을 사용하는 방법** [→ 가장 코드가 짧고 깔끔한 방법이므로 권장 ✅]
    
    ```python
    if (names := votes.get(key)) is None:
        votes[key] = names = []
    
    names.append(who)
    ```
    
5. **`dict` 타입이 제공하는 `setdefault` 메서드를 사용하는 방법**
    
    ```python
    names = votes.setdefault(key, [])
    
    names.append(who)
    ```
    
    - `setdefault` 메서드는 딕셔너리에 키가 없으면 제공받은 디폴트 값을 키에 연관시켜 딕셔너리에 대입한 다음, 키에 연관된 값을 반환한다.
    - 전달하는 디폴트 값이 별도로 복사되지 않고 **딕셔너리에 직접 대입**되므로, 디폴트 값을 전달할 때마다 해당 키의 존재 여부와 상관 없이 그 값을 새로 생성해야 한다. 따라서 성능이 크게 저하될 수 있다.
        
        > 디폴트 값으로 이용할 객체를 새로 생성하지 않고 재활용한다면 이상한 동작을 하게 되고 버그가 발생할 수 있다.
        > 
    - 실제로는 `setdefault` 보다 `defaultdict`를 사용하는 것으로 충분할 수 있다.

<br>

## Better Way 17: 내부 상태에서 원소가 없는 경우를 처리할 때는 `setdefault` 보다 `defaultdict`를 사용하라

- `collections` 내장 모듈에 있는 `defaultdict` 클래스는 키가 없을 때 자동으로 디폴트 값을 저장해 딕셔너리를 다룰 때 키가 없는 경우를 쉽게 처리할 수 있다.
    
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
    
- 임의의 키가 들어있는 딕셔너리가 어떻게 생성되었는지 모르는 경우, 딕셔너리의 원소에 접근하려면 우선 `get`을 사용해야 한다. 하지만 `setdefault`가 더 짧은 코드를 만들어내는 몇 가지 경우에는 `setdefault`를 사용하는 것도 고려해볼 수 있다.

<br>

## Better Way 18: `__missing__`을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라

### `setdefault`와 `defaultdict`의 한계점

- 다음의 경우에는 **`setdefault` 메서드를 사용하지 않아야** 한다.
    
    > 키 값의 존재 여부와 상관 없이 디폴트 값을 생성하는 동작이 항상 호출되기 때문이다.
    
    1. 디폴트 값을 생성하는 **비용**이 큰 경우
    2. 디폴트 값을 생성하는 과정에서 **예외가 발생**할 수 있는 상황인 경우
- **`defaultdict`에 전달되는 함수는 인자를 전달 받을 수 없으므로**, 접근에 사용한 키 값과 관련된 디폴트 값을 생성하는 것은 불가능하다.

### `dict` 타입의 하위 클래스 내 `__missing__` 특별 메서드

```python
def open_picture(profile_path):
    try:
        return open(profile_path, "a+b")
    except OSError:
        print(f"경로를 열 수 없습니다: {profile_path}")
        raise

class Pictures(dict):
    def __missing__(self, key):
        value = open_picture(key) # 1) 키에 해당하는 디폴트 값을 생성
        self[key] = value # 2) 딕셔너리에 삽입
        return value # 3) 호출한 쪽에 그 값을 반환

pictures = Pictures()
handle = pictures[path]
handle.seek(0)
image_data = handle.read()
```

- 키가 없는 경우를 처리하는 로직을 커스텀화 할 수 있다.
- **`__missing__` 메서드**는 다음의 동작을 구현해야 한다.
    1. 키에 해당하는 디폴트 값을 생성
    2. 딕셔너리에 삽입
    3. 호출한 쪽에 그 값을 반환
- 딕셔너리에 접근 시 해당 키가 없는 경우 `__missing__` 메서드가 호출되며, 해당 키가 존재하는 경우에는 호출되지 않는다.
