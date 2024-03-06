---
title: "[Python] 동시성(Concurrency): 쓰레드, 프로세스, 코루틴 (+ Spinner 예제)"
date: 2024-01-30 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, fluent python, concurrency, thread, process, coroutine]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

## 1. 동시성(Concurrency) vs. 병렬성(Parallelism)

동시성과 병렬성의 차이점을 표로 정리하면 다음과 같다.

| 구분 | 초점 | 무엇에 관한 것인지 |
| --- | --- | --- |
| 동시성 (concurrency) | **“dealing”** with lots of things at once | 구조   |
| 병렬성 (parallelism) | **“doing”** lots of things at once  | 실행   |

- **병렬성(parallelism)**은 **동시성(concurrency)에 포함**된다.
    
    > 즉, 모든 parallel system은 concurrent 하나, 모든 concurrent system이 parallel 한 것은 아니다.
    > 
    
    ![](/assets/img/posts/Python/Fluent-Python/2024-01-30-01.png){: .w-75}
    _ref: <https://en.wikipedia.org/wiki/File:Parallel-concurrent.png>_
    
- 대부분의 컴퓨팅 연산은 parallel 하지 않고 **concurrent** 하다.
    
    > 200개의 작업을 **parallel** 하게 수행하려면 200개의 CPU 코어가 필요하다.
    > 
    > 
    > 4개의 CPU 코어만 가지고 200개의 작업을 수행하는 것을 **concurrent** 하다고 한다.
    > 

- 파이썬의 **concurrent programming 관련 패키지**에는 `threading`, `multiprocessing`, `asyncio`가 있다.
    
    각 패키지에서 **동시성(concurrency)을 달성하는 수단**은 다음과 같다.
    
    | 패키지 이름     | 동시성 달성 수단                    |
    | --------------- | ----------------------------------- |
    | `threading`       | 쓰레드 (threads)                    |
    | `multiprocessing` | 프로세스 (processes)                |
    | `asyncio`         | 네이티브 코루틴 (native coroutines) |

<br>

### 관련 용어 정리

1. <span class="shlp">**동시성(concurrency)**</span>: the ability to “handle” multiple pending tasks
    - 태스크가 **(1) 한 번에 하나씩**, 혹은 **(2) 가능하다면 병렬적**으로 수행되도록 한다.
    - 싱글 코어 CPU는 OS 스케줄러가 multi-tasking을 지원(= pending task의 실행을 교차 배치)하는 경우, 동시성을 지원한다고 할 수 있다.

2. <span class="shlp">**병렬성(parallelism)**</span>: the ability to “execute” multiple computations at the same time
    - 이를 달성하기 위해서는 멀티 코어 GPU, 다중 CPUs, GPU, 다중 클러스터 등이 필요하다.

3. <span class="shlp">**실행 단위(execution unit)**</span>: general term for objects that execute code concurrently
    - 파이썬은 세 가지 실행 단위를 가진다.
        1. 쓰레드(thread)
        2. 프로세스(process)
        3. 코루틴(coroutine)
    - 각각은 독립적인 **상태(state)**와 **콜 스택(call stack)**을 가진다.

4. <span class="shlp">**프로세스(process)**</span>: an instance of a computer program while it is running
    - 메모리와 CPU 시간의 일부를 가진다.
    - 각 프로세스는 자신만의 메모리 공간을 가진다.
    - 프로세스 간 통신을 위해서는 파이프(pipe), 소켓(socket), 메모리 맵 파일(memory mapped file) 등을 사용해야 하나, 이는 **raw bytes만 전달**할 수 있다. 따라서 파이썬 객체가 전달되려면 raw bytes로 serialized 되어야 하는데, 이는 **(1) 비용이 많이 드는 작업**이며 **(2) 모든 파이썬 객체가 serializable 한 것도 아니다**.
    - 프로세스는 자식 프로세스를 생성하며, 이는 마찬가지로 부모 프로세스에 대해 완전히 독립적이다.
    - OS 스케줄러의 감독 하에 **선점형 멀티태스킹(preemptive multitasking)**을 지원한다.
        
        > OS 스케줄러가 선점한다는 것은, 주기적으로 다른 프로세스가 실행될 수 있도록 한다는 것이다. 따라서 이론적으로, frozen 프로세스가 전체 시스템을 freeze 할 수는 없다.
        > 
    - 멀티 프로세싱(multi-processing)을 활용하면 다음과 같은 구조가 되며, 이는 `multiprocessing` 모듈을 통해 구현한다. 멀티 코어 CPU라면 병렬(parallel) 수행이 가능하고, 싱글 코어 CPU라면 time slicing을 통해 동시(concurrent) 수행이 가능하다.
        
        ![](/assets/img/posts/Python/Fluent-Python/2024-01-30-02.png){: .w-75}
        
