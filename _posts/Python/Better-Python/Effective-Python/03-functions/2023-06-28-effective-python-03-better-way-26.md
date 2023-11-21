---
title: "[Better Way #26] functools.wraps을 사용해 함수 데코레이터를 정의하라"
date: 2023-06-28 21:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch03 functions]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 03. Functions"**을 읽고 정리한 내용입니다.

<br>

## 데코레이터 (decorator)

- 자신이 감싸고 있는 함수가 호출되기 전과 후에 코드를 추가로 실행해준다.
    - 즉, 자신이 감싸고 있는 함수의 입력 인자, 반환 값, 함수에서 발생한 오류에 접근할 수 있다.
    - ex) 재귀 함수를 디버깅할 때 유용하다.
- 데코레이터를 함수에 적용할 때는 **`@` 기호**를 사용한다.

  ```python
  def trace(func):
      def wrapper(*args, **kwargs):
          result = func(*args, **kwargs)
          print(f"{func.__name__}({args!r}, {kwargs!r}) "
                f"-> {result!r}")
          return result
      return wrapper

  @trace
  def fibonacci(n):
      if n in (0, 1):
          return n
      return (fibonacci(n - 2) + fibonacci(n - 1))
  ```
  ```python
  fibonacci(4)
  # fibonacci((0,), {}) -> 0
  # fibonacci((1,), {}) -> 1
  # fibonacci((2,), {}) -> 1
  # fibonacci((1,), {}) -> 1
  # fibonacci((0,), {}) -> 0
  # fibonacci((1,), {}) -> 1
  # fibonacci((2,), {}) -> 1
  # fibonacci((3,), {}) -> 2
  # fibonacci((4,), {}) -> 3
  # 3
  ```
    
    - `trace` 함수는 `wrapper` 함수를 반환한다.
    - 데코레이터로 사용되면 반환된 `wrapper` 함수가 모듈에 `fibonacci`라는 이름으로 등록되게 된다.
        
      ```python
      fibonacci = trace(fibonacci)
      ```
        
    - 기존 함수와 이름이 동일하지만 새롭게 decorated 된 함수(`fibonacci`)는 `wrapper`의 코드를 원래의 함수가 실행되기 전과 후에 실행한다.
- 데코레이터가 감싸고 있는 **원래 함수의 위치를 찾을 수 없기 때문**에 다음의 문제가 발생한다.
    1. **인트로스펙션**(introspection)을 수행하는 도구(ex. 디버거)가 잘못 동작할 수 있다. (ex. `help(fibonacci)` → 독스트링이 출력되지 않는다.)
    2. `pickle` **객체 직렬화**가 깨지게 된다.

<br>

## `functools.wraps` 함수

- 데코레이터 작성을 돕는 데코레이터이다.
- `wraps`를 `wrapper` 함수에 적용하면, **데코레이터 내부에 들어가는 함수**에서 **중요한 메타데이터를 복사**하여 `wrapper` 함수에 적용해준다.

```python
from functools import wraps

def trace(func):
    @wraps(func) # -- functools.wraps 사용
    def wrapper(*args, **kwargs):
        ...
    return wrapper

@trace
def fibonacci(n):
    """n번째 피보나치 수를 반환한다."""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```
```python
# (1) docstring이 출력됨
help(fibonacci)
# Help on function fibonacci in module __main__:
# 
# fibonacci(n)
#     n번째 피보나치 수를 반환한다.

# (2) pickle 객체 직렬화가 제대로 동작함
import pickle
print(pickle.dumps(fibonacci))
# b'\x80\x04\x95\x1a\x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\tfibonacci\x94\x93\x94.'
```
