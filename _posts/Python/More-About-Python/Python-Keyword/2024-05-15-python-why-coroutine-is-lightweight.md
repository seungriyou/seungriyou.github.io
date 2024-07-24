---
title: "[Python] Coroutine이 Thread 보다 가벼운/빠른 이유"
date: 2024-05-15 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, keyword, coroutine, thread]
math: true
---

흔히 coroutine은 thread보다 가볍다(lightweight), 혹은 빠르다(fast)고 알려져 있다. 그 이유는 무엇일까?

<br>

## [1] 자원 효율성 (Resource Efficiency)

coroutine은 thread에 비해 <span class="shl">**메모리와 시스템 자원**</span>을 훨씬 적게 사용한다. 각 **thread**는 **별도의 스택 영역**을 가지기 때문에 일정량의 메모리를 필요로 하기 때문에, 많은 thread를 생성하면 메모리 사용량이 급격히 증가할 수 있다.

반면, **coroutine**은 **스택 영역이 없어서** 각 coroutine에 대해 메모리를 할당할 필요가 없으며, 수천 개의 coroutine을 생성하더라도 메모리 측면에서 크게 부담이 가지 않는다.

> coroutine은 스택 영역을 가지지 않는다.
> 
> > ref: [링크1](https://stackoverflow.com/questions/70339355/are-python-coroutines-stackless-or-stackful) / [링크2](https://medium.com/@jooyunghan/stackful-stackless-%EC%BD%94%EB%A3%A8%ED%8B%B4-4da83b8dd03e) / [링크3](https://langdev.stackexchange.com/questions/697/what-are-the-benefits-of-stackful-vs-stackless-coroutines)
> 
> - 만약 stackful coroutine이라면 coroutine에서 다른 함수를 호출하여 그 함수에서 coroutine을 suspend 할 수 있다.
> - 하지만 파이썬의 coroutine은 stackless coroutine이며, 그 대신 이벤트 루프에 실행을 의존한다. 이로 인해 간결성과 성능 측면에서 장점을 가지게 된다.
{: .prompt-info}

<br>

## [2] 문맥 교환 비용 (Context Switching Overhead)

**thread** switching은 context switching을 포함하며, 이는 **운영체제와의 interaction**을 필요로 하므로 스케줄링과 스택 관리에 상당한 overhead가 발생한다.

하지만 **coroutine**은 파이썬 인터프리터(**이벤트 루프**)에 의해 관리되므로 사용자 수준에서 동작하며, switching 시 운영체제 수준의 개입이 필요하지 않기 때문에 훨씬 <span class="shl">**빠르게 switching**</span>이 가능하다. 이는 coroutine이 동일한 단일 thread 내에서 실행되기 때문이다.

<br>

## [3] 병행 처리 모델 (Concurrency Model)

**thread**는 운영체제 스케줄러가 언제 thread를 중지 및 실행할지 결정하는 **preemptive multitasking**을 채택한다.

반면, **coroutine**은 <span class="shl">**cooperative multitasking**</span>을 채택하여 프로그램의 특정 지점에서 **“자발적으로”** 제어를 양도한다. 이렇게 제어를 양도하는 것은 I/O 작업을 wait 할 때 주로 발생한다. 이러한 cooperative multitaksing은 **병행 작업 관리를 단순화**하며 overhead를 줄인다.

<br>

## [4] 디버깅 및 테스트의 단순화 (Simplified Debugging and Testing)

coroutine을 사용하면 multi-threading의 race condition과 deadlock을 피할 수 있기 때문에 디버깅과 테스트가 더 쉬워진다. 이는 coroutine을 사용할 때는 제어를 양도하는 지점이 개발자에 의해 명시적으로 정의되기 때문이다.


<br>

이러한 이유로 coroutine은 thread에 비해 가벼우며, 많은 I/O-bound 작업을 concurrent 하게 처리해야 하는 상황에서 유리하다.

<br>

## References

- <https://superfastpython.com/coroutines-faster-threads/>
- <https://superfastpython.com/asyncio-coroutines-faster-than-threads/>