5. <span class="shlp">**쓰레드(thread)**</span>: an execution unit within a single process
    - 프로세스가 실행을 시작하면, 싱글 쓰레드인 메인 쓰레드를 사용한다.
    - 프로세스는 여러 개의 쓰레드를 생성할 수 있다.
    - 하나의 프로세스 내의 쓰레드는 **같은 메모리 공간을 공유**한다.
        - **[장점]** 쓰레드 간에 파이썬 객체를 주고 받을 수 있다.
        - **[단점]** 여러 쓰레드가 같은 객체를 동시에(concurrently) 업데이트하면 데이터가 corrupt 될 수 있다.
    - OS 스케줄러의 감독 하에 **선점형 멀티태스킹**을 지원한다.
    - 같은 작업을 수행할 때, 프로세스보다 자원을 더 적게 사용한다.
    - 멀티 쓰레딩(multi-threading)을 활용하면 다음과 같은 구조가 되며, 이는 `multithreading` 모듈을 통해 구현한다. 마찬가지로 멀티 코어 CPU라면 병렬(parallel) 수행이 가능하고, 싱글 코어 CPU라면 time slicing을 통해 동시(concurrent) 수행이 가능하다.
        
        ![](/assets/img/posts/Python/Fluent-Python/2024-01-30-03.png){: .w-75}
        
6. <span class="shlp">**코루틴(coroutine)**</span>: a function that can suspend itself and resume later
    - 코루틴의 종류에는 다음의 두 가지가 있으며, 여기에서 다루는 코루틴은 네이티브 코루틴을 의미한다.
        
        
        | 분류            | 구현            |
        | --------------- | --------------- |
        | 클래식 코루틴   | 제너레이터 함수 |
        | 네이티브 코루틴 | `async def`       |

    - 코루틴은 주로 **이벤트 루프(event loop)**의 감독 하에 실행되며, 이는 **모두 같은 싱글 쓰레드에 존재**한다.
    - **비선점형 멀티태스킹(cooperative multitasking)**을 지원한다.
        - 각 코루틴은 **`yield`나 `await` 키워드**를 통해 명시적으로 제어를 양도해야 한다.
            
            > → 다른 코루틴이 **concurrently** 하게 (parallel X) 진행된다.
            > 
        - 즉, 코루틴 내의 blocking 코드는 이벤트 루프 및 다른 모든 코루틴의 실행을 block 하게 된다.
            
            > ↔ 선점형 멀티태스킹
            > 
    - 같은 작업을 수행할 때, 프로세스 및 쓰레드보다 자원을 더 적게 사용한다.

7. <span class="shlp">**큐(queue)**</span>: a data structure that lets us put and get items, usually in FIFO order
    - 분리된 실행 단위 간에 애플리케이션 데이터나 컨트롤 메시지(ex. 에러 코드, 종료 신호)를 교환하도록 지원한다.
        
        
        | 실행 단위        | 큐를 지원하는 패키지                  |
        | ---------------- | ------------------------------------- |
        | 쓰레드           | `queue` 패키지 (파이썬 표준 라이브러리) |
        | 프로세스, 코루틴 | `multiprocessing`, `asyncio` 패키지       |
    - `queue`와 `asyncio` 패키지에서는 FIFO가 아닌 queue도 지원한다. (ex. `LifoQueue`, `PriorityQueue`)

8. <span class="shlp">**락(lock)**</span>: an object that execution units can use to (1) synchronize their actions and (2) avoid corrupting data
    - 어떤 코드가 공유 자료 구조를 업데이트 중이라면, 관련된 락을 가지고 있어야 한다.
    - 이는 다른 부분들에게 “동일한 자료 구조에 접근하려면 락이 release 될 때까지 기다리라”는 신호를 보낸다.
    - 가장 간단한 락은 `mutex`이며, 동시성 모델에 따라 락의 구현은 달라진다.

