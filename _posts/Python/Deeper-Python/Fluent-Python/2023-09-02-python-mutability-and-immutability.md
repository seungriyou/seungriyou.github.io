---
title: "[Python] 가변성(Mutability)과 불변성(Immutability): 튜플은 항상 불변일까?"
date: 2023-09-02 19:50:00 +0900
categories: [Python, Deeper Python]
tags: [python, fluent python, mutability, immutability, tuple]
math: true
---

> 본문은 파이썬의 원리에 대해 **“Fluent Python (2nd Edition)”**을 참고하여 공부한 내용을 정리한 글입니다. (Ch06. Object References, Mutability, and Recycling)
> 

<br>

> 흔히 파이썬에서 “`tuple`은 불변이다” 라고 알려져 있다. 그런데, 과연 파이썬의 `tuple`은 어떠한 상황에서든 불변일까? 그리고 `tuple` 내에 변경 가능하다고 알려진 `list`가 들어있다면, 그 `list`는 변경 불가능할까?
{: .prompt-warning}

<br>

## TL;DR 📌

1. 변경이 불가(immutable)하다고 알려진 **`tuple`의 불변성**은 가지고 있는 **“참조” 자체에 대한 것**이며, “참조된 객체”에까지 확장되지 않는다.
2. 파이썬에서 함수의 parameter를 전달하는 매커니즘은 **call by sharing**이기 때문에 mutable types를 optional parameter의 default value로 사용하지 말아야 하며, 대신 `None`을 사용해야 한다.

<br>

## `tuple`의 상대적 불변성(Immutability)

파이썬의 컨테이너(ex. `list`, `dict`, `set`, `tuple` 등)는 객체들에 대한 참조를 담으며, 이 중에서 `tuple`은 변경이 불가하다고(immutable) 알려져 있다.

하지만 `tuple`이 참조하고 있는 객체가 변경 가능(mutable)하다면, 해당 객체는 변경될 수 있다.

즉, **`tuple`의 불변성**은 가지고 있는 <span class="hl">**“참조” 자체에 대한 것**</span>이며, **“참조된 객체”**에까지 확장되지 않는다.

<br>

> immutable collection에서 변하지 않는 것(= immutable)은 **그 안에 담긴 객체들의 identity**이다.
>
> 그 안에 담긴 객체들이 mutable 하다면, 그 객체의 값은 바뀔 수 있다.
{: .prompt-tip}

<br>

## "<span class="blue">Call by Sharing</span>": 파이썬의 Parameter Passing 매커니즘

### Call by Sharing

파이썬에서 **함수의 parameter를 전달**하는 매커니즘은 <span class="hl">**call by sharing**</span>이다. 이를 정리하면 다음과 같다.

- 함수의 각 parameter는 argument에 들어있는 각 **“참조의 copy”**를 받는다.
- 함수 내 **parameter**는 **실제 argument의 alias**가 된다.

> parameter와 argument는 다음과 같은 차이점이 있다.
> 
> | 용어 | 번역 | 의미 |
> | --- | --- | --- |
> | **parameter** | 매개변수 | 함수 및 메서드의 입력 변수(variable) 이름 |
> | **argument** | 인자 | 함수 및 메서드의 입력 값(value) |

<br>

따라서 **mutable object**가 parameter로 들어오게 되면, (1) 함수 내에서 그 **object 자체**를 바꿀 수는 있으나 (2) object의 **identity**를 바꾸지는 못한다.

| Type          | Value of argument variable |
| ------------- | -------------------------- |
| number, tuple | unchanged (**immutable**)      |
| list          | changed (**mutable**)          |

![](/assets/img/posts/Python/Fluent-Python/2023-09-02-01.png){: .w-50}

<br>

### [권장 사항] Mutable Type을 Parameter의 Default Value로 사용하지 말 것!

위와 같은 이유로 <span class="hl">**mutable types**를 **optional parameter의 default value로 사용하지 말아야**</span> 하며, 대신 **`None`을 사용**해야 한다.

