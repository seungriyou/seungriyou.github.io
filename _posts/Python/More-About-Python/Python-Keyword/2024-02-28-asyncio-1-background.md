---
title: "[Python] asyncio 파헤치기 - ① 배경 지식 정리"
date: 2024-02-28 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, asyncio]
math: true
image: 
  path: /assets/img/posts/Python/thumbnail.png
---
> 본문은 “Python Concurrency with Asyncio”를 읽고 재구성한 글임을 밝힙니다.
> 

<br>

파이썬 3.4에서부터는 `asyncio` 라이브러리를 통해 동시성(concurrency)을 지원한다. 본 포스팅에서는 `asyncio`에 대해 자세히 알아보기 앞서, 관련 배경 지식들을 쭉 나열하며 정리해보도록 하겠다.

<br>

## 1. I/O-Bound & CPU-Bound

어떤 연산이 I/O-bound 인지 CPU-bound 인지 판단하는 것은, **“연산이 더 빠르게 수행되는 것을 제한하고 있는 요소가 무엇인지”**에 대한 것이다. 예를 들어 CPU-bound 연산은 CPU의 사양이 더 좋아진다면, I/O-bound 연산은 I/O 장치의 성능이 더 좋아진다면 더 빠르게 수행될 수 있는 것이다. 따라서 다음과 같이 정리할 수 있다.

- <span class="shlp">**CPU-bound 연산**</span>: 전형적인 계산 혹은 파이썬 코드가 수행되는 연산
- <span class="shlp">**I/O-bound 연산**</span>: 대부분의 시간이 네트워크나 I/O 장치에 대해 대기하는 데에 소요되는 연산

<br>

많은 어플리케이션, 특히나 현대의 웹 어플리케이션은 <span class="red">**I/O 연산(I/O-bound 연산)**</span>의 영향을 자주 받는다.

> **I/O 연산의 예시**: 인터넷으로부터 데이터를 다운로드 받는 것, 마이크로서비스 간 네트워크 통신을 하는 것, 데이터베이스에 여러 쿼리를 보내는 것, 웹 서버로 요청을 보내는 것, 디스크로부터 파일을 읽는 것 등
> 

이들은 모두 시간이 <span class="red">**오래 소요될 수 있는 작업(long-running task)**</span>들이다. 따라서 이러한 연산들을 모두 **순차적으로(sequentially)** 실행하게 된다면 성능에 부정적인 영향을 미칠 수밖에 없다는 것을 알 수 있다. 그렇다면 long-running task를 어떻게 실행해야 할까? 여기에서 **동기**와 **비동기**가 등장한다.

<br>

## 2. 동기(Synchronous) & 비동기(Asynchronous)

동기와 비동기의 핵심적인 차이점은 **“다음 작업 수행 시 기존 작업의 완료 여부를 따지는지”**에 관한 것이다.

- <span class="shlp">**동기(synchronous)**</span>: 요청한 작업이 완료되어야만 다음 작업을 수행한다.
    
    > 요청된 순서대로 작업이 수행된다.
    > 
- <span class="shlp">**비동기(asynchronous)**</span>: 요청한 작업의 완료 여부를 따지지 않고 다음 작업을 수행한다.
    
    > 요청된 순서대로 작업이 수행되는 것을 보장하지 않는다.
    > 

![](/assets/img/posts/Python/More-About-Python/2024-02-28-01.png){: .w-75}
_ref: <https://medium.com/from-the-scratch/wtf-is-synchronous-and-asynchronous-1a75afd039df>_

<br>

따라서 **동기 어플리케이션**에서는 코드가 **순차적(sequentially)**으로 실행된다. 하지만 앞서 다루었듯, 순차적으로 실행되는 작업들 중 하나가 I/O 연산과 같은 long-running task라면? 해당 작업이 완료될 때까지 다른 모든 작업들이 꼼짝없이 기다려야 하는 상황이 발생할 것이다. 

<br>

반면, **비동기 어플리케이션**에서는 특정 long-running task가 메인 어플리케이션으로부터 분리되어 **백그라운드(background)**에서 실행될 수 있다. 동작을 정리하면 다음과 같다.

1. 시스템은 해당 long-running task에 dependent 하지 않은 **다른 작업을 수행**할 수 있다.
2. 해당 long-running task가 **완료되면, 그 사실을 공지**받고 결과값을 사용할 수 있다.

