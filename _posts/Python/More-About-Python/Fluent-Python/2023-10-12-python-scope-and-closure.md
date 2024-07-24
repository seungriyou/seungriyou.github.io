---
title: "[Python] 스코프(Scope)와 클로저(Closure)"
date: 2023-10-12 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, scope, closure, nonlocal, global]
math: true
---

## TL;DR 📌

1. 파이썬에서는 어떠한 변수 혹은 함수에 접근 가능한지를 판단하기 위해 스코프 체인을 따라 해당 변수 및 함수의 정의를 찾아나간다. 이때, 확인하는 순서는 `Local` → `Enclosed` → `Global` → `Built-in` 순이다.
2. `global`과 `nonlocal` 키워드를 이용하여 외부 스코프의 변수를 내부 스코프에서도 사용할 수 있다. 이때, `global`은 global scope에 정의된 전역 변수를 사용할 수 있도록 하고, `nonlocal`은 enclosed scope에 정의된 변수를 사용할 수 있도록 한다.
3. 클로저(closure)란 자신을 둘러싼 스코프(= enclosing scope)를 기억하는 함수이며, 다음의 조건을 만족해야 한다.
    1. 해당 함수는 어떤 함수 내에 중첩된 **내부 함수**여야 한다.
    2. 해당 함수는 자신을 둘러싼 함수 내에 정의되어 있는 **변수를 참조**해야 한다.
    3. 해당 함수를 둘러싼 함수는 이 **함수를 반환**해야 한다.
4. 클로저가 참조하는, enclosing scope에 정의된 변수를 자유 변수(free variable)라 한다.
5. 내부 함수(Local Scope)에서 외부 함수의 지역 변수(Enclosing Scope)에 접근하는 세 가지 방법의 동작은 다음과 같다.
    1. **[참조]** 내부 함수에서는 외부 함수에 정의된 지역 변수도 참조할 수 있다.
    2. **[대입]** 내부 함수에서 외부 함수의 지역 변수에 새로운 값을 대입한다면 **새로운 변수를 정의**하는 것으로 취급된다.
    3. **[변경]** 해당 변수가 **mutable** 하다면 내부 함수에서 변경할 수 있으나, **immutable** 하다면 내부 함수에 새로운 지역 변수를 생성 & 정의하게 된다.

<br>

## 1. 스코프(Scope)

### 1-1. 스코프란?

스코프(scope)란, **프로그램 실행의 한 지점에서 어떤 변수(혹은 함수)에 접근 가능한지**에 관한 것이다. 프로그래밍 언어마다 스코프 규칙이 다르며, 변수와 함수는 정의된 스코프가 종료된 순간부터는 더이상 접근 불가하다.

파이썬의 스코프에는 네 가지 종류가 있다.

| Scope Type         | Description                                  |
| ------------------ | -------------------------------------------- |
| Local Scope (L)    | 함수 혹은 클래스 내부에 정의됨               |
| Enclosed Scope (E) | enclosing function 내부에 정의됨 (중첩 함수) |
| Global Scope (G)   | 최상단에 정의됨                              |
| Built-in Scope (B) | 파이썬 built-in 모듈에 reserved 됨           |

<br>

### 1-2. 파이썬의 Scope Resolution 규칙: LEGB

스코프는 중첩된 형태로 존재한다. 따라서 프로그램이 어떤 변수에 접근하려 할 때, 중첩된 스코프 체인(scope chain)을 따라 해당 변수를 정의했는지를 찾아나간다. 이러한 과정을 **scope resolution**이라 하며, `Local` → `Enclosed` → `Global` → `Built-in` 순으로 찾아나가기 때문에 **LEGB 규칙**이 적용된다고 부른다.

![](/assets/img/posts/Python/Fluent-Python/2023-10-12-01.png){: .w-75}

<br>

### 1-3. Variable Shadowing을 피하는 방법: `global` / `nonlocal` 키워드

