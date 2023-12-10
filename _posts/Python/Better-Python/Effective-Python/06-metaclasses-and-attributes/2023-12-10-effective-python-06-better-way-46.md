---
title: "[Better Way #46] 재사용 가능한 @property 메서드를 만들려면 디스크립터를 사용하라"
date: 2023-12-10 13:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, property]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

## TL;DR

1. `@property` 메서드의 동작과 검증 기능을 **재사용**하고 싶다면 **디스크립터 클래스**를 만들어라.
2. 디스크립터 클래스를 만들 때는 **메모리 누수**를 방지하며 **각 인스턴스에 대해 따로 값을 추적**할 수 있도록 하기 위해 **`WeakKeyDictionary`**를 사용하라.
3. **`__getattribute__`**가 **디스크립터 프로토콜**을 사용해 애트리뷰트 값에 접근하는 방식을 정확히 이해하라.

<br>

---

<br>

`@property` 내장 기능의 가장 큰 문제점은 <span class="hl">**재사용성**</span>이다.

1. `@property`가 데코레이션하는 메서드를 같은 클래스에 속하는 여러 애트리뷰트로 사용할 수 없다.
2. 서로 무관한 클래스 사이에서 `@property` 데코레이터를 적용한 메서드를 재사용할 수도 없다.

따라서 동일한 기능을 다시 사용하고 싶다면 **같은 `@property` 메서드**와 **대응되는 setter 메서드**를 번거롭게 다시 작성해야 한다.

이처럼 `@property` 메서드를 재사용하려는 경우에는 **디스크립터 프로토콜**을 적용할 수 있다.

<br>

## 디스크립터 프로토콜

- **애트리뷰트 접근**을 해석하는 방법을 정의한다.
- 디스크립터 클래스는 **`__get__`과 `__set__` 메서드**를 제공하며, 이 두 메서드를 통해 다른 코드 없이도 **`@property` 메서드**와 그에 **대응되는 setter 메서드**로 구현하려 했던 동작을 **재사용 가능한 형태로 구현**할 수 있다.
- 같은 로직을 한 클래스 안에 속한 여러 다른 애트리뷰트에 적용할 수 있으므로, 디스크립터가 믹스인보다 낫다.

<br>

### 디스크립터 애트리뷰트에 대한 접근을 파이썬이 처리하는 방법 (`__getattribute__`)

- 다음과 같이, **`Grade`의 인스턴스를 클래스 애트리뷰트로** 가지는 `Exam` 클래스를 살펴보자.
    
  ```python
  class Grade:
      def __get__(self, instance, instance_type):
          ...
      
      def __set__(self, instance, value):
          ...
  ```
  
  ```python
  class Exam:
      # 클래스 애트리뷰트
      math_grade = Grade()
      writing_grade = Grade()
      sciend_grade = Grade()
  ```
    
- **프로퍼티 대입**은 다음과 같이 **`__set__` 메서드**로 해석된다.
    
  ```python
  exam = Exam()
  exam.writing_grade = 40
  
  # Exam.__dict__['writing_grade'].__set__(exam, 40)
  ```
    
- **프로퍼티 접근**은 다음과 같이 **`__get__` 메서드**로 해석된다.
    
  ```python
  exam.writing_grade
  
  # Exam.__dict__['writing_grade'].__get__(exam, Exam)
  ```
    
- 이는 **`object`의 `__getattribute__` 메서드**의 동작이다.
    1. **`Exam` 인스턴스**에 `writing_grade` 라는 이름의 애트리뷰트가 있는지 확인하고, 있으면 그것을 사용한다.
    2. **`Exam` 인스턴스**에 해당 애트리뷰트가 없으면, **`Exam` 클래스**의 애트리뷰트를 대신 사용한다.
    3. **클래스 애트리뷰트**가 **`__get__`과 `__set__` 메서드가 정의된 객체**라면, **디스크립터 프로토콜**을 따른다.

<br>

### Wrong Way #1

다음과 같은 코드에서는 **`Exam` 클래스가 처음 정의될 때**, 각 클래스 애트리뷰트에 대한 **`Grade` 인스턴스가 단 한 번만 생성**된다.