9. <span class="shlp">**경합(contention)**</span>: dispute over a limited asset
    - **자원 경합**(resource contention)은 여러 실행 단위가 동일한 공유 자원에 접근하려 할 때 발생한다.
    - **CPU 경합**(CPU contenction)은 OS 스케줄러로부터 CPU 시간을 할당받기 위해 compute-intensive 프로세스/쓰레드들이 대기해야 할 때 발생한다.

<br>

## 2. 쓰레드(Threads) & 프로세스(Processes) vs. 코루틴(Coroutines)

> 여기서 “코루틴”이란 “네이티브 코루틴”을 의미한다.
> 

<span class="shl">**쓰레드와 프로세스의 단점**</span>에는 다음과 같은 것들이 있다.

1. 이들을 시작하는 것은 쉽지만, 언제 실행이 완료됐는지 tracking 할 수 없다.
2. 실행의 결과 값이나 에러를 얻으려면 별도의 message queue와 같은 communication channel을 이용해야 한다.
3. 이들을 실행할 때 비용이 많이 든다.
    
    > startup cost를 amortize 하려면 이들을 worker로 만들 수도 있다. 하지만 어떻게 worker를 중단할 것인지, 안전하게 작업을 중단하려면 어떻게 해야하는지(ex. 메시지, 큐) 등과 같이 고려해야 할 점이 많아진다.
    > 

<br>

그에 반해, <span class="shl">**코루틴의 장점**</span>은 다음과 같다.

1. 실행할 때 비용이 적게 든다.
2. `await` 키워드를 통해 코루틴을 실행하면, 그로부터 반환되는 값을 받는 것이 쉽다.
3. 안전하게 작업을 중단할 수 있고, exception을 처리하기 쉽다.

하지만 코루틴이 비동기 프레임워크에 의해 시작되는 경우, 쓰레드와 프로세스처럼 monitor 하기 어렵다는 **단점**도 존재한다. 또한, **GIL**로 인해 <span class="blue">쓰레드와 코루틴은 CPU-intensive 작업에는 적합하지 않다</span>.

<br>

### GIL(Global Interpreter Lock) 관련 정리

1. 각 **파이썬 인터프리터 인스턴스**는 <span class="red">**하나의 프로세스**</span>이다.
    
    > **파이썬 인터프리터(Python Interpreter)**
    >
    > 파이썬으로 작성된 코드를 한 줄씩 읽으면서 실행하는 프로그램
    {: .prompt-info}
    
    - `multiprocessing`, `concurrent.futures` 라이브러리를 통해 추가적인 파이썬 프로세스를 실행할 수 있다.
    - `subprocess` 라이브러리를 통해 언어와 상관 없이 외부 프로그램을 수행하는 프로세스를 실행할 수 있다.

2. **파이썬 인터프리터**는 사용자 프로그램과 메모리 가비지 컬렉터를 실행하기 위해 <span class="red">**싱글 쓰레드**</span>를 사용한다.
    - `threading`, `concurrent.futures` 라이브러리를 통해 추가적인 파이썬 쓰레드를 실행할 수 있다.

3. 객체 참조를 포함한 **내부 인터프리터 상태에 대한 접근**은 **GIL(Global Interpreter Lock)**이라는 락에 의해 제어된다.
    - CPU 코어 개수와 상관 없이, **어떤 시점에 하나의 파이썬 쓰레드만이 GIL**을 가질 수 있다. 즉, 어떤 시점에 **하나의 파이썬 쓰레드만이** 파이썬 코드를 실행할 수 있다.

4. 파이썬 쓰레드가 GIL을 독점하는 것을 방지하기 위해, 파이썬 인터프리터는 디폴트로 **현재의 파이썬 쓰레드를 5ms마다 중지**하여 **GIL을 release** 한다.
    - 여러 쓰레드가 GIL을 기다리고 있다면, OS 스케줄러가 그 중 하나를 선택한다.
    - 프로그래머는 GIL에 대한 권한이 없다.

5. **시스템 콜을 호출**하는 모든 파이썬 표준 라이브러리 함수는 **GIL을 release** 한다.
    - ex) 디스크 입출력, 네트워크 입출력, `time.sleep()` 등

6. 파이썬/C API 레벨에 통합된 extension을 통해 **non-Python인, GIL-free 쓰레드**를 생성할 수 있다.
    - 이들은 파이썬 객체를 변경할 수는 없으나, 버퍼 프로토콜을 지원하는 객체(ex. `numpy.array`)의 메모리 단에서 read/write를 수행할 수 있다.