위에서 다룬 LEGB 규칙으로 인해, <span class="blue">**내부 스코프에서 정의된 변수**</span>와 <span class="blue">**외부 스코프에서 정의된 변수**</span>의 **이름이 동일**한 경우에는 **variable shadowing**이 발생한다. 이는 더 좁은 스코프에 정의된 변수가 더 넓은 스코프에 정의된 변수를 가리게 되어(shadowing), 내부 스코프에서는 외부 스코프에서 정의된 변수를 사용할 수 없는 현상이다.

> 정확히 말하면, **내부 스코프에서 동일한 이름의 변수에 값을 대입(assign)하는 순간부터** variable shadowing이 발생한다.
>
> > 내부 스코프에서 “정의”된 변수란, 어떠한 “값이 대입”되었다는 뜻이다!
> 
> 만약, 내부 스코프에서 동일한 이름의 변수에 값을 대입하는 행위 없이 해당 변수를 **참조 혹은 (가변 객체일 때) 변경만** 한다면, 문제 없이 접근할 수 있다.
{: .prompt-warning}

<br>

그렇다면, 내부 스코프에서 (1) 외부 스코프에서 정의되어 있으면서 (2) 동일한 이름을 가진 변수에 **값을 대입하거나 (불변 객체일 때) 변경**하려면 어떻게 해야할까?

바로 `global`, `nonlocal` 키워드를 이용하면 된다.

- **`global` 키워드** : global scope에 정의된 변수, 즉 **전역 변수**를 내부 스코프에서도 사용 가능
- **`nonlocal` 키워드** : enclosed scope에 정의된 변수, 즉 중첩 함수에서 **외부 함수의 지역 변수**를 내부 스코프에서도 사용 가능

<br>

### 1-4. 파이썬의 Variable Look-up Logic 총정리

- `global` 키워드가 붙어있다면, **전역 스코프(global scope)**에서 찾는다.
- `nonlocal` 키워드가 붙어있다면, 해당 변수가 정의된 곳으로부터 **가장 가까운 surrounding 함수의 지역 스코프(enclosed/nonlocal scope)**에서 찾는다.
- 변수가 함수의 파라미터이거나 함수 내부에서 값이 대입된 경우라면, 해당 함수의 **지역 스코프(local scope)**에서 찾는다.
- 변수가 파라미터도 아니고 대입된 경우도 아니라면, 다음과 같이 가까운 순서로 찾는다.
    1. **enclosed(nonlocal) scope** : surrounding 함수의 local scopes에서 찾는다.
    2. **global scope** : nonlocal scope에서 찾지 못하는 경우, global scope에서 찾는다.
    3. **built-in scope** : global scope에서 찾지 못하는 경우, `__builtins__.__dict__`에서 찾는다.

<br>

## 2. 클로저(Closure)

> 중첩 함수의 상황, 즉 enclosed scope에 대해서 더 자세히 다뤄보자! 바로 여기에서 클로저가 등장한다.

### 2-1. 클로저란?

![Untitled](/assets/img/posts/Python/Fluent-Python/2023-10-12-02.png)

파이썬에서 **클로저(closure)** `f`란, `f` 내부에서 (1) 전역 변수도, (2) `f`의 지역 변수도 아닌 변수들을 참조하는 확장된 스코프를 가지는 함수이다. 즉, 클로저란 **자신을 둘러싼 스코프(= enclosing scope)를 기억하는 함수**라고 할 수 있다.

<br>

어떤 함수(`averager()`)가 <span class="red">**클로저이려면 만족해야 하는 조건**</span>은 다음과 같다.

1. 해당 함수는 어떤 함수 내에 중첩된 **내부 함수**여야 한다.
2. 해당 함수는 자신을 둘러싼 함수(`make_averager()`) 내에 정의되어 있는 **변수를 참조**해야 한다.
3. 해당 함수를 둘러싼 함수는 이 **함수를 반환**해야 한다.

<br>

### 2-2. 자유 변수(Free Variable)

클로저는 확장된 스코프를 가지므로 **(1) 전역 변수가 아니면서(non-global variable) (2) 함수 외부에 정의되어 있는 변수(external variable)**에 접근할 수 있는데, 이러한 변수를 **자유 변수(free variable)**라고 한다.