즉, 비동기를 통해 많은 <span class="shl">**I/O 연산**을 **동시에 수행**</span>할 수 있게 되므로 어플리케이션의 속도를 향상시킬 수 있는 것이다. 하지만 여기에서 말하는 **“동시에”**란, 하나의 시점에 모든 연산이 “동시에" 수행되는 것을 의미하는 것이 아니다. 그렇다면, 이 “동시에”란 무엇을 의미하는 것일까?

<br>

## 3. 동시성(Concurrency) & 병렬성(Parallelism)

> 동시성에 관련된 자세한 내용 및 예제 코드는 다음의 포스팅을 참조한다: [[Python] 동시성(Concurrency): 쓰레드, 프로세스, 코루틴 (+ Spinner 예제)](/posts/python-concurrency-spinner/)
{: .prompt-info}

우선, 이해를 돕기 위해 제빵사가 두 개의 케이크를 만드는 상황을 가정해보자. 이때, 동시성과 병렬성은 각각 다음과 같이 빗대어 표현할 수 있다.

- **동시성(concurrency)**: 한 명의 제빵사가 두 개의 케이크를 동시에 만드는 상황
    
    ![](/assets/img/posts/Python/More-About-Python/2024-02-28-02.png){: .w-75}
    
- **병렬성(parallelism)**: 두 명의 제빵사가 각각 한 개의 케이크를 동시에 만드는 상황
    
    ![](/assets/img/posts/Python/More-About-Python/2024-02-28-03.png){: .w-75}
    

<br>

이러한 개념을 컴퓨터에 적용해보자! 두 개의 어플리케이션이 실행되고 있다고 가정한다.

- **동시성만 지원하는 시스템**: 하나의 CPU 코어에서 두 어플리케이션을 번갈아가며 실행한다. 이때, 시간 간격을 짧게 하면 마치 동시에 실행되는 것처럼 보일 것이다.
    
    ![](/assets/img/posts/Python/More-About-Python/2024-02-28-04.png){: .w-75}
    
- **병렬성을 지원하는 시스템**: 두 어플리케이션을 각 CPU 코어에서 동시에 병렬적으로 실행한다.
    
    ![](/assets/img/posts/Python/More-About-Python/2024-02-28-05.png){: .w-75}
    

<br>

> 즉, 앞서 언급했던 “**비동기**를 통해 여러 I/O 연산을 **동시에 수행**”하는 것은 <span class="red">**동시성(concurrency)**</span>에 관한 것이다.
{: .prompt-tip}

<br>

종합해보면, 다음과 같이 정리할 수 있다.

- <span class="shlp">**동시성(concurrency)**</span>
    - 여러 작업이 독립적으로, 번갈아가며 수행되면서 마치 동시에 실행되는 것처럼 동작하는 것이다.
    - 작업(쓰레드, 프로세스)이 번갈아가며 수행될 때 **선점형 멀티태스킹(preemptive multitasking)**이 사용되므로, **싱글 코어 CPU**에서도 동시성이 달성될 수 있다.
        
        > 선점형 멀티태스킹, 싱글 코어에 대해서는 다음 섹션에서 다룬다.
        > 
- <span class="shlp">**병렬성(parallelism)**</span>
    - 반드시 두 개 이상의 작업을 같은 시점에 실행하는 것이다.
    - 싱글 코어 CPU에서는 불가능하다. **멀티 코어 CPU에서만** 가능하다.

![](/assets/img/posts/Python/More-About-Python/2024-02-28-06.png){: .w-75}
_ref: <https://en.wikipedia.org/wiki/File:Parallel-concurrent.png>_

<br>

> **병렬적(in parallel)**이라는 것은, 작업이 <span class="shl">**(1) simultaneously**</span>, 그리고 <span class="shl">**(2) concurrently**</span> 하게 수행된다는 것이다.
>
> - 즉, 병렬성은 동시성을 내포하고 있으나, 동시성은 병렬성을 내포하지 않는다.
> - **멀티 쓰레드 어플리케이션**이 **멀티 코어 CPU**에서 실행된다면, **병렬성** 및 **동시성**을 모두 달성하고 있는 것이다.
{: .prompt-info}

<br>

### 3-1. 멀티태스킹(Multitasking)

멀티태스킹이란, **다수의 작업이 CPU와 같은 공용 자원을 나누어 사용하는 것**을 의미한다. 멀티태스킹에는 다음과 같이 두 가지 종류가 있다.

