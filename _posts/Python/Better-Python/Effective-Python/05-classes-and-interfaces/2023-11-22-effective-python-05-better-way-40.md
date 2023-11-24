---
title: "[Better Way #40] super로 부모 클래스를 초기화하라 (+ MRO)"
date: 2023-11-22 12:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces, mro]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>

자식 클래스에서 부모 클래스를 초기화하는 올바른 방법에 대해 알아보자.

<br>

## 부모 클래스의 `__init__` 메서드를 직접 호출할 때의 문제점

> 부모 클래스를 초기화할 때, 부모 클래스의 `__init__` 메서드를 직접 호출하는 것은 권장하지 않는 방법이다. 그 이유는 무엇일까?
{: .prompt-danger}

- <span class="hl">**다중 상속**</span>을 사용하는 경우, 상위 클래스의 `__init__` 메서드를 직접 호출하면 **모든 하위 클래스에서 `__init__` 호출의 순서가 정해져 있지 않아** 프로그램이 예측할 수 없는 방식으로 작동할 수 있다.
    
  다음과 같이 클래스 정의에서 **부모 클래스를 나열한 순서**와 **부모 클래스의 생성자를 호출하는 순서**가 일치하지 않는다면 잘못된 결과를 얻을 수 있다.
  
  ```python
  class MyBaseClass:
      def __init__(self, value):
          self.value = value
          
  class TimesTwo:
      def __init__(self):
          self.value *= 2
          
  class PlusFive:
      def __init__(self):
          self.value += 5
  
  class AnotherWay(MyBaseClass, PlusFive, TimesTwo):
      def __init__(self, value):
          # 부모 클래스의 순서와 부모 클래스의 생성자를 호출하는 순서가 일치하지 X
          MyBaseClass.__init__(self, value)
          TimesTwo.__init__(self)
          PlusFive.__init__(self)
  ```
  
  ```python
  foo = AnotherWay(5)
  print(f"(5 + 5) * 2 = 20이 나와야 하지만, 실행 결과는 {foo.value}")
  # (5 + 5) * 2 = 20이 나와야 하지만, 실행 결과는 15
  ```
    
- <span class="hl">**다이아몬드 상속**</span>을 사용하는 경우, **공통 조상 클래스의 `__init__` 메서드가 여러 번 호출**될 수 있으므로 코드가 예기치 못한 방식으로 작동할 수 있다.
    
  > **다이아몬드 상속**
  > 
  > 
  > 어떤 클래스가 두 가지 서로 다른 클래스를 상속하는데, 두 상위 클래스의 상속 계층을 거슬러 올라가면 같은 조상 클래스가 존재하는 경우이다.
  {: .prompt-info}
  
  다음과 같은 경우, **상속 다이아몬드의 정점에 있는 `MyBaseClass.__init__`이 두 번 실행**되어, 중간에 값이 다시 초기화되므로 예상한 결과가 출력되지 않는다.
  
  ```python
  class TimesSeven(MyBaseClass):
      def __init__(self, value):
          MyBaseClass.__init__(self, value)
          self.value *= 7
          
  class PlusNine(MyBaseClass):
      def __init__(self, value):
          MyBaseClass.__init__(self, value)
          self.value += 9
  
  class ThisWay(TimesSeven, PlusNine):
      def __init__(self, value):
          TimesSeven.__init__(self, value)
          PlusNine.__init__(self, value)      # -- 호출 시, MyBaseClass.__init__이 다시 호출되면서 self.value가 다시 5로 돌아간다!
  ```
  
  ```python
  foo = ThisWay(5)
  print(f"(5 * 7) + 9 = 44가 나와야 하지만, 실행 결과는 {foo.value}")
  # (5 * 7) + 9 = 44가 나와야 하지만, 실행 결과는 14
  ```
  

<br>

## 부모 클래스를 초기화 할 때는 `super().__init__`을 호출하자

```python
class TimesSevenCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value *= 7

class PlusNineCorrect(MyBaseClass):
    def __init__(self, value):
        super().__init__(value)
        self.value += 9

class GoodWay(TimesSevenCorrect, PlusNineCorrect):
    def __init__(self, value):
        super().__init__(value)
```