정리하면, 클로저가 참조하는, 즉 enclosing scope에 정의된 변수가 자유 변수인 것이다.

> 위의 예시에서 `series` 변수는 `averager()` 함수를 기준으로 **자유 변수**이다. 그 이유는 다음과 같다.
> 
> 1. `series`는 외부 함수인 `make_averager()`의 지역 변수로, 전역 변수가 아니다.
> 2. `series`는 내부 함수인 `averager()`의 지역 스코프에 bound 되지 않는, 함수 외부에 정의된 변수이다.

<br>

**자유 변수가 존재 가능한 상황**은 다음과 같다.

1. 해당 함수가 다른 함수 안에 중첩(nested)된 경우
2. 해당 변수가 외부 함수(outer function)의 지역 스코프(local scope)의 일부인 경우

<br>

> 클로저로 인해 defining scope가 unavailable한 상황에서 해당 함수가 호출될 때도 자유 변수의 binding을 사용할 수 있게 된다.
{: .prompt-tip}

<br>

### 2-3. 클로저의 특징

- 일반적으로는 함수가 종료되면 해당 local scope에 정의된 모든 변수는 메모리에서 제거되지만, **클로저를 사용하면 외부 함수가 종료되어도 내부 함수가 enclosing scope에 정의된 변수(= 외부 함수의 지역 변수)를 참조**할 수 있다. 이러한 동작은 외부 함수가 제거되어도 가능하다.
- 이처럼 외부 함수의 지역 변수와 코드를 묶어서 사용할 수 있다는 점에서 편리하기는 하지만, 한편으로는 클로저가 참조하는 변수의 생명 주기가 연장되므로 **가비지 컬렉션(garbage collection) 측면에서는 메모리 누수(memory leak)**가 발생 가능하기 때문에 주의해야 한다.
- 가독성 측면에서는 별로 좋지 않기 때문에, 클로저의 동작이 복잡해진다면 **클래스로** 만드는 것을 권장한다.

<br>

### 2-4. 내부 함수(Local Scope)에서 외부 함수의 지역 변수(Enclosing Scope)에 접근하기

> enclosing scope에 정의된 변수에 접근하는 세 가지 경우(**참조, 대입, 변경**)에 대해 알아보자!

1. 내부 함수에서 외부 함수에 정의된 지역 변수를 <span class="red">**참조**</span>하는 것은 항상 가능하다. (→ 클로저와 자유 변수)
2. 내부 함수에서 외부 함수의 지역 변수에 <span class="red">**새로운 값을 대입**</span>하는 경우, 해당 변수가 내부 함수의 스코프에 정의되어 있지 않기 때문에 영향을 끼칠 수 없으므로 **새롭게 변수를 정의**하는 것으로 취급된다.
    - 해당 변수의 가변성이 mutable인지, immutable인지와는 관련 없이 동일하게 적용된다.
    - 즉, 외부 함수의 지역 변수가 가리키는 객체와의 binding이 끊기고 **새롭게 생성된 객체에 binding** 되게 된다. 이렇게 새로 정의된 (하지만 변수 이름은 동일한) 변수의 스코프는 **local scope**가 된다.
        
        > 본래 의도한 바는 해당 변수의 값만 업데이트되는 것이지만, 그렇게 동작하지 않는다! 변수는 값을 담는 상자가 아닌 **라벨**에 불과하다는 사실을 잊지 말자!
        > 
    - 만약 어떤 변수가 현재 스코프에 이미 정의되어 있는 경우라면, 해당 변수의 재할당은 의도한대로 이루어지게 된다.
