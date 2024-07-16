---
title: "[Better Way #53] 블로킹 I/O의 경우 스레드를 사용하고 병렬성을 피하라"
date: 2023-12-29 09:50:00 +0900
categories: [Python, Effective Python]
tags: [python, effective python, ch07 concurrency and parallelism, thread, blocking io, singlethread, multithread, gil]
math: true
---

> 본문은 **"파이썬 코딩의 기술 (Effective Python, 2판)"**의 **"Chapter 07. Concurrency and Parallelism"**을 읽고 정리한 내용입니다.

<br>

## 1. 전역 인터프리터 락(Global Interpreter Lock, GIL)

파이썬의 표준 구현인 CPython은 두 단계를 거쳐 파이썬 프로그램을 실행한다.

1. 소스코드를 구문 분석하여 **바이트코드(bytecode)** 로 변환한다.
2. 바이트코드를 스택 기반 **인터프리터**를 통해 실행한다.

<br>

바이트코드 인터프리터에는 파이썬 프로그램이 실행되는 동안 일관성 있게 유지해야 하는 상태가 존재하는데, CPython은 이를 **전역 인터프리터 락(Global Interpreter Lock, GIL)** 을 이용하여 일관성을 강제로 유지한다.

- 근본적으로 GIL은 상호 배제 락(뮤텍스, mutex)이며, CPython이 **선점형 멀티스레드**로 인해 영향을 받는 것을 방지한다.
- 선점형 멀티스레드에서는 <span class="shl">한 스레드가 다른 스레드의 실행을 중간에 **인터럽트(interrupt)**</span>시키고 제어를 가져올 수 있다.
    
    > 이러한 인터럽트가 예기치 못한 때에 발생하면 인터프리터의 상태(ex. 가비지 컬렉터의 참조 카운터)가 일관성 있게 유지되지 못할 수 있다.
    
- GIL을 통해 CPython 자체와 CPython이 사용하는 C 확장 모듈이 실행되며 인터럽트가 함부로 발생하는 것을 방지한다.

<br>

따라서 다른 언어로 작성된 프로그램에서는 멀티스레드를 통해 프로그램이 여러 CPU 코어를 동시에 활용할 수 있지만, 파이썬의 경우에는 <span class="shl">**GIL로 인해 여러 스레드 중 어느 하나만 진행**</span>되게 된다.

파이썬에서 계산량이 많은 작업(ex. 약수 찾기)을 **싱글스레드**와 **멀티스레드**로 작성해보고, 실행 시간에 어떤 차이가 있는지 살펴보자.

<br>

### [1] 싱글스레드 프로그램

하나의 스레드에서 순차적으로 `factorize`를 실행한다.

```python
def factorize(number):
    for i in range(1, number + 1):
        if number % i == 0:
            yield i
```

```python
import time

numbers = [2139079, 1214759, 1516637, 1852285]
start = time.time()

for number in numbers:
    list(factorize(number))

end = time.time()
delta = end - start

print(f"총 {delta:.3f} 초 걸림")
# 총 0.195 초 걸림
```

<br>

### [2] 멀티스레드 프로그램

여러 스레드에서 `factorize`를 실행한다.

```python
from threading import Thread

class FactorizeThread(Thread):
    def __init__(self, number):
        super().__init__()
        self.number = number

    def run(self):
        self.factors = list(factorize(self.number))
```

```python
start = time.time()

threads = []
for number in numbers:
    thread = FactorizeThread(number)
    thread.start()
    threads.append(thread)

# 모든 스레드가 끝날 때까지 기다림
for thread in threads:
    thread.join()

end = time.time()
delta = end - start
print(f"총 {delta:.3f} 초 걸림")
# 총 0.205 초 걸림
```

하지만 오히려 싱글스레드 프로그램보다 시간이 더 오래 걸리는 것을 확인할 수 있다.

이는 GIL로 인해 **각 CPU 코어를 동시에 사용하지 못할** 뿐만 아니라, **lock 충돌 및 스케줄링 부가 비용**이 발생하기 때문이다.

<br>

## 2. 멀티스레드가 적합한 상황 🌟

