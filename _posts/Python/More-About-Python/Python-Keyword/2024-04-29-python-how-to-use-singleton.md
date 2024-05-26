---
title: "[Python] Singleton을 사용하는 다섯 가지 방법"
date: 2024-04-29 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, keyword, singleton]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

## 1. Singleton Pattern

### 1-1. Singleton Pattern을 사용하는 이유

singleton pattern은 **어떤 클래스의 인스턴스가 전체 시스템에서 단 하나만 존재하도록** 하기 위해 사용한다.

예를 들어 어떤 서버 프로그램에서 configuration 정보가 `AppConfig` 클래스로 존재한다고 가정하자. 만약 프로그램의 많은 곳에서 해당 정보를 사용하고자 한다면, 각각의 사용처에서 `AppConfig` 클래스의 인스턴스를 생성하게 될 것이다. 이는 **메모리 자원의 낭비**로 이어지게 될 것이기 때문에, 전체 프로그램 런타임에서 단 한 개의 `AppConfig` 인스턴스만을 생성하기 위해서 singleton pattern을 도입할 수 있다.

<br>

### 1-2. 장점

1. 인스턴스가 단 하나만 존재하도록 보장하여, 불필요한 인스턴스 생성을 방지하고 **메모리 낭비**를 줄일 수 있다.
2. singleton 인스턴스에는 애플리케이션 어디에서든 접근할 수 있기 때문에, **데이터 공유**와 **전역 상태 관리** 측면에서 편리하다.
3. 애플리케이션의 특정 부분에서 유지되어야 하는 설정이나 상태를 **중앙에서 관리**할 수 있다.

<br>

### 1-3. 단점

1. 서로 독립적으로 수행되어야 하는 **단위 테스트** 시, 서로 다른 테스트 간에 상태가 공유되어 테스트가 어려워진다.
2. **의존성**을 숨기기 때문에 코드 가독성과 유지보수성이 떨어질 수 있다.
3. 잘못 구현된 singleton은 멀티 쓰레드 환경에서 **동기화 문제**로 인해 인스턴스가 여러 개 생성될 수 있다.
4. **결합도**가 증가하며, **전역 상태**를 가지기 때문에 예상치 못한 부작용이 발생할 수 있다.

<br>

### 1-4. Lazy Initialization

lazy initialization이란, singleton 클래스의 인스턴스가 처음(클래스 로딩 시점)부터 생성되는 것이 아닌, **실제로 필요한 시점에 처음으로 생성**되도록 하는 기법이다.

이를 사용하면 다음과 같은 이점이 있다.

1. 처음부터 인스턴스를 생성하지 않기 때문에 애플리케이션 시작 시점에서 불필요한 리소스를 할당하지 않아도 된다.
2. 애플리케이션 초기 로드 시간과 메모리 사용량을 줄일 수 있다.

따라서 lazy initialization은 singleton 인스턴스를 생성하는 비용이 큰 경우나, instantiate 시 필요로 하는 자원이 나중에 접근 가능한 경우에 유용하게 사용된다.

이때, thread-safe 하게 구현하기 위해서 동기화 메커니즘을 사용해야 하며, 성능 저하에 유의해야 한다. 또한, 코드가 복잡해져 코드 가독성과 유지보수성이 떨어질 수 있다.

<br>

## 2. Python에서 Singleton을 사용하는 다섯 가지 방법

### 2-1. 모듈

파이썬의 모듈은 기본적으로 singleton 모드로 동작한다. 왜냐하면, **모듈은 처음 import 될 때 `.pyc` 파일(바이트 코드)을 생성**하고, 그 후로 import 될 때는 다시 모듈 코드를 실행하는 것이 아니라 **이미 생성된 `.pyc` 파일을 로드**하기 때문이다.

따라서 singleton object를 사용하기 위해서는 어떤 모듈에 관련된 함수와 데이터를 정의하기만 하면 된다.

<br>

singleton class를 사용하고 싶다면 다음과 같이 모듈 내에 인스턴스를 생성하고,

```python
# mysingleton.py

class Singleton(object):
    def foo(self):
        pass
singleton = Singleton()
```

다른 파일에서 모듈로 import 하여 사용하면 된다.

```python
from mysingleton import singleton
```

<br>

