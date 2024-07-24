---
title: "[Python] 클래식 코루틴(Classic Coroutine)"
date: 2024-02-05 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, coroutine, classic coroutine]
math: true
---

## 1. 코루틴(Coroutine)

파이썬에서 **`tuple` 인스턴스**는 다음과 같이 두 가지로 사용될 수 있다.

1. **레코드 (record)**
    - 아이템의 개수가 정해져있으며, 각 아이템은 다른 타입을 가질 수 있다.
    - 타입 힌트 예시: `city: tuple[str, str, int]`
2. **불변 리스트 (immutable lists)**
    - 아이템의 개수가 정해져있지 않으며, 모든 아이템은 같은 타입을 가진다.
    - 타입 힌트 예시: `domains: tuple[str, ...]`

<br>

파이썬에서는 **`generator`**도 마찬가지로, 다음의 두 가지로 사용될 수 있다.

1. <span class="shlp">**이터레이터 (iterator)**</span>
2. <span class="shlp">**코루틴 (coroutine)**</span>

<br>

### 1-1. 클래식 코루틴 vs. 네이티브 코루틴

#### [1] 클래식 코루틴(classic coroutine)
- `my_coro.send(data)` 호출을 통해 데이터를 소비하는 <span class="red">**제너레이터 함수**</span>이다.
- 데이터를 읽어들일 때 `yield`를 사용한다.
- 다른 클래식 코루틴에게 위임할 때 `yield from`을 사용한다.
- `asyncio`에 의해 지원되지 않는다.
- 타입 힌트: `Generator`를 사용한다.
  
    > `typing` 모듈 (< Python 3.9) / `collections.abc` 모듈 (≥ Python 3.9)
    > 
    
    ```python
    Generator[YieldType, SendType, ReturnType]
    ```
    
    > 혼동에 주의하자! 타입 이름은 **`Generator`**이지만 사실 **클래식 코루틴**을 나타낸다. 일반적인 제너레이터는 이터레이터(`Iterator`)에 해당하기 때문이다.
    {: .prompt-warning}
    
    - **`SendType`**: `gen.send(x)` 호출 시 `x`의 타입 (코루틴일 때만 O)
    - **`ReturnType`**: 반환하는 값의 타입 (코루틴일 때만 O)
        
        > 코루틴이 아닌 이터레이터로 동작하는 제너레이터는 값을 반환하지 않는다. 이때는 `None`과 `typing.NoReturn`을 모두 사용할 수 있다.
        > 
    - **`YieldType`**: `next(it)` 호출 시 반환되는 값의 타입

> 사실상 **클래식 코루틴**이란 `yield` 키워드를 가지는 **제너레이터 함수**이며, **코루틴 객체**란 실질적으로 **제너레이터 객체**이다.
{: .prompt-info}
    
#### [2] 네이티브 코루틴(native coroutine)
- `async def`로 정의된 코루틴이다.
- 다른 네이티브 코루틴에게 위임할 때 `await` 키워드를 사용한다.
- 파이썬 3.5+부터는 “코루틴”이라 하면 “네이티브 코루틴”을 의미한다.
- 타입 힌트: `Coroutine`을 사용한다.
    
    ```python
    Coroutine[YieldType, SendType, ReturnType]
    ```
        

<br>

### 1-2. 제너레이터 vs. 코루틴

- **제너레이터**는 이터레이션 시 <span class="shl">**데이터를 생성**</span>한다.
- **코루틴**은 <span class="shl">**데이터를 소비**</span>하며, 이터레이션과 관련이 없다.

<br>

## 2. 클래식 코루틴(Classic Coroutine)

제너레이터 함수라고 생각할 수 있는 클래식 코루틴의 예제 코드를 살펴보며, 동작 과정에 대해 알아보자.

> 본래 “코루틴”이라고 하면 주로 “네이티브 코루틴”을 지칭하나, 편의를 위해 본 섹션에서는 “클래식 코루틴”을 “코루틴”이라 칭하겠다.
> 

### 2-1. Running Average 계산 코루틴

```python
from collections.abc import Generator

def averager() -> Generator[float, float, None]:
    total = 0.0
    count = 0
    average = 0.0
    
    while True:
        term = yield average  # -- term은 caller로부터 send 된 값을 저장한다.
        total += term
        count += 1
        average = total / count
```

running average를 계산하는 **클래식 코루틴(= 제너레이터)**인 `averager()`는 **무한 루프**를 통해 클라이언트가 값을 보내는 한 계속해서 `average` 값을 yield 한다.