- <span class="shlp">**선점형 멀티태스킹(preemptive multitasking)**</span>
    - **OS**가 **time slicing**을 이용하여 작업을 번갈아가면서 수행한다. 실행 중인 작업에 할당된 time slice가 소진되면 OS는 해당 작업이 완료되지 않았더라도 다른 작업으로 switch 하는데, 이를 선점(preempting)이라 한다.
        
        > OS 스케줄러가 선점한다는 것은, 주기적으로 다른 프로세스가 실행될 수 있도록 한다는 것이다. 따라서 이론적으로, frozen 작업이 전체 시스템을 freeze 할 수는 없다.
        > 
    - 멀티 <span class="red">**쓰레드**</span> 혹은 멀티 <span class="red">**프로세스**</span>를 번갈아 수행할 때에 해당한다.

- <span class="shlp">**비선점형 멀티태스킹(cooperative multitasking)**</span>
    - 실행할 작업을 선택하는 것을 OS에게 맡기지 않고, 어플리케이션 코드에 **다른 작업을 수행할 시점을 명시적으로 나타냄**으로써 수행한다.
    - 어플리케이션에서 수행되는 작업들이 “여기에서 작업을 잠시 멈출테니 그 동안 다른 작업을 수행해도 됩니다” 라고 명시적으로 말하는 것과 비슷하다. 이러한 점에서 협력적(cooperative)이라 표현한다.
    - 여러 <span class="red">**코루틴**</span>을 수행할 때에 해당한다.

<br>

`asyncio`는 동시성을 달성하기 위해 <span class="shl">**비선점형 멀티태스킹**</span>을 사용한다. 비선점형 멀티태스킹의 장점에는 다음과 같은 것들이 있다.

1. <span class="ulr">**Less resource intensive**</span>
    
    선점형 멀티태스킹에서는 OS가 작업(쓰레드, 프로세스) 간에 switching을 수행하려면 **문맥 교환(context switch)**이 수반된다. 이때, OS는 진행 중이던 작업에 관한 정보를 저장 및 로드해야하므로 비용이 발생한다.
    
2. <span class="ulr">**Granularity**</span>
    
    선점형 멀티태스킹은 time slicing 기반이기 때문에, 작업이 switch 되는 시점이 **최적의 시점**이 아닐 수 있다. 반면, 비선점형 멀티태스킹에서는 작업을 중지할 시점을 명시적으로 표시할 수 있으므로 적절한 시점을 프로그래머가 직접 지정할 수 있다.
    

<br>

### 3-2. 싱글 코어(Single-Core) & 멀티 코어(Multi-Core)

우선, **CPU와 코어의 관계**에 대해 간단히 알아보면 다음과 같다.

- **CPU(central processing unit, 중앙 처리 장치)**: 컴퓨터 시스템에서 모든 산술 및 논리 연산을 수행하는 핵심 장치
    - 기억 / 연산 / 제어라는 기본 동작을 수행한다.
    - 명령을 수행하기 위해 다음의 단계를 따른다.
        
        > fetch → decode → execute
        > 
- **코어(core)**: CPU의 핵심 구성 요소로, 프로그램의 명령을 읽고 수행하는 처리 장치
    - 이전에는 CPU 한 개당 코어가 하나 존재하는 형태인 **싱글 코어(single-core)** CPU를 사용했으나, 2005년에 CPU 한 개당 코어가 n개 존재하는 형태인 **멀티 코어(multi-core)** CPU가 처음 등장하였다.

<br>

본래 **CPU는 한 번에 하나의 작업(프로세스, 쓰레드)을 처리할 수 있다**고 알려져 있으나, 사실 이는 **싱글 코어 CPU**(더 자세히 말하면, 단일 CPU이며 하이퍼 쓰레딩(hyper-threading)이 지원되지 않는 경우)일 때에 해당한다.

**싱글 코어와 멀티 코어의 멀티 태스킹 방법**을 비교해보면 다음과 같다.

- <span class="shlp">**싱글 코어(single-core)**</span>: 한 번에 단 하나의 작업만을 처리할 수 있으며, OS 수준에서 시분할 기법(time sharing)을 통해 여러 작업 간에 빠르게 전환하며 실행함으로써 마치 동시에 여러 작업이 수행되는 것처럼 보이게 한다. (실제로는 동시에 실행되는 것이 X)
    
    ![](/assets/img/posts/Python/More-About-Python/2024-02-28-07.png){: .w-75}
    _ref: <https://www.guru99.com/cpu-core-multicore-thread.html>_
    
