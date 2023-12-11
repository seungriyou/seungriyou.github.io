---
title: "[Better Way #47] 지연 계산 애트리뷰트가 필요하면 __getattr__, __getattribute__, __setattr__을 사용하라"
date: 2023-12-11 10:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

## TL;DR

1. `__getattr__`와 `__setattr__`를 사용해 **객체의 애트리뷰트를 지연해 가져오거나 저장**할 수 있다.
2. `__getattr__`와 `__getattribute__`의 차이점을 이해하라.
    
    
    | 메서드 | 호출 시점 |
    | --- | --- |
    | `__getattr__` | 애트리뷰트가 존재하지 않을 때만 호출된다. |
    | `__getattribute__` | 애트리뷰트에 접근할 때마다 항상 호출된다. |

1. **애트리뷰트에 접근할 때마다** 호출되는 `__getattribute__`와 `__setattr__`에서 **무한 재귀**를 피하려면 **`super()`, 즉 `object` 클래스**에 있는 메서드를 사용하여 인스턴스 애트리뷰트에 접근하라.

<br>

---

<br>

파이썬 **`object` 훅**을 사용하여 시스템을 서로 접합하는 제너릭 코드를 쉽게 작성할 수 있다.

`object` 훅에는 대표적으로 `__getattr__`, `__getattribute__`, `__setattr__`가 있다.

<br>

## `__getattr__` 메서드

- 어떤 클래스 안에 `__getattr__` 메서드 정의가 있으면, 이 <span class="hl">**객체의 인스턴스 딕셔너리에서 찾을 수 없는 애트리뷰트에 접근할 때마다**</span> `__getattr__`이 호출된다.
- 인스턴스 딕셔너리에 **존재하는 애트리뷰트**에 접근할 때는 `__getattr__`이 호출되지 않지만, **존재하지 않는 애트리뷰트**에 접근할 때는 `__getattr__`이 호출된다.
    
    > 따라서 `__getattr__`에서 `setattr`를 수행해 인스턴스 딕셔너리에 해당 애트리뷰트를 추가한다면, 두 번째로 해당 애트리뷰트에 접근할 때부터는 `__getattr__`이 호출되지 않는다.
    {: .prompt-info}
    
- **스키마가 없는 데이터에 지연 계산으로 접근**하는 등의 활용이 필요할 때 유용하다.
    
    > 스키마가 없는 데이터에 접근하면 `__getattr__`이 한 번 실행되면서 프로퍼티를 적재하는 작업을 처리할 수 있도록 구현할 수 있기 때문이다. 이 경우, 그 후 모든 데이터 접근은 기존 결과를 읽게 된다.

```python
class LazyRecord:
    def __init__(self):
        self.exists = 5
    
    def __getattr__(self, name):
        value = f"{name}를 위한 값"
        setattr(self, name, value)
        return value

class LoggingLazyRecord(LazyRecord):
    def __getattr__(self, name):
        print(f"* 호출: __getattr__({name!r}), "
              f"인스턴스 딕셔너리 채워 넣음")
        result = super().__getattr__(name)  # 무한 재귀 방지 및 실제 프로퍼티 값 가져오기 위해 super() 사용
        print(f"* 반환: {result!r}")
        return result
```

```python
data = LoggingLazyRecord()
print(f"이전: {data.__dict__}")
print(f"첫 번째 foo: {data.foo}")
print(f"두 번째 foo: {data.foo}")
print(f"이후: {data.__dict__}")

# 이전: {'exists': 5}
# * 호출: __getattr__('foo'), 인스턴스 딕셔너리 채워 넣음
# * 반환: 'foo를 위한 값'
# 첫 번째 foo: foo를 위한 값
# 두 번째 foo: foo를 위한 값
# 이후: {'exists': 5, 'foo': 'foo를 위한 값'}
```

<br>

## `__getattribute__` 메서드

- <span class="hl">**객체의 애트리뷰트에 접근할 때마다**</span> 호출된다. 즉, 이미 **존재하는 애트리뷰트에 접근할 때에도** 호출된다.
- **존재하지 않는 프로퍼티**에 동적으로 접근하는 경우에는 **`AttributeError`**가 발생한다.
- 프로퍼티에 접근할 때마다 항상 전역 트랜잭션 상태를 검사하는 등의 작업을 수행할 수 있다.
- 부가 비용이 많이 들고 성능에 부정적인 영향을 끼치기도 하나, 때로는 이런 비용을 감수할 만한 가치가 있는 경우도 있다.