7. GIL에 대한 contention은 **compute-intensive 파이썬 쓰레드를 느리게** 한다.
    - compute-intensive 쓰레드를 다룰 때에는 **sequential & single-threaded 코드**가 더 간단하고 빠르다.
    - 이는 GIL로 인해 어떤 시점에 단 하나의 쓰레드만 파이썬 코드를 실행할 수 있기 때문이다.

8. **멀티 코어**에서 CPU-intensive 파이썬 코드를 병렬로 실행하려면, **멀티 프로세스**를 사용해야 한다.

<br>

> **코루틴**의 경우, **이벤트 루프**와 **모든 코루틴**이 **싱글 쓰레드에 존재**하므로 <span class="red">**GIL의 영향을 받지 않는다**</span>.
>
> 멀티 쓰레드를 사용해도 되지만, 베스트 프랙티스는 **하나의 쓰레드에서 이벤트 루프와 모든 코루틴이 실행**되는 것이다.
{: .prompt-info}

<br>

## 3. 동시성 프로그래밍 세계의 Hello World: <span class="pink">Spinner</span> 구현

다음과 같은 세 가지의 패키지를 이용하여 동시성 프로그램에서의 hello world에 해당하는 spinner를 구현해보자.

| 패키지 이름     | 동시성 달성 수단                    |
| --------------- | ----------------------------------- |
| `threading`       | 쓰레드 (threads)                    |
| `multiprocessing` | 프로세스 (processes)                |
| `asyncio`         | 네이티브 코루틴 (native coroutines) |

spinner는 3초 동안 돌게 되며, 실행 예시는 다음과 같다.

![](/assets/img/posts/Python/Fluent-Python/2024-01-30-04.gif){: .w-75}
_[3] 코루틴을 이용한 Spinner (`asyncio`)의 실행 결과_

<br>

### [1] 쓰레드를 이용한 Spinner (`threading`)

```python
import itertools
import time
from threading import Thread, Event

# spin()은 새로운 thread `spinner`에서 수행된다.
# threading.Event instance인 done을 이용하여 thread 간 signal로 사용한다.
def spin(msg: str, done: Event) -> None:
    for char in itertools.cycle(r'\|/-'):
        status = f'\r{char} {msg}'
        print(status, end='', flush=True)

        # Event.wait(timeout=s)의 경우, timeout이 경과하면 False, 다른 thread에 의해 Event.set() 되면 True를 반환한다.
        # 이때, 다른 thread가 Event.set()을 호출하여 True를 반환하기 전까지는 blocked 된다.
        if done.wait(.1):
            break   # Event.set()이 호출되면 무한 루프를 종료한다.

    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')

# slow()는 main thread에 의해 수행된다.
def slow() -> int:
    # time.sleep()으로 인해 호출처인 main thread가 blocked 되며, GIL이 released 된다.
    # 따라서 다른 thread인 spinner가 진행된다.
    time.sleep(3)
    return 42

def supervisor() -> int:
    # main thread가 done Event를 set 하면, spinner thread가 바로 인지하고 exit 한다.
    done = Event()

    # 새로운 thread 생성 시, target으로 함수를, args로 해당 함수의 argument를 전달한다.
    spinner = Thread(target=spin, args=('thinking!', done))

    # <Thread(Thread-1 (spin), initial)>를 출력한다.
    # initial: thread의 상태로, 아직 시작하지 않았음을 뜻한다.
    print(f'spinner object: {spinner}')

    # spinner thread를 시작한다.
    spinner.start()

    # slow()를 호출함으로써 main thread가 blocked 된다.
    # 그 동안 spinner thread가 동작한다.
    result = slow()

    # time.sleep(3) -> 3초가 지나면 main thread가 blocked 상태에서 벗어나 Event를 set하여
    # spinner thread의 for loop을 break 시킨다.
    done.set()

    # spinner thread가 종료할 때까지 대기한다.
    spinner.join()

    # result는 slow()의 return 값이 된다.
    return result

def main() -> None:
    result = supervisor()
    print(f'Answer: {result}')
    

if __name__ == '__main__':
    main()
```

우선, **`supervisor()`에서 `slow()`를 호출할 때의 흐름**을 살펴보자.