클래식 코루틴 내 `yield` 문은 **코루틴을 suspend** 하고, 다음의 동작을 수행한다.

1. 호출자에게 **결과를 yield** 한다.
    
    > 이전에 계산해둔 `average` 값을 yield 한다.
    > 
2. 그 후, 호출자로부터 **전달(`.send()`)된 값을 전달받는다**.
    
    > 따라서 `term`은 호출자로부터 전달된 값을 저장하게 된다.
    > 

<br>

이처럼 코루틴을 통해 객체 애트리뷰트나 클로저 없이도 <span class="blue">**컨텍스트(context)**</span>를 가질 수 있다. 이러한 컨텍스트는 다음 `.send()` 까지 **코루틴이 suspended 된 동안 유지**된다.

따라서 코루틴을 통해 activation 간 local state를 유지할 수 있으므로, 비동기 프로그래밍에서 콜백(callback) 대신 사용된다.

> **콜백(callback)**
>
> <span class="hl">특정 이벤트가 발생하면 자동으로 호출되어 실행되는 함수</span>이다. 사용자 정의 코드에서 실행되는 일반적인 함수와는 다르게, 반대로 함수 내에서 사용자 정의 코드를 호출한다. 이는 다른 코드의 인자로 넘겨주는 실행 가능한 코드이며, 콜백을 전달받는 코드는 해당 콜백을 필요에 따라 즉시 실행할 수도, 나중에 실행할 수도 있다.
{: .prompt-info}

<br>

위와 같이 구현한 `averager()`를 사용하는 코드를 살펴보자.

```python
coro_avg = averager()

next(coro_avg)   # -- priming the coroutine
# 0.0
coro_avg.send(10)
# 10.0
coro_avg.send(30)
# 20.0
coro_avg.send(5)
# 15.0
coro_avg.send(20)
# 16.25
coro_avg.close()  # -- explicitly terminate

coro_avg.send(40) # -- already closed
# StopIteration:
```

1. 우선, 처음에 `next(coro_avg)`를 호출함으로써 **코루틴 프라이밍(priming the coroutine)**을 수행한다.
    - 이는 코루틴에서 초기값을 yield 함으로써 **첫 `yield` 문까지 진전**시키기 위한 단계로, `next(coro_avg)` 혹은 `coro_avg.send(None)`을 호출함으로써 수행한다.
    - 이때, 코루틴은 호출자가 보낸 값을 `yield` 라인에서 suspended 되어있는 경우에만 받을 수 있기 때문에 `.send()` 시에는 `None`이 아닌 값을 전달하면 안 된다.
2. 각 activation 이후, 코루틴은 `yield` 키워드에서 suspended 되며, 호출자로부터 값이 `send` 되는 것을 기다린다.
3. 제너레이터에 대한 유효한 참조가 더 이상 없을 때면 가비지 컬렉트(garbage collect) 되므로, 대부분의 경우 제너레이터를 terminate 할 필요가 없기 때문에 무한 루프로 구현한다.
4. 만약 명시적으로 제너레이터를 terminate 하고 싶다면 `.close()` 메서드를 사용한다.
    - **`.close()` 메서드**는 **suspended `yield` 문**에서 **`GeneratorExit`**을 발생시킨다.
    - 코루틴 함수에서 처리되지 않으면 제너레이터를 terminate 한다.
    - `GeneratorExit`은 코루틴을 감싸고 있는 제너레이터 객체에 의해 catch 된다.
    - **`.close()` 된 코루틴에 `.send()`를 호출**하면 **`StopIteration`**이 발생한다.
    
    > 정리하면, **`.close()` 메서드를 호출**하면 **`GeneratorExit`** exception이 발생한다.
    >
    > 이렇게 `.close()` 된 코루틴에 **한 번 더 `.send()` 메서드를 호출**하면 **`StopIteration` exception**이 발생한다.
    {: .prompt-tip}
    

<br>

### 2-2. Running Average 계산 코루틴에서 값 반환하기

우선, 코루틴에서 반환할 값의 타입인 `Result` 클래스와, 코루틴을 중지시키고 값을 반환시키기 위해 코루틴에게 send 할 `Sentinel` 클래스를 선언한다.

```python
from collections.abc import Generator
from typing import Union, NamedTuple, TypeAlias

class Result(NamedTuple):
    count: int  # type: ignore
    average: float

class Sentinel:
    def __repr__(self):
        return f'<Sentinel>'

STOP = Sentinel()

SendType: TypeAlias = float | Sentinel  # -- Python 3.10+
```