3. 외부 함수의 지역 변수를 내부 함수에서 <span class="red">**변경**</span>할 때의 동작은 **해당 변수의 가변성에 따라** 달라지게 된다.
    
    
    | 변수의 데이터 타입 | 예시 | 내부 함수에서의 변경 가능 여부 |
    | --- | --- | --- |
    | 가변 (mutable) | `list` | 외부 함수의 지역 변수이더라도 mutable하므로 `.append()`<br>등의 함수를 통해 변경이 가능하다. |
    | 불변 (immutable) | `numbers`,<br>`strings`,<br>`tuples` | immutable 객체의 update는 새로운 값을 대입함으로써 이루어진다.<br>하지만 1번에서 언급했듯이 외부 함수의 지역 변수에 새로운 값을 대입하는 것은<br>불가능하며, **새로운 지역 변수를 implicitly 하게 생성**하게 된다.<br>→ <span class="hl">더이상 자유 변수가 아니게 되며, 클로저에 저장되지 않는다!</span> |


<br>

> 관련 포스팅: [[Better Way #21] 변수 영역과 클로저의 상호작용 방식을 이해하라](/posts/effective-python-03-better-way-21/)
{: .prompt-info}


<br>

> **총 정리**
>
> 1. **[참조]** 내부 함수에서는 외부 함수에 정의된 지역 변수도 참조할 수 있다. (→ **클로저**)
> 2. **[대입]** 내부 함수에서 외부 함수의 지역 변수에 새로운 값을 대입한다면 **새로운 변수를 정의**하는 것으로 취급된다.
> 3. **[변경]** 해당 변수가 **mutable** 하다면 내부 함수에서 변경할 수 있으나, **immutable** 하다면 내부 함수에 새로운 지역 변수를 생성 & 정의하게 된다.
{: .prompt-danger}

<br>

### 2-5. 클로저에서 `nonlocal` 키워드 사용하기

`nonlocal` 키워드를 통해 내부 함수에서 외부 함수의 지역 변수에 **다시 값을 대입**하더라도 새롭게 정의된 지역 변수가 아닌 **자유 변수로 사용할 수 있다**.

```python
def make_averager():
    count = 0
    total = 0
    
    def averager(new_value):
        nonlocal count, total   # -- new local variables (X) / free variables (O)
        count += 1
        total += new_value
        return total / count
    
    return averager
```

> **리마인드**
> 
> `nonlocal`과 `global`은 비슷하나, 값 대입의 대상이 되는 변수의 종류가 다르다.
> 
> | 구분 | 값 대입 대상이 되는 변수 |
> | --- | --- |
> | `global`  | 전역 변수 (global variable) |
> | `nonlocal`  | 외부 함수의 지역 변수 (local variable) |

<br>

## 3. Dynamic Scope vs. Lexical Scope

first-class function을 가지는 언어에서 어떤 함수가 (1) 정의된 스코프와 (2) 호출된 스코프가 다르다면, 자유 변수를 어떻게 평가해야 할까?

이때, **자유 변수의 평가가 이루어지는 시점**에 따라 다음의 두 가지로 나눌 수 있다.

| 구분 | 자유 변수의 평가 시점 | Description |
| --- | --- | --- |
| dynamic scope |  해당 함수가 **호출**된 시점의 영향을 받는다. | 프로그래머가 해당 함수가 제대로 동작하기 위한 환경을<br>설정하려면 **함수 내부**를 알아야 한다. |
| lexical scope | 해당 함수가 **정의**된 시점의 영향을 받는다. | **클로저**가 필요하다. |

<br>

> 파이썬은 클로저를 통해 lexical scope를 지원한다.
{: .prompt-info}

<br>

## References

- “Fluent Python (2nd Edition)”, Ch09. Decorators and Closures
- <https://www.geeksforgeeks.org/scope-resolution-in-python-legb-rule/>
- <https://www.geeksforgeeks.org/variable-shadowing-in-python/>
- <https://medium.com/@dannymcwaves/a-python-tutorial-to-understanding-scopes-and-closures-c6a3d3ba0937>
- <https://benmyers.dev/blog/scope/>
- <https://gsbang.tistory.com/entry/Python-클로저Closure>
- <https://shoark7.github.io/programming/python/closure-in-python>
- <https://dojang.io/mod/page/view.php?id=2366>
- <https://www.geeksforgeeks.org/python-closures/>
- <https://velog.io/@fhwmqkfl/TILPython.-Scope>
