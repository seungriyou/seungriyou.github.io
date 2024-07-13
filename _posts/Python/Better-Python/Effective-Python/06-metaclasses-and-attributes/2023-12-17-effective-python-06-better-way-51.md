---
title: "[Better Way #51] 합성 가능한 클래스 확장이 필요하면 메타클래스보다는 클래스 데코레이터를 사용하라"
date: 2023-12-17 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, decorator, metaclass]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

**메타클래스**는 서로 쉽게 합성할 수 없지만 **클래스 데코레이터**는 여러 개를 충돌 없이 사용할 수 있기 때문에, <span class="shl">클래스를 확장하면서 합성해야 할 때 **클래스 데코레이터**를 사용하는 것이 적합</span>하다.

> **클래스 데코레이터**란 <span class="ulr">`class` 인스턴스를 파라미터로 받아서 이를 변경한 클래스나 새로운 클래스를 반환해주는 함수</span>이다.
{: .prompt-info}

예시로 어떤 클래스의 모든 메서드를 감싸서 메서드에 전달되는 인자, 반환 값, 발생한 예외를 모두 출력하는 기능을 추가해보자.

<br>

## 클래스 메서드에 기능 추가하기

### [1] 함수 데코레이터

다음과 같은 디버깅 데코레이터를 정의한다.

```python
from functools import wraps

def trace_func(func):
    if hasattr(func, "tracing"):  # 단 한 번만 데코레이터를 적용한다.
        return func

    @wraps(func)
    def wrapper(*args, **kwargs):
        result = None
        try:
            result = func(*args, **kwargs)
            return result
        except Exception as e:
            result = e
            raise
        finally:
            print(f"{func.__name__}({args!r}, {kwargs!r}) -> {result!r}")

    wrapper.tracing = True

    return wrapper
```

그리고 위의 데코레이터를 새로운 클래스 내 메서드들에 달아준다.

```python
class TraceDict(dict):
    @trace_func
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    @trace_func
    def __setitem__(self, *args, **kwargs):
        return super().__setitem__(*args, **kwargs)

    @trace_func
    def __getitem__(self, *args, **kwargs):
        return super().__getitem__(*args, **kwargs)
```

다음의 코드를 실행해보면 의도한 대로 잘 동작하는 것을 확인할 수 있다.

```python
trace_dict = TraceDict([("안녕", 1)])
trace_dict["거기"] = 2
trace_dict["안녕"]
try:
    trace_dict["존재하지 않음"]
except KeyError:
    pass
```

```
__init__(({'안녕': 1}, [('안녕', 1)]), {}) -> None
__setitem__(({'안녕': 1, '거기': 2}, '거기', 2), {}) -> None
__getitem__(({'안녕': 1, '거기': 2}, '안녕'), {}) -> 1
__getitem__(({'안녕': 1, '거기': 2}, '존재하지 않음'), {}) -> KeyError('존재하지 않음')
```

<br>

> 하지만 이러한 방식에는 꾸미려는 **모든 메서드를 `@trace_func` 데코레이터를 사용해서 재정의해야 한다**는 문제가 있다.
{: .prompt-danger} 

<br>

### [2] 메타클래스

반면, 메타클래스를 이용하면 클래스에 속한 모든 메서드를 자동으로 감쌀 수 있다. 다음과 `Meta.__new__`에 데코레이터로 감싸는 동작을 추가한 메타클래스를 작성한다.

```python
import types

trace_types = (
    types.MethodType,
    types.FunctionType,
    types.BuiltinFunctionType,
    types.BuiltinMethodType,
    types.MethodDescriptorType,
    types.ClassMethodDescriptorType,
)

class TraceMeta(type):
    def __new__(meta, name, bases, class_dict):
        klass = super().__new__(meta, name, bases, class_dict)

        for key in dir(klass):
            value = getattr(klass, key)
            if isinstance(value, trace_types):
                # 데코레이터로 감싸기
                wrapped = trace_func(value)
                setattr(klass, key, wrapped)

        return klass
```

이번에는 `TraceDict` 클래스에 메타클래스를 `TraceMeta`로 지정해준다. 내부 메서드를 새롭게 재정의할 필요가 없음에 유의하자.

```python
class TraceDict(dict, metaclass=TraceMeta):
    pass
```

다음과 같이 잘 동작하는 것을 확인할 수 있다. (`__setitem__`은 출력이 안 되는 듯하다..)

```python
trace_dict = TraceDict([("안녕", 1)])
trace_dict["거기"] = 2
trace_dict["안녕"]
try:
    trace_dict["존재하지 않음"]
except KeyError:
    pass
```

```
__new__((<class '__main__.TraceDict'>, [('안녕', 1)]), {}) -> {}
__getitem__(({'안녕': 1, '거기': 2}, '안녕'), {}) -> 1
__getitem__(({'안녕': 1, '거기': 2}, '존재하지 않음'), {}) -> KeyError('존재하지 않음')
```

