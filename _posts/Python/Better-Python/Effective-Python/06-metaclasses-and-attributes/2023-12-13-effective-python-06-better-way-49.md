---
title: "[Better Way #49] __init_subclass__를 사용해 클래스 확장을 등록하라"
date: 2023-12-13 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, inheritance, metaclass]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

**메타클래스를 이용해 클래스를 확장**하는 다른 용례로 <span class="shl">**프로그램이 자동으로 타입을 등록**</span>하는 것이 있다.

간단한 식별자를 이용해 그에 해당하는 클래스를 찾는 역검색을 하고 싶을 때 이런 기능이 유용하다.

예시로 <span class="ulr">**직렬화 및 역직렬화 기능**</span>을 구현해보고, 이를 메타클래스 및 `__init_subclass__`를 이용하여 점차 개선해보며 살펴보자.

<br>

## 직렬화 및 역직렬화 기능: 구현하기

### [1] 데이터 타입을 미리 알아야 가능한 코드

다음과 같은 `Serializable` 클래스를 상속하여 불변 데이터 구조를 쉽게 직렬화할 수 있다.

```python
import json

class Serializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({"args": self.args})
```

```python
# Point2D: 불변 데이터 구조
class Point2D(Serializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point2D({self.x}, {self.y})"

point = Point2D(5, 3)
print("객체:", point)
print("직렬화한 값:", point.serialize())

# 객체: Point2D(5, 3)
# 직렬화한 값: {"args": [5, 3]}
```

<br>

이러한 `Serializable`을 상속하여 데이터를 역직렬화하는 클래스 `Deserializable`을 작성할 수 있다.

```python
class Deserializable(Serializable):
    @classmethod
    def deserialize(cls, json_data):
        params = json.loads(json_data)
        return cls(*params["args"])
```

```python
class BetterPoint2D(Deserializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point2D({self.x}, {self.y})"

before = BetterPoint2D(5, 3)
print("이전:", before)
data = before.serialize()
print("직렬화한 값:", data)
after = BetterPoint2D.deserialize(data)
print("이후:", after)

# 이전: Point2D(5, 3)
# 직렬화한 값: {"args": [5, 3]}
# 이후: Point2D(5, 3)
```

<br>

> 이러한 방식에는 **직렬화할 데이터의 타입(`Point2D`, `BetterPoint2D` 등)을 미리 알고 있는 경우**에만 사용할 수 있다는 문제가 있다. JSON으로 직렬화할 클래스가 아주 많더라도 JSON 문자열을 적당한 파이썬 object로 **역직렬화하는 함수는 공통으로 하나만 있는 것**이 이상적이다.
{: .prompt-danger}

<br>

### [2] 역직렬화 기능을 공통 함수로 뺀 코드

```python
class BetterSerializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({"class": self.__class__.__name__, "args": self.args})

    def __repr__(self):
        name = self.__class__.__name__
        args_str = ", ".join(str(x) for x in self.args)
        return f"{name}({args_str})"
```

```python
registry = {}

def register_class(target_class):
    # registry에 target_class 이름으로 등록하기
    registry[target_class.__name__] = target_class

def deserialize(data):
    params = json.loads(data)
    name = params["class"]
    target_class = registry[name]
    return target_class(*params["args"])
```

<br>

이러한 `deserialize` 함수가 항상 제대로 동작하려면 나중에 역직렬화할 모든 클래스에서 `register_class` 함수를 호출해야 한다.

```python
class EvenBetterPoint2D(BetterSerializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

# register_class 함수를 호출해야 함!
register_class(EvenBetterPoint2D)

before = EvenBetterPoint2D(5, 3)
print("이전:", before)
data = before.serialize()
print("직렬화한 값:", data)
after = deserialize(data)
print("이후:", after)

# 이전: EvenBetterPoint2D(5, 3)
# 직렬화한 값: {"class": "EvenBetterPoint2D", "args": [5, 3]}
# 이후: EvenBetterPoint2D(5, 3)
```

<br>

