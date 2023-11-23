---
title: "[Better Way #41] 기능을 합성할 때는 믹스인 클래스를 사용하라"
date: 2023-11-23 12:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces, mixin]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>

> **TL; DR**
>
> 1. **믹스인을 합성**하면 단순한 동작으로부터 더 복잡한 기능을 만들어낼 수 있다.
> 2. 인스턴스 애트리뷰트와 `__init__`을 사용하는 **다중 상속은 피해야** 한다. **믹스인을 사용**하여 구현할 수 있는 기능이라면 믹스인을 사용하자!
> 3. 믹스인 클래스가 **클래스별로 특화된 기능을 필요**로 한다면, **인스턴스 수준에서 끼워넣을 수 있는 기능**(→ **정해진 메서드**를 통해 해당 기능을 인스턴스가 제공하게 만듦)을 활용하라.
> 4. 믹스인에는 필요에 따라 **인스턴스 메서드**는 물론, **클래스 메서드**도 포함될 수 있다.
{: .prompt-tip}

<br>

파이썬은 다중 상속을 지원하는 객체지향 언어이지만, **다중 상속**으로 인해 골치 아픈 상황이 생길 수 있으므로 **최대한 피하는 것을 권장**한다. 그렇다면 다중 상속이 제공하는 편의와 캡슐화가 필요하다면 어떻게 해야 할까?

바로, **믹스인(mix-in)**을 활용한다!

<br>

## 믹스인(Mix-in) 클래스

- 자식 클래스가 사용할 **메서드 몇 개만 정의**하는 클래스이다.
    
    > 즉, 해당 믹스인을 상속하는 모든 클래스에서 믹스인이 제공하는 기능을 사용할 수 있다.
    
- 자체 애트리뷰트 정의가 없기 때문에 믹스인 클래스의 `__init__` 메서드를 호출할 필요가 없다.
- 믹스인을 **합성하거나 계층화**하여 **반복적인 코드를 최소화**하고 **재사용성을 최대화**할 수 있다.

<br>

### 믹스인의 특징

1. 파이썬에서는 동적인 상태 접근이 가능하므로, **제너릭인 기능**을 믹스인 안에 한 번만 작성해두면 **다른 여러 클래스에 적용**할 수 있다.
    
    > 필요 시에는 기존 믹스인의 기능을 다른 기능으로 **오버라이드(override)** 하여 변경할 수도 있다.
    {: .prompt-info}

2. 믹스인을 사용하면 **인스턴스의 동작**이나 **클래스의 동작** 중 어느 것이든 하위 클래스에 추가할 수 있다.
    
    > 인스턴스의 동작은 **인스턴스 메서드**, 클래스의 동작은 **클래스 메서드(`@classmethod`)**를 통해 추가한다.
    {: .prompt-info}

<br>

믹스인의 특징에 대해 예시와 함께 자세히 알아보자!

<br>

## [예시] 믹스인 메서드 오버라이드

### (1) 파이썬 객체를 딕셔너리로 바꾸는 믹스인 `ToDictMixin` 작성

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output
    
    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict()
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value
```

<br>

이 믹스인을 사용하여 이진 트리를 딕셔너리로 변경해보자.

```python
class BinaryTree(ToDictMixin):
    def __init__(self, value, left=None, right=None):
        self.value = value
        self.left = left
        self.right = right
        

tree = BinaryTree(10,
                  left=BinaryTree(7, right=BinaryTree(9)),
                  right=BinaryTree(13, left=BinaryTree(11)))

pprint.pprint(tree.to_dict())
```

<br>

이진 트리를 나타내는 파이썬 객체가 딕셔너리로 변환되었다!

```python
{'left': {'left': None,
          'right': {'left': None, 'right': None, 'value': 9},
          'value': 7},
 'right': {'left': {'left': None, 'right': None, 'value': 11},
           'right': None,
           'value': 13},
 'value': 10}
```

<br>

### (2) 메서드 오버라이드를 통한 순환 참조 방지 처리

> 필요 시에는 기존 믹스인의 기능을 다른 기능으로 **오버라이드(override)** 하여 변경할 수도 있다.
{: .prompt-info}

하지만 **순환 참조가 존재하는 이진 트리**의 경우(ex. `BinaryTree`에 대한 참조를 저장하는 `BinaryTree`의 하위 클래스), `ToDictMixin.to_dict`는 무한 루프에 빠지게 된다. 

이를 처리하려면 다음과 같이 **문제가 발생하는 특정 메서드(`_traverse`)를 오버라이드**하여 문제가 되는 값만 처리하게 만들어서 믹스인이 무한 루프에 빠지지 않도록 한다!

```python
class BinaryTreeWithParent(BinaryTree):
    def __init__(self, value, left=None, right=None, parent=None):
        super().__init__(value, left=left, right=right)
        self.parent = parent
    
    # 믹스인 메서드 오버라이드를 통해 parent로 인한 순환 참조로 발생하는 무한 루프 방지
    def _traverse(self, key, value):
        # -- 부모를 가리키는 참조라면, 부모의 숫자 값을 삽입함으로써 순환 참조 방지
        if (isinstance(value, BinaryTreeWithParent) and key == 'parent'):
            return value.value
        # -- 부모를 가리키는 참조가 아니라면, super 내장 함수를 통해 디폴트 믹스인 호출
        else:
            return super()._traverse(key, value)

