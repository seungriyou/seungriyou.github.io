---
title: "[Better Way #54] 스레드에서 데이터 경합을 피하기 위해 Lock을 사용하라"
date: 2023-12-30 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch07 concurrency and parallelism, thread, multithread, gil, lock, mutex]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 07. Concurrency and Parallelism"**을 읽고 정리한 내용입니다.

<br>

GIL이 동시 접근을 보장해주는 락 역할을 해주는 것처럼 보이지만, 실제로는 그렇지 않다.

파이썬 스레드는 GIL로 인해 한 순간에 단 하나만 실행될 수 있으나, <span class="shl">파이썬 인터프리터에서 어떤 스레드가 데이터 구조에 대해 수행하는 연산은 **연속된 두 바이트코드 사이에서 언제든 인터럽트**될 수 있기 때문</span>이다. 따라서 여러 스레드가 같은 데이터 구조에 동시에 접근하면 위험하다.

<br>

## 1. Race Condition 발생 상황

센서 네트워크에서 광센서를 통해 빛이 들어온 경우를 샘플링하는 예시를 살펴보자. 시간이 지나면서 빛이 들어온 횟수를 병렬적으로 모두 세고 싶다면 다음과 같은 클래스를 사용해 셀 수 있다.

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self, offset):
        self.count += offset
```

센서를 읽을 때는 blocking I/O를 수행하므로 센서마다 작업자 스레드를 할당한다고 하자.

```python
def worker(sensor_index, how_many, counter):
    for _ in range(how_many):
        # 센서를 읽는다
        ...
        counter.increment(1)
```

다음과 같이 병렬로 센서마다 하나씩 `worker` 스레드를 실행하고, 모든 스레드가 값을 다 읽을 때까지 기다린다.

> 실행 시 카운터 값이 500000이 나올 수가 있으나, `found` 값은 일반적으로 500000 보다 작은 값이 나오게 된다.
> 

```python
from threading import Thread

how_many = 10**5
counter = Counter()

threads = []
for i in range(5):
    thread = Thread(target=worker, args=(i, how_many, counter))
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

expected = how_many * 5
found = counter.count
print(f"카운터 값은 {expected}여야 하는데, 실제로는 {found} 입니다.")
# 카운터 값은 500000여야 하는데, 실제로는 481384 입니다.
```

<br>

> 이러한 현상은 파이썬 인터프리터가 <span class="shl">실행되는 **모든 스레드의 실행 시간을 거의 비슷하게** 만들기 위해 실행 중인 스레드를 일시 중단시키고 다른 스레드를 실행시키는 일(**인터럽트**)을 반복</span>하기 때문에 발생한다.
{: .prompt-danger}

<br>

## 2. 파이썬 인터프리터의 동작

프로그래머는 파이썬이 스레드를 언제 일시 중단시킬지 알 수 없다. 심지어 <span class="shl">**원자적(atomic)**인 것처럼 보이는 연산</span>을 수행하는 도중에도 파이썬이 스레드를 일시 중단시킬 수도 있다.

<br>

예를 들어, `Counter` 객체의 `increment` 메서드 내 동작인 `counter.count += 1`은 간단해 보인다. 하지만 어떤 객체의 애트리뷰트에 대한 `+=` 연산자는 실제로 다음의 세 가지 연산으로 이루어져 있다.

```python
value = getattr(counter, "count")
result = value + 1
setattr(counter, "count", result)
```

즉, `increment` 동작을 수행하는 파이썬 스레드는 세 연산 사이에서 일시 중단될 수 있으며, 스레드 간 연산 순서가 뒤섞이며 `value`의 이전 값을 대입하는 경우가 발생할 수 있다.

```python
# 스레드 A
value_a = getattr(counter, "count")
# 인터럽트로 인해 스레드 B로 컨텍스트 전환
value_b = getattr(counter, "count")
result_b = value_b + 1
setattr(counter, "count", result_b)
# 다시 스레드 A로 컨텍스트 전환 (스레드 B가 카운터를 증가시켰던 결과가 사라짐)
result_a = value_a + 1
setattr(counter, "count", result_a)
```

<br>

> 이와 같은 경우를 해결하기 위해 파이썬은 `threading` 내장 모듈에서 여러 도구를 제공하며, 가장 간단한 도구로 **`Lock` 클래스**가 있다.
{: .prompt-tip}

<br>

## 3. `threading.Lock` 클래스

`threading.Lock` 클래스는 **상호 배제 락(뮤텍스)**이며, 이를 통해 `Counter` 클래스가 여러 스레드의 동시 접근으로부터 자신의 현재 값을 보호할 수 있다.

한 번에 단 하나의 스레드만 락을 획득할 수 있으며, 다음과 같이 `with` 문을 사용할 수 있다.

```python
from threading import Lock

class LockingCounter:
    def __init__(self):
        self.lock = Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset
```

이전과 같이 `worker` 스레드를 사용하되 `LockingCounter`를 사용할 수 있다.

```python
counter = LockingCounter()

threads = []
for i in range(5):
    thread = Thread(target=worker, args=(i, how_many, counter))
    threads.append(thread)
    thread.start()

for thread in threads:
    thread.join()

expected = how_many * 5
found = counter.count
print(f"카운터 값은 {expected}여야 하는데, 실제로는 {found} 입니다.")
# 카운터 값은 500000여야 하는데, 실제로는 500000 입니다.
```

<br>

이제는 예상과 실행 결과가 들어맞는 것을 확인할 수 있다.
