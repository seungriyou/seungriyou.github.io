---
title: "[Python] 메타클래스(Metaclass)"
date: 2024-04-23 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, keyword, metaclass]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

> 본문은 참고한 블로그 및 자료를 제가 이해하기 쉽게 다시 정리한 글임을 밝힙니다.

<br>

## 1. 메타클래스(Metaclass)

파이썬에서는 **모든 것이 객체**이기 때문에 **클래스도 그 자체로 객체**이다. 그렇다면 클래스라는 객체를 생성하는 클래스는 무엇일까? 이처럼 <span class="ulr">**“클래스의 클래스”**</span>를 **메타클래스(metaclass)**라고 한다.

파이썬에서 기본적으로 사용하는 **메타클래스는 `type`**이다.

> 이 `type`은 우리가 어떤 자료형의 타입을 알아내기 위해 사용하는 `type()`이 맞다!
> 

```python
class MyClass:
    pass
    
type(MyClass)
# <class 'type'>
```

<br>

### 1-1. `type` 활용하기

> 이러한 메타클래스 `type`을 활용하는 방법 두 가지를 살펴보자.

1. <span class="ulr">**클래스 만들기**</span>
    - `()` = 상속 클래스 정의를 담는 tuple
    - `{}` = 클래스에서 정의할 속성과 메서드를 담는 dict
 
    ```python
   MyClass = type('MyClass', (), {})
   MyClass
   # <class '__main__.MyClass'>
    ```
    
2. <span class="ulr">**커스텀 메타클래스 만들기**</span>
    
    `type`을 상속받아 커스텀 메타클래스를 정의하고, 이를 적용할 클래스의 `metaclass` 파라미터로 지정하면 된다. 이러한 **커스텀 메타클래스를 통해서 원하는 방법으로 클래스가 동작하도록 제어**할 수 있다.
    
    예를 들어 **클래스 `MyClass`가 생성될 때 속성 `a`가 반드시 정수가 되어야 하는 상황**을 가정해보자.
    
    이러한 검증 과정을 `MyMetaClass`의 `__new__` 메서드에 명시하면 된다.
    
    ```python
   class MyMetaclass(type):
       def __new__(cls, clsname, bases, dct):
           assert type(dct['a']) is int, 'Attribute `a` is not an integer.'
           return type.__new__(cls, clsname, bases, dct)
   
   class MyClass(metaclass=MyMetaclass):
       a = 3.14
    ```
    
    실행 결과, 다음과 같이 assert 문이 실행되는 것을 확인할 수 있다.
    
    ```
   Traceback (most recent call last):
     File "/Users/.../test.py", line 7, in <module>
       class MyClass(metaclass=MyMetaclass):
     File "/Users/.../test.py",, line 4, in __new__
       assert type(dct['a']) is int, 'Attribute `a` is not an integer.'
              ^^^^^^^^^^^^^^^^^^^^^
   AssertionError: Attribute `a` is not an integer.
    ```
    

<br>

## 2. 메서드 관점에서 살펴본 인스턴스 생성 내부 동작

### 2-1. 클래스

| 메서드 이름     | 동작                                        |
| --------------- | ------------------------------------------- |
| `__new__` 메서드  | 클래스의 인스턴스를 생성한다. (메모리 할당) |
| `__init__` 메서드 | 생성된 인스턴스를 초기화한다.               |
| `__call__` 메서드 | 인스턴스를 실행한다.                        |

> `__init__` 메서드는 **생성자가 아니라는 것**에 주의해야 한다! 실제 생성은 `__new__` 메서드에서 수행하며, `__init__` 메서드에서는 생성된 인스턴스를 초기화할 뿐이다.
{: .prompt-danger}

<br>

### 2-2. 메타클래스

메타클래스 또한 `__new__`, `__init__`, `__call__` 메서드를 가지고 있다. 예제를 통해 동작 양상을 살펴보자.