> 이러한 방식은 FastAPI 오픈소스 프로젝트인 [**polar**](https://github.com/polarsource/polar)에서 사용되고 있다.
> 
> - [polar/advertisement/service.py](https://github.com/polarsource/polar/blob/main/server/polar/advertisement/service.py)
>     
>   ```python
>   class AdvertisementCampaignService:
>      async def get(
>           self, session: AsyncSession, id: uuid.UUID, allow_deleted: bool = False
>       ) -> AdvertisementCampaign | None:
>          ...
>   
>   advertisement_campaign_service = AdvertisementCampaignService()
>   ```
>     
> - [polar/advertisement/endpoints.py](https://github.com/polarsource/polar/blob/main/server/polar/advertisement/endpoints.py)
>     
>   ```python
>   from .service import advertisement_campaign_service
>   ```
{: .prompt-info}

<br>

### 2-2. 데코레이터

다음과 같은 데코레이터 `Singleton`을 살펴보자.

```python
def Singleton(cls):
    _instance = {}

    def _singleton(*args, **kargs):
        if cls not in _instance:
            _instance[cls] = cls(*args, **kargs)
        return _instance[cls]

    return _singleton
```

데코레이터 함수인 `Singleton`이 호출될 때마다 새로운 `_singleton` 함수(클로저)가 생성된다.

이때, **LEGB 규칙**으로 인해 클로저는 enclosing scope에 위치한 `_instance`를 사용할 수 있다. 즉, 각 `_singleton` 함수가 실행될 때마다 사용되는 `_instance`는 어디서나 동일한 객체이므로 공유된다.

<br>

```python
@Singleton
class A(object):
    a = 1

    def __init__(self, x=0):
        self.x = x

a1 = A(2)
a2 = A(3)

print(a1) # <__main__.A object at 0x100e7ef90>
print(a2) # <__main__.A object at 0x100e7ef90>
```

위와 같이 `A` 클래스를 작성하면 `A()`를 호출할 때마다 `_singleton` 함수가 호출되고, 각 호출마다 사용되는 `_instances`는 동일하기 때문에 해당 클래스 `A`는 처음 단 한 번만 instantiate 된다. 이렇게 instantiate 된 후로는 `_instances` 딕셔너리에서 꺼내와 사용하게 되므로 singleton으로 동작한다.

<br>

### 2-3. 클래스

언뜻 보기에 다음과 같이 class method `instance()`를 이용해서 `Singleton` 클래스를 작성하면, `Singleton.instance()`를 호출할 때 singleton 객체를 얻을 수 있을 것처럼 보일 수 있다.

```python
class Singleton(object):

    def __init__(self):
        pass

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            Singleton._instance = Singleton(*args, **kwargs)
        return Singleton._instance
```

<br>

하지만 **멀티 쓰레딩** 환경이라면? `time.sleep(1)`을 사용하여 I/O operation이 존재하는 상황을 연출한 코드의 실행 결과를 살펴보자.

```python
class Singleton(object):

    def __init__(self):
        time.sleep(1) # simulate I/O operation
    ...
    

import threading

def task(arg):
    obj = Singleton.instance()
    print(obj)

for i in range(10):
    t = threading.Thread(target=task,args=[i,])
    t.start()
```

예상과 다르게 다음과 같이 매 호출마다 새로운 `Singleton` 인스턴스가 생성됨을 확인할 수 있다.

```
<__main__.Singleton object at 0x10141d090>
<__main__.Singleton object at 0x10141e2d0>
<__main__.Singleton object at 0x10141eb90>
<__main__.Singleton object at 0x10141e5d0>
<__main__.Singleton object at 0x10141e8d0>
<__main__.Singleton object at 0x10141d3d0>
<__main__.Singleton object at 0x10141d6d0>
<__main__.Singleton object at 0x10141d9d0>
<__main__.Singleton object at 0x10141dcd0>
<__main__.Singleton object at 0x10141dfd0>
```

<br>

이를 해결하기 위해서는 **lock**을 사용할 수 있다. unlocked 부분은 concurrent 하게, locked 부분은 serial 하게 실행되도록 하는 것이다. 하지만 성능 측면에서는 좋지 않다.

```python
import time
import threading

class Singleton(object):
    _instance_lock = threading.Lock()

    def __init__(self):
        time.sleep(1)

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            with Singleton._instance_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = cls(*args, **kwargs)
        return Singleton._instance
```

```latex
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
<__main__.Singleton object at 0x104dbd150>
```

<br>

### 2-4. `__new__` 메서드

2-3의 코드에서, `instance()` 메서드 부분을 `__new__` 메서드로 변경한다. 이렇게 하면 별도의 메서드를 호출하여 instantiate 할 필요 없이, 일반적인 클래스를 instantiate 하는 것처럼 사용할 수 있다.

```python
import time
import threading

class Singleton(object):
    _instance_lock = threading.Lock()

    def __init__(self):
        time.sleep(1)

    def __new__(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            with Singleton._instance_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = object.__new__(cls)
        return Singleton._instance

def task(arg):
    obj = Singleton()   # 일반적인 클래스처럼 생성할 수 있다.
    print(obj)

for i in range(10):
    t = threading.Thread(target=task,args=[i,])
    t.start()
```

```
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
<__main__.Singleton object at 0x104d1cf50>
```

<br>

### 2-5. 메타클래스

> 메타클래스에 대해서는 [**[Python] 메타클래스(Metaclass)**](/posts/python-metaclass/) 포스팅을 참고하자.
{: .prompt-tip}

메타클래스의 `__call__` 메서드에 어떤 클래스가 생성될 때마다 필요한 동작을 명시할 수 있다. `__call__` 메서드에는 이전에 `__new__` 메서드에 작성했던 내용을 작성하면 된다.

```python
import threading

class SingletonType(type):
    _instance_lock = threading.Lock()
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, "_instance"):
            with SingletonType._instance_lock:
                if not hasattr(cls, "_instance"):
                    cls._instance = super(SingletonType, cls).__call__(*args, **kwargs)
        return cls._instance

class Foo(metaclass=SingletonType):
    def __init__(self, name):
        self.name = name

def task(arg):
    obj = Foo('name')
    print(obj)

for i in range(10):
    t = threading.Thread(target=task, args=[i, ])
    t.start()
```

```
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
<__main__.Foo object at 0x1010bd2d0>
```

<br>

## References

- <https://medium.com/@nathanbyers13/the-singleton-pattern-pros-cons-and-best-practices-9e4d256132de>
- <https://medium.com/geekculture/python-five-ways-to-write-singleton-b6afbed65680>
