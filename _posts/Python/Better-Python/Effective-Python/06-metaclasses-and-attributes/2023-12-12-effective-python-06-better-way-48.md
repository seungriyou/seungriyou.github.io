---
title: "[Better Way #48] __init_subclass__를 사용해 하위 클래스를 검증하라"
date: 2023-12-12 10:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, validation, inheritance, metaclass]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

## 1. 메타클래스의 가장 간단한 활용법

메타클래스의 대표적인 사용 목적은 <span class="shl">어떤 클래스(하위 클래스)가 제대로 구현됐는지 **검증**</span>하는 것이다.

- 복잡한 클래스 계층을 설계할 때, 어떤 스타일을 강제로 지키도록 할 때
- 메서드를 오버라이드하도록 요청할 때
- 클래스 애트리뷰트 사이에 엄격한 관계를 가지도록 할 때

이처럼 검증에 메타클래스를 사용하면, **프로그램 시작 시 클래스가 정의된 모듈을 처음 import 할 때와 같은 시점에 검증**이 이루어지므로 **예외가 훨씬 빨리 발생**할 수 있다.

<br>

## 2. 일반적인 객체에 대해 메타클래스가 작동하는 방법

메타클래스는 **`type`을 상속**해 정의되며, 기본적으로 **`__new__` 메서드**를 통해 자신과 연관된 클래스의 내용을 받는다.

> **메타클래스의 `__new__` 메서드**
>
> `class` 문의 모든 본문이 처리된 직후에 호출된다.
>
> > **[[Python] 메타클래스(Metaclass)](/posts/python-metaclass/)** 포스팅 참고
{: .prompt-info}

- 메타클래스 정의
    
  ```python
  class Meta(type):
      def __new__(meta, name, bases, class_dict):
          print(f"* 실행: {name}의 metaclass {meta}.__new__")
          print("부모 클래스들:", bases)
          print(class_dict)
          return type.__new__(meta, name, bases, class_dict)
  ```
    
- 메타클래스를 지정한 클래스
    
  ```python
  class MyClass(metaclass=Meta):
      stuff = 123
      
      def foo(self):
          pass
  ```
  
  ```
  * 실행: MyClass의 metaclass <class '__main__.Meta'>.__new__
  부모 클래스들: ()
  {'__module__': '__main__', '__qualname__': 'MyClass', 'stuff': 123, 'foo': <function MyClass.foo at 0x106f05a20>}
  ```
    
- 메타클래스를 지정한 클래스를 상속하는 클래스
    
  ```python
  class MySubclass(MyClass):
      other = 567
      
      def bar(self):
          pass
  ```
  
  ```
  * 실행: MySubclass의 metaclass <class '__main__.Meta'>.__new__
  부모 클래스들: (<class '__main__.MyClass'>,)
  {'__module__': '__main__', '__qualname__': 'MySubclass', 'other': 567, 'bar': <function MySubclass.bar at 0x106f05b40>}
  ```
    
<br>

위의 코드를 통해 **메타클래스는 다음과 같은 요소에 접근할 수 있음**을 알 수 있다.

- **`meta`**: 클래스 이름
- **`bases`**: 클래스가 상속하는 부모 클래스들
    
    > 모든 클래스가 상속하는 `object`는 명시적으로 들어있지 않다.
    
- **`class_dict`**: `class` 본문에 정의된 모든 클래스 애트리뷰트
    
    > 해당 클래스가 상속하는 부모 클래스의 본문에 정의된 클래스 애트리뷰트는 포함하지 않는다.

<br>

## 3. 연관된 클래스 정의 전, 파라미터 검증하기

### [1] 메타클래스

> 다각형을 표현하는 타입을 만든다고 가정하자.

검증을 수행하는 특별한 메타클래스를 정의하고, `Meta.__new__`에 파라미터 검증 기능을 추가한다.

이 메타클래스를 모든 다각형 클래스 계층 구조의 기반 클래스로 사용한다.

- **메타클래스 (변의 개수 검증)**
    
  ```python
  class ValidationPolygon(type):
      def __new__(meta, name, bases, class_dict):
          # Polygon 클래스의 하위 클래스만 검증한다
          if bases:
              if class_dict["sides"] < 3:
                  raise ValueError("다각형 변은 3개 이상이어야 함")
          return type.__new__(meta, name, bases, class_dict)
  ```
    
- **다각형 클래스**
    
  ```python
  class Polygon(metaclass=ValidationPolygon):
      sides = None  # 하위 클래스는 이 애트리뷰트에 값을 지정해야 한다
  
      @classmethod
      def interior_angles(cls):
          return (cls.sides - 2) * 180
  ```
    