<br>

> 하지만 이 방식에도 문제점이 있다. 바로 상위 클래스가 메타클래스를 이미 정의한 경우, **즉 메타클래스가 합성될 때 `metaclass conflict` 오류가 발생한다**는 점이다.
{: .prompt-danger}

```python
class OtherMeta(type):
    pass

class SimpleDict(dict, metaclass=OtherMeta):
    pass

class TraceDict(SimpleDict, metaclass=TraceMeta):
    pass
```

```
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Cell In[10], line 9
      5 class SimpleDict(dict, metaclass=OtherMeta):
      6     pass
----> 9 class TraceDict(SimpleDict, metaclass=TraceMeta):
     10     pass

TypeError: metaclass conflict: the metaclass of a derived class must be a (non-strict) subclass of the metaclasses of all its bases
```

물론, 이를 해결하기 위해 **<span class="shl">메타클래스 상속</span>(`OtherMeta`가 `TraceMeta` 상속)**을 활용할 수 있다.

그러나 다음과 같은 경우에는 메타클래스 상속 또한 활용할 수 없다.

1. <span class="ulg">라이브러리에 있는</span> 메타클래스를 사용하는 경우 (코드를 변경할 수 없기 때문)
2. 유틸리티 메타클래스(여기에서는 `TraceMeta`)를 <span class="ulg">여러 개</span> 사용하고 싶은 경우

즉, 메타클래스를 사용하는 방식은 적용 대상 클래스에 대한 제약이 너무 많다.

```python
class TraceMeta(type): ...

class OtherMeta(TraceMeta):
    pass

class SimpleDict(dict, metaclass=OtherMeta):
    pass

class TraceDict(SimpleDict, metaclass=TraceMeta):
    pass
```

<br>

### [3] 클래스 데코레이터 (권장 ✅)

바로 이러한 문제를 해결하고자 파이썬은 **클래스 데코레이터**를 지원한다.

> **클래스 데코레이터**는 함수 데코레이터처럼 사용할 수 있으며, 데코레이터 함수는 인자로 받은 클래스를 적절히 변경해서 재생성해야 한다.
{: .prompt-info}

```python
def my_class_decorator(klass):
    klass.extra_param = "안녕"
    return klass

@my_class_decorator
class MyClass:
    pass

print(MyClass)
print(MyClass.extra_param)
# <class '__main__.MyClass'>
# 안녕
```

<br>

앞서 다뤘던 `TraceMeta.__new__` 메서드의 핵심 부분을 별도의 함수로 옮겨서 어떤 클래스에 속한 모든 메서드와 함수에 `trace_func`을 적용하는 클래스 데코레이터를 다음과 같이 구현할 수 있다.

```python
def trace(klass):
    for key in dir(klass):
        value = getattr(klass, key)
        if isinstance(value, trace_types):
            wrapped = trace_func(value)
            setattr(klass, key, wrapped)

    return klass
```

이 데코레이터를 다음과 같이 클래스에 적용하면 메타클래스를 사용했을 때와 같은 결과를 얻을 수 있다.

```python
@trace
class TraceDict(dict):
    pass
```

```python
trace_dict = TraceDict([("안녕", 1)])
trace_dict["거기"] = 2
trace_dict["안녕"]
try:
    trace_dict["존재하지 않음"]
except KeyError:
    pass
```

```
__new__((<class '__main__.TraceDict'>, [('안녕', 1)]), {}) -> {}
__getitem__(({'안녕': 1, '거기': 2}, '안녕'), {}) -> 1
__getitem__(({'안녕': 1, '거기': 2}, '존재하지 않음'), {}) -> KeyError('존재하지 않음')
```

<br>

또한, 클래스 데코레이터를 사용하면 **클래스에 이미 메타클래스가 지정되어 있어도 데코레이터를 추가로 사용**할 수 있다.

> <span class="shl">클래스를 확장하면서 합성이 가능</span>하려면 **클래스 데코레이터**를 사용하는 것을 권장한다.
{: .prompt-tip}

```python
class OtherMeta(type):
    pass

@trace
class TraceDict(dict, metaclass=OtherMeta):
    pass
```

```python
trace_dict = TraceDict([("안녕", 1)])
trace_dict["거기"] = 2
trace_dict["안녕"]
try:
    trace_dict["존재하지 않음"]
except KeyError:
    pass
```

```
__new__((<class '__main__.TraceDict'>, [('안녕', 1)]), {}) -> {}
__getitem__(({'안녕': 1, '거기': 2}, '안녕'), {}) -> 1
__getitem__(({'안녕': 1, '거기': 2}, '존재하지 않음'), {}) -> KeyError('존재하지 않음')
```