```python
class MyMetaClass(type):
    def __new__(cls, *args, **kwargs):
        print('metaclass __new__')
        return super().__new__(cls, *args, **kwargs)

    def __init__(cls, *args, **kwargs):
        print('metaclass __init__')
        super().__init__(*args, **kwargs)

    def __call__(cls, *args, **kwargs):
        print('metaclass __call__')
        return super().__call__(*args, **kwargs)

class MyClass(metaclass=MyMetaClass):
    def __new__(cls, *args, **kwargs):
        print('child __new__')
        obj = super().__new__(cls)
        return obj

    def __init__(self):
        print('child __init__')

    def __call__(self):
        print('child __call__')

print('=============================================')
print('---- Instantiate MyClass:')
obj = MyClass()
print('---- Call MyClass Instance:')
obj()

print('=============================================')
print(f'{type(MyClass)=}')
```

```
metaclass __new__
metaclass __init__
=============================================
---- Instantiate MyClass:
metaclass __call__
child __new__
child __init__
---- Call MyClass Instance:
child __call__
=============================================
type(MyClass)=<class '__main__.MyMetaClass'>
```

<br>

위의 코드에서 알 수 있는 점은 다음과 같다.

1. `MyClass`의 타입은 `MyMetaClass`이다.
2. 메타클래스 `MyMetaClass`를 상속한 **클래스 `MyClass`의 정의가 로드되는 순간**, 이미 `MyMetaClass`는 생성 및 초기화 된다. 따라서 **`MyMetaClass`의 `__new__`, `__init__` 메서드**가 차례로 호출된다.
    
    `MyClass`가 로드되는 순간에 `MyClass`라는 객체가 생성된 것이고, 객체가 생성되었다는 것은 클래스의 생성자가 호출되었다는 의미이다. 따라서 `MyClass`를 instantiate 하기 전에 `MyMetaClass`는 instantiate 된다.
    
3. 실제로 **클래스 `MyClass`를 instantiate 할 때**에는 `MyMetaClass` 인스턴스가 실행된다. 즉, 다음과 같이 실행되는 것이다.
    
    ```python
   MyClass() == (MyMetaClass())()
    ```
    
    따라서 **`MyMetaClass`의 `__call__` 메서드**가 호출된다. 그리고 `MyMetaClass`의 `__call__` 메서드에서는 인스턴스의 생성자 `__new__` 메서드를 호출하기 때문에, `MyClass`의 인스턴스가 생성되는 것이다.
    

<br>

## 3. 활용 방안

앞서, **메타클래스를 통해서 원하는 방법으로 클래스가 동작하도록 제어**할 수 있다고 했었다. 이때, 지금까지 살펴봤던 내용 중 떠올려야 할 **메타클래스의 핵심 특징**은 다음과 같다:

> 어떤 메타클래스를 상속받은 클래스를 **instantiate** 할 때마다, **메타클래스의 `__call__` 메서드가 호출**된다.
{: .prompt-info}

따라서 메타클래스의 `__call__` 메서드에 <span class="shl">**어떤 클래스가 생성될 때마다 필요한 동작**을 명시</span>할 수 있는 것이다.

가장 대표적인 예시로 **singleton 기능**을 메타클래스를 통해 제공할 수 있다.

```python
class SingletonType(type):
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, "_instance"):
            cls._instance = super(SingletonType, cls).__call__(*args, **kwargs)
        return cls._instance

class Foo(metaclass=SingletonType):
    def __init__(self, name):
        self.name = name
```

```python
obj1 = Foo('name')
obj2 = Foo('name1')

print(obj1, obj2)
# <__main__.Foo object at 0x101646e90> <__main__.Foo object at 0x101646e90>
print(obj1 is obj2)
# True
```

<br>

## References

- <https://stackoverflow.com/questions/44178162/call-method-of-type-class>
- <https://alphahackerhan.tistory.com/34>
- <https://dodokim.medium.com/python-metaclass-7f224d92a0a2>
- <https://weeklyit.code.blog/2019/12/24/2019-12월-3주-python의-__init__과-__new__/>
- <https://wikidocs.net/21056>