<br>

**변의 개수가 3개 이상인 다각형 클래스**를 선언하면 오류 없이 사용할 수 있다.

```python
class Triangle(Polygon):
    sides = 3

class Rectangle(Polygon):
    sides = 4

class Nonagon(Polygon):
    sides = 9

assert Triangle.interior_angles() == 180
assert Rectangle.interior_angles() == 360
assert Nonagon.interior_angles() == 1260
```

하지만, **변의 개수가 2개 이하인 클래스**를 정의하면, `class` 본문이 처리된 직후 에러가 발생한다.

```python
# 변이 두 개 이하인 클래스를 정의하면 프로그램이 아예 시작하지도 않음

print("class 이전")

class Line(Polygon):
    sides = 2  # sides에 3 미만 값을 설정

print("class 이후")
```

```
class 이전
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
Cell In[12], line 5
      1 # 변이 두 개 이하인 클래스를 정의하면 프로그램이 아예 시작하지도 않음
      3 print('class 이전')
----> 5 class Line(Polygon):
      6     sides = 2   # sides에 3 미만 값을 설정
      8 print('class 이후')

Cell In[7], line 6
      4 if bases:
      5     if class_dict["sides"] < 3:
----> 6         raise ValueError("다각형 변은 3개 이상이어야 함")
      7 return type.__new__(meta, name, bases, class_dict)

ValueError: 다각형 변은 3개 이상이어야 함
```

<br>

### [2] `__init_subclass__` 메서드

위와 같은 다각형의 변 개수를 검증하는 기능을 다각형 클래스의 메타클래스 `ValidatePolygon`이 아닌 `__init_subclass__` 메서드를 통해서 구현할 수도 있다.

이러한 방식의 **장점**은 다음과 같은 것들이 있다.

- 코드가 훨씬 짧아진다.
- 메타클래스를 작성할 필요가 없다.
- 메타클래스에서 클래스 애트리뷰트 `sides`에 접근하기 위해서는 `class_dict["sides"]`와 같이 접근해야 했으나, `__init_subclass__` 메서드에서는 `cls.sides`로 접근할 수 있다.

<br>

`__init_subclass__` 메서드에 검증 기능을 추가한 `BetterPolygon` 클래스를 정의한다.

```python
class BetterPolygon:
    sides = None  # 하위 클래스에서 이 애트리뷰트의 값을 지정해야 함

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError("다각형 변은 3개 이상이어야 함")

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180
```

<br>

> 모든 코드가 메타클래스를 이용한 방법에서와 동일하게 동작한다.
> 

**변의 개수가 3개 이상인 다각형 클래스**를 선언하면 오류 없이 사용할 수 있다.

```python
class Hexagon(BetterPolygon):
    sides = 6

assert Hexagon.interior_angles() == 720
```

하지만, **변의 개수가 2개 이하인 클래스**를 정의하면, `class` 본문이 처리된 직후 에러가 발생한다.

```python
print("class 이전")

class Point(BetterPolygon):
    sides = 1

print("class 이후")
```

```
class 이전
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
Cell In[15], line 3
      1 print('class 이전')
----> 3 class Point(BetterPolygon):
      4     sides = 1
      6 print('class 이후')

Cell In[13], line 7
      5 super().__init_subclass__()
      6 if cls.sides < 3:
----> 7     raise ValueError('다각형 변은 3개 이상이어야 함')

ValueError: 다각형 변은 3개 이상이어야 함
```

<br>

> 클래스를 정의할 때, 파라미터를 검증하기 위해 메타클래스를 이용하는 방법 또는 `__init_subclass__` 메서드를 이용하는 방법을 적용할 수 있다는 것을 알아보았다.
>
> 하지만 **검증해야 할 요소가 여러 개라면?** 이러한 경우에는 어떤 방법이 더 유리할까?
{: .prompt-danger}

<br>

## 4. 검증 요소 여러 개 추가하기

### [1] 메타클래스

메타클래스를 이용한 검증 방식에서는 검증 요소를 한 번에 여러 개 추가할 수 없다. 왜냐하면 **클래스 정의마다 메타클래스는 단 하나만 지정**할 수 있기 때문이다.

<br>

다음의 코드에서는 `Filled`가 `ValidateFilled` 메타클래스를, `Polygon`이 `ValidationPolygon` 메타클래스를 지정하고 있으므로 두 메타클래스가 충돌하여 에러가 발생한다.

