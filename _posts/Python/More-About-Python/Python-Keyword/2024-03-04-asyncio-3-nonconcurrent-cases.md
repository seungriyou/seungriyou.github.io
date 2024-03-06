---
title: "[Python] asyncio 파헤치기 - ③ 동시성이 달성되지 않는 상황"
date: 2024-03-04 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, asyncio, nonconcurrent, blocking, cpu bound, io bound]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---

> 본문은 “Python Concurrency with Asyncio”를 읽고 재구성한 글임을 밝힙니다.
> 

<br>

이번 포스팅에서는 실제로 `asyncio`를 사용하는 상황을 연출하면서 주의해야 할 점에 대해 알아보도록 하겠다.

<br>

## 1. 실습 준비

실습을 진행하기 전, 코루틴의 실행 시간을 측정하기 위한 데코레이터를 작성하자.

```python
# util/async_timer.py

import functools
import time
from typing import Callable, Any

def async_timed():
    def wrapper(func: Callable) -> Callable:
        @functools.wraps(func)
        async def wrapped(*args, **kwargs) -> Any:
            print(f"starting {func} with args {args} {kwargs}")
            start = time.time()
            try:
                return await func(*args, **kwargs)
            finally:
                end = time.time()
                total = end - start
                print(f"finished {func} in {total:.4f} second(s)")
        
        return wrapped
    
    return wrapper
```

```python
# util/__init__.py

...
from util.async_timer import async_timed
```

> 데코레이터(decorator)는 <span class="shl">**어떤 함수의 코드를 수정하지 않으면서 기능을 추가할 수 있도록**</span> 돕는 패턴이다. 데코레이터를 통해 어떤 함수를 intercept 하여, 실행 전후에 추가 동작을 수행할 수 있다.
>
> 더 자세한 내용은 [[Python] 데코레이터(Decorator)](/posts/python-decorator/) 포스팅을 참고한다.
{: .prompt-info}

<br>

이렇게 생성한 `async_timed` 데코레이터의 동작을 확인하기 위해 다음과 같이 동시성이 달성된 코드에 적용해보겠다.

```python
import asyncio
from util import async_timed

@async_timed()
async def delay(delay_seconds: int) -> int:
    print(f"sleeping for {delay_seconds} second(s)")
    await asyncio.sleep(delay_seconds)
    print(f"finished sleeping for {delay_seconds} second(s)")
    
    return delay_seconds

@async_timed()
async def main():
    task_one = asyncio.create_task(delay(2))
    task_two = asyncio.create_task(delay(3))
    
    await task_one
    await task_two
    
asyncio.run(main())
```

결과를 살펴보면 두 개의 `delay` 작업이 각각 **주어진 시간(각각 2초, 3초)만큼 잘 수행**되었으나, 이들이 순차적이 아닌 **동시적으로 수행**되었기에 `main` 작업이 5초(= 2+3)가 아닌 **3초만에 완료**되었음을 확인할 수 있다.

```
starting <function main at 0x103944700> with args () {}
starting <function delay at 0x102a03370> with args (2,) {}
sleeping for 2 second(s)
starting <function delay at 0x102a03370> with args (3,) {}
sleeping for 3 second(s)
finished sleeping for 2 second(s)
finished <function delay at 0x102a03370> in 2.0022 second(s)
finished sleeping for 3 second(s)
finished <function delay at 0x102a03370> in 3.0015 second(s)
finished <function main at 0x103944700> in 3.0033 second(s)
```

이렇게 데코레이터를 통해 측정된 시간을 기반으로, 예시 코드를 실행해보면서 **동시성이 달성되었는지 여부를 판단**해보자.

<br>

## 2. `asyncio`를 사용하더라도 동시성을 달성할 수 없는 경우: <span class="red">Blocking</span> 상황

이러한 성능 상의 이점을 맛보고 나면 어플리케이션의 모든 곳에 `asyncio`를 사용하고 싶어질 수 있다. 하지만 다음의 두 가지 경우에는 `asyncio`를 적용하더라도 성능 향상을 달성할 수 없다.

