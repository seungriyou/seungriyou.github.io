---
title: "[Python] 추상 기반 클래스(Abstract Base Class), collections.abc 모듈"
date: 2023-10-05 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, abc, abstract base class, collections, isinstance, issubclass]
math: true
---

## 1. `collections.abc` 모듈

### 1-1. 추상 기반 클래스(Abstract Base Class, ABC)

파이썬의 `collections.abc` 내장 모듈 안에는 **컨테이너(container) 타입에 정의해야 하는 전형적인 메서드**를 제공하는 **추상 기반 클래스(abstract base class, ABC)** 정의가 들어있다.

이러한 ABC는 **인터페이스를 정의하는 방법을 제공**함으로써 <span class="shl">**덕 타이핑을 보완**</span>한다. 

> 실제로는 상속 관계가 아니지만 `isinstance()`나 `issubclass()`로는 `True`로 인식되는 **virtual subclasses** 개념을 도입한다.
> 

<br>

### 1-2. `collections.abc`를 이용하여 커스텀 컨테이너 타입 정의하기

커스텀 컨테이너 타입을 정의할 때는 `collections.abc` 모듈에서 제공하는 **ABC를 상속**받아 **정의된 abstract method를 구현**하는 것을 권장한다. 

> 해당 custom container 타입이 정상적으로 작동하기 위해 필요한 interface 및 method를 **제대로 구현하도록 보장**할 수 있다.
{: .prompt-tip}

하지만 **간단한 경우**에는 **built-in container 타입**(ex. `list`, `tuple`, `set`, `dict`)을 상속받아 구현해도 된다. 

> 관련 포스팅: [**[Better Way #43] 커스텀 컨테이너 타입은 collections.abc를 상속하라**](/posts/effective-python-05-better-way-43/)
{: .prompt-info}

<br>

## 2. `isinstance()`와 `issubclass()`가 동작하는 세 가지 방법

> ref: <https://docs.python.org/3/library/collections.abc.html>

1. <span class="shl">**newly written class**의 경우</span>, ABC를 **직접 상속(direct inheritance)**할 수 있으므로 직접 상속된 ABC와 비교한다.

   - 상속받은 interface에 정의된 abstract methods를 모두 구현해야 한다.
   - 나머지 mixin methods는 inheritance로부터 받아오거나 override 될 수 있다.
   - 그 외의 methods는 필요에 따라 추가될 수 있다.

   ```python
   class C(Sequence):                      # Direct inheritance
       def __init__(self): ...             # Extra method not required by the ABC
       def __getitem__(self, index):  ...  # Required abstract method
       def __len__(self):  ...             # Required abstract method
       def count(self, value): ...         # Optionally override a mixin method
   ```

   ```python
   issubclass(C, Sequence)
   True
   isinstance(C(), Sequence)
   True
   ```
    <br>

2. <span class="shl">**existing classes**나 **built-in classes**의 경우</span>, ABC의 **virtual subclasses**로 등록될 수 있으므로 **등록된 ABC와 비교**한다. 

   - **모든 abstract methods**와 **모든 mixin methods**를 포함한 전체 API를 구현해야 한다. 단, API의 나머지 부분에 의해 **자동으로 추론될 수 있는 methods는 예외**이다.
     
     > 아래 예제에서, class `D`는 `__getitem__`, `__len__` method를 구현하므로, 본래 `Sequence`라면 구현해야 할 `__contains__`, `__iter__`, `__reversed__` method를 구현하지 않아도 된다. (automatically inferred)
     
   - `isinstance()`와 `issubclass()`는 **full interface가 구현되었는지** 여부에 의존한다.

   ```python
   class D:                                 # No inheritance
       def __init__(self): ...              # Extra method not required by the ABC
       def __getitem__(self, index):  ...   # Abstract method
       def __len__(self):  ...              # Abstract method
       def count(self, value): ...          # Mixin method
       def index(self, value): ...          # Mixin method

   Sequence.register(D)                     # Register instead of inherit
   ```

   ```python
   issubclass(D, Sequence)
   True
   isinstance(D(), Sequence)
   True
   ```
    <br>

3. <span class="shl">**일부 simple interfaces(ex. `Iterable`)**의 경우</span>, **required methods가 정의**되어 있고, 그것이 `None`으로 설정되지 않았다면 directly recognizable 하다. 

    > interface의 method 간 관계가 method name 만으로 추론될 수 있는 것은 아니기 때문에, **complex interface의 경우에는 이 방법이 지원되지 않는다**.
    > 
    > → ex) class 내에 method `__getitem__`, `__len__`, `__iter__`가 존재한다고 해서 `Sequence`와 `Mapping`을 구분할 수 있지는 않다.
    > 

   ```python
   class E:
       def __iter__(self): ...
       def __next__(next): ...
   ```

   ```python
   issubclass(E, Iterable)
   True
   isinstance(E(), Iterable)
   True
   ```
    

<br>

## 3. 기본 Collection 타입

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-01.png){: .w-75}

- 위 UML의 class들은 **ABC(abstract base classes)**이다.
- <span class="shlp">**ABC를 구성하는 method의 종류**</span>
    
    | method 종류 | 설명 |
    | --- | --- |
    | **abstract method** | concrete subclasses(ex. list, dict)를 통해 구현되어야 한다. (italic으로 작성됨) |
    | **concrete method**<br>(mixin methods) |  concrete implementation을 가지고 있으므로, subclasses가 상속받아 사용할 수 있다.  |