- <span class="shlp">**멀티 코어(multi-core)**</span>: 여러 작업(멀티 프로세스/쓰레드)을 각 코어에 분배하여 병렬 처리(parallel execution)가 가능하기 때문에 실제로 동시에 여러 작업을 처리할 수 있으나, 개발자가 프로그램 구현 시 직접 병렬 프로그래밍 관련 작업을 해주어야 한다.
    
    ![](/assets/img/posts/Python/More-About-Python/2024-02-28-08.png){: .w-50}
    _ref: <https://www.guru99.com/cpu-core-multicore-thread.html>_
    

<br>

> 즉, 정리하면 다음과 같다. (하이퍼 쓰레딩이 적용되지 않은, 단일 CPU 환경 가정)
> 
> 1. **싱글 코어 CPU**에서의 멀티태스킹은 한 시점에 여러 작업이 동시에 수행되는 것처럼 보이지만 실제로는 여러 작업이 빠르게 번갈아가며 실행되는 것이다. 이를 **시분할 기법(time slicing)**이라고 한다.
> 2. **멀티 코어 CPU** 환경일 때 비로소 한 시점에 여러 작업이 동시에 수행되는 **병렬 처리**가 가능하다.
{: .prompt-tip}

<br>

### 3-3. 파이썬에서 동시성을 달성하는 방법

> 파이썬에서는 크게 다음의 세 가지 수단으로 동시성을 달성한다. 이런 것들이 있다는 것 정도만 알고 가자!
> 

| 패키지 이름     | 동시성 달성 수단                    |
| --------------- | ----------------------------------- |
| `threading`       | 쓰레드 (threads)                    |
| `multiprocessing` | 프로세스 (processes)                |
| `asyncio`         | 네이티브 코루틴 (native coroutines) |

<br>

## 4. 프로세스(Process) & 쓰레드(Thread) 그리고 코루틴(Coroutine)

> 지금까지 “작업”을 처리하는 방법에 대해 다뤄보았고, **“작업”**이란 **프로세스** 또는 **쓰레드**를 의미한다. 이것에 대해 자세히 알아보자, **코루틴**에 대해서도 간단히 알아보자.
> 

### 4-1. 프로세스(Process)

![](/assets/img/posts/Python/More-About-Python/2024-02-28-09.png){: .w-75}

- 프로세스란 **실행 중인 어플리케이션**을 의미하며, 이는 다른 어플리케이션에서 접근할 수 없는 메모리 공간을 가지고 있다.
- OS 스케줄러의 감독 하에 **선점형 멀티태스킹**을 지원한다.
- **멀티 코어 CPU**에서는 여러 개의 프로세스를 동시에, 한꺼번에 실행할 수 있다. 반면, **싱글 코어 CPU**에서 여러 개의 프로세스를 수행하려면 선점형 멀티태스킹을 기반으로(= OS가 time slicing을 통해 여러 개의 프로세스를 switching 하며) 수행할 수밖에 없다.

<br>

### 4-2. 쓰레드(Thread)

![](/assets/img/posts/Python/More-About-Python/2024-02-28-10.png){: .w-75}

- 쓰레드란 **OS에 의해 관리될 수 있는 가장 작은 단위**이며, **프로세스의 가벼운 버전**이라고 생각할 수 있다.
- 하나의 프로세스는 최소 하나의 쓰레드를 가지고 있으며, 이는 **메인 쓰레드(main thread)**라고 불린다. 또한, 쓰레드를 추가로 생성하는 것도 가능한데, 이렇게 생성된 쓰레드는 **워커(worker) 혹은 백그라운드 작업(background task)**이라 불린다.
- 쓰레드는 프로세스와는 다르게 자신만의 메모리를 가지지 못하지만, 자신을 생성한 프로세스의 메모리를 공유한다.
- OS 스케줄러의 감독 하에 **선점형 멀티태스킹**을 지원한다. (프로세스와 동일)
- 멀티 코어 CPU와 싱글 코어 CPU에서 여러 개의 쓰레드를 수행하는 방식이 프로세스와 동일하다.

<br>

### 4-3. 멀티 쓰레딩(Multi-Threading) & 멀티 프로세싱(Multi-Processing)

