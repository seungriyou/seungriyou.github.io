---
title: "[Python] 데코레이터(Decorator)"
date: 2023-10-20 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, decorator, functools]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

## TL;DR 📌

1. 데코레이터를 통해 어떤 타겟 함수에 해당 함수의 코드의 수정 없이 동작을 추가할 수 있다. 이는 타겟 함수를 (데코레이터 내부에 정의된) 데코레이터 함수로 타겟 함수를 감싼 것으로 교체함으로써 동작한다.
2. 표준 라이브러리 `functools`에서 제공하는 데코레이터를 활용하면 memoization, 메서드 오버로딩 등의 동작을 편리하게 사용할 수 있다.
3. 파라미터 데코레이터는 파라미터를 받을 수 있는 데코레이터로, 한 번 더 중첩된 함수로 구현된다.
4. 데코레이터는 클래스 형태로도 구현할 수 있는데, 데코레이터가 복잡하다면 클래스로 구현하는 것을 권장한다.

<br>

## 1. 데코레이터(Decorator) 개요

### 1-1. 데코레이터의 역할

파이썬에서는 데코레이터를 어떠한 타겟 함수(= decorated function)에 달아줌으로써 **해당 함수의 코드를 수정하지 않고도 동작을 추가하거나 동작 방식을 변경**할 수 있다.

그렇다면, 데코레이터는 어떻게 함수의 동작을 수정할 수 있는지에 대해 알아보자.

<br>

### 1-2. 데코레이터의 동작 원리

다음의 코드는 **타겟 함수의 실행 시간을 측정 후 출력하는 기능을 추가**하는 **데코레이터** `clock`의 코드이다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-01.png){: .w-75}

이렇게 작성한 데코레이터를 다음과 같이 타겟 함수 `factorial`의 정의에 달아주면 어떻게 동작할까?

```python
@clock
def factorial(n):
    return 1 if n < 2 else n * factorial(n - 1)
```

바로 다음의 코드와 동일한 동작을 하게 된다.

```python
def factorial(n):
    return 1 if n < 2 else n * factorial(n - 1)

factorial = clock(factorial)    # factorial이 clock(factorial)로 교체된다!
```

즉, 쉽게 말하면 **타겟 함수 `factorial`**이 **`clock(factorial)`으로 교체**된다는 것이다.

> 파이썬의 데코레이터는 **타겟 함수를 다른 함수로 교체하는 syntactic sugar** 이다. 파이썬에서 데코레이터의 본질은 **함수** 또는 **또 다른 callable**이라는 것을 기억하자!
{: .prompt-danger}

<br>

이렇게 타겟 함수를 교체하기 위해 **대부분의 데코레이터에는 내부 함수(`clocked`)를 정의**하고 **그것을 반환**하도록 한다. 이때, 이 내부 함수를 <span class="blue">**데코레이터 함수(decorator function)**</span>이라고 하며, 클로저에 의존하게 된다.

> 위의 코드에서는 `factorial(n)`을 호출하게 되면 `clocked(n)`이 실행되게 된다.
> 

이러한 데코레이터 함수 내부에서는 **타겟 함수를 실행**하며, **타겟 함수의 실험 시점의 앞이나 뒤에 추가 기능**을 넣기도 한다. 이 예제에서는 타겟 함수 `factorial`의 실행 시간을 측정하기 위해 **데코레이터 함수 `clocked` 내부에서 `func(*args)` 호출 전과 후에** 시간을 측정하고, 그 차이를 출력한다.

```python
# 데코레이터 함수
def clocked(*args):
    # ==== 타겟 함수 실행 전 ====
    t0 = time.perf_counter()

    # ==== * 타겟 함수 실행 * ====
    result = func(*args)

    # ==== 타겟 함수 실행 후 ====
    elapsed = time.perf_counter() - t0
    ...
```

<br>

### 1-3. 데코레이터의 특징

- 데코레이터 함수(= 데코레이터의 내부 함수)는 **타겟 함수와 동일한 인자**를 받아야 한다.
    - 이러한 특징을 **transparent** 하다고 한다.
    - `*args`, `**kwargs`를 권장한다.
- 데코레이터는 주로 타겟 함수가 반환해야 하는 것을 그대로 반환한다. 하지만 때에 따라 추가 처리를 하기도 한다.
- 데코레이터와 타겟 함수는 주로 서로 다른 모듈에 위치한다.

<br>

## 2. 표준 라이브러리 `functools`에서 제공하는 데코레이터