1. 메인 쓰레드에서 `slow()`를 호출한다.
2. `slow()` 내 `time.sleep(e)`으로 인해 호출처인 메인 쓰레드가 3초간 blocked 되며, GIL이 릴리즈 된다.
3. 메인 쓰레드가 blocked 되어 있는 동안 다른 쓰레드인 spinner 쓰레드가 실행된다.
4. 메인 쓰레드의 `time.sleep(3)`에 설정했던 3초가 모두 지나면, OS 스케줄러가 해당 메인 쓰레드의 작업이 완료되었음을 인지하고 다시 작업을 재개하기 위해 스케줄한다.

이러한 흐름으로 `slow()`의 실행이 완료되면, 이어서 `done.set()`을 실행하게 된다.

<br>

따라서 `spin`과 `slow`는 concurrent 하게 실행된다고 할 수 있다.

| 함수 | 호출하는 쓰레드                            |
| ---- | ------------------------------------------ |
| `spin` | 새로운 spinner 쓰레드 (메인 쓰레드가 생성) |
| `slow` | 메인 쓰레드                                |

<br>

이어서 **`threading.Event` 클래스**에 대해 살펴보자.

`Event`는 파이썬에서 쓰레드 간 협력에 사용되는 신호 메커니즘 중 가장 간단한 형태이다. `Event` 인스턴스는 내부에 boolean flag 값을 가지는데, 그 값은 다음과 같이 설정된다.

| 시점                | Event 인스턴스 내 boolean flag 값 |
| ------------------- | --------------------------------- |
| 생성 및 시작 시     | `False`                             |
| `Event.set()` 호출 시 | `True`                              |

`Event` 관련 메서드 및 동작을 살펴보면 다음과 같다.

1. `Event` 인스턴스를 생성하면 boolean flag 값이 `False`로 설정된다.
2. 어떤 쓰레드가 `Event.wait(timeout=s)`를 호출하면, **(1) boolean flag 값이 `False`**이며 **(2) (주어진 경우) `timeout`이 끝나지 않을 때까지** 해당 쓰레드가 **block** 된다.
3. 다른 쓰레드가 `Event.set()`을 호출하게 되면 boolean flag 값이 `True`로 바뀌게 된다.
4. `Event.wait(timeout=s)`의 반환 값을 정리하면 다음과 같다.
    
    
    | 반환 값 | 상황 |
    | --- | --- |
    | `False` | `timeout이` 완료된 경우<br>(`timeout` 내에 flag 값이 `True`가 되지 못 함) |
    | `True` | 다른 쓰레드에 의해 `Event.set()` 된 경우<br>(flag 값이 `True`가 됨) |

> **`main` 쓰레드**가 **`done` event 인스턴스를 `.set()`** 하면, **`spinner` 쓰레드**가 바로 인지하고 exit 한다.
>
> → 파이썬에서는 **쓰레드를 종료**하는 API가 없기 때문에, **종료하기 위해 메시지**를 보내야 한다. 이때, 메시지는 **event 인스턴스를 set하는 것**이라고 볼 수 있다!
{: .prompt-info}

<br>

### [2] 프로세스를 이용한 Spinner (`multiprocessing`)

```python
import itertools
import time
from multiprocessing import Process, Event

# multiprocessing.Event는 class가 아닌 function이므로 Mypy type hint로 사용할 수 없다.
# 해당 function은 synchronize.Event instance를 반환하므로, synchronize를 import 하여 type hint로 사용한다.
from multiprocessing import synchronize

# done의 type hint만 변경되었으며, 나머지 동작은 thread 예제와 동일하다.
def spin(msg: str, done: synchronize.Event) -> None:
    for char in itertools.cycle(r'\|/-'):
        status = f'\r{char} {msg}'
        print(status, end='', flush=True)
        if done.wait(.1):
            break

    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')

def slow() -> int:
    time.sleep(3)
    return 42

def supervisor() -> int:
    done = Event()

    # Process의 사용법은 Thread와 유사하다.
    **spinner = Process(target=spin, args=('thinking!', done))**

    # <Process name='Process-1' parent=91303 initial>를 출력한다.
    # 이때, parent는 해당 코드를 실행하는 Python instance의 PID이다.
    print(f'spinner object: {spinner}')

    spinner.start()

    result = slow()

    done.set()

    spinner.join()

    return result

def main() -> None:
    result = supervisor()
    print(f'Answer: {result}')
    

if __name__ == '__main__':
    main()
```