즉, `Exam` 인스턴스가 생성될 때마다 매번 새로운 `Grade` 인스턴스가 생성되는 것이 아닌, `Exam` 클래스가 정의될 때 한 번 생성된 **인스턴스를 계속해서 재사용**하게 되는 것이다.

```python
class Grade:
    def __init__(self):
        self._value = 0
        
    def __get__(self, instance, instance_type):
        return self._value
    
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError("점수는 0과 100 사이입니다.")
        self._value = value
```

```python
class Exam:
    # 클래스 애트리뷰트 (Exam 클래스 정의 시 단 한 번만 생성)
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()
```

따라서 다음과 같이 한 `Exam` 인스턴스의 애트리뷰트만 새로운 값으로 할당하더라도, <span class="hl">**전체 인스턴스의 애트리뷰트 값이 모두 같은 값으로 할당**</span>되게 되는 문제가 발생한다.

```python
first_exam = Exam()
first_exam.writing_grade = 82
first_exam.science_grade = 99
print(f"쓰기: {first_exam.writing_grade}")
print(f"과학: {first_exam.science_grade}")
# 쓰기: 82
# 과학: 99

second_exam = Exam()
second_exam.writing_grade = 75
print(f"두 번째 쓰기 점수: {second_exam.writing_grade} -> 맞음")
print(f"첫 번째 쓰기 점수: {first_exam.writing_grade} -> 틀림")
# 두 번째 쓰기 점수: 75 -> 맞음
# 첫 번째 쓰기 점수: 75 -> 틀림
```

<br>

### Wrong Way #2

위의 문제점을 해결하기 위해, 다음과 같이 `Grade` 클래스가 **각 `Exam` 인스턴스에 대해 따로 값을 추적**하도록 딕셔너리를 이용하여 구현할 수 있다.

```python
class Grade:
    def __init__(self):
        self._values = {}    # 각 Exam 인스턴스에 대해 따로 값을 추적하기 위한 딕셔너리
        
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)
    
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError("점수는 0과 100 사이입니다.")
        self._values[instance] = value
```

하지만 이러한 경우에는 프로그램이 실행되는 동안 `__set__` 호출에 전달된 **모든 `Exam` 인스턴스에 대한 참조를 저장**하기 때문에 **인스턴스에 대한 reference counter가 절대로 0이 될 수 없고**, 그로 인해 **garbage collector**가 인스턴스 메모리를 결코 재활용할 수 없다.

즉, <span class="hl">**메모리 누수(leak)**</span>가 발생한다는 문제가 발생한다.

<br>

### Correct Way: `WeakKeyDictionary`

위의 문제점을 해결하기 위해서 파이썬 **`weakref` 내장 모듈**에서 제공하는 **`WeakKeyDictionary`**를 사용할 수 있다.

> **`WeakKeyDictionary`**는 딕셔너리에 객체를 저장할 때 일반적인 strong reference 대신 **weak reference**를 사용한다. 그리고 파이썬의 **garbage collector**는 <span class="hl">weak reference로만 참조되는 객체가 사용 중인 메모리를 언제든지 재활용</span>할 수 있다.
{: .prompt-info}

```python
from weakref import WeakKeyDictionary

class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()    # WeakKeyDictionary로 변경!
        
    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return self._values.get(instance, 0)
    
    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError("점수는 0과 100 사이입니다.")
        self._values[instance] = value
```

따라서 `WeakKeyDictionary`에 저장된 인스턴스가 더 이상 쓰이지 않는다면, 즉 해당 객체를 가리키는 모든 strong reference가 사라졌다면, garbage collector가 해당 메모리를 재활용할 수 있어 <span class="hl">**메모리 누수**</span>가 발생하지 않는 방법으로 <span class="hl">**각 인스턴스에 대해 따로 값을 추적**</span>할 수 있게 된다.

```python
first_exam = Exam()
first_exam.writing_grade = 82
second_exam = Exam()
second_exam.writing_grade = 75
print(f"두 번째 쓰기 점수: {second_exam.writing_grade} -> 맞음")
print(f"첫 번째 쓰기 점수: {first_exam.writing_grade} -> 맞음")

# 두 번째 쓰기 점수: 75 -> 맞음
# 첫 번째 쓰기 점수: 82 -> 맞음
```
