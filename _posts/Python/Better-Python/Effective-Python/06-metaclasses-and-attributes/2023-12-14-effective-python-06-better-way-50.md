---
title: "[Better Way #49] __set_name__으로 클래스 애트리뷰트를 표시하라"
date: 2023-12-14 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, descriptor, property, metaclass]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

메타클래스를 이용하면 <span class="shl">클래스가 정의된 후, 클래스가 실제로 사용되기 이전 시점에 **프로퍼티를 변경하거나 표시**</span>할 수 있다.

애트리뷰트가 포함된 클래스 내부에서 애트리뷰트 사용을 더 자세히 관찰하고자 <span class="blue">**디스크립터**</span>를 사용할 때 이런 방식을 활용한다.

> 디스크립터에 관련된 내용은 **[[Better Way #46] 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라](/posts/effective-python-06-better-way-46/)** 포스팅을 참고한다.
{: .prompt-info}

이를 확인하기 위해 예시로 고객 데이터베이스의 로우(row)을 표현하는 새 클래스를 정의하는 경우를 살펴보자.

<br>

## 클래스 애트리뷰트 디스크립터: 구현하기

각 컬럼(column)에 해당하는 프로퍼티를 클래스에 정의하려 할 때, 애트리뷰트와 컬럼 이름을 연결하는 디스크립터 클래스 `Field`는 다음과 같다.

```python
class Field:
    def __init__(self, name):
        self.name = name
        self.internal_name = "_" + self.name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, "")

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

<br>

이렇게 컬럼 이름을 `Field` 디스크립터에 저장하고 나면, `setattr` 내장 함수를 사용해 **인스턴스별 상태를 직접 인스턴스 딕셔너리 `__dict__`에 저장**할 수 있어 나중에 `getattr`로 인스턴스의 상태를 읽을 수 있게 된다.

이러한 방식으로 <span class="shl">인스턴스 딕셔너리에 데이터를 저장하도록 하면 **`weakref` 내장 모듈을 사용하지 않고도 메모리 누수를 피할 수 있다**</span>.

> `weakref` 내장 모듈을 사용한 디스크립터는 [**이 곳**](/posts/effective-python-06-better-way-46/#correct-way-weakkeydictionary)을 참고한다.
{: .prompt-info}

<br>

데이터베이스 로우를 표현하는 클래스를 정의하려면 다음과 같이 애트리뷰트별로 컬럼 이름을 지정하면 된다.

```python
class Customer:
    # 클래스 애트리뷰트
    first_name = Field("first_name")
    last_name = Field("last_name")
    prefix = Field("prefix")
    suffix = Field("suffix")

cust = Customer()
print(f"이전: {cust.first_name!r} {cust.__dict__}")
cust.first_name = "뽀로로"
print(f"이후: {cust.first_name!r} {cust.__dict__}")

# 이전: '' {}
# 이후: '뽀로로' {'_first_name': '뽀로로'}
```

<br>

하지만 이 방식에는 클래스 안에서 필드 이름을 좌변에 이미 정의했는데(`field_name =`), <span class="shl">같은 문자열을 `Field` 디스크립터에게 중복해서 전달(`Field("field_name")`)한다</span>는 문제점이 있다.

파이썬이 `Customer` 클래스 정의를 처리하는 순서는 우변 → 좌변 순이므로, `Field` 인스턴스가 자신이 대입될 클래스의 애트리뷰트 이름을 미리 알 수 없기 때문이다.

<br>

## 클래스 애트리뷰트 디스크립터: 개선하기

### [1] 메타클래스

이러한 중복을 줄이기 위해서 **메타클래스**를 사용할 수 있다.

> 메타클래스를 사용하면 `class` 문에 직접 훅을 걸어 **`class` 본문이 끝나자마자 필요한 동작을 수행**할 수 있다.
{: .prompt-tip}

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        for key, value in class_dict.items():
            if isinstance(value, Field):
                value.name = key
                value.internal_name = "_" + key
        cls = type.__new__(meta, name, bases, class_dict)
        return cls

# 데이터베이스 로우를 표현하는 모든 클래스는 다음 클래스를 상속하여 메타클래스를 사용해야 함
class DatabaseRow(metaclass=Meta):
    pass
```

<br>

`Field` 디스크립터의 생성자가 컬럼 이름을 받는 대신 `Meta.__new__` 메서드가 애트리뷰트를 설정해주므로, 다음과 같이 코드를 변경한다.

```python
class Field:
    def __init__(self):
        # 다음의 두 정보를 메타클래스가 채워준다!
        self.name = None
        self.internal_name = None

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, "")

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

<br>

이를 이용하면 데이터베이스 로우에 대응하는 클래스를 다음과 같이 정의할 수 있으며, 여기에는 **중복이 없음**을 확인할 수 있다.

```python
class BetterCustomer(DatabaseRow):
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()

cust = BetterCustomer()
print(f"이전: {cust.first_name!r} {cust.__dict__}")
cust.first_name = "크롱"
print(f"이후: {cust.first_name!r} {cust.__dict__}")

# 이전: '' {}
# 이후: '크롱' {'_first_name': '크롱'}
```

<br>

하지만 여전히 `DatabaseRow`를 상속하는 것을 잊어버리거나 클래스 계층 구조로 인한 제약 때문에 `DatabaseRow`를 상속할 수 없는 경우에 문제가 발생할 수 있다. 즉, <span class="shl">`DatabaseRow`를 상속하지 않으면 코드가 깨진다</span>.

```python
class BrokenCustomer:
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()

cust = BrokenCustomer()
cust.first_name = "에디"
```

```
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Cell In[12], line 9
      5     suffix = Field()
      8 cust = BrokenCustomer()
----> 9 cust.first_name = '에디'

Cell In[9], line 13
     12 def __set__(self, instance, value):
---> 13     setattr(instance, self.internal_name, value)

TypeError: attribute name must be string, not 'NoneType'
```

<br>

### [2] `__set_name__` 특별 메서드

위 문제를 해결하기 위해 메타클래스 대신 디스크립터에 **`__set_name__` 특별 메서드**(파이썬 3.6+ 지원)를 사용할 수 있다.

> **클래스가 정의될 때마다** 파이썬은 해당 클래스 안에 들어 있는 디스크립터 인스턴스의 `__set_name__`을 호출한다. `__set_name__`은 <span class="shl">디스크립터 인스턴스를 소유하고 있는 **클래스**와 디스크립터 인스턴스가 대입될 **애트리뷰트 이름**을 인자로</span> 받기 때문에, `Meta.__new__`에서 하던 일을 디스크립터의 `__set_name__`에서 처리할 수 있다.
{: .prompt-info}

```python
class Field:
    def __init__(self):
        self.name = None
        self.internal_name = None

    def __set_name__(self, owner, name):
        # 클래스가 생성될 때 모든 디스크립터에 대해 이 메서드가 호출됨!
        self.name = name
        self.internal_name = "_" + name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, "")

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

```python
class FixedCustomer:
    first_name = Field()
    last_name = Field()
    prefix = Field()
    suffix = Field()

cust = FixedCustomer()
print(f"이전: {cust.first_name!r} {cust.__dict__}")
cust.first_name = "루피"
print(f"이후: {cust.first_name!r} {cust.__dict__}")

# 이전: '' {}
# 이후: '루피' {'_first_name': '루피'}
```