> 관련 예시: [[Better Way #24] None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라](/posts/effective-python-03-better-way-24/)

- default value는 **function이 정의된 시점(= module이 로드된 시점)에 evaluated** 되고, 이 default value가 function object의 attribute가 된다.
- (1) default value가 mutable object이고 (2) 그것을 function 내부에서 바꾼다면, 그 change는 해당 function의 **모든 future call에 영향**을 미칠 것이다.
- mutable value를 받을 수 있는 parameter의 default value를 설정할 때는 주로 `None`을 사용하여 다음과 같이 처리한다.
    
  ```python
  def __init__(self, passengers=None):
      if passengers is None:
          self.passengers = [] # -- new empty list
      else:
          self.passengers = list(passengers) # -- copy!
  ```
    
  > `else` 문 내부의 assignment에서, **`passengers`로 aliasing** 대신 **`list(passengers)`로 copy** 하는 이유
  > 
  > 1. internal handling이 original argument에 영향을 미치지 못한다.
  >     
  >     > **principle of least astonishment** - aliasing을 하게 되면 `.remove()`나 `.append()` 등으로 조작했을 때 actual argument의 original list가 변하게 된다.
  >     > 
  > 2. `passengers` parameter가 `list`가 아닌 `tuple`, `set` 등의 다른 iterable로 들어와도 된다.
  >     
  >     > `list` constructor는 아무 iterable이나 받을 수 있으므로, `.remove()`, `.append()` 등의 필요한 연산을 지원한다는 것을 보장할 수 있다.
  >     >
  {: .prompt-info}
    

<br>

## 복합 할당 연산자(Augmented Assignment)

일반적인 단일 할당 연산자는 복사(copy)를 만들지 않지만, **복합 할당 연산자(ex. `+=`, `*=`)**는 좌변의 변수의 종류에 따라 다르게 동작한다.

| Lefthand Variable | Behavior                   |
| ----------------- | -------------------------- |
| **immutable** object  | create new object          |
| **mutable** object    | modify the object in place |

이때, 좌변에 위치한 immutable object를 가리키는 변수에 새로운 값을 할당하는 것은 **다른 object로 새롭게 binding 하는 것**이라 할 수 있다. 이를 <span class="hl">**rebinding**</span>이라 한다.

<br>

## (CPython) Immutables 관련 트릭들

1. immutable type인 `tuple`, `str`, `bytes`, `frozenset`에 **shallow copy를 만드는 방법**(ex. `[:]`, `.copy()`, constructor를 사용(ex. `tuple(t)`))을 적용하는 경우, copy를 만드는 것이 아니라 **same object에 대한 reference를 return** 한다.
    
    > **`frozenset`**
    >
    > - mutable 한 set와 다르게 immutable 한 객체이다.
    > - hashable elements만 담을 수 있다. (value of hashable object는 변할 수 없기 때문)
    {: .prompt-info}
    
2. **string literal**이나 자주 이용되는 **small integers**(ex. `0`, `1`, `-1`)의 경우, **interning**이라는 optimization technique을 통해 관리한다. 
    
    > 하지만 그 기준이나 구현 방법이 알려져 있지 않기 때문에 interning에 의존해서는 안되며, equality를 비교할 때는 항상 `==`을 이용해야 한다.
    > 
    
    ![](/assets/img/posts/Python/Fluent-Python/2023-09-02-02.png){: .w-50}
    

<br>

### 정리

- 파이썬에서 함수는 **arguments의 copy**를 받고, **arguments는 항상 references**이다.
- referenced objects가 mutable 하다면 **value**는 변경될 수 있으나, **identity**는 변경될 수 없다.
- 함수 내에서 **rebinding**을 수행하는 것은 함수 외부에 **아무 영향을 미치지 못한다**.

<br>

> object identity는 objects가 mutable 할 때만 중요하다.
>
> mutable objects는 thread를 이용한 프로그래밍이 어려운 주요 이유 중 하나이다. 적절한 **synchronization** 없이 threads가 object를 변경하면 corrupted data가 발생할 수 있다.
{: .prompt-danger}