> 이 방식의 문제점은 **`register_class` 함수 호출을 잊어버릴 수 있다**는 것이다. 클래스 데코레이터도 마찬가지로, 호출을 잊어버리는 실수가 발생 가능하다.
> 
> 
> ```python
> class Point3D(BetterSerializable):
>     def __init__(self, x, y, z):
>         super().__init__(x, y, z)
>         self.x = x
>         self.y = y
>         self.z = z
> 
> # register_class 호출을 잊어버림!
> 
> point = Point3D(5, 9, -4)
> data = point.serialize()
> deserialize(data)
> ```
> 
> ```
> ---------------------------------------------------------------------------
> KeyError                                  Traceback (most recent call last)
> Cell In[9], line 12
>      10 point = Point3D(5, 9, -4)
>      11 data = point.serialize()
> ---> 12 deserialize(data)
> 
> Cell In[7], line 10
>       8 params = json.loads(data)
>       9 name = params["class"]
> ---> 10 target_class = registry[name]
>      11 return target_class(*params["args"])
> 
> KeyError: 'Point3D'
> ```
{: .prompt-danger}

<br>

## 직렬화 및 역직렬화 기능: 개선하기

### [1] 메타클래스를 이용한 구현

메타클래스를 이용하면 프로그래머가 `BetterSerializable`을 사용하는 의도를 감지하여 항상 제대로 `register_class`를 호출해줄 수 있다.

<span class="shl">**메타클래스는 하위 클래스가 정의될 때 `class` 문을 가로채서 이러한 추가적인 동작(= 새로운 타입 등록)을 수행**</span>할 수 있다.

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        cls = type.__new__(meta, name, bases, class_dict)
        # class 문을 가로채 class를 등록함
        register_class(cls)
        return cls
```

```python
class RegisteredSerializable(BetterSerializable, metaclass=Meta):
    pass
```

<br>

이렇게 메타클래스를 지정하면 `RegisteredSerializable`의 하위 클래스를 정의할 때 `register_class` 함수가 호출되고 `deserialize`가 항상 제대로 동작하는 것을 보장할 수 있다.

```python
class Vector3D(RegisteredSerializable):
    def __init__(self, x, y, z):
        super().__init__(x, y, z)
        self.x, self.y, self.z = x, y, z

before = Vector3D(10, -7, 3)
print("이전:", before)
data = before.serialize()
print("직렬화한 값:", data)
print("이후:", deserialize(data))

# 이전: Vector3D(10, -7, 3)
# 직렬화한 값: {"class": "Vector3D", "args": [10, -7, 3]}
# 이후: Vector3D(10, -7, 3)
```

<br>

### [2] `__init_subclass__` 메서드를 이용한 구현

**`__init_subclass__` 특별 클래스 메서드**는 파이썬 3.6부터 도입된 방식으로, 이를 사용하면 <span class="shl">**클래스를 정의할 때 커스텀 로직을 제공**</span>할 수 있다.

이를 이용하면 혼동하기 쉬운 **메타클래스 구문을 대체**할 수 있다.

```python
class BetterRegisteredSerializable(BetterSerializable):
    def __init_subclass__(cls):
        super().__init_subclass__()
        # class 문을 가로채 class를 등록함
        register_class(cls)
```

```python
class Vector1D(BetterRegisteredSerializable):
    def __init__(self, magnitude):
        super().__init__(magnitude)
        self.magnitude = magnitude

before = Vector1D(6)
print("이전:", before)
data = before.serialize()
print("직렬화한 값:", data)
print("이후:", deserialize(data))

# 이전: Vector1D(6)
# 직렬화한 값: {"class": "Vector1D", "args": [6]}
# 이후: Vector1D(6)
```

<br>

## 정리

이처럼 <span class="shl">**클래스를 확장(ex. 클래스 등록, 파라미터 검증 등)**하는 데에 **메타클래스 혹은 `__init_subclass__`**를 사용</span>할 수 있으며, 해당 동작을 잊어버릴 일이 없다고 보장할 수 있다.

> 표준적인 **메타클래스** 방식보다는 **`__init_subclass__`**가 가독성 측면에서 더 권장된다.
{: .prompt-tip}

본문의 예시와 같이 직렬화/역직렬화인 경우 잘 작동하며, 객체-관계 매핑(ORM), 확장성 플러그인 시스템, 콜백 훅에도 마찬가지로 잘 동작한다.

<br>

| 구분        | 방법                                                 |
| ----------- | ---------------------------------------------------- |
| <span class="blue">함수</span> 확장   | 데코레이터                                           |
| <span class="blue">클래스</span> 확장 | 메타클래스, `__init_subclass__` 특별 클래스 메서드 |
