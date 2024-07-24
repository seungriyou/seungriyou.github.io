---
title: "[Python] Data Class Builder: collections.namedtuple, typing.NamedTuple, @dataclass"
date: 2023-09-20 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, namedtuple, dataclass]
math: true
---

## Data Class Builder란?

데이터(`lat`, `lon`)를 저장하는 역할만을 수행하는 단순한 클래스 `Coordinate`를 생각해보자.

```python
class Coordinate:
    def __init__(self, lat, lon):
        self.lat = lat
        self.lon = lon

moscow = Coordinate(55.756, 37.617)
```

이렇게 단순히 생성한 `Coordinate` 클래스는 다음과 같은 단점을 가진다.

1. `object`로부터 상속받은 `__repr__`는 object의 주소를 반환하기 때문에, 다음과 같이 **객체 자체를 출력해서 확인할 때** 유용하지 않다.
    
    ```python
    moscow
    # <__main__.Coordinate at 0x10662f0d0>
    ```
    
2. `object`로부터 상속받은 `__eq__`는 `is`와 동일하게 object ID를 비교하므로 `==` 연산자로 클래스 내 필드를 비교할 수 없으며, 두 `Coordinate`를 비교하려면 **각 attribute에 대해 모두 comparison**을 진행해야 한다.
    
    ```python
    moscow == Coordinate(lat=55.756, lon=37.617)
    # False
    ```
    

<br>

이러한 단점들로 인해, <span class="hl">**간단히 데이터를 담기 위한 용도의 클래스를 생성할 때 유용**</span>하게 사용할 수 있는 **data class builder**가 등장하였다.

<br>

위의 `Coordinate` 클래스를 **세 가지 data class builder**인 `collections.namedtuple`, `typing.NamedTuple`, `@dataclass`를 이용한 방법으로 바꾸어보고, 각각의 특징에 대해 알아보도록 하자!

<br>

## [1] `collections.namedtuple`

```python
from collections import namedtuple
Coordinate = namedtuple('Coordinate', 'lat lon')

moscow = Coordinate(55.756, 37.617)
```

1. `__repr__`가 저장된 데이터의 값을 보여주므로 유용하다.
    
    ```python
    moscow
    # Coordinate(lat=55.756, lon=37.617)
    ```
    
2. `__eq__` 또한 저장된 각 attribute의 값을 비교하므로 `==`가 meaningful 하다.
    
    ```python
    moscow == Coordinate(lat=55.756, lon=37.617)
    # True
    ```
    
3. `tuple`의 subclass인 class를 생성한다.
    
    ```python
    issubclass(Coordinate, tuple)
    ```
    

> 관련 포스팅: [[Better Way #37] 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라](/posts/effective-python-05-better-way-37/)
{: .prompt-info}

<br>

## [2] `typing.NamedTuple`

```python
from typing import NamedTuple

# 다음의 두 가지 형태가 모두 가능하다.
Coordinate = NamedTuple('Coordinate',
                        [('lat', float), ('lon', float)])
Coordinate = NamedTuple('Coordinate', lat=float, lon=float)
```

1. 기본적인 특징은 **`collections.namedtuple`과 동일**하다.
2. **class statement**로도 사용할 수 있다.
    
    ```python
    class Coordinate(NamedTuple):
        lat: float
        lon: float
        
        def __str__(self):
            ns = 'N' if self.lat >= 0 else 'S'
            we = 'E' if self.lon >= 0 else 'W'
            return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
    
    moscow = Coordinate(55.756, 37.617)
    print(moscow)
    # 55.8°N, 37.6°E
    ```
    
3. `tuple`의 subclass이지만 `NamedTuple`의 subclass가 아니다.
    
    ```python
    issubclass(Coordinate, NamedTuple)
    # -- TypeError: issubclass() arg 2 must be a class, a tuple of classes, or a union
    ```
    
4. `get_type_hints()` 함수로 타입 힌트를 확인할 수 있다.
    
    ```python
    from typing import get_type_hints
    
    # type hint를 확인할 수 있다.
    get_type_hints(Coordinate)
    ```

<br>

## [3] `@dataclass`

```python
# class statement example

from dataclasses import dataclass

@dataclass(frozen=True)
class Coordinate:
    lat: float
    lon: float
    
    def __str__(self):
        ns = 'N' if self.lat >= 0 else 'S'
        we = 'E' if self.lon >= 0 else 'W'
        return f'{abs(self.lat):.1f}°{ns}, {abs(self.lon):.1f}°{we}'
```

1. inheritance나 metaclass와 관련이 없는 class decorator 이다.
2. 상속을 받지 않으므로 `object`의 subclass이다.
3. `frozen=True`는 해당 instance가 변경될 수 없다는 것을 나타낸다.

<br>

## 세 가지 Data Class Builder 비교

![](/assets/img/posts/Python/Fluent-Python/2023-09-20-01.png)

<br>

## References

- “Fluent Python (2nd Edition)”,  Ch05. Data Class Builders
