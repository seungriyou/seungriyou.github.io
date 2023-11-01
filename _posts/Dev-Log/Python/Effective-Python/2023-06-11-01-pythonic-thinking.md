---
title: "[Effective Python] 01. Pythonic Thinking"
date: 2023-06-11 10:50:00 +0900
categories: [Dev-Log, Python]
tags: [python, effective python, book]     # TAG names should always be lowercase
---

> 본문은 "파이썬 코딩의 기술 (Effective Python, 2판)"을 읽고 정리한 내용입니다.

<br>

## Better Way 1: 사용 중인 파이썬의 버전을 알아두라

이제 파이썬 3 버전을 사용해야 하며, 파이썬 버전은 다음과 같이 확인할 수 있다.

```bash
python --version
# Python 3.10.11
```

```python
import sys
print(sys.version_info)
print(sys.version)

# sys.version_info(major=3, minor=10, micro=11, releaselevel='final', serial=0)
# 3.10.11 | packaged by conda-forge | (main, May 10 2023, 19:01:19) [Clang 14.0.6 ]
```

<br>

## Better Way 2: PEP 8 스타일 가이드를 따르라

> <https://peps.python.org/pep-0008/>
> 

### 공백(Whitespace)

- 탭 대신 **스페이스**를 사용해 들여쓰기하라.
- 문법적으로 중요한 들여쓰기에는 **4칸 스페이스**를 사용하라.
- 라인 길이는 **79개 문자 이하**여야 한다.
- 긴 식을 다음 줄에 이어서 쓰는 경우, 일반적인 들여쓰기보다 **4 스페이스를 더** 들여쓴다.
- 파일 안에서 **각 함수와 클래스 사이**에는 빈 줄을 **두 줄** 넣는다.
- 클래스 안에서 **메서드와 메서드 사이**에는 빈 줄을 **한 줄** 넣는다.
- 딕셔너리에서 키와 콜론 사이에는 공백 X, 키와 값을 같이 넣는 경우에는 콜론 다음에 스페이스를 하나 넣는다.
- 변수 대입에서 `=` 전후에는 스페이스를 하나씩만 넣는다.
- 타입 표기를 덧붙이는 경우, 변수 이름과 콜론 사이에 공백 X, 콜론과 타입 정보 사이에는 스페이스를 하나 넣는다.

### 명명 규약

- **함수, 변수, 애트리뷰트**: snake_case (`lowercase_underscore`)
- **보호되어야 하는 인스턴스 애트리뷰트**: 밑줄로 시작 (`_leading_underscore`)
- **비공개(private) 인스턴스 애트리뷰트**: 밑줄 두 개로 시작 (`__leading_underscore`)
- **클래스(예외도 포함)**: PascalCase (`CapitalizedWord`)
- **모듈 수준의 상수**: 모든 글자를 대문자로하고, 단어 사이를 밑줄로 연결 (`ALL_CAPS`)
- **클래스 내 인스턴스 메서드**: 호출 대상 객체를 가리키는 첫 번째 인자로 `self` 사용
- **클래스 메서드**: 클래스를 가리키는 첫 번째 인자로 `cls` 사용

### 식과 문