**동시성을 달성**하기 위해서는 작업을 여러 개 사용하면 된다. 이를 멀티 쓰레딩, 멀티 프로세싱이라 한다.

하지만 <span class="shlp">**멀티 쓰레딩**</span>의 경우, 파이썬에서는 **오직 I/O-bound 작업에 사용할 때만** 동시성의 효과를 얻을 수 있다. 이는 파이썬의 큰 특징인 **GIL** 때문이다.

> **GIL(전역 인터프리터 락)**에 대해서는 [[Python] GIL(Global Interpreter Lock, 전역 인터프리터 락)](/posts/python-global-interpreter-lock/) 을 참고하자!
{: .prompt-info}

따라서 파이썬에서는 <span class="shlp">**멀티 프로세싱**</span>을 통해 자식 프로세스에게 작업을 분배함으로써 **CPU-bound 작업에 대해서도** 동시성의 효과를 얻을 수 있다.

![](/assets/img/posts/Python/More-About-Python/2024-02-28-11.png){: .w-75}

<br>

### 4-4. 코루틴(Coroutine)

> `asyncio`는 **GIL이 I/O 연산에서 릴리즈 된다**는 점을 통해, <span class="red">**하나의 쓰레드(single-thread)에서도 동시성을 달성**</span>할 수 있는 방법을 제공한다. 이는 주로 **코루틴**을 통해 달성된다.

- 코루틴이란 주로 <span class="shl">**동시성을 적용하려는 I/O 작업**</span>을 수행하기 위한 것으로, **쓰레드의 가벼운 버전**이라고 생각하면 쉽다.
- 주로 **이벤트 루프(event loop)**의 감독 하에 실행되며, 이는 <span class="shl">**모두 같은 싱글 쓰레드(single-thread)에 존재**</span>한다.
    
    > 이벤트 루프에 대해서는 이어서 다룬다.
    > 
- 여러 개의 쓰레드를 사용할 수 있듯이 코루틴 또한 여러 개를 사용할 수 있으며, **어떤 I/O-bound 코루틴이 완료될 때까지 기다리는 것**과 동시에 **다른 코드를 수행**할 수 있으므로 동시성을 달성할 수 있다. 하지만 하나의 쓰레드에서 수행되는 것이라 하더라도 <span class="shl">**여전히 GIL에 종속적**</span>이기 때문에, **CPU-bound 작업**을 concurrent 하게 실행하려면 **멀티 프로세싱**이 필요하다.
    
    > `asyncio`는 코루틴을 통해 <span class="blue">**하나의 쓰레드**</span> 내에서도 <span class="blue">**I/O-bound 작업의 동시성**</span>을 달성할 수 있다! 하지만 CPU-bound 작업의 동시성을 달성하려면 여전히 멀티 프로세싱이 필요하다.
    {: .prompt-tip}
    
- 프로세스 및 쓰레드와는 달리 <span class="shl">**비선점형 멀티태스킹**</span>을 지원한다. 따라서 다른 작업을 수행할 시점을 명시적으로 표시해야 한다.

<br>

## 5. 소켓(Socket)

> 앞서, `asyncio`는 **코루틴**을 통해 <span class="red">**하나의 쓰레드**</span> 내에서도 <span class="red">**(I/O-bound 작업의) 동시성**</span>을 달성할 수 있다는 것을 알게 되었다. 이는 어떻게 가능한 것일까? 이는 시스템 레벨에서는 **I/O 연산이 완전히 동시에(concurrently) 수행**되기 때문이다.
> 

소켓이란 **네트워크를 통해 데이터를 주고 받는 것에 대한 low-level 추상화**이며, **우편함**이라고 생각하면 쉽다. 따라서 **(1) 바이트를 송신**하는 것과 **(2) 바이트를 수신**하는 것의 두 가지 연산을 지원한다.

즉, 어떤 URL로부터 데이터를 받아오기 위해서는, 해당 서버에 연결하기 위한 소켓을 열고, 소켓에 요청을 작성한 후, 서버가 결과를 반환하기를 기다린다. 이 과정을 그림으로 나타내면 다음과 같다.

![](/assets/img/posts/Python/More-About-Python/2024-02-28-12.png){: .w-75}