```python
class ValidatingRecord:
    def __init__(self):
        self.exists = 5
        
    def __getattribute__(self, name):
        print(f"* 호출: __getattr__({name!r})")
        try:
            value = super().__getattribute__(name)
            print(f"* {name!r} 찾음, {value!r} 반환")
            return value
        except AttributeError:
            value = f"{name}를 위한 값"
            print(f"* {name!r}를 {value!r}로 설정")
            setattr(self, name, value)
            return value
```

```python
data = ValidatingRecord()

print(f"exists: {data.exists}")
print(f"첫 번째 foo: {data.foo}")
print(f"두 번째 foo: {data.foo}")

# * 호출: __getattr__('exists')
# * 'exists' 찾음, 5 반환
# exists: 5
# * 호출: __getattr__('foo')
# * 'foo'를 'foo를 위한 값'로 설정
# 첫 번째 foo: foo를 위한 값
# * 호출: __getattr__('foo')
# * 'foo' 찾음, 'foo를 위한 값' 반환
# 두 번째 foo: foo를 위한 값
```

> 프로퍼티가 존재하는지 검사하는 **`hasattr` 내장 함수**나, 프로퍼티 값을 꺼내오는 **`getattr` 내장 함수**도 동작 전에 **애트리뷰트 이름을 인스턴스 딕셔너리**에서 검색한다.
>
> 따라서 이때도 `__getattr__` 메소드와 `__getattribute__` 메서드가 호출되게 되는데, 그 차이점은 다음과 같다.
> 
> - **`__getattr__` 메서드**: `hasattr`나 `getattr`이 인스턴스 딕셔너리에 존재하지 않는 애트리뷰트를 인자로 가질 때만 호출된다.
> - **`__getattribute__` 메서드**: `hasattr`나 `getattr`이 호출될 때마다 함께 호출된다.
{: .prompt-info}

<br>

## `__setattr__` 메서드

인스턴스의 애트리뷰트에 직접 대입하든, `setattr` 내장 함수를 통해서든 <span class="hl">**값을 대입할 때마다**</span> 항상 호출된다.

```python
class SavingRecord:
    def __setattr__(self, name, value):
        # TODO: 데이터를 데이터베이스 레코드에 저장하는 코드
        ...
        super().__setattr__(name, value)

class LoggingSavingRecord(SavingRecord):
    def __setattr__(self, name, value):
        print(f"* 호출: __setattr__({name!r}, {value!r})")
        super().__setattr__(name, value)
```

```python
data = LoggingSavingRecord()
print(f"이전: {data.__dict__}")
data.foo = 5
print(f"이후: {data.__dict__}")
data.foo = 7
print(f"최종: {data.__dict__}")

# 이전: {}
# * 호출: __setattr__('foo', 5)
# 이후: {'foo': 5}
# * 호출: __setattr__('foo', 7)
# 최종: {'foo': 7}
```

<br>

## 주의할 점: `__getattribute__`, `__setattr__` <span class="hl">무한 재귀</span> 피하기

`__getattribute__`와 `__setattr__`는 원하든 원하지 않든 **어떤 객체의 모든 애트리뷰트에 접근할 때마다 함수가 호출된다는 문제점**이 있다.

> 이러한 무한 재귀 문제를 피하려면 **`super()`에 있는, 즉, `object` 클래스에 있는 메서드**를 사용하여 **인스턴스 애트리뷰트**에 접근해야 한다.
{: .prompt-tip}

<br>

**`__getattribute__`**를 사용할 때 무한 재귀 문제를 해결하는 예시를 살펴보자.

다음과 같이 어떤 객체와 관련된 딕셔너리에 키가 있을 때만 이 객체의 애트리뷰트에 접근하는 코드를 작성하면, `__getattribute__`가 `self._data`에 접근해서 `__getattribute__`가 다시 호출되기 때문에 **무한 재귀 문제**가 발생한다.

```python
class BrokenDictionaryRecord:
    def __init__(self, data):
        self._data = {}
    def __getattribute__(self, name: str):
        print(f"* 호출: __getattribute__({name!r})")
        return self._data[name]


data = BrokeData = BrokenDictionaryRecord({"foo": 3})
data.foo
# RecursionError: maximum recursion depth exceeded while calling a Python object
```

이를 해결하기 위해서는 **`super().__getattribute__`를 호출**해서 인스턴스 애트리뷰트 딕셔너리에서 값을 가져와야 한다.

```python
class DictionaryRecord:
    def __init__(self, data):
        self._data = data
    
    def __getattribute__(self, name):
        print(f"* 호출: __getattribute__({name!r})")
        data_dict = super().__getattribute__("_data") # -- super()!
        return data_dict[name]


data = BrokeData = DictionaryRecord({"foo": 3})
print(f"foo: {data.foo}")
# * 호출: __getattribute__('foo')
# foo: 3
```
