---
title: "[Python] 컨텍스트 매니저 프로토콜(Context Manager Protocol)과 with 문"
date: 2023-11-05 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, context manager, with]
math: true
---

## 1. 컨텍스트 매니저와 `with` 블록

> **이터레이터(iterator)**가 `for` 문을 제어하기 위해 존재하듯, **컨텍스트 매니저 객체(context manager object)**는 `with` 문을 제어하기 위해 존재한다.
{: .prompt-info}

### 1-1. `with` 문

코드 블록이 어떠한 이유로든 종료되더라도, 해당 블록 이후에 특정 연산이 수행될 수 있도록 보장한다.

즉, `try` / `finally` 구조를 단순화했다고 생각할 수 있다.

<br>

### 1-2. 컨텍스트 매니저 인터페이스(Context Manager Interface)

컨텍스트 매니터 인터페이스는 다음의 두 메서드를 포함한다.

1. **`__enter__` 메서드**: `with` 블록에 진입할 때 호출된다.
    - **return 값**은 **`as` 절에 등장하는 타겟 변수**에 bound 된다.
        
        > `as` 절은 optional 이다. 사용자에게 전달할 유용한 객체가 없는 경우, 컨텍스트 매니저는 `None`을 return 하기 때문이다.
        {: .prompt-warning}
        
    - 보통 **컨텍스트 매니저 객체(`self`)**를 return 하나, **다른 객체**를 return 할 수도 있다.
2. **`__exit__` 메서드**: 어떠한 이유로든 `with` 블록이 완료/종료될 때 호출된다.
    - `__enter__` 메서드에서 어떤 것이 return 되었든 간에, 컨텍스트 매니저 객체에서 호출된다.

<br>

파이썬에서 컨텍스트 매니저를 가장 보편적으로 사용하는 상황은 파일을 다룰 때이다.

```python
with open('scripts/ch10/10-3.py') as fp:
    src = fp.read(60)

# with 블록을 벗어났지만 여전히 fp variable은 사용가능하다.
fp
# <_io.TextIOWrapper name='scripts/ch10/10-3.py' mode='r' encoding='UTF-8'>

fp.closed, fp.encoding
# (True, 'UTF-8')

# 하지만 파일이 이미 closed 되었으므로 더 이상 읽어들일 수 없다.
fp.read(60)
# ValueError: I/O operation on closed file.
```

<br>

### <span class="blue">[컨텍스트 매니저 생성 방법 #1]</span> 클래스 기반 컨텍스트 매니저

> **`__enter__` 메서드와 `__exit__` 메서드를 구현**한 클래스의 인스턴스를 컨텍스트 매니저로 사용할 수 있다.
{: .prompt-tip}

```python
import sys

class LookingGlass:
    
    def __enter__(self):
        self.original_write = sys.stdout.write
        sys.stdout.write = self.reverse_write
        return 'JABBERWOCKY'
    
    def reverse_write(self, text):
        self.original_write(text[::-1])
    
    def __exit__(self, exc_type, exc_value, traceback):
        sys.stdout.write = self.original_write
        if exc_type is ZeroDivisionError:
            print('Please DO NOT divide by zero!')
            return True
```

```python
with LookingGlass() as what:
    print('Alice, Kitty, and Snowdrop')
    print(what)
# pordwonS dna ,yttiK ,ecilA
# YKCOWREBBAJ

what
# 'JABBERWOCKY'

print('Back to normal')
# Back to normal
```

- **`__enter__` 메서드**
    - 타겟 변수 `what` 에 `__enter__` 메서드의 return 값이 bound 된다.
    - `with` 블록 내에서는 `__enter__` 내에서 설정한 대로 동작한다.

- **`__exit__` 메서드**
    - 아무 이상이 없다면, `__exit__(None, None, None)`이 호출될 것이다.
    - 만약 `__exit__` 메서드가 **`None`** 또는 **any falsey value**를 return 한다면, `with` 블록 내에서 발생한 exception은 propagate 된다.
    - 세 가지 인자는 다음과 같다.
        - `exc_type`: the exception class
        - `exc_value`: the exception instance
        - `traceback`: traceback object

<br>

> Python 3.10부터는 **parenthesized context manager**를 통해 **nested `with` blocks**를 대체할 수 있다.
> 
> 
> ```python
> with (
>     CtxManager1() as ex1,
>     CtxManager2() as ex2,
>     CtxManager3() as ex3,
> ):
>     ...
> ```
{: .prompt-tip}

<br>

## 2. `contextlib` 유틸리티

`contextlib`는 파이썬 표준 라이브러리에 포함된 패키지로, 컨텍스트 매니저를 생성, 결합, 사용하는 데에 유용한 함수, 클래스, 데코레이터를 제공한다.