- 긍정적인 식을 부정하지 말고(`if not a is b`) 부정을 내부에 넣는다. (`if a is not b`)
- **빈 컨테이너나 시퀀스**를 검사할 때는 길이를 0과 비교하지 말고 `False`로 취급된다는 사실을 이용한다. (`if not 컨테이너`)
- **비어있지 않은 컨테이너나 시퀀스**르 검사할 때도 길이를 0과 비교하지 말고 `True`로 취급된다는 사실을 이용한다.
- `if`문, `for`문, `while` 루프, `except` 복합문을 한 줄로 작성하지 말고 여러 줄에 나눠 배치한다.
- 식을 한 줄 안에 다 쓸 수 없다면 **식을 괄호로 둘러싸고 줄바꿈과 들여쓰기를 추가**해서 읽기 쉽게 한다.
- 여러 줄에 걸쳐 식을 쓸 때는 줄이 계속된다는 표시를 하는 `\` 보다는 **괄호**를 사용한다.

### 임포트(Import)

- `import` 문은 항상 파일 맨 앞에 위치한다.
- 모듈 임포트 시 **절대 경로**를 사용한다. 반드시 **상대 경로**로 임포트해야 하는 경우, `from . import foo`와 같이 명시한다.
- **표준 라이브러리 모듈 - 서드 파티 모듈 - 커스텀 모듈** 순서로 섹션을 나눈다. 각 섹션에서는 알파벳 순서로 임포트한다.

<br>

## Better Way 3: `bytes`와 `str`의 차이를 알아두라

| 데이터 종류 | 인스턴스 타입 | 변환 시 사용하는 메서드 | 설명 |
| --- | --- | --- | --- |
| 이진 데이터 | `bytes` | `bytes.decode` | 부호가 없는 8바이트 데이터 |
| 유니코드 데이터 | `str` | `str.encode` | 사람이 사용하는 언어의 문자를 표현하는 유니코드 코드 포인트 |

### 유니코드 샌드위치

- 유니코드 데이터를 인코딩 및 디코딩하는 부분을 인터페이스의 가장 먼 경계 지점에 위치시키는 방식이다.
- 프로그램의 핵심 부분은 **유니코드 데이터 `str`**을 사용해야 하며, 문자 인코딩에 대해 어떠한 가정도 해서는 안 된다.
- 입력으로 다양한 텍스트 인코딩을, 출력으로 한 가지 인코딩(ex. UTF-8)을 제한할 수 있다.
- 일반적으로 시스템 디폴트 인코딩 방식은 `UTF-8`이다.

### 두 가지 도우미 함수

1. `to_str`: 항상 유니코드 데이터 `str` 반환

    ```python
    def to_str(bytes_or_str):
        if isinstance(bytes_or_str, bytes):
            value = bytes_or_str.decode("utf-8")
        else:
            value = bytes_or_str
        return value # str 인스턴스

    print(repr(to_str(b"foo"))) # 'foo'
    print(repr(to_str("bar"))) # 'bar'
    print(repr(to_str(b"\xed\x95\x9c"))) # '한'
    ```
  
2. `to_bytes`: 항상 이진 데이터 `bytes` 반환
  
    ```python
    def to_bytes(bytes_or_str):
        if isinstance(bytes_or_str, str):
            value = bytes_or_str.encode("utf-8")
        else:
            value = bytes_or_str
        return value # bytes 인스턴스
    
    print(repr(to_bytes(b"foo"))) # b'foo'
    print(repr(to_bytes("bar"))) # b'bar'
    print(repr(to_bytes(b"\xed\x95\x9c"))) # b'\xed\x95\x9c'
    ```
  

### `bytes`와 `str` 다룰 때 주의할 점

1. `>`, `==`, `+`, `%`와 같은 연산자를 사용할 때, 두 타입을 섞어서 사용하면 서로 호환되지 않는다.
2. 파일 핸들과 관련한 연산(`open`)들에서 모드를 다르게 지정해주어야 한다.
    
    > 시스템 인코딩을 항상 검사하고, 디폴트 인코딩이 의심스러운 경우에는 명시적으로 `open`에 `encoding` 파라미터를 전달한다.
    > 
    
    | 데이터 종류 | open 시 사용하는 모드 |
    | --- | --- |
    | 유니코드 데이터 `str` | 텍스트 모드 사용 (`w`, `r`) |
    | 이진 데이터 `bytes` | 이진 모드 사용 (`wb`, `rb`) |

<br>

## Better Way 4: C 스타일 형식 문자열을 `str.format`과 쓰기보다는 f-문자열을 통한 인터폴레이션을 사용하라

### 형식화 (Formatting)

> 미리 정의된 문자열에 데이터 값을 끼워넣어 사람이 보기 좋은 문자열로 저장하는 과정

1. **C 스타일 형식화**: `%` 형식화 연산자 사용 (tuple, dictionary로 값 전달)
2. **고급 문자열 형식화 (파이썬 3+)**: 내장함수 `format`, `str.format` 사용 (위치 지정자 `{}` 사용)
3. **f-문자열 (파이썬 3.6+) [이 방법을 권장 ✅]**: 형식 문자열 앞에 `f` 문자를 붙이고, 인터폴레이션 수행 

```python
key = "my_var"
value = 1.234

f_string = f"{key:<10} = {value:.2f}"
c_tuple = "%-10s = %.2f" % (key, value)
str_args = "{:<10} = {:.2f}".format(key, value)
str_kw = "{key:<10} = {value:.2f}".format(key=key, value=value)
c_dict = "%(key)-10s = %(value).2f" % {"key": key, "value": value}