### 2-1. `functools.wraps`

데코레이터 작성을 돕는 데코레이터로, 타겟 함수로부터 메타데이터를 복사해와서 데코레이터 함수에 적용해준다.

```python
import functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        ...
```

> 관련 포스팅: [[Better Way #26] functools.wraps을 사용해 함수 데코레이터를 정의하라](/posts/effective-python-03-better-way-26/)
{: .prompt-info}

<br>

### 2-2. `functools.cache` / `functools.lru_cache`

memoization을 구현할 수 있는 데코레이터이다.

> memoization이란, 이전 결과들을 저장하여 반복적인 계산을 피하는 방법이다.
> 

다음과 같이 재귀 함수의 대표적인 예시인 `fibonacci` 함수에 이를 적용한다면, unique 한 `n`의 값 각각에 대해 단 한 번씩만 `finbonacci` 함수가 호출되어 연산의 중복을 방지하므로 성능이 크게 개선된다.

```python
import functools

from clockdeco import clock

# stacked decorators
@functools.cache
@clock 
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 2) + fibonacci(n - 1)
```

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-02.png){: .w-50}

> **데코레이터를 중첩해서 사용한다면? (stacked decorator)**
> 
> **`@`**는 데코레이터 함수를 **바로 밑의 함수에 적용**하는 syntax sugar이기 때문에 **밑에서부터 적용**된다.
> 
> 예를 들면, 다음의 경우에는 `beta` → `alpha` 순서로 적용된다.
> 
> ![](/assets/img/posts/Python/Fluent-Python/2023-10-20-03.png){: .w-50}
> 
> 이렇게 stack 하는 것은 데코레이터가 transparent 하다는 특성을 가지기 때문에 가능하다.
{: .prompt-tip}

<br>

그렇다면 **`@lru_cache`와 `@cache`**는 어떤 점이 다를까?

우선, `@cache`는 파이썬 3.9 이후에 도입된 기능으로, `@lru_cache(maxsize=None)`과 동일하게 동작한다.

즉, `@lru_cache`의 경우에는 `@cache`와는 다르게 메모리 사용량을 `maxsize` 파라미터를 통해 제한할 수 있다.

> **LRU(Least Recently Used)**는 공간이 부족할 때 항목을 삭제하기 위한 정책으로, memoization으로 인해 `maxsize` 만큼의 메모리가 모두 소진되었을 때 최근 참조 시점이 가장 빠른 항목부터 삭제하는 방법이다.
{: .prompt-info}

따라서 long-running 프로세스의 경우에는, `@cache` 보다는 `@lru_cache`를 사용해서 `maxsize` 파라미터를 지정하는 것을 권장한다.

> **`@lru_cache`의 파라미터**
> 
> | 파라미터 | Default 값 | 설명 |
> | --- | --- | --- |
> | `maxsize` | 128 | - 캐시에 저장 가능한 최대 항목 개수이다.<br>- **2의 제곱수**를 사용해야 최적의 성능을 낼 수 있다.<br>- `None`으로 지정하는 경우, LRU 로직이 종료되며, 이는 항목이 절대 삭제되지 않아<br>너무 많은 메모리를 소비할 수 있는 `@cache`와 동일하게 동작한다. |
> | `typed` | `False` | `f(1)`와 `f(1.0)`을 호출할 때, `False`인 경우에는 하나의 항목에 저장되며,<br>`True`인 경우에는 구분된 항목에 저장된다. |

<br>

### 2-3. `functools.singledispatch`

Java의 **메서드 오버로딩(method overloading)**과 비슷한 기능을 지원하도록 돕는 데코레이터이다.

> 메서드/함수 오버로딩이란, **동일한 이름의 메서드/함수**를 **다른 파라미터 타입이나 개수**로 동작 가능하도록 지원하는 기능이다.
>
> 파이썬은 메서드 오버로딩을 지원하지 않는다.
{: .prompt-info}

`@singledispatch` 데코레이터를 붙이는 함수는 **첫 번째 인자의 타입을 기준**으로 하는 **제네릭 함수(generic function)의 entrypoint**가 된다. 이때, 용어를 정리하면 다음과 같다.

- **제네릭 함수 (generic function)**: <span class="hl">타입에 따라</span> 동일한 연산을 각각 다른 방법으로 수행하는 함수들의 집합
- **싱글 디스패치 (single dispatch)**: <span class="hl">하나의 인자</span>를 기준으로 구현체를 고르는 방법 (여기에서는 첫 번째 인자)

<br>

다음은 `htmlize()`라는 함수에 오버로딩을 적용한 예시이다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-04.png){: .w-75}

