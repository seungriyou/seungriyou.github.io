---
title: "[Python] 가비지 컬렉션(Garbage Collection): 객체를 삭제하려면?"
date: 2023-09-06 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, garbage collection]
math: true
---

## TL;DR 📌

1. `del`은 객체(objects)를 삭제하는 것이 아니라 **참조(references)를 삭제**한다.
2. 파이썬에서 object는 명시적으로 제거되지 않으며, object가 **unreachable** 해지면 자동으로 **garbage-collected** 된다.
3. CPython의 가비지 컬렉터(garbage collector)는 **reference counting** 기반으로 동작한다. 어떤 object의 **`refcount`가 0에 도달**하면 해당 object는 곧바로 제거된다.
4. `weakref.finalize` 함수를 이용하여 object가 제거될 때 호출될 callback function을 등록할 수 있다. 이는 weak references를 이용한다.

<br>

## `del`과 객체의 삭제 시점

파이썬에서 더 이상 사용하지 않을 object를 삭제하려는 목적으로 `del`을 사용해 본 경험이 종종 있을 것이다.

하지만 `del`은 객체(objects) 자체를 삭제하는 것이 아니라, 해당 객체에 대한 <span class="hlp">참조(references)를 삭제</span>한다.

> `del`은 function이 아니라 statement이다. 따라서 `del(x)`이 아니라 `del x`라고 작성한다. 하지만 `del(x)`이라고 작성해도 동작은 한다.
> 

<br>

그렇다면 실제로 object가 삭제되는 것은 언제일까?

이는 해당 object에 대한 <span class="hl">모든 reference가 삭제되어 그 object가 unreachable해졌을 때</span>이다!

<br>

## 가비지 컬렉션(Garbage Collection) (w/ Reference Counting)

> CPython에서 사용하는 primary algorithm for **garbage collection**이다.
> 

파이썬의 **가비지 컬렉터(garbage collector)**는 삭제된 변수가 어떤 object의 마지막 참조(last reference)이면, 즉 <span class="hl">**참조의 개수가 0에 도달**하면 해당 object를 메모리로부터 제거</span>한다.

<br>

이때, 어떤 object의 <span class="hlb">**참조의 개수가 감소하는 경우**</span>는 다음의 두 가지 경우가 있다.

1. `del` statement로 **reference를 삭제**하는 경우
2. 변수를 **rebind** 하는 경우
    
    > 이미 존재하던 변수에 새로운 값을 할당하는 것은 bind 된 object 자체를 바꾸는 것이 아니라, **다른 object로 bind 하는 것**이다(→ **rebinding**). rebinding 시, 그 variable이 last reference였다면 해당 object는 garbage collected 된다.
    > 

![](/assets/img/posts/Python/Fluent-Python/2023-09-06-01.png){: .w-50}

<br>

파이썬에서 각 object는 자신을 가리키고 있는 **참조의 개수(= `refcount`)**를 가지고 있으며, 이 **`refcount`가 0에 도달**하면 해당 object는 곧바로 제거된다. 그리고 이러한 동작을 **가비지 컬렉션(garbage collection)**이라 한다.

<br>

### 파이썬의 구현체별 가비지 컬렉션의 동작

- **CPython**:
    - **reference counting** 방법으로 수행된다. 이 방법은 구현이 간단하지만 reference cycle이 있을 때는 memory leak이 발생하기 쉽다.
    - `__del__` method(object에 defined 된 경우)를 호출하고, object에 할당된 memory를 free 시킨다.
- **CPython 2.0**:
    - **generational garbage collector**를 도입하여 **reference cycle**로 인해 **keep alive 되는 unreachable objects**를 dispose 할 수 있도록 하였다.
    - reference cycle이란, outstanding reference를 통해서도 unreachable하며, 모든 mutual references가 group 내에서만 존재하는 것을 의미한다.
- **그 외**: reference counting에 의존하지 않는 garbage collector를 사용하기도 한다.


<br>

> 파일을 close 할 때는 reference counting에 의존하지 않고 explicitly 하게 close 하는 것을 권장한다.
> 
> 
> → `with` statement를 사용하여 file이 open 중에 exception이 발생했더라도 closed 된다는 것을 보장한다.
> 
> ```python
> with open("test.txt", "wt", encoding="utf-8") as fp:
>     fp.write("1, 2, 3")
> ```
{: .prompt-tip}

<br>

## `weakref.finalize`

> `weakref.finalize`는 객체가 제거될 때 호출될 **callback function**을 등록하는 함수이다. 이 함수는 어떤 원리로 동작할까?
>

![](/assets/img/posts/Python/Fluent-Python/2023-09-06-02.png)

- `finalize` function으로 전달되는 `s1` reference는 해당 target object(`{1, 2, 3}`)를 monitor 하고 callback을 호출하기 위해서 전달된다.
- `finalize`는 reference에 해당하는 target object의 **weak reference**를 가지고 있다.
    
    > **weak references**
    > 
    > - target object의 **reference count를 증가시키지 않는다**. 즉, target object가 **garbage collected 되는 것을 방지하지 않는다**. (→ do not keep an object alive)
    > - **caching application**에서 유용하게 사용된다.
    {: .prompt-info}

- `weakref` 모듈은 `WeakValueDictionary`, `WeakKeyDictionary`, `WeakSet` 등의 collection도 제공한다.

<br>

## References

- “Fluent Python (2nd Edition)”, Ch06. Object References, Mutability, and Recycling
