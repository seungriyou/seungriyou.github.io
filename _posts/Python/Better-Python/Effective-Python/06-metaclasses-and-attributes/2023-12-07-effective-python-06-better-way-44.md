---
title: "[Better Way #44] 세터와 게터 메서드 대신 평범한 애트리뷰트를 사용하라"
date: 2023-12-07 13:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch06 metaclasses and attributes, property]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 06. Metaclasses and Attributes"**을 읽고 정리한 내용입니다.

<br>

파이썬에서는 명시적인 setter나 getter 메서드를 구현할 필요가 없다.

따라서 다음과 같이 <span class="hl">**항상 단순한 공개 애트리뷰트로부터 구현을 시작**</span>하고, **가급적이면 setter나 getter 메서드를 사용하지 말아야** 한다.

```python
class Resistor:
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 0
        self.current = 0

r1 = Resistor(50e3)

# setter 및 getter 없이, 직접적으로 애트리뷰트에 접근하면 된다.
r1.ohms = 10e3
r1.ohms += 5e3
```

<br>

하지만 나중에 <span class="hl">**애트리뷰트가 설정될 때 특별한 동작을 수행해야 한다**</span>면, 애트리뷰트를 **`@property` 데코레이터와 대응하는 `setter` 애트리뷰트**를 사용할 수도 있다.

이러한 경우에 대해서 알아보자.

<br>

## `@property` 메서드를 사용하는 경우

### (1) 어떠한 애트리뷰트가 설정될 때, 다른 애트리뷰트에 영향을 미치는 경우
    
이와 같이 하면 `voltage` 프로퍼티에 대입할 때마다 `voltage` setter 메서드가 호출되어, 객체의 `current` 애트리뷰트가 함께 갱신된다.

```python
class VoltageResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0
    
    @property
    def voltage(self):              # getter
        return self._voltage
    
    @voltage.setter
    def voltage(self, voltage):     # setter
        self._voltage = voltage
        self.current = self._voltage / self.ohms
```

```python
r2 = VoltageResistance(1e3)
print(f"이전: {r2.current:.2f} 암페어")
r2.voltage = 10
print(f"이후: {r2.current:.2f} 암페어")

# 이전: 0.00 암페어
# 이후: 0.01 암페어
```
   
<br>
 
### (2) 타입을 검사하거나 클래스 프로퍼티에 전달된 값에 대한 검증을 수행하고 싶은 경우
  
다음의 코드에서는 모든 저항값이 0 ohm 보다 큰지 확인하게 된다.

```python
class BoundedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
    
    @property
    def ohms(self):
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        if ohms <= 0:
            raise ValueError(f"저항 > 0이어야 합니다. 실제 값: {ohms}")
        self._ohms = ohms
```

```python
r3 = BoundedResistance(1e3)
r3.ohms = 0

# ValueError: 저항 > 0이어야 합니다. 실제 값: 0
```

생성자에 잘못된 값을 넘기는 경우에도 객체 생성이 끝나기 전에 즉시 저항 값인 `ohms`를 검증하는 코드가 실행된다.
이는 `super().__init__(ohms)` 실행 시 `self.ohms = ohms`가 호출되어 setter 메서드가 호출되기 때문이다.

```python
BoundedResistance(-5)

# ValueError: 저항 > 0이어야 합니다. 실제 값: -5
```

<br>

### (3) 부모 클래스에 정의된 애트리뷰트를 불변으로 만들고 싶은 경우
    
이와 같이 하면 객체 생성 후 프로퍼티에 값을 대입했을 때 예외가 발생한다.

```python
class FixedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
    
    @property
    def ohms(self):
        return self._ohms
    
    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError("ohms는 불변 애트리뷰트입니다.")
        self._ohms = ohms
```

```python
r4 = FixedResistance(1e3)
r4.ohms = 2e3

# AttributeError: ohms는 불변 애트리뷰트입니다.
```

<br>

## `@property` 메서드 (getter, setter) 사용 시 주의점

1. 최소 놀람의 법칙(rule of least surprise)에 따라, 게터나 세터 메서드가 **예기치 않은 동작을 수행하지 않도록** 해야 한다.
    
    > ex) 게터 프로퍼티 메서드 안에서 다른 애트리뷰트를 설정하면 안 된다.
    
    > 관련이 있는 **객체 상태를 `@property.setter` 메서드 안에서만 변경**하는 것을 권장한다.
    {: .prompt-info}
    
2. **복잡하거나 느린 연산**의 경우에는 `@property` 메서드가 아닌 일반적인 메서드를 사용해야 한다.
    
    > ex) 동적으로 모듈 임포트, 시간이 오래 걸리는 도우미 함수 호출, I/O 수행, 데이터베이스 질의 등
    
3. `@property` 메서드는 하위 클래스 사이에서만 공유될 수 있으므로, 서로 관련이 없는 클래스 사이에 같은 `@property` 메서드 구현을 공유하기 위해서는 **디스크립터(descriptor)**를 사용한다.
    
    > → “Better Way 46”에서 다룬다.