우선, `multiprocessing` 패키지는 다음과 같은 특징을 가진다.

- concurrent 작업들을 **별도의 파이썬 프로세스**들에서 수행한다.
- `multiprocessing.Process` 인스턴스를 생성하면, 새로운 파이썬 인터프리터가 백그라운드에서 자식 프로세스로 실행된다.
- 각 <span class="blue">**파이썬 프로세스는 자신만의 GIL**</span>을 가지므로, **가능한 모든 CPU 코어를 사용**할 수 있게 된다. 다만, OS 스케줄러에 의존하게 된다.
- `multiprocessing.Event`는 클래스가 아닌 함수이므로 mypy 타입 힌트로 사용할 수 없다. 대신, 해당 함수는 `synchronize.Event` 인스턴스를 반환하므로 이것을 사용한다.

<br>

코드를 살펴보면 기본 API는 `threading`과 `multiprocessing`이 비슷해보이지만, 쓰레드를 프로세스로 변경함으로써 **프로세스 간에 파이썬 객체를 공유하지 못하게 되므로** 구현 복잡도가 올라가게 된다.

- 프로세스 경계를 넘나드는 객체들은 **serialized / deserialized** 되어야 한다.
- 현재 코드에서는 **`Event` state**가 해당되며, 이는 low-level OS 세마포어로 구현되어 있다.

<br>

### [3] 코루틴을 이용한 Spinner (`asyncio`)

```python
import itertools
import asyncio

# async def로 정의된 것은 모두 native coroutine이다.

# 이전 예제들과 다르게 Event instance가 필요하지 않다.
async def spin(msg: str) -> None:
    for char in itertools.cycle(r'\|/-'):
        status = f'\r{char} {msg}'
        print(status, end='', flush=True)

        try:
            # 다른 coroutine을 blocking 하지 않으면서 pause 하기 위해서
            # time.sleep() 대신 await asyncio.sleep()을 사용한다.
            await asyncio.sleep(.1)

        except **asyncio.CancelledError**:
            # Task.cancel() method가 호출되면 CancelledError가 발생한다.
            break

    blanks = ' ' * len(status)
    print(f'\r{blanks}\r', end='')

async def slow() -> int:
    # 마찬가지로, time.sleep() 대신 await asyncio.sleep()을 사용한다.
    await asyncio.sleep(3)
    return 42

async def supervisor() -> int:
    # (1) spin()의 eventual execution을 schedule 한다.
    # (2) asyncio.Task instance를 반환한다.
    spinner = asyncio.create_task(spin('thinking!'))

    # <Task pending name='Task-2' coro=<spin() running at /path/to/spinner_async.py:5>>를 출력한다.
		# 현재 spinner Task는 pending 상태이다.
    print(f'spinner object: {spinner}')

    # await 키워드를 통해 호출하였으므로,
    # slow()가 return 할 때까지 현재 coroutine인 supervisor를 block 한다.
    result = await slow()

    # Task.cancel() method를 통해 해당 spin() coroutine 내에서
    # CancelledError exception을 raise 한다.
    spinner.cancel()

    return result

# main()은 유일한 regular function이다.
def main() -> None:
    # (1) event loop를 시작하여 coroutine을 실행한다.
    # (2) supervisor() return 할 때까지 main은 blocked 된다.
    result = asyncio.run(supervisor())
    print(f'Answer: {result}')
    

if __name__ == '__main__':
    main()
```

쓰레드, 프로세스와 코루틴의 대표적인 차이점은 <span class="shl">**“관리 주체가 누구냐”**</span>는 것이다.

| 실행 단위 | 관리 주체 |
| ---- | ---- |
| 쓰레드, 프로세스 | **OS 스케줄러**가 CPU 시간을 할당한다. |
| 코루틴  | **application-level 이벤트 루프(event loop)**가 pending coroutines로<br>이루어진 queue를 관리하며, 하나씩 실행한다. |

<br>

**이벤트 루프, 라이브러리 코루틴, 사용자 코루틴**은 모두 <span class="red">**하나의 쓰레드(single thread)에서 실행**</span>된다는 점에 주의하자. 따라서 어떤 코루틴에서 시간이 소요된다면 이는 이벤트 루프 및 전체 코루틴을 느리게 만든다. 또한, 하나의 쓰레드에서 실행되므로 실행 단위 간 시그널링을 위한 객체를 필요로 하지 않기 때문에, `spin` 함수에 `slow`의 작업이 끝났음을 알리기 위한 `Event` 인자가 필요하지 않다.