이러한 과정에서 “기다리는” 단계가 존재하기 때문에, 기본적으로 소켓은 **blocking**이다. 하지만 **OS 레벨에서는 소켓이 <span class="blue">non-blocking</span> 모드로 동작**할 수 있다. 따라서 **소켓에 요청을 작성한 후** 결과가 반환되기까지를 block 하며 기다리는 것이 아니라, **곧바로 다른 작업을 수행**할 수 있다. 그리고 서버로부터 **결과가 반환되면 OS가 그 사실을 알려주게 된다**.

이를 통해 `asyncio`는 파이썬이 **하나의 쓰레드에서** 수행되다가 I/O 연산을 마주치면, 그것을 OS에게 넘기고 다른 작업을 수행함으로써 **동시성을 달성**한다.

<br>

## 6. 이벤트 루프(Event Loop)

이벤트 루프란 `asyncio`에서 핵심적인 부분이다. 본래 이벤트 루프는 널리 사용되는 디자인 패턴으로, **무한 루프를 돌면서 이벤트나 메시지를 담고 있는 큐(queue)에서 하나씩 꺼내와 처리하는 형태**이다.

`asyncio`에서는 이벤트 루프가 <span class="shl">**task으로 이루어진 큐**</span>를 가지게 된다. 이때, **task는 코루틴을 감싼 것**이다.

> 코루틴은 (1) I/O-bound 작업을 마주치면 **실행을 중단**하고, (2) 이벤트 루프로 하여금 **다른 작업을 수행**하도록 한다.
{: .prompt-info}

<br>

**이벤트 루프의 iteration**에서 수행되는 동작을 기술하면 다음과 같다.

1. 실행되어야 할 task를 찾고, 그것을 **I/O 연산에 도달할 때까지 실행**한다.
2. I/O 연산에 도달하면 **해당 task를 중지**하고, **OS에게** I/O 작업이 완료될 때까지 소켓(socket)을 확인할 것을 요청한다.
3. I/O 작업이 완료될 때가지 기다리는 것이 아닌, 다른 **task를 찾아 실행**한다.

이때, 모든 iteration에서는 **완료된 I/O 작업이 있는지 확인**하고, 완료된 것이 있다면 **해당 작업과 관련된 task를 재개**한다.

![](/assets/img/posts/Python/More-About-Python/2024-02-28-13.png){: .w-75}

<br>

따라서 다음의 코드와 같이 CPU-bound 작업과 I/O-bound 작업이 섞인 task 3개를 실행하면, 그림과 같은 방식으로 동작한다.

```python
def make_request():
    cpu_bound_setup()
    io_bound_web_request()
    cpu_bound_postprocess()

task_one = make_request()
task_two = make_request()
task_three = make_request()
```

![](/assets/img/posts/Python/More-About-Python/2024-02-28-14.png)

<br>

## 7. `asyncio`

지금까지 살펴본 개념들을 토대로, `asyncio`를 설명하면 다음과 같다.

> `asyncio`는 **코루틴(coroutine)**들을 **싱글 쓰레드(single-threaded) 이벤트 루프(event loop)**에서 **비동기(asynchronous)** 방식으로 실행하며 **동시성(concurrency)**을 달성하도록 지원하는 라이브러리이다.
{: .prompt-tip}

이때, <span class="shl">**비동기 방식으로 동시성을 달성하는 과정**</span>은 다음과 같다.

1. 특정 task를 실행하다가 **I/O-bound 연산**을 마주치면 <span class="blue">**실행을 중지**</span>한다.
2. 백그라운드에서 I/O-bound 연산이 완료될 때까지 기다리는 것은 OS에게 맡기고, 곧바로 <span class="blue">**다른 task를 실행**</span>한다. I/O-bound 연산이 완료되면 OS가 알려준다.

<br>

> 여기까지 `asyncio`에 대해 알아보기 전 필요한 배경 지식들을 키워드 별로 다뤄보았다. 이어지는 포스팅에서는 기본적인 `asyncio`의 사용법과 핵심 개념들에 대해 알아보자.
> 

<br>

## References

- “Python Concurrency with Asyncio”, Ch01. Getting to Know `asyncio`
- <https://medium.com/from-the-scratch/wtf-is-synchronous-and-asynchronous-1a75afd039df>
- <https://www.guru99.com/cpu-core-multicore-thread.html>
- <https://da-nyee.github.io/posts/operation-system-synchronous-asynchronous/>
- <https://velog.io/@richpin/동기비동기-feat.-Python>
