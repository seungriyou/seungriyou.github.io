---
title: "[Python] Thread-Local과 Context-Local"
date: 2024-05-03 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, keyword, thread-local, context-local, thread, coroutine]
math: true
---

> 본문에서 다루는 예제 코드는 **[Python threading.local 와 ContextVar 비교](https://daco2020.tistory.com/799)** 포스팅을 참고했음을 밝힙니다.

<br>

**ASGI 애플리케이션**에서는 여러 task가 single thread 기반으로 concurrent 하게 수행된다. 따라서 각 task 마다 어떠한 데이터를 안전하게 관리하려면, thread-local이 아닌 **context-local**한 방법을 사용해야 한다.

thread-local과 context-local이 무엇인지, 그리고 각각을 달성하는 방법은 무엇인지 알아보자.

<br>

## 1. Thread-Local

### 1-1. Thread-Local이란?

모든 파이썬 프로그램은 기본적으로 main thread라고 불리는 하나의 thread에서 실행되는데, 추가적으로 thread가 필요하다면 `threading.Thread` 모듈을 통해 지원한다. 이때, 각 thread 별로 관리되는 데이터를 저장할 수 있는데, 이러한 성질을 **thread-local** 이라고 한다. 또한, 이러한 데이터를 저장하는 공간을 **thread-local storage**, 줄여서 TLS라고 부른다.

<br>

### 1-2. `threading.local()`

파이썬에서는 `threading` 모듈에서 제공하는 `local()`을 이용하여 각 thread 마다 관리되는 데이터를 저장할 수 있다. 이를 사용하면 **multi-threading 환경**에서도 **안전하게 각 thread에 속하는 데이터를 보존**할 수 있다.

<br>

이를 활용한 예제 코드는 다음과 같다.

```python
import threading

# Create threading.local object.
thread_local_data = threading.local()

def thread_set():
    thread_local_data.value = "A"

def thread_get():
    value = getattr(thread_local_data, "value", "none")
    print(f"Thread-local data is {value}")

def thread_execute():
    thread_set()
    thread_get()

thread = threading.Thread(target=thread_execute)
thread.start()
thread.join()

# output: 'Thread-local data is A'
```

<br>

## 2. Context-Local

### 2-1. Context-Local이란?

thread-local과 비슷하지만, **context**는 thread에 국한되지 않는다. 따라서 context-local에서는 각 thread 뿐만 아니라, **각 asynchronous task(ex. `asyncio.Task`)별로도 context를 유지할 수 있다**.

<br>

### 2-2. `contextvars.ContextVar`

파이썬에서는 `contextvars` 모듈의 context variable `ContextVar`을 통해 **context 단위로 관리되는 값**을 저장할 수 있다.

이는 같은 context 내에서 **연쇄적인 호출을 통해 variable을 넘겨야 할 때** 편리하고, **multi-threading** 환경과 **asynchronous** 환경에서 사용 가능하다.

> `contextvars`는 `asyncio`를 지원하므로, 추가 설정 없이 async 환경에서 사용할 수 있다.

<br>


context variable을 선언할 때는 `contextvars.ContextVar(name, default)`로 정의하며, 해당 클래스는 다음과 같은 주요 메소드들를 가진다.

1. **`get()`**: 특정 context에서의 context variable 값을 반환한다. 아무 값도 가지고 있지 않은 경우, `default` 값이 설정되었다면 해당 값을, 아니라면 에러를 발생시킨다.
2. **`set(value)`**: 인자로 받은 값을 특정 context의 context variable 값으로 설정한다.
3. **`reset(token)`**: context variable의 값을 `set(value)`가 호출되기 전의 값으로 리셋한다. 이때 사용하는 `token`은 `set(value)`를 호출했을 때의 반환 값이다.

> `contextvars.ContextVar`의 실제 구현 코드: [링크](https://github.com/MagicStack/contextvars/blob/d13d478ee17a61b1f28bea67097357556ffb942b/contextvars/__init__.py#L81)
{: .prompt-info}

<br>

이를 활용한 예제 코드는 다음과 같다.

```python
from contextvars import ContextVar

# Create ContextVar object. (can set `default`)
context_var = ContextVar("context_var", default="...!")

def context_set():
    context_var.set("B")

def context_get():
    value = context_var.get()
    print(f"Context-local data is {value}")

def context_execute():
    context_set()
    context_get()

context_execute()

# output: 'Context-local data is B'
```

<br>

### 2-3. `asyncio.Task`

[`asyncio` 공식 문서](https://docs.python.org/3/library/asyncio-task.html#task-object)에서 `Task` 항목을 살펴보면 다음과 같은 글을 찾을 수 있다.

> An optional keyword-only *context* argument allows specifying a custom [`contextvars.Context`](https://docs.python.org/3/library/contextvars.html#contextvars.Context) for the *coro* to run in. If no *context* is provided, the Task copies the current context and later runs its coroutine in the copied context.
> 

<br>

즉, `Task`는 `contextvars` 모듈을 지원하며, `Task`가 생성될 때 `context`가 따로 주어지지 않으면 현재 context를 복사(`contextvars.copy_context()`)하고 나중에 해당 context에서 coroutine을 실행한다. 이러한 동작을 통해 `Task` 별로 context를 가지게 되는 것 같다.

<br>

- [`asyncio.Task`의 실제 구현 코드](https://github.com/python/cpython/blob/92fab3356f4c61d4c73606e4fae705c6d8f6213b/Lib/asyncio/tasks.py#L119)를 살펴보면, `asyncio`의 `Task` instance 마다 `_context` attribute를 가지게 되는 것을 확인할 수 있다.
    
  ```python
  class Task(futures._PyFuture):  # Inherit Python Task implementation
                                  # from a Python Future implementation.

      """A coroutine wrapped in a Future."""

      ...

      def __init__(self, coro, *, loop=None, name=None, context=None,
                    eager_start=False):
          super().__init__(loop=loop)

          ...

          if context is None:
              self._context = contextvars.copy_context()
          else:
              self._context = context
  ```
    
- [`contextvars.copy_context()`의 실제 구현 코드](https://github.com/MagicStack/contextvars/blob/d13d478ee17a61b1f28bea67097357556ffb942b/contextvars/__init__.py#L184)를 살펴보면, 다음과 같은 동작을 통해 context의 복사본을 얻을 수 있음을 확인할 수 있다.
    
  ```python
  def copy_context():
      return _get_context().copy()

  def _get_context():
      ctx = getattr(_state, 'context', None)
      if ctx is None:
          ctx = Context()
          _state.context = ctx
      return ctx

  def _set_context(ctx):
      _state.context = ctx

  _state = threading.local()
  ```
    
- [`contextvars.Context` 클래스의 실제 구현 코드](https://github.com/MagicStack/contextvars/blob/master/contextvars/__init__.py#L24)를 살펴보면 다음과 같다.
    
  ```python
  class Context(collections.abc.Mapping, metaclass=ContextMeta):

      def __init__(self):
          self._data = immutables.Map()
          self._prev_context = None

      def run(self, callable, *args, **kwargs):
          if self._prev_context is not None:
              raise RuntimeError(
                  'cannot enter context: {} is already entered'.format(self))

          self._prev_context = _get_context()
          try:
              _set_context(self)
              return callable(*args, **kwargs)
          finally:
              _set_context(self._prev_context)
              self._prev_context = None

      def copy(self):
          new = Context()
          new._data = self._data
          return new
  ```
  

<br>

### 2-4. 사용 시 주의사항

- `ContextVar`는 closure에서 생성되면 안 되며, top module level에서 생성되어야 한다.
- 현재 context에서 새로운 값을 context variable에 `set(value)`를 통해 설정하려면, 반환된 token 값을 저장하는 것이 좋다. 이는 추후 context variable을 이전 값으로 `reset(token)`하는 데에 사용할 수 있다.
- `get()` 및 `reset(token)` 시에 예외 처리를 항상 잘 해야 한다.
- concurrent 환경에서 사용할 때는 다른 코드에 예상치 못한 영향을 미치지 않는지 확인해야 한다.

<br>

## 3. `threading.local()` vs. `contextvars.ContextVar`

### 3-1. 공통점

특정 thread 혹은 context에 local한 데이터를 저장하거나 접근하는 방법을 제공한다.

즉, **<span class="shl">multi-threading 환경</span>에서는 비슷하게 동작**한다.

<br>

### 3-2. 차이점

**<span class="shl">async 환경</span>인 coroutine**에서의 동작이 다르다.

파이썬에서는 **여러 coroutine(asynchronous tasks)이 하나의 thread를 공유**하기 때문에 thread-local을 보장한다고 해서 coroutine 별로 locality를 보장할 수 없으며, 예측 불가능한 결과를 불러올 수 있다.

- `threading.local()`은 **thread-local**이기 때문에, 각 thread 마다 storage를 가지며 그곳에 데이터를 저장한다.
- 반면, `contextvar`은 **context-local**이기 때문에, thread-local의 특성을 가질 뿐만 아니라 concurrent 환경에서 각 task 마다 storage를 가진다.

> async 환경에서는 `contextvar`를 사용해야 한다!
{: .prompt-danger}

![](/assets/img/posts/Python/More-About-Python/2024-05-03-01.png){: .w-75}
_ref: <https://daco2020.tistory.com/799>_

<br>

### 3-3. Async 환경에서의 예시

#### [1] `threading.local()`
하나의 thread에서 모든 coroutine이 실행되므로 데이터가 안전하게 유지되지 않고 덮어씌워진다.
    
```python
import asyncio
import threading

thread_local_data = threading.local()

async def execute(name):
    await asyncio.sleep(0.1)
    print(f"MBTI type of student {name} is {getattr(thread_local_data, 'value', 'none')}")

async def set_A():
    thread_local_data.value = "INFJ"
    print(f"MBTI type of Student A: {thread_local_data.value}")
    await execute("A")

async def set_B():
    thread_local_data.value = "CUTE"
    print(f"MBTI type of Student B: {thread_local_data.value}")
    await execute("B")

async def main():
    await asyncio.gather(set_A(), set_B())

asyncio.run(main())

# MBTI type of Student A: INFJ
# MBTI type of Student B: CUTE
# MBTI type of student A is CUTE <-- 덮어씌워짐!
# MBTI type of student B is CUTE
```

#### [2] `contextvars.ContextVar`
각 coroutine 별로 context variable 값이 안전하게 유지된다.
    
```python
import asyncio
from contextvars import ContextVar

context_var_data: ContextVar[str] = ContextVar("context_var_data")

async def execute(name):
    await asyncio.sleep(0.1)
    print(f"MBTI type of student {name} is {context_var_data.get('none')}")

async def set_A():
    context_var_data.set("INFJ")
    print(f"MBTI type of Student A: {context_var_data.get()}")
    await execute("A")

async def set_B():
    context_var_data.set("CUTE")
    print(f"MBTI type of Student B: {context_var_data.get()}")
    await execute("B")

async def main() -> None:
    await asyncio.gather(set_A(), set_B())

asyncio.run(main())

# MBTI type of Student A: INFJ
# MBTI type of Student B: CUTE
# MBTI type of student A is INFJ <-- 값이 덮어씌워지지 않고 안전하게 유지됨
# MBTI type of student B is CUTE
```
    

<br>

## References

- <https://superfastpython.com/thread-local-data/>
- <https://thinhdanggroup.github.io/context-var/>
- <https://velog.io/@kjyggg/threading.local-과-contextvars>
- <https://daco2020.tistory.com/799> (예제 코드)
- <https://kobybass.medium.com/python-contextvars-and-multithreading-faa33dbe953d>
- <https://stackoverflow.com/questions/63105799/understanding-python-contextvars>
- <https://www.geeksforgeeks.org/context-variables-in-python/>
- <https://valarmorghulis.io/tech/201904-contextvars-and-thread-local/>
- <https://docs.python.org/3/library/contextvars.html#asyncio-support>
- <https://peps.python.org/pep-0567/>
- <https://github.com/python/cpython/blob/main/Lib/_threading_local.py>
- <https://github.com/MagicStack/contextvars/blob/master/contextvars/__init__.py>
- <https://stackoverflow.com/questions/70748053/using-contextvar-to-keep-track-of-async-loop-in-python>