<br>

네이티브 코루틴(native coroutine)은 `async def`로 정의된다. 이러한 <span class="shlp">**코루틴을 실행하는 방법**</span>에는 세 가지가 있다.

1. **`asyncio.run(coro())`**
    
    | --- | --- |
    | **호출처** | 일반적인 함수 (ex. `main()`) |
    | **동작**   | 주로 모든 비동기 코드의 **entrypoint가** 되는 코루틴 객체(ex. `supervisor()`)를 실행한다. |
    | **설명**   | - `coro()`가 반환할 때까지 **호출처는 block** 된다.<br>- 반환값은 **`coro()`의 반환값**이다.<br>- 현재의 쓰레드에 **새로운 이벤트 루프(event loop)**를 설정하고, 해당 이벤트 루프에서 인자로 넘어오는<br>**코루틴을 task로 예약**하여 **실행**한 뒤, 해당 task의 실행이 완료되면 이벤트 루프를 닫는다. |
    
    > **이벤트 루프(event loop)**
    >
    > 무한 루프를 돌며 매 루프마다 작업(= task)을 하나씩 실행시키는 로직이다.
    {: .prompt-info}
    
2. **`asyncio.create_task(coro())`**
    
    | --- | --- |
    | **호출처** | 코루틴                                  |
    | **동작**   | 언젠가는 실행될 **다른 코루틴을 예약**한다. |
    | **설명**   | - 현재 코루틴을 **suspend 하지 않는다**.<br>- **`Task` 인스턴스**를 반환한다.<br>- 해당 task의 실행이 **event loop에 의해 즉시 예약**된다. (즉시 실행 X)<br>- `asyncio.run()` 이후, **task를 추가로 생성하여 실행**함으로써 **concurrency를** 달성하기 위해 사용한다. |
    
    > **태스크(task)**
    >
    >`Task` 인스턴스에 해당하며, 하나의 코루틴에서부터 출발하는 하나의 실행 흐름이다.
    {: .prompt-info}
    
3. **`await coro()`**
    
    | --- | --- |
    | **호출처** | 코루틴                                                 |
    | **동작**   | `coro()`에 의해 반환되는 코루틴 객체에게 제어를 넘긴다.  |
    | **설명**   | - 현재 코루틴을 **suspend** 한다. (`coro()`가 반환할 때까지)<br>- `await` expression의 값은 **`coro()`의 반환값**이다.<br>- 코루틴 내에서만 사용할 수 있다. |
    
    > **코루틴 체인(coroutine chain)**
    >
    > `await`는 코루틴 내에서만 사용할 수 있으며, 이를 통해 다른 코루틴을 호출하며 코루틴 체인을 형성한다.
    {: .prompt-info}
    

<br>

> 그냥 **`coro()`를 호출**하는 것은 곧바로 **코루틴 객체를 반환**하기는 하지만, `coro` 함수의 body를 **실행하지는 않는다**. 코루틴의 body를 실행하는 것은 **이벤트 루프**의 역할이다!
{: .prompt-tip}

<br>

그런데 왜 코루틴을 사용하는 코드에서는 `slow()`에서 **`time.sleep(3)` 대신 `await asyncio.sleep(3)`**을 사용할까?

다음과 같이 `time.sleep(3)`을 사용한다고 가정해보자.

```python
async def slow() -> int:
    time.sleep(3)
    return 42
```

이렇게 `slow()`를 변경하면, spinner가 보이지 않는 채로 3초가 흐른 후 프로그램이 종료되게 된다. 이는 명시적으로 추가 쓰레드나 프로세스를 시작하지 않은 이상, **`asyncio`가 오직 하나의 실행 흐름**을 가지기 때문이다.

> `asyncio`에서 concurrency는 하나의 코루틴에서 다른 하나로 제어가 전달되며 달성된다!
> 

이때의 동작 흐름을 살펴보면 다음과 같다.