> ref: <https://docs.python.org/3/library/contextlib.html>

### <span class="blue">[컨텍스트 매니저 생성 방법 #2]</span> `@contextmanager` 기반 컨텍스트 매니저

```python
import contextlib
import sys

@contextlib.contextmanager
def looking_glass():
    original_write = sys.stdout.write
    
    def reverse_write(text):
        original_write(text[::-1])
    
    sys.stdout.write = reverse_write   # with 블록 진입 시점, 즉 __enter__ 호출 시 실행
    yield 'JABBERWORKY'                # __enter__가 return 하는 값, 즉 as 절의 타겟 변수에 bind
    sys.stdout.write = original_write  # with 블록 마지막 시점, 즉 __exit__ 호출 시 실행
```

```python
with looking_glass() as what:
    print('Alice, Kitty, and Snowdrop')
    print(what)
# pordwonS dna ,yttiK ,ecilA
# YKROWREBBAJ

print('back to normal')
# back to normal
```

**`@contextlib.contextmanager`**는 **하나의 `yield`**를 가지는 **제너레이터 함수**로부터 컨텍스트 매니저를 생성할 수 있도록 하는 데코레이터이다.

- `yield` 하는 값은 `__enter__` 메서드가 return 하는 값이어야 한다.
- `__enter__`, `__exit__` 메서드를 구현하는 클래스로 제너레이터 함수를 wrap 하므로, 전체 클래스를 작성할 필요가 없다.

<br>

decorated 된 제너레이터 함수는 `yield`를 기준으로 다음과 같이 나뉜다.

- **`yield` 문 이전**: `with` 블록 진입 시점, 즉, 인터프리터가 `__enter__`를 호출할 때 실행된다.
- **`yield` 값**: `__enter__`가 return 하는 값으로, `as` 절의 타겟 변수에 bind 된다.
- **`yield` 문 이후**: `with` 블록의 마지막에서 `__exit__`이 호출될 때 실행된다.

<br>

데코레이터에 의해 생성된 클래스가 가지는 `__enter__`, `__exit__` 메서드의 동작은 각각 다음과 같다.

- **`__enter__` 메서드**
    1. 제너레이터 함수를 호출하여 제너레이터 객체 `gen`을 얻는다.
    2. `yield` 문까지 전진시키기 위해 `next(gen)`을 호출한다.
    3. `next(gen)`에서 yield 된 값을 반환하고, `as` 절의 변수에 bind 한다.

- **`__exit__` 메서드**
    1. `exc_type`으로 exception이 전달되었으면 `gen.throw(exception)`이 호출된다. 이때, `yield` 라인에서 exception이 raise 된다.
    2. exception이 전달되지 않았다면 `next(gen)`이 호출되어 `yield` 이후의 동작을 재개한다.

<br>

### [예외 처리] 클래스 기반 vs. `@contextmanager` 기반

```python
import contextlib
import sys

@contextlib.contextmanager
def looking_glass():
    original_write = sys.stdout.write
    
    def reverse_write(text):
        original_write(text[::-1])
    
    sys.stdout.write = reverse_write
    msg = ''
    try:
        yield 'JABBERWOCKY'
    except ZeroDivisionError:
        msg = 'Please DO NOT divide by zero!'
    finally:
        sys.stdout.write = original_write
        if msg:
            print(msg)
```

**클래스 기반 컨텍스트 매니저**에서 **`__exit__` 메서드의 return 값**은 exception과 관련이 있다.

다음의 표는 return 값에 따라 달라지는 인터프리터의 동작을 기술한다.

| Return Value | Description | Action of the Interpreter |
| ---- | ---- | ---- |
| a truthy value | exception이 **handling 되었음**을 의미한다. | exception을 **suppress** 한다.  |
| no explicitly returned value | 인터프리터는 `None`을 받는다. | exception을 **propagate** 한다. |

하지만 **`@contextmanager` 기반 컨텍스트 매니저**에서 `__exit__` 메서드는 제너레이터로 보내진 **모든 exception은 handling 되었다고 가정**한다.

따라서 `@contextmanager`를 사용할 때는 `yield` 주변에 `try`/`finally` (혹은 `with` 블록) 처리가 필요하다.

<br>

### 2-1. 데코레이터로 사용하기

`@contextmanager`로 decorated 된 제너레이터를 다시 데코레이터로 사용할 수도 있다.

```python
@looking_glass()
def verse():
    print('The time has come')

verse()
# emoc sah emit ehT

print('back to normal')
# back to normal
```

<br>

## References

- “Fluent Python (2nd Edition)”, Ch18. with, match, and else Blocks