```python
class ValidateFilled(type):
    def __new__(meta, name, bases, class_dict):
        # Filled 클래스의 하위 클래스만 검증한다
        if bases:
            if class_dict["color"] not in ("red", "green"):
                raise ValueError("지원하지 않는 color 값")
        return type.__new__(meta, name, bases, class_dict)

class Filled(metaclass=ValidateFilled):
    color = None  # 모든 하위 클래스에서 이 애트리뷰트의 값을 지정해야 한다
```

```python
class RedPentagon(Filled, Polygon):
    color = "red"
    sides = 5
```

```
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
Cell In[17], line 1
----> 1 class RedPentagon(Filled, Polygon):
      2     color = 'red'
      3     sides = 5

TypeError: metaclass conflict: the metaclass of a derived class must be a (non-strict) subclass of the metaclasses of all its bases
```

<br>

따라서 메타클래스를 이용하여 검증을 여러 단계로 만들기 위해서는 **`type` 정의를 복잡한 계층으로 설계**해야 한다. `ValidatePolygon`을 메타클래스로 가지는 `Polygon` 클래스를 상속하고, `ValidateFilledPolygon`을 메타클래스로 가지는 `FilledPolygon`을 상속하여 만들면 된다.

```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        # 루트 클래스가 아닌 경우에만 검증
        if not class_dict.get("is_root"):
            if class_dict["sides"] < 3:
                raise ValueError("다각형 변은 3개 이상이어야 함")
        return type.__new__(meta, name, bases, class_dict)


# NOTE: is_root는 Polygon과 FilledPolygon이 상속되지 않고 그냥 쓰일 때에만 class_dict.get()으로 접근 가능
class Polygon(metaclass=ValidatePolygon):
    is_root = True
    sides = None  # 하위 클래스에서 지정해주어야 함
   
    
class ValidateFilledPolygon(ValidatePolygon):
    def __new__(meta, name, bases, class_dict):
        # 루트 클래스가 아닌 경우에만 검증
        if not class_dict.get("is_root"):
            if class_dict["color"] not in ("red", "green"):
                raise ValueError("지원하지 않는 color 값")
        return super().__new__(meta, name, bases, class_dict)
```

```python
class FilledPolygon(Polygon, metaclass=ValidateFilledPolygon):
    is_root = True
    color = None  # 하위 클래스에서 지정해주어야 함
```

- 검증 통과한 경우
    
  ```python
  class GreenPentagon(FilledPolygon):
      color = "green"
      sides = 5
  
  greenie = GreenPentagon()
  assert isinstance(greenie, Polygon)
  ```
    
- 색 검증에 실패한 경우
    
  ```python
  class OrangePentagon(FilledPolygon):
      color = "orange"
      sides = 5
  ```
  
  ```
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  Cell In[26], line 2
        1 # 색 검증 확인하기
  ----> 2 class OrangePentagon(FilledPolygon):
        3     color = "orange"
        4     sides = 5
  
  Cell In[15], line 15
        13 if not class_dict.get("is_root"):
        14     if class_dict["color"] not in ("red", "green"):
  ---> 15         raise ValueError("지원하지 않는 color 값")
        16 return super().__new__(meta, name, bases, class_dict)
  
  ValueError: 지원하지 않는 color 값
  ```
    
- 변 개수 검증에 실패한 경우
  
  ```python
  class RedLine(FilledPolygon):
      color = "red"
      sides = 2
  ```
  
  ```
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  Cell In[27], line 2
        1 # 변 개수 검증 확인하기
  ----> 2 class RedLine(FilledPolygon):
        3     color = "red"
        4     sides = 2
  
  Cell In[15], line 16
        14     if class_dict["color"] not in ("red", "green"):
        15         raise ValueError("지원하지 않는 color 값")
  ---> 16 return super().__new__(meta, name, bases, class_dict)
  
  Cell In[15], line 6
        4 if not class_dict.get("is_root"):
        5     if class_dict["sides"] < 3:
  ----> 6         raise ValueError("다각형 변은 3개 이상이어야 함")
        7 return type.__new__(meta, name, bases, class_dict)
  
  ValueError: 다각형 변은 3개 이상이어야 함
  ```
    
<br>

> 이처럼 **메타클래스**를 이용하여 여러 검증 요소를 추가하면 다음과 같은 **단점**이 있다.
> 
> - 모든 로직을 중복 정의해야 하므로 코드 재사용이 줄어든다.
> - 불필요한 준비 코드가 늘어난다.
> - 합성성이 저해된다.
{: .prompt-danger}

<br>

### [2] `__init_subclass__` 메서드