CPython에서도 멀티코어를 활용할 수 있는 방법이 존재한다. 하지만 이는 `Thread` 클래스를 사용하지 않으며(`concurrent.futures`를 사용), 상당한 노력이 필요하다. 이러한 한계에도 불구하고 파이썬이 스레드를 지원하는 이유는 다음과 같은 것들이 있다.

1. 멀티스레드를 사용하면 <span class="shl">**프로그램이 동시에 여러 일을 하는 것처럼**</span> 보이게 만들기 쉽다.
    
    > GIL로 인해 한 순간에 스레드 중 하나만 앞으로 진행할 수 있음에도 불구하고, CPython이 **어느 정도 균일하게 각 스레드를 실행**시켜주기 때문에 멀티스레드를 통해 여러 함수를 동시에 실행할 수 있다.
    
2. <span class="shl">**blocking I/O**</span>를 다루기 위해서이다.
    
    > blocking I/O는 파이썬이 특정 <span class="blue">**시스템 콜**</span>을 사용할 때 일어난다. 시스템 콜을 통해 파이썬 프로그램은 **OS가 대신 외부 환경과 상호작용(ex. 파일 쓰기/읽기, 네트워크 통신, 디스플레이 장치 통신 등)하도록 요청**한다. 스레드를 사용하면 OS가 시스템 콜 요청에 응답하는 데 걸리는 시간 동안 **파이썬 프로그램이 다른 작업**을 할 수 있다.
    
    > (하나의 스레드 기준) 시스템 콜 → blocking I/O → GIL 릴리즈
    {: .prompt-info}

<br>

멀티스레드를 통해 성능 향상을 기대할 수 있는 **blocking I/O 상황**을 살펴보자!

<br>

### [1] 싱글스레드 프로그램

느린 시스템 콜인 `select`를 통해, 운영체제에게 0.1초 동안 block한 다음 제어를 돌려달라고 요청하는 상황을 가정하자.

```python
import select
import socket

def slow_systemcall():
    select.select([socket.socket()], [], [], 0.1)
```

이 시스템 콜을 싱글스레드에서 순차적으로 실행하면, 실행에 필요한 시간이 linear하게 증가하는 것을 확인할 수 있다.

```python
start = time.time()

for _ in range(5):
    slow_systemcall()

end = time.time()
delta = end - start
print(f"총 {delta:.3f} 초 걸림")
# 총 0.520 초 걸림
```

이 프로그램의 문제점은 `slow_systemcall`이 실행되는 동안 프로그램이 아무 진전을 할 수 없다는 것이다. 프로그램의 메인 스레드가 `select` 시스템 콜에 의해 blocked 되기 때문이다.

**blocking I/O**(ex. 헬리콥터에 신호 보내기)와 **계산 작업**(ex. 다음 이동 계산)을 동시에 수행해야 한다면, <span class="shl">**시스템 콜을 스레드로 옮기는 것**</span>을 고려해야 한다.

<br>

### [2] 멀티스레드 프로그램

다음 코드에서는 `slow_systemcall` 함수를 여러 스레드에서 따로따로 호출한다.

```python
start = time.time()

# blocking I/O -- 여러 직렬 포트 및 헬리콥터와 통신
threads = []
for _ in range(5):
    # 시스템 콜 스레드
    thread = Thread(target=slow_systemcall)
    thread.start()
    threads.append(thread)

# 계산 작업 -- 시스템 콜 스레드가 끝나기 전에 헬리콥터의 움직임 계산
def compute_helicopter_location(index): ...

for i in range(5):
    compute_helicopter_location(i)

for thread in threads:
    thread.join()

end = time.time()
delta = end - start
print(f"총 {delta:.3f} 초 걸림")
# 총 0.113 초 걸림
```

이와 같이 병렬화한 버전은 순차적으로 실행한 경우보다 시간이 1/5로 줄어든다.

<br>

> GIL은 파이썬 프로그램이 병렬로 실행되지 못하게 방지하지만, 시스템 콜에는 영향을 미칠 수 없다. 이는 <span class="ulr">**파이썬 스레드가 시스템 콜을 하기 전에 GIL을 릴리즈**</span>하고, <span class="ulr">**시스템 콜에서 반환되자마자 GIL을 다시 획득**</span>하기 때문이다.
{: .prompt-tip}