assert c_tuple == c_dict == f_string
assert str_args == str_kw == f_string
```

> 연속된 string은 서로 연결되므로 다음과 같이 여러 줄로 나누어 작성할 수 있다.
> 
> ```python
> for i (item, count) in enumerate(pantry):
>   print(f"#{i+1}: "
> 	f"{item.title():<10s} = "
> 	f"{round(count)}")
> ```
{: .prompt-info}

<br>

## Better Way 5: 복잡한 식을 쓰는 대신 도우미 함수를 작성하라

- **복잡한 식**은 도우미 함수로 옮기고, 특정 로직을 **반복 적용**하려면(단지 두세 번에 불과할지라도) 도우미 함수를 작성하는 것을 권장한다.
- 복잡하지만 한 줄 짜리인 식에 `bool` 연산자 **`or`나 `and`**를 사용하는 것보다 **`if`/`else` 식**을 사용하는 편이 더 가독성이 좋다.
- 코드를 줄여쓰는 것보다 가독성을 좋게 하는 것이 더 가치있다.
- ex) query string을 parsing 하는 예제
  
  ```python
  from urllib.parse import parse_qs
  
  my_values = parse_qs("빨강=5&파랑=0&초록=", keep_blank_values=True)
  print(repr(my_values)) # {'빨강': ['5'], '파랑': ['0'], '초록': ['']}
  ```
  
  ```python
  # query string을 parsing하는 동작이 반복적으로 필요할 경우, 도우미 함수를 작성
  def get_first_int(values, key, default=0):
      found = values.get(key, [""])
      if found[0]:
          return int(found[0])
      return default
  
  red = get_first_int(my_values, "빨강")
  green = get_first_int(my_values, "초록")
  opacity = get_first_int(my_values, "투명도")
  print(f"빨강: {red!r}") # 빨강: 5
  print(f"초록: {green!r}") # 초록: 0
  print(f"투명도: {opacity!r}") # 투명도: 0
  ```
    

<br>

## Better Way 6: 인덱스를 사용하는 대신 대입을 사용해 데이터를 언패킹하라

> 언패킹(unpacking)이란, 한 문장 안에서 여러 값을 대입할 수 있는 문법이다.
> 

### 언패킹 구문의 동작 원리

```python
def bubble_sort(a):
    for _ in range(len(a)):
        for i in range(1, len(a)):
            if a[i] < a[i - 1]:
                a[i - 1], a[i] = a[i], a[i - 1] # 언패킹