- **각 top ABC**들은 **single special method**를 가지고 있으며, **`Collection` ABC는 다음의** <span class="shl">**세 가지 essential interfaces**</span>를 통합한다. 따라서 **모든 collection은 다음을 구현**해야 한다.
    
    
    | essential interface | 역할                                    |
    | ------------------- | --------------------------------------- |
    | `Iterable` | `for문`, unpacking 및 iteration 관련 지원 |
    | `Sized` | `len` built-in function 지원              |
    | `Container`| `in` operator 지원                        |
    
    > **concrete class**는 실제로 interface를 **상속받지 않아도** 된다. 내부의 **special method를 구현**하면 interface를 만족한다고 간주된다.
    {: .prompt-info}
    
- **`Collection`의** <span class="shl">**세 가지 주요 specialization**</span>은 다음과 같다.
    
    
    | specialization | 설명                                              |
    | -------------- | ------------------------------------------------- |
    | `Sequence`       | `list`, `str과` 같은 built-in의 interface를 formalize |
    | `Mapping`        | `dict`, `collections.defaultdict` 등으로 구현         |
    | `Set`            | built-in `set`과 `fronzeset`의 interface              |

<br>

## 4. `collections.abc` 모듈의 UML 클래스 다이어그램

> **built-in classes**는 virtual subclass로 정의되므로, subclass 관계는 **`collections.abc`**에서 제공하는 classes에 대해서만 알아보자.
>
> - **document**
>    
>   - [collections.abc — Abstract Base Classes for Containers](https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes)
>   - [collections — Container datatypes](https://docs.python.org/3/library/collections.html)
>    
> - **source code**
>     
>   - <https://github.com/python/cpython/blob/main/Lib/_collections_abc.py>
>   - <https://github.com/python/cpython/blob/main/Lib/collections/__init__.py>
>   - <https://github.com/python/cpython/blob/main/Lib/_collections_abc.py>
{: .prompt-tip}

<br>

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-02.png){: .w-75}
_ref: <https://github.com/python/cpython/issues/76652> (consistency problem이 있다고 하니 참고용으로만 보기)_

<br>

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-03.png){: .w-75}

<br>

### [1] `Sequence`

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-04.png){: .w-75}

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-05.png){: .w-75}

| ABC 이름 | 상속 받는 ABC | 소스 코드 |
| --- | --- | --- |
| `UserString` | `Sequence`        | [LINK](https://github.com/python/cpython/blob/main/Lib/collections/__init__.py#L1347) |
| `UserList`   | `MutableSequence` | [LINK](https://github.com/python/cpython/blob/main/Lib/collections/__init__.py#L1214) |

<br>

### [2] `Mapping`

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-06.png){: .w-75}

| ABC / class 이름 | 상속 받는 ABC / built-in class | 소스 코드  |
| ---- | ---- | ---- |
| `UserDict` | `MutableMapping` | [LINK](https://github.com/python/cpython/blob/main/Lib/collections/__init__.py#L1117) |
| `ChainMap` | `MutableMapping` | [LINK](https://github.com/python/cpython/blob/main/Lib/collections/__init__.py#L985) |
| `Counter` | `dict` | [LINK](https://github.com/python/cpython/blob/main/Lib/collections/__init__.py#L545) |
| `OrderedDict` | `dict` | [LINK](https://github.com/python/cpython/blob/main/Lib/collections/__init__.py#L83) |
| `defaultdict` | `dict`  | [LINK](https://github.com/python/cpython/blob/e57ecf6bbc59f999d27b125ea51b042c24a07bd9/Lib/test/test_descrtut.py#L16) |

<br>

> **custom mapping**을 구현하고 싶을 때는 위에 나타난 ABC를 subclassing 하는 것보다 **`collections.UserDict`를 확장**하거나 **`dict`를 wrap** 하는 것이 편리하다.
>
> > `collections.UserDict`나 standard library의 모든 concrete mapping classes는 **모두 basic `dict`를 encapsulate** 하고 있으므로 **hash table 기반**으로 생성된다. 따라서, **key가 hashable** 해야 한다는 limitation을 공유하게 된다.
{: .prompt-tip}

<br>

### [3] `Set`

![](/assets/img/posts/Python/Fluent-Python/2023-10-05-07.png){: .w-75}
<br>

> mutable 할 필요가 없으면 `MutableXXX`가 아닌 `XXX`를 subclassing 하는 것이 정확하다.
{: .prompt-warning}

<br>

## References

- <https://dongjunlee.github.io/code%20reading/CodeReading-3_ABC/#collectionsabc>
- <https://github.com/python/cpython/issues/76652>
- <https://sangmoonoh.medium.com/just-class-diagram-for-python-3-collections-abstract-base-classes-e1eafde6ad25>

<br>

## Additional Keyword

### `__subclasscheck__`, `__subclasshook__`

- <https://www.geeksforgeeks.org/__subclasscheck__-and-__subclasshook__-in-python/>
- <https://docs.python.org/3/library/abc.html#abc.ABCMeta.__subclasshook__>
- <https://github.com/python/cpython/blob/e57ecf6bbc59f999d27b125ea51b042c24a07bd9/Lib/_py_abc.py#L108>
- <https://github.com/python/cpython/blob/e57ecf6bbc59f999d27b125ea51b042c24a07bd9/Lib/abc.py#L121>
