---
title: "[Better Way #42] 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라"
date: 2023-11-24 10:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>

## TL;DR
1. 파이썬 컴파일러는 **비공개 애트리뷰트**를 자식 클래스나 클래스 외부에서 사용하지 못하도록 **엄격히 금지하지 않는다**.
2. 작성하고 있는 클래스의 하위 클래스를 정의하는 사람들이 해당 클래스의 애트리뷰트를 사용하지 못하도록 막기보다는, 애트리뷰트를 사용해 더 많은 일을 할 수 있도록 허용하라.
3. 비공개 애트리뷰트로 (클래스 외부나 하위 클래스의) 접근을 막으려하기보다는, **보호된 필드를 사용하며 문서에 적절한 가이드**를 남겨라.
4. **비공개 애트리뷰트**는 **코드 작성을 제어할 수 없는 하위 클래스에서 이름 충돌이 일어나는 경우를 막고 싶을 때만** 사용할 것을 권장한다.

<br>

---

<br>

파이썬에서 클래스 애트리뷰트에 대한 가시성은 **공개(public)**와 **비공개(private)** 뿐이다. 비공개 애트리뷰트에 대해서 자세히 알아보자!

<br>

## 비공개 애트리뷰트 (private)

- 클래스 애트리뷰트 **이름 앞에 밑줄을 두 개(`__`)** 붙이면 비공개 필드가 된다.
- **어디에서 비공개 애트리뷰트에 접근 가능한지**에 대해 정리하면 다음과 같다.
    
    
    | 비공개 애트리뷰트 접근 여부 | 위치 |
    | --- | --- |
    | 불가능 ❌ | 클래스 외부, 하위 클래스 |
    | 가능 ⭕ | 해당 클래스 내 메서드, 클래스 메서드<br>(해당 class 블록 내부) |

- 비공개 애트리뷰트의 동작은 <span class="hl">**애트리뷰트 이름을 바꾸는 단순한 방식**</span>으로 구현되기 때문에, 이 방식을 알고 나면 비공개 애트리뷰트에 접근할 수 없던 곳(ex. 클래스 외부, 하위 클래스)에서도 비공개 애트리뷰트에 접근할 수 있게 된다.
  - 다음과 같은 코드에서 각 **`__private_field`의 실제 이름**은 다음과 같이 상이하다.
      
    ```python
    class MyParentObject:
        def __init__(self):
            self.__private_field = 71   # 실제 이름: _MyParentObject__private_field
            
    class MyChildObject(MyParentObject):
        def get_private_field(self):
            return self.__private_field # 실제 이름: _MyChildObject__private_field
    ```
    
    즉, 부모의 비공개 애트리뷰트를 자식 애트리뷰트에서 접근하면 <span class="hl">**단지 이름이 존재하지 않는다는 이유로 오류가 발생**</span>하는 것이다.
    
    ```python
    baz = MyChildObject()
    baz.get_private_field()
    # AttributeError: 'MyChildObject' object has no attribute '_MyChildObject__private_field'
    ```
      
  - 이러한 방식을 알고 나면 다음과 같이 실제 이름으로, 억지로 접근이 가능해진다.
      
    ```python
    assert baz._MyParentObject__private_field == 71
    ```
    
    ```python
    print(baz.__dict__)
    # {'_MyParentObject__private_field': 71}
    ```
      
<br>

> 이처럼 파이썬은 **비공개 애트리뷰트에 대해 가시성을 엄격하게 제한하지 않는다**. 하지만 내부에 몰래 접근함으로써 발생할 수 있는 피해를 줄이고자 파이썬 개발자는 **스타일 가이드에 정해진 명명 규약**을 지켜야 한다.
{: .prompt-info}

<br>

## 보호 애트리뷰트 (protected)

- 파이썬 스타일 가이드를 통해 **관례적으로 사용**되는 가시성으로, 필드 이름 앞에 **밑줄이 하나(`_`)**만 있으면 보호(protected) 필드로 간주한다.

  > 관례적으로 사용하는 것일 뿐, 강제적인 것은 아니다!

- 보호 필드는 클래스 외부에서 사용하는 경우 조심해야 한다는 뜻을 가지고 있다.

<br>

## 애트리뷰트 가시성 관련 팁

### (1) 하위 클래스나 클래스 외부에서 사용하면 안 되는 <span class="hl">내부 API</span>를 표현하는 경우

#### 권장하지 않는 방법
> **비공개 필드(`__`)**를 사용한다.
{: .prompt-danger}

- 누군가가 해당 클래스의 하위 클래스를 만들 때, 비공개 애트리뷰트로 인해 확장이나 오버라이드가 번거로워질 수 있다.
- 비공개 애트리뷰트의 실제 이름을 사용하여 억지로 확장할 경우, 확장 과정에서 참조가 올바르지 않게 되어 하위 클래스가 깨지기 쉬워 진다.

#### 권장하는 방법
> 상속을 허용하는 클래스(부모 클래스) 쪽에서 **보호 애트리뷰트(`_`)**를 사용하고, **모든 보호 필드에 문서를 추가**한다.
{: .prompt-tip}

- API 내부에 있는 필드 중에서 **어떤 필드를 하위 클래스에서 변경할 수 있고, 어떤 필드를 그대로 둬야 하는지** 명시한다.
- 코드를 안전하게 확장할 수 있는 방법을 다른 프로그래머는 물론 미래의 자신에게도 안내할 수 있다.

<br>

### (2) 비공개 애트리뷰트는 <span class="hl">하위 클래스의 필드와 이름이 충돌할 수 있는 경우</span>에만 사용

- 공개 API에 속한 클래스를 작성할 때는, 하위 클래스 작성이 프로그래머의 제어 밖에서 일어날 것이다.
- 애트리뷰트의 이름이 흔한 이름(ex. `value`)일 때, **하위 클래스 작성 시에 충돌**이 발생할 가능성이 높다.
- 이러한 충돌 위험성을 줄이려면, **부모 클래스(공개 API) 쪽에서 자식 클래스의 애트리뷰트 이름이 자신의 애트리뷰트 이름과 겹치는 일을 방지**하기 위해 **비공개 애트리뷰트**를 사용할 수 있다.
- 충돌하는 경우
    
  ```python
  class ApiClass:
      def __init__(self):
          self._value = 5
      
      def get(self):
          return self._value
  
  class Child(ApiClass):
      def __init__(self):
          super().__init__()
          self._value = 'hello'   # 충돌
  ```
  
  ```python
  a = Child()
  print(f"{a.get()}와 {a._value}는 달라야 한다.")
  # hello와 hello는 달라야 한다.
  ```
  
- 충돌을 방지한 경우
    
  ```python
  class ApiClass:
      def __init__(self):
          self.__value = 5    # 비공개 애트리뷰트
      
      def get(self):
          return self.__value  # 비공개 애트리뷰트
  
  class Child(ApiClass):
      def __init__(self):
          super().__init__()
          self._value = 'hello'   # 충돌 X
  ```
  
  ```python
  a = Child()
  print(f"{a.get()}와 {a._value}는 달라야 한다.")
  # 5와 hello는 달라야 한다.
  ```