1. 제네릭 함수의 entrypoint로 설정할 `htmlize()` 함수 위에 `@singledispatch` 데코레이터를 붙인다.
2. 각 구현체 함수들에는 `@<base>.register` 데코레이터를 붙인다.
3. 런타임에 `htmlize()` 함수 호출 시 주어지는 첫 번째 인자의 타입을 이용하여 함수 정의가 구현체들 중에서 정해진다.
    
    > 구현체 함수들의 이름에는 정해진 규칙이 없으며, 예시의 `_`는 이를 잘 나타낸다.
    > 
4. 구현체 함수를 정의한 순서와 상관 없이, 가장 구체적으로 매칭되는 타입이 선택된다.
    
    > ex) `bool`은 `numbers.Integral`의 서브타입이지만 `bool`로 간주된다.
    > 
5. 인자의 타입을 함수 정의에 명시하기 어려운 경우, `@<base>.register(type)`의 형태로 타입을 지정해도 된다.
6. 두 개 이상의 타입을 하나의 구현체에 등록하려면 stack 하면 된다.

<br>

> 이때, 등록하는 타입으로는 concrete implementations(ex. `int`, `list`) 보다 `ABC`s(abstract classes)나 `typing.Protocol`(ex. `numbers.Integral`, `abc.MutableSequence`)을 이용하는 것을 권장한다.
>
> → 이미 존재하거나 미래에 새로 생길 클래스는 `ABC`s의 서브 클래스일 것이기 때문이다!
{: .prompt-tip}

<br>

이러한 `singledispatch`의 장점은 다음과 같다.

1. 하나의 함수에 긴 `if` / `elif` / … 블록을 작성하는 것보다 훨씬 낫다.
2. 각 모듈에서 해당 모듈이 지원하는 타입에 대해 구현체 함수를 등록할 수 있다. 따라서 실제로는 제네릭 함수의 구현체는 서로 같은 모듈에 다 같이 위치하지 않는 경우가 많다.

<br>

## 3. 데코레이터의 종류

### 3-1. 파라미터 데코레이터(Parameterized Decorator)

파라미터를 받을 수 있는 데코레이터를 구현하려면 함수를 한 단계 더 중첩해야 하며, 가장 상위 함수에서 해당 파라미터를 받아야 한다.

1. **데코레이터 팩토리(decorator factory)**: 파라미터를 받고, 실제 데코레이터를 반환한다.
2. **내부 함수(inner function)**: 실제 데코레이터이다.

<br>

따라서 위에서 살펴본 `clock` 데코레이터를 `fmt` 파라미터를 받을 수 있도록 다시 구현하면 다음과 같다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-05.png){: .w-75}

위의 코드에서 **데코레이터 팩토리**는 `clock`, **실제 데코레이터**는 `decorate` 함수가 된다.

<br>

아무 파라미터 없이 실행하면, `DEFAULT_FMT`로 설정된 포맷대로 출력된다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-06.png){: .w-50}

다음과 같이 `fmt` 파라미터에 다른 포맷을 넣으면 다른 포맷으로 출력된다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-07.png){: .w-50}

<br>

### 3-2. 클래스 데코레이터 (Class-Based Decorator)

`__call__` 메서드를 구현한 클래스는 그 자체로도 데코레이터로 사용될 수 있다.

> `__call__` 메서드 안에는 데코레이터 함수가 중첩되어 있고, 이를 반환한다.
> 

이때, **클래스 자체**가 **파라미터 데코레이터 팩토리**가 된다.

<br>

`clock` 데코레이터를 클래스 데코레이터로 다시 구현하면 다음과 같다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-20-08.jpeg){: .w-75}

`__call__` 메서드로 인해 `clock` 인스턴스는 callable이 되며, 이것이 호출되면 인스턴스는 타겟 함수를 데코레이터 함수인 `clocked`로 wrap 한다.

복잡한 데코레이터의 경우에는 이렇게 클래스 데코레이터로 구현하는 것이 가독성이 좋다.

<br>

## References

- “Fluent Python (2nd Edition)”, Ch09. Decorators and Closures
- <https://realpython.com/primer-on-python-decorators/>
- <https://docs.python.org/3/library/functools.html#functools.cache>
- <https://invrtd-h.tistory.com/92>
