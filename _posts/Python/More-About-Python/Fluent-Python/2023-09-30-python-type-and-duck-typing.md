---
title: "[Python] 타입(Type): 덕 타이핑(Duck Typing), 이름 기반 타이핑(Nominal Typing)"
date: 2023-09-30 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, type, duck typing, nominal typing]
math: true
---

## TL;DR 📌

1. 파이썬에서 타입은 **“지원되는 연산의 집합”에 의해 정의**된다.
2. **덕 타이핑(duck typing)**이란, 파이썬이 런타임에서 실행될 때에 해당한다. 객체의 선언된 타입은 전혀 상관이 없으며, 실제로 **그 객체가 어떤 연산을 지원하는지**만 중요하다.
3. **이름 기반 타이핑(nominal typing)**이란, 파이썬에 타입 힌트를 추가하고 `mypy` 타입 검사를 수행할 때에 해당한다. 정적으로 강제되므로 런타임에서 발생할 수 있는 버그를 일찍 잡는 데에 유용하다.

<br>

## 1. 파이썬의 타입(Type)

### 1-1. 점진적 타이핑(Gradual Type System)

> 파이썬에서는 `mypy`를 이용한 **타입 힌트**를 통해 점진적 타이핑(gradual typing)을 도입할 수 있다.

**점진적 타이핑(gradual type system)**이란,

1. **타입이 명시된** 일부 변수 및 식에 대해서는 <span class="red">**컴파일 타임**</span>에(= **정적 타이핑**(static typing)) 타입의 정확성을 검사하며
2. **타입이 명시가 되어 있지 않은** 일부 식들에 대해서는 <span class="red">**런타임**</span>에(= **동적 타이핑**(dynamic typing)) 타입 에러가 발생할 수도 있는

타입 시스템이다.

<br>

이러한 **점진적 타이핑의 특징**에는 다음과 같은 것들이 있다.

1. <span class="shlp">**필수가 아니다.**</span>
    
    타입 힌트가 없는 코드에서 경고문을 출력하면 안 된다. 대신, 타입 체커가 객체의 타입을 결정할 수 없다면, 모든 타입과 호환되는 `Any` 타입으로 간주한다.
    
    - 타입 체커를 만족시키는 타입 힌트를 작성하기 어렵다면, 해당 타입 힌트는 제외하고 작성할 수 있다.
    - 타입 힌트는 모든 레벨에서 옵셔널하다. (주석을 통해 코드의 특정 라인이 무시되도록 할 수도 있다.)
2. <span class="shlp">**런타임에서 타입 에러를 잡지 않는다.**</span>
    
    타입 힌트는 정적 타입 체커, 린터, 그리고 IDE에서 경고문을 띄우는 데에 사용되며, 런타임에서 발생 가능한 타입 에러를 방지하지는 않는다.
    
3. <span class="shlp">**성능을 향상시키지 않는다.**</span>

<br>

### 1-2. 타입은 Supported Operation에 의해 정의된다!

파이썬에서 **타입**이란 <span class="blue">**“지원되는 연산의 집합”에 의해 정의**</span>된다.

예시로, 다음의 코드를 살펴보자.

```python
from collections import abc

def double(x: abc.Sequence):
    return x * 2
```

위의 코드에서 `double` 함수의 파라미터 `x`의 타입은 `*` 연산을 지원하는, 즉 `__mul__` 메서드를 구현 및 상속 받은 타입(ex. numeric, sequence, N-dim numpy array 등)이어야 한다.

그러나 **타입 힌트**에서 `x`를 `abc.Sequence` 타입으로 지정했고, 이 타입에서는 `*` 연산을 지원하지 않기 때문에 <span class="red">**타입 체크 단계**</span>에서 **reject** 되게 된다.

하지만 <span class="red">**실제 런타임**</span>에서는 **타입 힌트가 무시**되기 때문에 `x`가 타입 힌트에 위반되는 numeric이든, 타입 힌트에 적합한 sequence이든 **문제 없이 실행** 된다.

<br>

## 2. 점진적 타이핑의 타입에 대한 두 가지 관점

### 2-1. 덕 타이핑(Duck Typing)

> 파이썬이 **런타임에서 실행될 때**에 해당한다. (dynamic)
{: .prompt-tip}

- Smalltalk, <span class="blue">**Python**</span>, JavaScript, Ruby 등에서 채택되었다.
- 객체는 타입을 가지지만, 변수 및 파라미터는 타입을 가지지 않는다.
- 객체의 선언된 타입은 전혀 상관이 없으며, 실제로 **그 객체가 어떤 연산을 지원하는지**만 중요하다.
- **오직 런타임**에서 **객체의 연산이 시도될 때**만 강제된다.
- 이름 기반 타이핑(nominal typing) 보다 더 유연하지만, 발생 가능한 타입 관련 에러를 미리 잡을 수 없으므로 런타임에서 더 많은 에러를 허용한다.

<br>

### 2-2. 이름 기반 타이핑(Nominal Typing)

> 파이썬에 타입 힌트를 추가하고 **`mypy` 타입 검사를 수행할 때**에 해당한다. (static)
{: .prompt-tip}

- C++, Java, C# 등에서 채택되었고, <span class="blue">**annotated Python**</span>에 의해 지원된다.
- 객체와 변수는 모두 타입을 가진다.
    
    > 하지만 객체는 런타임에서만 존재하므로, 타입 체커는 **변수 및 파라미터가 타입 힌트로 annotated 된 코드**만 확인한다.
    > 
- 프로그램의 일부라도 실행하지 않고, 소스 코드에 대해서 **정적으로(statically) 강제**된다.
- 덕 타이핑보다 더 엄격하므로 일찍 버그를 잡는 데에 유용하다.

<br>

### 2-3. 코드 예시

```python
class Bird:
    pass

class Duck(Bird):
    def quack(self):
        print('Quack!')

def alert_bird(birdie: Bird) -> None:
    birdie.quack()
```

- <span class="shl">**덕 타이핑**</span>: 런타임에서 `alert_bird()` 함수가 실행 가능하다면, 즉 `birdie.quack()`을 호출할 수 있다면 이 컨텍스트에서 `birdie`는 `Duck`이다. (파이썬 런타임에서 타입 힌트는 무시된다!)
- <span class="shl">**이름 기반 타이핑**</span>: `Bird` 클래스는 `.quack()` 메서드를 지원하지 않으므로, `mypy` 타입 체커 단계에서 `alert_bird()` 함수는 illegal 하다.

<br>

> 파이썬은 **런타임**에 타입 힌트에 대해서는 전혀 신경쓰지 않고, **덕 타이핑만 사용**한다. 하지만, **이름 기반 타이핑**인 **타입 체커**를 사용한다면 런타임에서 발생 가능한 많은 에러를 방지할 수 있다.
{: .prompt-info}

<br>

### 2-4. 장단점 비교

|   | 장점 | 단점 |
| --- | --- | --- |
| <span class="shl">**덕 타이핑**</span> | 더 유연하다. | 런타임에서 에러를 발생시킬 수 있는 unsupported operations를<br>허용한다. |
| <span class="shl">**이름 기반 타이핑**</span> | 런타임에서 발생 가능한 에러를<br>미리 검출할 수 있다. | 때때로 실제로는 실행 가능한 코드를 reject 하기도 한다.  |

<br>

## References

- “Fluent Python (2nd Edition)”, Ch08. Type Hints in Functions
- <https://en.wikipedia.org/wiki/Gradual_typing>