하지만 `__init_subclass__`를 사용하여 여러 검증 요소를 추가하면 **`super` 내장 함수를 이용해 부모나 형제자매 클래스의 `__init_subclass__`를 호출**해주는 한, 여러 단계로 이루어진 검증 구조를 쉽게 정의할 수 있다. 또한, 이는 **다중 상속**과도 잘 어우러진다.

> `__init_subclass__` 정의 안에서 **`super().__init_subclass__`를 호출**해 여러 계층에 걸쳐 클래스를 검증하고 다중 상속을 제대로 처리할 수 있다.
{: .prompt-tip}

<br>

**<span class="shl">다중 상속</span> —** `__init_subclass__` 메서드에서 변의 개수를 검증했던 `BetterPolygon` 클래스와, 다음과 같이 `__init_subclass__` 메서드에서 색을 검증하는 `Filled` 클래스를 모두 상속해보자. 두 클래스 `BetterPolygon`과 `Filled`는 모두 `super().__init_subclass__()`를 호출하므로 하위 클래스가 생성될 때 각각의 검증 로직이 실행된다.

```python
class Filled:
    color = None  # 하위 클래스에서 이 애트리뷰트 값을 지정해야 함

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.color not in ("red", "green", "blue"):
            raise ValueError("지원하지 않는 color 값")
```

- 검증 통과한 경우
    
  ```python
  class RedTriangle(Filled, BetterPolygon):
      color = "red"
      sides = 3
  
  ruddy = RedTriangle()
  assert isinstance(ruddy, Filled)
  assert isinstance(ruddy, BetterPolygon)
  ```
    
- 변 개수 검증에 실패한 경우
    
  ```python
  print("class 이전")
  
  class BlueLine(Filled, Polygon):
      color = "blue"
      sides = 2
  
  print("class 이후")
  ```
  
  ```
  class 이전
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  Cell In[32], line 5
        1 # 변의 수를 잘못 지정한 경우
        3 print("class 이전")
  ----> 5 class BlueLine(Filled, Polygon):
        6     color = "blue"
        7     sides = 2
  
  Cell In[15], line 6
        4 if not class_dict.get("is_root"):
        5     if class_dict["sides"] < 3:
  ----> 6         raise ValueError("다각형 변은 3개 이상이어야 함")
        7 return type.__new__(meta, name, bases, class_dict)
  
  ValueError: 다각형 변은 3개 이상이어야 함
  ```
    
- 색 검증에 실패한 경우
  
  ```python
  print("class 이전")
  
  class BeigeSquare(Filled, Polygon):
      color = "beige"
      sides = 4
  
  print("class 이후")
  ```
  
  ```
  class 이전
  ---------------------------------------------------------------------------
  ValueError                                Traceback (most recent call last)
  Cell In[33], line 6
        1 # 변의 수를 잘못 지정한 경우
        3 print("class 이전")
  ----> 6 class BeigeSquare(Filled, Polygon):
        7     color = "beige"
        8     sides = 4
  
  Cell In[15], line 7
        5     if class_dict["sides"] < 3:
        6         raise ValueError("다각형 변은 3개 이상이어야 함")
  ----> 7 return type.__new__(meta, name, bases, class_dict)
  
  Cell In[28], line 7
        5 super().__init_subclass__()
        6 if cls.color not in ('red', 'green', 'blue'):
  ----> 7     raise ValueError("지원하지 않는 color 값")
  
  ValueError: 지원하지 않는 color 값
  ```
    

<br>

**<span class="shl">다이아몬드 상속</span> —** 다음의 코드에서 `Bottom` 클래스 → `Top` 클래스에 이르는 상속 경로가 두 가지(`Left`를 통하는 경우, `Right`를 통하는 경로)이지만, 각 클래스마다 `Top.__init_subclass__`는 단 한 번만 호출된다.

> 다이아몬드 상속의 경우, **[[Better Way #40] super로 부모 클래스를 초기화하라 (+ MRO)](/posts/effective-python-05-better-way-40/)** 포스팅을 참고한다.
{: .prompt-info}

```python
class Top:
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f"{cls}의 Top")

class Left(Top):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f"{cls}의 Left")

class Right(Top):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f"{cls}의 Right")

class Bottom(Left, Right):
    def __init_subclass__(cls):
        super().__init_subclass__()
        print(f"{cls}의 Bottom")
```

```
<class '__main__.Left'>의 Top
<class '__main__.Right'>의 Top
<class '__main__.Bottom'>의 Top
<class '__main__.Bottom'>의 Right
<class '__main__.Bottom'>의 Left
```