1. 태스크 혹은 코루틴 내의 <span class="shl">**CPU-bound 코드**</span>를 **멀티 프로세싱 없이** 사용하는 경우
2. <span class="shl">**blocking I/O-bound API**</span>를 **멀티 쓰레딩 없이** 사용하는 경우

<br>

그리고 이를 한 마디로 쉽게 정리하면 **“<span class="red">blocking</span> 연산이 멀티 프로세싱 및 멀티 쓰레딩 없이 수행될 때는 `asyncio`를 적용하더라도 동시성을 달성할 수 없다”**고 할 수 있다.

> 일반적으로, **(1) 긴 시간이 소요되는 <span class="shl">CPU-bound 연산</span>**과 **(2) 코루틴이 아닌 <span class="shl">I/O-bound 연산</span>**을 **<span class="red">blocking</span> 연산**이라 표현한다.
{: .prompt-tip}

이러한 blocking 연산은 <span class="shlp">**이벤트 루프 쓰레드를 block** 하며, **다른 코루틴이 실행될 수 없도록** 하기 때문</span>이다.

<br>

이 두 가지 경우에 대해 예시와 함께 살펴보자.

<br>

### 2-1. CPU-Bound 코드 (w/o 멀티 프로세싱)

다음과 같이 CPU-bound 코드 두 개를 `asyncio`를 통해 동시에 실행하려 해보자.

```python
import asyncio
from util import async_timed

@async_timed()
async def cpu_bound_work() -> int:
    counter = 0
    for i in range(100_000_000):
        counter = counter + 1
    return counter

@async_timed()
async def main():
    task_one = asyncio.create_task(cpu_bound_work())
    task_two = asyncio.create_task(cpu_bound_work())
    await task_one
    await task_two

asyncio.run(main())
```

하지만 태스크를 생성 및 예약했음에도 불구하고, 다음과 같이 동시적이 아닌 **순차적으로 수행되는 것**을 확인할 수 있다. 따라서 `main` 작업이 두 번의 `cpu_bound_work` 작업 시간의 합으로 나타난다.

```
starting <function main at 0x1035a0700> with args () {}
starting <function cpu_bound_work at 0x102f8b370> with args () {}
finished <function cpu_bound_work at 0x102f8b370> in 2.7899 second(s)
starting <function cpu_bound_work at 0x102f8b370> with args () {}
finished <function cpu_bound_work at 0x102f8b370> in 2.7878 second(s)
finished <function main at 0x1035a0700> in 5.5781 second(s)
```

> `asyncio`는 **싱글 쓰레드** 동시성 모델을 가지고 있기 때문에, 여전히 **싱글 쓰레드와 GIL의 제약**을 받기 때문이다.
{: .prompt-tip}

<br>

이러한 CPU-bound 코드를 동시에 처리하려면 **멀티 프로세싱**을 사용해야 하며, `async` / `await` 문법을 사용하고 싶다면 **`asyncio`에게 프로세스 풀(process pool)에서 태스크를 실행할 것**을 명시해야 한다.

> 관련된 자세한 내용은 본 포스팅에서 다루지 않겠다.
> 

<br>

### 2-2. Blocking I/O-Bound API (w/o 멀티 쓰레딩)

기존 라이브러리의 I/O-bound 연산을 코루틴으로 수행하는 것 또한 CPU-bound 연산과 동일한 양상을 보인다. 왜냐하면 이러한 **I/O-bound API는 메인 쓰레드를 block** 하기 때문이다.

> 코루틴 내에서 **blocking API**를 호출하면 <span class="shl">**이벤트 루프 쓰레드 자체가 block**</span> 되기 때문에 다른 코루틴이나 태스크도 수행될 수 없다.
{: .prompt-tip}

이러한 **blocking API**에는 `requests` 라이브러리에서 제공하는 연산이나 `time.sleep`과 같은 것들이 있다.

<br>

`requests` 라이브러리를 사용한 예시 코드는 다음과 같다.

