---
title: "[Better Way #35] 제너레이터 안에서 throw로 상태를 변화시키지 말라"
date: 2023-07-09 22:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch04 comprehensions and generators]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 04. Comprehensions and Generators"**을 읽고 정리한 내용입니다.

<br>

## `throw` 메서드

- `throw` 메서드를 통해 제너레이터가 마지막으로 실행한 `yield` 식의 위치에서 예외를 다시 발생시킬 수 있다.
- `throw`를 사용하는 방법은 가독성을 해친다.

```python
def timer(period):
    current = period
    while current:
        current -= 1
        try:
            yield current
        except Reset:
            current = period

def run():
    it = timer(4)
    while True:
        try:
            if check_for_reset():
                current = it.throw(Reset())
            else:
                current = next(it)
        except StopIteration:
            break
        else:
            announce(current)

run()
```

<br>

## `throw` 메서드를 사용하지 않고 예외적인 동작을 제공하는 방법

- **이터러블 컨테이너 객체(`__iter__` 메서드 구현)**를 사용해 **상태가 있는 클로저를 정의**하는 것이다. (예외적인 경우에 상태 전이)
- 제너레이터와 예외를 함께 사용해야 한다면, 비동기 기능을 사용할 때 더 좋게 구현할 수 있는 경우도 많다.

```python
class Timer:
    def __init__(self, period):
        self.current = period
        self.period = period
    
    def reset(self):
        self.current = self.period
    
    def __iter__(self):
        while self.current:
            self.current -= 1
            yield self.current

def run():
    timer = Timer(4)
    for current in timer:
        if check_for_reset():
            timer.reset()
        announce(current)

run()
```