- `Result` 클래스는 `NamedTuple`을 상속 받아 생성하므로 `.count()` 메서드를 가지고 있는 `tuple`의 하위 클래스가 된다. 따라서 mypy 오류가 뜨지 않도록 `# type: ignore` comment를 추가한다.
- sentinel 값은 코루틴이 데이터를 그만 collect 하고 결과 값을 반환하도록 한다.
- 코루틴의 타입 힌트로 추가할 `SendType`을 선언한다. 이는 `float` 혹은 `Sentinel`이 될 수 있다.
    
    > 이때, 파이썬 3.10+에서는 `typing.Union` 대신 `|`를 이용하고 `TypeAlias`를 사용하는 것을 권장한다.
    > 
    
    ```python
    SendType = Union[float, Sentinel] # <= Python 3.9
    SendType: TypeAlias = float | Sentinel  # > Python 3.10
    ```
    

<br>

다음은 새로운 코루틴 `averager2()` 이다.

```python
def averager2(verbose: bool = False) -> Generator[None, SendType, Result]:
    total = 0.0
    count = 0
    average = 0.0

    while True:
        term = yield
        if verbose:
            print('received:', term)
        if isinstance(term, Sentinel):
            break
        total += term
        count += 1
        average = total / count

    return Result(count, average)
```

- 이 코루틴은 데이터를 yield 하지 않으므로 `YieldType`이 `None`이며, `SendType`의 데이터를 받아 동작이 끝나면 `Result` 튜플을 반환한다.
- 위의 코드와 같이 `yield`를 사용하는 것은, **데이터를 consume 하도록 설계된 코루틴**에서만 가능하다.
    
    > `None`을 yield 하지만, `.send(term)`으로부터 `term`을 전달받는다.
    > 
- 전달받은 `term`이 `Sentinel`인 경우, 루프로부터 break 하고 `return` 문에 도달한다.
    
    > `Sentinel`이 send 된 경우에만 `return` 문이 실행된다.
    > 

<br>

이렇게 작성한 `averager2()`의 동작을 살펴보자.

1. <span class="shlp">**코루틴이 return 되기 전에 terminate 되는 경우**</span>
    
   ```python
   coro_avg = averager2()
   next(coro_avg) # -- priming the coroutine
   coro_avg.send(10)
   coro_avg.send(30)
   coro_avg.send(6.5)
   coro_avg.close()
   ```
    
    - [예제 #1]에서와 다르게 데이터를 yield 하지 않으므로, `.send()`를 해도 yield 되는 값이 없다.
    - `Result`가 return 되지 않은 상황에서 `.close()`가 호출된 경우, `yield` 라인에서 `GeneratorExit` exception이 발생하므로 `return` 문에 도달할 수 없다.

2. <span class="shlp">**코루틴에게 sentinel 값을 send 하여 값을 반환 받는 경우**</span>
    
   ```python
   coro_avg = averager2()
   next(coro_avg) # -- priming the coroutine
   coro_avg.send(10)
   coro_avg.send(30)
   coro_avg.send(6.5)
   
   try:
       coro_avg.send(STOP)
   except StopIteration as exc:
       result = exc.value
   
   result
   # Result(count=3, average=15.5)
   ```
    
    - **`STOP` sentinel**을 통해 코루틴이 **루프로부터 break** 하고 **`Result`가 반환**되면, 코루틴을 감싸고 있는 제너레이터 객체가 **`StopIteration`을 raise** 한다.
    - `StopIteration` 객체는 **`return` 문의 값을 저장**하고 있는 **`value` 애트리뷰트**를 가진다.

<br>

또한, 다음과 같이 값을 반환하는 제너레이터를 위임할 수도 있다.

```python
def compute():
    res = yield from averager2(True)    # -- averager2의 return value를 저장한다.
    print('computed:', res)
    return res

comp = compute()

for v in [None, 10, 20, 30, STOP]:
    try:
        comp.send(v)
    except StopIteration as exc:
        result = exc.value

# received: 10
# received: 20
# received: 30
# received: <Sentinel>
# computed: Result(count=3, average=20.0)

result
# Result(count=3, average=20.0)
```

<br>

> 새로운 coroutine-based framework를 처음부터 개발하는 것이 아닌 이상, `.send()`나 `.throw()` 메서드를 사용하여 코루틴을 by hand로 실행하는 것을 추천하지 않는다.
{: .prompt-info}

<br>

## References

- “Fluent Python (2nd Edition)”, Ch17. Iterators, Generators, and Classic Coroutines
- <https://www.fluentpython.com/extra/classic-coroutines/>
- <https://blog.humminglab.io/posts/python-coroutine-programming-1/>