1. `time.sleep(3)`으로 인해 유일한 쓰레드인 메인 쓰레드가 blocked 되기 때문에 spinner가 동작하지 않는다.
2. 3초가 지나면 메인 쓰레드가 unblocked 되며, `slow()`가 return 하게 된다.
3. 곧바로 다음 코드를 실행하므로 `spinner` task를 `cancel`한다. 따라서 제어 흐름이 `spin` 코루틴의 body에 도달하지 못한다.

<br>

> **`asyncio` 코루틴**에서 **절대 `time.sleep(s)`을 사용하지 말아야 한다**. 유일한 쓰레드인 **메인 쓰레드가 blocked** 되므로 전체 프로그램이 중지되기 때문이다.
>
> 만약 코루틴에서 일정 시간동안 대기를 해야 한다면, **`await asyncio.sleep(DELAY)`를 사용**하자. 이는 **제어를 다시 `asyncio` event loop으로 넘겨주므로**, 다른 pending 코루틴을 실행할 수 있다.
{: .prompt-danger}

<br>

### 쓰레드 vs. 코루틴

앞서 다룬 [1]번과 [3]번의 코드의 `supervisor()` 함수를 기준으로, **쓰레드(`Thread`)와 코루틴(`Task`)을 비교**해보면 다음과 같다.

1. `asyncio.Task`는 `threading.Thread`와 비슷하다.
2. `Task`는 코루틴 객체를 drive 하고, `Thread`는 callable을 호출한다.
3. 코루틴은 **`await` 키워드**를 통해 명시적으로 제어를 넘긴다.
4. `Task` 객체를 직접 instantiate 하지 않고, `asyncio.create_task(...)`에 코루틴을 넘김으로써 얻는다.
5. `Task` 객체는 `asyncio.create_task()`로부터 반환되었다면 **이미 실행되도록 스케줄된 것**이지만, `Thread`는 `start` 메서드를 통해 명시적으로 실행되어야 한다.
6. 쓰레드 버전에서의 `supervisor`에서는 `slow`가 **일반 함수**이며 메인 쓰레드에 의해 직접적으로 실행되는 반면, 코루틴 버전에서의 `supervisor`에서는 `slow`가 `await`에 의해 driven 되는 **코루틴**이다.
7. `Thread`를 외부에서 종료하는 API가 없기 때문에 **signal(ex. `Event` 객체)**을 보내야 한다. 하지만 `Task`의 경우, **`cancel()` 메서드**를 통해 `await`에서 `CancelledError`를 raise 함으로써 종료시킬 수 있다.
8. 코루틴 버전에서의 `supervisor`는 `main` 함수에서 `asyncio.run`에 의해 시작되어야 한다.

<br>

추가로, **쓰레드와 코루틴의 가장 큰 차이점**은 <span class="shl">**스케줄러의 영향을 받는지**</span> 여부이다.

- **쓰레드**의 경우, <span class="blue">**스케줄러에 의해 언제든지 interrupt**</span> 될 수 있다. 따라서 multi-step operation이 중단될 경우에 대해 대비하기 위해서, 멀티 쓰레드 간 임계 영역(critical section)을 보호하기 위한 락(lock)을 관리하는 등의 **동기화(synchronization) 기법**을 이용해야 한다.
- 하지만 **코루틴**의 경우, 코드는 <span class="blue">**스케줄링 interruption에 대해 영향을 받지 않는다**</span>. 늘 명시적으로 `await`를 사용해야 남은 프로그램을 실행할 수 있으며, 락과 같은 동기화 기법이 필요하지 않다. 코루틴은 **싱글 쓰레드**인 **파이썬 인터프리터 쓰레드에 존재**하므로 코루틴이 **한 순간에 단 하나만 실행**될 수 있기에, 그 자체로 **이미 “동기화”**되어 있다고 볼 수 때문이다. 만약 제어를 그만두고 싶다면 `await`를 통해 스케줄러에게 제어를 넘기면 된다.

<br>

## References

- “Fluent Python (2nd Edition)”, Ch19. Concurrency Models in Python
- <https://docs.python.org/3/library/threading.html#event-objects>
- <https://realpython.com/python-sleep/>
- <https://thrivemyway.com/how-to-wait-in-python/>
- <https://docs.python.org/ko/3.11/library/time.html#time.sleep>
- <https://www.shiksha.com/online-courses/articles/mastering-the-art-of-delay-with-python-sleep/>
- <https://velog.io/@codingskynet/sleep-함수는-어떻게-동작하는-것일까>