```python
import asyncio
import requests
from util import async_timed

@async_timed()
async def get_example_status() -> int:
    return requests.get("http://www.example.com").status_code

@async_timed()
async def main():
    task_1 = asyncio.create_task(get_example_status())
    task_2 = asyncio.create_task(get_example_status())
    task_3 = asyncio.create_task(get_example_status())
    
    await task_1
    await task_2
    await task_3

asyncio.run(main())
```

결과를 살펴보면 동시성이 달성되지 못하고 순차적으로 수행되었음을 확인할 수 있다.

```
starting <function main at 0x10605a9e0> with args () {}
starting <function get_example_status at 0x105e09bd0> with args () {}
finished <function get_example_status at 0x105e09bd0> in 0.3163 second(s)
starting <function get_example_status at 0x105e09bd0> with args () {}
finished <function get_example_status at 0x105e09bd0> in 0.3171 second(s)
starting <function get_example_status at 0x105e09bd0> with args () {}
finished <function get_example_status at 0x105e09bd0> in 0.2895 second(s)
finished <function main at 0x10605a9e0> in 0.9233 second(s)
```

<br>

이처럼 대부분의 API는 blocking 방식으로 동작하기 때문에 `asyncio`와 사용하더라도 동작하지 않는다. 따라서 <span class="shl">**(1) 코루틴을 지원**</span>하며 <span class="shl">**(2) non-blocking 소켓을 사용**</span>하는 라이브러리를 찾아 사용해야 한다. 그렇지 않으면 **여전히 blocking 방식**으로 동작하여 동시에(concurrently) 실행되지 않고 **순차적(sequentially)으로 실행**된다.

이러한 라이브러리의 예시로는 `aiohttp` 같은 것들이 있다. 이는 **non-blocking 소켓**을 사용하고 **코루틴을 반환**하여 동시성을 달성할 수 있도록 지원한다.

> `sqlalchemy`를 `asyncio`와 함께 사용할 때, `aiopg`, `aiomysql`와 같이 비동기를 지원하는 드라이버를 사용해야 했던 이유가 바로 이것이다!
{: .prompt-info}

<br>

만약 `requests` 라이브러리를 사용해야 하고 `async` 문법 또한 사용하고 싶은 경우라면 **멀티 쓰레딩**을 사용해야 하며, **`asyncio`에게 thread pool executor에서 실행**할 것을 명시해야 한다.

> 관련된 자세한 내용은 본 포스팅에서 다루지 않겠다.
> 

<br>

## 3. 디버그 모드

이전 포스팅에서 다루었듯, 코루틴은 항상 어플리케이션의 어딘가에서 `await`와 함께 호출되어야 한다. 하지만 개발 과정에서 실수로 `await`를 빼먹는 경우가 발생할 수 있는데, 이를 방지하기 위해 **디버그(`debug`) 모드**를 사용할 수 있다. **디버그 모드에서 지원되는 기능**은 다음과 같다.

1. 100ms 이상 소요되는 코루틴 혹은 태스크가 수행될 때 로그 메시지를 남겨준다.
2. 코루틴을 `await` 하지 않으면 exception을 발생시킨다.

<br>

이러한 디버그 모드를 사용하는 방법에는 세 가지가 있다.

1. **`asyncio.run`에 `debug` 파라미터 `True`로 넘기기**
    
    ```python
    asyncio.run(coroutine(), debug=True)
    ```
    
2. **command-line argument `-X dev` 지정하기**
    
    ```bash
    python3 -X dev program.py
    ```
    
3. **환경 변수 `PYTHONASYNCIODEBUG` 사용하기**
    
    ```bash
    PYTHONASYINCIODEBUG=1 python3 program.py
    ```
    

<br>

> 지금까지 `asyncio`에 대한 아주 기초적인 내용들을 다뤄보았다. 어려우면서도 방대한 내용이다보니 앞으로도 `asyncio`에 대해 더 알아봐야 할 것 같다.
> 

<br>

## References

- “Python Concurrency with Asyncio”, Ch02. `asyncio` Basics
- <https://docs.python.org/3/library/asyncio-dev.html>