root = BinaryTreeWithParent(10)
root.left = BinaryTreeWithParent(7, parent=root)
root.left.right = BinaryTreeWithParent(9, parent=root.left)
pprint.pprint(root.to_dict())
```

<br>

변환 시 `parent`로 인한 순환 참조가 발생하지 않으므로 잘 작동한다.

```python
{'left': {'left': None,
          'parent': 10,
          'right': {'left': None, 'parent': 7, 'right': None, 'value': 9},
          'value': 7},
 'parent': None,
 'right': None,
 'value': 10}
```

<br>

### (3) `BinaryTreeWithParent`를 애트리뷰트로 가지는 클래스

`_traverse` 메서드 오버라이드 결과, `BinaryTreeWithParent` 인스턴스인 `tree_with_parent`를 애트리뷰트로 저장하는 클래스도 자동으로 `ToDictMixin`을 문제 없이 사용할 수 있다.

```python
class NamedSubTree(ToDictMixin):
    def __init__(self, name, tree_with_parent):
        self.name = name
        self.tree_with_parent = tree_with_parent
        

my_tree = NamedSubTree('foobar', root.left.right)
pprint.pprint(my_tree.to_dict())
```

```python
{'name': 'foobar',
 'tree_with_parent': {'left': None, 'parent': 7, 'right': None, 'value': 9}}
```

<br>

## [예시] 믹스인 합성 (w/ 인스턴스 메서드, 클래스 메서드)

### (1) 새로운 믹스인 `JsonMixin` 생성

> 믹스인을 사용하면 **인스턴스의 동작(→ 인스턴스 메서드)**이나 **클래스의 동작(→ 클래스 메서드, `@classmethod`)** 중 어느 것이든 하위 클래스에 추가할 수 있다.
{: .prompt-info}

- **직렬화**(파이썬 데이터 → JSON)와 **역직렬화**(JSON → 파이썬 데이터), 즉 **양방향 변환**이 가능하도록 하는 믹스인이다.
- **`JsonMixin`의 하위 클래스**는 다음과 같은 **요구사항**을 만족해야 한다.
    1. `__init__` 메서드가 키워드 인자를 받아야 한다.
    2. `to_dict` 메서드를 제공해야 한다.

```python
import json

class JsonMixin:
    # 클래스의 동작 (클래스 메서드)
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data)
        return cls(**kwargs)                # <req1> __init__ 메서드가 키워드 인자를 받아야 한다.
    
    # 인스턴스의 동작 (인스턴스 메서드)
    def to_json(self):
        return json.dumps(self.to_dict())   # <req2> to_dict 메서드를 제공해야 한다.
```

<br>

### (2) 양방향 변환 동작 확인하기

```python
class DatacenterRack(ToDictMixin, JsonMixin):
    def __init__(self, switch=None, machines=None):
        self.switch = Switch(**switch)
        self.machines = [
            Machine(**kwargs) for kwargs in machines
        ]
    
class Switch(ToDictMixin, JsonMixin):
    def __init__(self, ports=None, speed=None):
        self.ports = ports
        self.speed = speed
    
class Machine(ToDictMixin, JsonMixin):
    def __init__(self, cores=None, ram=None, disk=None):
        self.cores = cores
        self.ram = ram
        self.disk = disk
```

```python
serialized = """{
    "switch": {"ports": 5, "speed": 1e9},
    "machines": [
        {"cores": 8, "ram": 32e9, "disk": 5e12},
        {"cores": 4, "ram": 16e9, "disk": 1e12},
        {"cores": 2, "ram": 4e9, "disk": 500e9}
    ]
}"""
```

```python
deserialized = DatacenterRack.from_json(serialized)     # 파이썬 객체로 역직렬화
roundtrip = deserialized.to_json()                      # JSON으로 직렬화

assert json.loads(serialized) == json.loads(roundtrip)
```

<br>

> `JsonMixin`을 적용하려고 하는 클래스 상속 계층의 **상위 클래스에 이미 `JsonMixin`을 적용한 클래스가 있어도** 아무런 문제가 없다.
{: .prompt-tip}