```python
foo = GoodWay(5)
print(f"7 * (5 + 9) = 98이 나와야 하며, 실제 실행 결과도 {foo.value}")
# 7 * (5 + 9) = 98이 나와야 하며, 실제 실행 결과도 98
```

- **`super`**는 상속 다이아몬드 계층의 <span class="hl">**공통 상위 클래스를 단 한 번만 호출하도록 보장**</span>한다.
- `super`를 사용하면 추후 공통 상위 클래스의 이름을 변경하거나, 하위 클래스가 상속 받는 상위 클래스를 변경하더라도 각각의 `__init__` 메서드 정의를 바꿀 필요가 없다.
- **두 가지 파라미터**를 넘길 수 있는데, `object` 인스턴스를 초기화할 때는 두 파라미터를 지정할 필요가 없으므로 **부모 클래스를 초기화할 때는 아무 인자 없이 호출**하자!
    
    > 파이썬 컴파일러가 자동으로 올바른 파라미터(`__class__`, `self`)를 넣어주기 때문이다.
    > 
    > 
    > ```python
    > # 다음은 동일하다.
    > super(__class__, self).__init__(value)
    > super().__init__(value)
    > ```
    {: .prompt-info}

    - **두 가지 파라미터의 종류**
        
        | --- | --- |        
        | **첫 번째 파라미터** | 접근하고 싶은 MRO 뷰를 제공할 부모 타입 |
        | **두 번째 파라미터** | 지정한 타입의 MRO 뷰에 접근할 때 사용할 인스턴스 |
    - `super`에 두 파라미터를 제공해야 하는 경우는 **자식 클래스에서 상위 클래스의 특정 기능에 접근해야 하는 경우** 뿐이다.

<br>

> 그런데 `GoodWay`가 **<`TimesSevenCorrect`, `PlusNineCorrect`> 순서**로 상속받도록 하였는데, 실제 실행 결과를 살펴보면 `PlusNineCorrect`의 동작이 먼저 일어나 `+ 9`가 우선적으로 실행되게 된다. 그 이유는 **MRO** 때문이다!
{: .prompt-warning}

<br>

## MRO (Method Resolution Order)

- <span class="hl">**상위 클래스를 초기화하는 순서를 정의**</span>하며, [**C3 linearization**](https://en.wikipedia.org/wiki/C3_linearization)이라는 알고리즘을 사용한다.
- 파이썬은 **MRO**를 통해 **상위 클래스 초기화 순서**와 **다이아몬드 상속 문제**를 해결한다.
- 각 클래스의 <span class="red">**`__init__`이 호출된 순서의 역순**으로 **실제 초기화 작업이 수행**</span>된다!
    
  ```python
  for cls in GoodWay.mro():   # 클래스 메서드 mro()를 통해 MRO 순서를 살펴본다
      print(repr(cls))
  ```
  
  ![](/assets/img/posts/Python/Effective-Python/2023-11-22.png){: style="max-width: 70%"}

- 참고로, `GoodWay`의 **상속 순서를 반대로** 하면 다음과 같은 결과가 나오게 된다.
    
  ```python
  class GoodWay(PlusNineCorrect, TimesSevenCorrect):  # -- 상속 순서 반대
      def __init__(self, value):
          super().__init__(value)
  ```
  
  ```python
  foo = GoodWay(5)
  print(f"순서가 바뀌어 (7 * 5) + 9 = 44이 나와야 하며, 실제 실행 결과도 {foo.value}")
  # 순서가 바뀌어 (7 * 5) + 9 = 44이 나와야 하며, 실제 실행 결과도 44
  ```
  
  ```python
  for cls in GoodWay.mro():   # 클래스 메서드 mro()를 통해 MRO 순서를 살펴본다
      print(repr(cls))
  ```    
  ![](/assets/img/posts/Python/Effective-Python/2023-11-22-02.png){: style="max-width: 70%"}