names = ["프레즐", "당근", "쑥갓", "베이컨"]
bubble_sort(names)
print(names) # ['당근', '베이컨', '쑥갓', '프레즐']
```

1. 대입문의 우항 `(a[i], a[i - 1])`이 계산된다.
2. 그 결과값이 이름이 없는 새로운 tuple에 저장된다.
3. 대입문의 좌항에 있는 언패킹 패턴 `(a[i - 1], a[i])`을 통해 임시 tuple의 값이 각각의 변수에 저장된다.
4. 이름이 없는 임시 tuple이 사라진다.

<br>

## Better Way 7: `range` 보다는 `enumerate`를 사용하라

### `enumerate` 내장 함수

- 이터레이터를 지연 계산 제너레이터(lazy generator)로 감싼다.
- `(루프 인덱스, 이터레이터의 다음 값)` 쌍을 넘겨준다(`yield`).
- 두 번째 파라미터로 어디부터 수를 세기 시작할지 지정할 수 있다. (default: `0`)
    
  ```python
  flavor_list = ["바닐라", "초콜릿", "피칸", "딸기"]
  for i, flavor in enumerate(flavor_list, 1):
      print(f"{i}: {flavor}")
    
  # 1: 바닐라
  # 2: 초콜릿
  # 3: 피칸
  # 4: 딸기
  ```
    

<br>

## Better Way 8: 여러 이터레이터에 대해 나란히 루프를 수행하려면 `zip`을 사용하라

### `zip` 내장 함수

- 둘 이상의 이터레이터를 지연 계산 제너레이터를 사용해 묶어준다.
- 각 이터레이터의 다음 값이 들어있는 튜플을 반환하며, 이를 `for` 문에서 바로 언패킹할 수 있다.
- 입력 이터레이터의 길이가 서로 다른 경우, **아무 경고 없이 가장 짧은 이터레이터의 길이까지만** 튜플을 반환한다.
    
    > 따라서 이터레이터의 길이가 서로 같지 않을 것으로 예상한다면 **`itertools.zip_longest()` 함수**를 사용하자.
    > 해당 함수는 존재하지 않는 값을 전달된 `fillvalue` 파라미터의 값으로 대신한다. (default: None)
    > 
    {: .prompt-info}
    

<br>

## Better Way 9: `for`나 `while` 루프 뒤에 `else` 블록을 사용하지 말라

### 루프 뒤의 `else`문

> 루프는 그 자체로 의미가 명확해야 하지만, `else` 블록을 사용하면 동작이 직관적이지 않고 혼동을 야기할 수 있다. 따라서 루프 뒤에 `else` 블록을 사용하지 말아야 한다.
>

1. 루프 안에서 `break` 문을 사용하면 `else` 블록이 실행되지 않는다. (즉, 루프가 반복되는 도중에 `break`를 만나지 않은 경우에만 실행된다.)
2. 빈 시퀀스에 대한 루프를 실행하면 `else` 블록이 바로 실행된다. (ex. `for x in []:`)
3. `while` 루프의 조건이 처음부터 `False`인 경우에도 `else` 블록이 바로 실행된다. (ex. `while False:`)

### 대체할 도우미 함수의 두 가지 구현

> **`else` 문을 이용하여 구현한 두 수가 서로소인지 검사하는 로직**
> 
> 
> → 공약수일 가능성이 있는 모든 수를 이터레이션하며 두 수를 나눌 수 있는지 검사)
> 
> ```python
> a = 4
> b = 9
> 
> for i in range(2, min(a, b) + 1):
>     print("검사 중", i)
>     if a % i == 0 and b % i == 0:
>         print("서로소 아님")
>         break
> else:
>     print("서로소")
> 
> # 검사 중 2
> # 검사 중 3
> # 검사 중 4
> # 서로소
> ```


1. 원하는 조건을 찾자마자 빠르게 반환하는 방식
    
    ```python
    def coprime(a, b):
        for i in range(2, min(a, b) + 1):
            if a % i == 0 and b % i == 0:
                return False
        return True

    assert coprime(4, 9)
    assert not coprime(3, 6)
    ```
    
2. 루프 안에서 원하는 대상을 찾았는지 나타내는 결과 변수를 도입하는 방식
    
    ```python
    def coprime_alternate(a, b):
        is_coprime = True
        for i in range(2, min(a, b) + 1):
            if a % i == 0 and b % i == 0:
                is_coprime = False
                break
        return is_coprime

    assert coprime_alternate(4, 9)
    assert not coprime_alternate(3, 6)
    ```
    

<br>

## Better Way 10: 대입식을 사용해 반복을 피하라

### 대입식 (Assignment Expression)

- 코드 중복 문제를 해결하고자 파이썬 3.8+부터 지원되고 있는 구문이다.
- **왈러스 연산자(walrus operator) `:=`** 를 사용한다.
- 일반 대입문 `=`이 쓰일 수 없는 위치에서 변수에 값을 대입할 수 있어 유용하다.
- 같은 식이나 같은 대입문을 여러 번 되풀이하는 부분을 대입식으로 대체할 수 있다.

### [사용 예시 1] 해당 블록 안에서만 사용하는 변수를 강조하지 않기

> 값을 가져와서 그 값을 조건에 맞게 검사한 이후, 해당 블록 안에서만 사용하는 패턴
> 
> 
> ```python
> fresh_fruit = {
>     "사과": 10,
>     "바나나": 8,
>     "레몬": 5
> }
> 
> def make_lemonade(count):
>     print("Make Lemonade")
> 
> def make_cider(count):
>     print("Make Cider")
>     
> def out_of_stock():
>     print("Out of Stock!")
>     
> def slice_bananas(count):
>     print("Slice Bananas")
> 
> class OutOfBananas(Exception):
>     pass
> 
> def make_smoothies(count):
>     print("Make Smoothies")
> ```


- `count` 변수는 `if` 블록 안에서만 사용하므로 `:=` 를 사용할 수 있다.
    
  ```python
  if count := fresh_fruit.get("레몬", 0):
      make_lemonade(count)
  else:
      out_of_stock()
  ```
    
- 대입식이 더 큰 식의 일부분으로 쓰일 때는 괄호로 둘러싸야 한다.
    
  ```python
  if (count := fresh_fruit.get("사과", 0)) >= 4:
      make_cider(count)
  else:
      out_of_stock()
  ```
    
- `if`-`else` 블록에서만 사용하는 `count` 변수를 블록 밖에 선언하지 않는다.
    
  ```python
  if (count := fresh_fruit.get("바나나", 0)) >= 2:
      pieces = slice_bananas(count)
  else:
      pieces = 0 # if 문 이전에 미리 선언해도 ok
      
  try:
      smoothies = make_smoothies(pieces)
  except OutOfBananas:
      out_of_stock()
  ```
    

### [사용 예시 2] switch/case 문 대신하기

```python
if (count := fresh_fruit.get("바나나", 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get("사과", 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get("레몬", 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = "nothing"
```

### [사용 예시 3] do/while 루프 대신하기

```python
def pick_fruit():
    print("Pick Fruit")

def make_juice(fruit, count):
    print("Make Juice")
```

- loop-and-a-half 방식으로 구현한 버전 (w/ 무한 루프)
    
  ```python
  bottles = []
  while True:
      fresh_fruit = pick_fruit()
      if not fresh_fruit:
          break # 중간에서 끝내기

      for fruit, count in fresh_fruit.items():
          batch = make_juice(fruit, count)
          bottles.extend(batch)
  ```
    
- `:=`을 통해 구현한 버전 [✅ 권장!]
    
  ```python
  bottles = []
  while fresh_fruit := pick_fruit():
      for fruit, count in fresh_fruit.items():
          batch = make_juice(fruit, count)
          bottles.extend(batch)
  ```
    

> 파이썬에서 제공하지 않는 **switch/case 문**이나 **do/while 루프**를 대입식을 통해 깔끔하게 구현할 수 있다.
{: .prompt-info}
