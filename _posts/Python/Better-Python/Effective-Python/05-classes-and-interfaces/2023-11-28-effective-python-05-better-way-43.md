---
title: "[Better Way #43] 커스텀 컨테이너 타입은 collections.abc를 상속하라"
date: 2023-11-28 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch05 classes and interfaces]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 05. Classes and Interfaces"**을 읽고 정리한 내용입니다.

<br>

파이썬 컨테이너 타입을 이용하여 **커스텀 컨테이너 타입**을 만들고 싶다면 다음의 두 가지 방법 중 하나를 사용한다.

1. 간편하게 사용할 경우에는 **파이썬 컨테이너 타입(ex. `list`, `dict`)을 직접 상속**하여 구현한다.
2. 커스텀 컨테이너를 제대로 구현하려면 수많은 메서드를 구현해야 하므로 **`collection.abc`에 정의된 인터페이스를 상속**하여 구현한다.
    
    > 커스텀 컨테이너 타입이 정상적으로 작동하기 위해 필요한 **인터페이스**와 **기능**을 제대로 구현하도록 보장할 수 있다!
    {: .prompt-info}

<br>

그렇다면, 왜 커스텀 컨테이너를 제대로 구현하기 위해서는 `collection.abc`를 상속해야 하는지 **시퀀스 타입**을 예시로 알아보자!

<br>

## 파이썬 컨테이너 타입을 직접 상속받아 커스텀 컨테이터 타입 정의 시 주의할 점

1. 커스텀 컨테이너 타입을 정의할 때, 작성하는 클래스가 올바른 컨테이너 타입이 되려면 <span class="hl">**특정 메서드를 반드시 구현**</span>해야 한다.
    
    > 커스텀 시퀀스 타입의 경우, **특별 메서드 `__len__`과 `__getitem__`**을 구현해야 한다.
    
2. 필요한 특정 메서드를 모두 구현했더라도, 파이썬 프로그래머가 리스트 인스턴스에서 <span class="hl">**기대할 수 있는 모든 시퀀스의 의미 구조를 제공할 수 없을 수도**</span> 있다.
    
    > 커스텀 시퀀스 타입을 구현하기 위해 `__len__`과 `__getitem__`을 구현했더라도, 이것으로는 충분하지 않다. 파이썬 프로그래머가 시퀀스 타입에 있을 것이라고 예상하는 **`count`나 `index` 메서드**가 없기 때문이다!
    

<br>

이러한 이유로 파이썬 컨테이너 타입을 직접 상속받아 커스텀 컨테이너 타입을 직접 정의하는 것은 생각보다 훨씬 더 어렵다.

`collections.abc` 모듈을 사용한다면 이를 어떻게 해결할 수 있을까?

<br>

## 커스텀 컨테이너 타입 정의 시 `collections.abc` 모듈을 사용하는 이유

1. `collections.abc` 모듈은 <span class="hl">**추상 기반 클래스 정의**</span>를 제공한다.
    
    > 추상 기반 클래스는 **컨테이너 타입에 정의해야 하는 전형적인 메서드**를 모두 제공하므로, 하위 클래스 작성 시 이 중 필요한 메서드의 구현을 잊어버리면 **에러**를 통해 실수한 부분을 알 수 있다.
    
    
    ```python
    from collections.abc import Sequence
    
    class BadType(Sequence):
        pass
    ```
    
    ```python
    foo = BadType()
    # TypeError: Can't instantiate abstract class BadType with abstract methods __getitem__, __len__
    ```
    
2. 추상 기반 클래스가 요구하는 모든 메서드를 구현하면, <span class="hl">**그 외의 추가 메서드 구현을 상속 받아**</span> 자연스럽게 얻을 수 있다.
    
    > 커스텀 시퀀스 타입의 경우, 파이썬 프로그래머가 시퀀스 타입에 있을 것이라고 예상하는 **`count`나 `index` 메서드와 같은 추가 메서드**의 구현을 따로 하지 않아도 된다!
    > 
    
    ```python
    class BetterNode(Sequence):
        def __init__(self, value, left=None, right=None):
            self.value = value
            self.left = left
            self.right = right
        
        def _traverse(self):
            if self.left is not None:
                yield from self.left._traverse()
            yield self
            if self.right is not None:
                yield from self.right._traverse()
            
        def __getitem__(self, index):
            for i, item in enumerate(self._traverse()):
                if i == index:
                    return item.value
            raise IndexError(f"인덱스 범위 초과: {index}")
        
        def __len__(self):
            for count, _ in enumerate(self._traverse(), 1):
                pass
            return count
    ```
    
    ```python
    tree = BetterNode(
        10,
        left=BetterNode(
            5,
            left=BetterNode(2),
            right=BetterNode(
                6,
                right=BetterNode(7)
            )
        ),
        right=BetterNode(
            15,
            left=BetterNode(11)
        )
    )
    ```
    
    ```python
    print("트리:", list(tree))
    print("7의 인덱스:", tree.index(7))
    print("10의 개수:", tree.count(10))
    # 트리: [2, 5, 6, 7, 10, 11, 15]
    # 7의 인덱스: 3
    # 10의 개수: 1
    ```
    

<br>

따라서 `Set`이나 `MutableMapping`과 같이 파이썬의 관례에 맞춰 구현해야 하는 특별 메서드가 훨씬 많은 더 복잡한 컨테이너 타입을 구현할 때는 더욱 더 `collections.abc`의 이점이 커진다.
