---
title: "[Python] 파이썬의 동작 원리: 인터프리터 언어, 컴파일 언어, 그리고 CPython"
date: 2023-12-09 19:50:00 +0900
categories: [Python, More About Python]
tags: [python, cpython, interpreter, compiler, byte code]
math: true
---

## 1. 인터프리터 언어 vs. 컴파일 언어

### 1-1. 인터프리터 언어(Interpreted Language)

- 인터프리터를 통해 프로그래밍 언어의 소스 코드를 한 줄씩 읽어들이며 곧바로 실행하는 언어이다. 따라서 별도의 실행 파일이 생성되지 않는다.
    
    > **인터프리터(interpreter)**
    > 
    > high-level 언어로 작성된 프로그램을 한 줄씩 읽어들이며 low-level 언어로 번역하고, 즉시 실행시키는 프로그램
    {: .prompt-info}

- 컴파일을 수행하지 않으므로 별도의 컴파일 시간이 필요하지 않고 프로그램 수정이 간단하며 해당 플랫폼에 지원하는 인터프리터만 준비되어 있다면 곧바로 실행이 가능하다.

- 실행 시에 줄 단위로 번역 및 실행이 수행되므로 실행이 느릴 수 있으며, 런타임 환경에서 코드가 실행되기 때문에 프로그램 실행 전에 버그를 발견하기 어렵다.

- ex) Python, JavaScript, SQL, Ruby

<br>

### 1-2. 컴파일 언어(Compiled Language)

- 컴파일러를 통해 컴파일 타임에 전체 소스 코드를 한 번에 기계어로 변환 후 실행 파일을 만드는(= 빌드) 언어이다.
    
    > **컴파일러(compiler)**
    > 
    > high-level 언어로 작성된 소스코드를 low-level 언어로 번역하고, linking을 통해 실행 가능한 실행 파일을 생성하는 프로그램
    {: .prompt-info}

- 컴파일 단계와 실행 단계가 분리되어 있다. 컴파일은 단 한 번만 수행하며, 실행은 컴파일 단계를 거치지 않고 실행 파일을 실행함으로써 할 수 있다.

- 실행 시에는 컴파일을 하지 않아도 되므로 코드 실행 속도가 상대적으로 빠르나, 소스 코드의 규모가 클 경우 컴파일 시간이 오래 걸릴 수 있으며 컴파일 시 지정된 플랫폼에 최적화되도록 컴파일 되므로 특정 플랫폼에서는 동작하지 않을 수 있다.

- ex) C, C++, C#, Go

<br>

### 1-3. 총 정리

| | 컴파일 언어  | 인터프리터 언어 |
| ---- | ---- | ---- |
| **기계어 번역 단위** | 프로그램 전체 | 소스 코드 한 줄 |
| **기계어 번역 시점** | 컴파일 타임 (실행 전 단계) | 런타임 |
| **기계어 번역 수행** | **컴파일**: 실행에 앞서 단 한 번 수행 | **인터프리트**: 코드 라인 실행마다 반복 수행 |
| **실행 파일 생성 여부** | O  | X |
| **번역 속도** | ⬇️  | ⬆️   |
| **실행 속도 (경향성)**  | ⬆️ (컴파일 / 실행 단계 분리)  | ⬇️ (인터프리트 단계 반복 수행)  |
| **특징**  | 미리 컴파일 수행하므로 런타임 전에 오류 발견 가능 | 동적 타이핑 지원 |
| **예시 언어** | C, C++, C#, Go 등 | Python, JavaScript 등 |

<br>

## 2. 파이썬 코드 실행 시 동작 원리

파이썬은 대표적인 인터프리터 방식 언어로 알려져 있다. 그렇다면 파이썬 코드가 인터프리터를 거치면 곧바로 실행되는 코드로 변환되는 것일까?

아니다. 실제로는 인터프리터 내부에서 <span class="shl">**컴파일러(compiler)가 코드를 바이트 코드(byte code)로 컴파일**</span>하고, 이렇게 생성된<span class="shl"> **바이트 코드를 파이썬 가상머신(Python virtual machine)이 하나씩 읽어들여 실행**</span>함으로써 동작하게 된다.

> 엄밀히 말하면 파이썬은 **컴파일 방식 + 인터프리터 방식** 언어라고 볼 수 있을 것이다!
{: .prompt-danger}

<br>

> **파이썬 코드를 실행할 때 수행되는 동작**
> 
> 1. 컴파일러가 소스 코드를 받는다.
> 2. 컴파일러는 소스 코드의 각 라인의 문법(syntax)을 확인한다.
> 3. 만약 컴파일러가 에러를 마주치면, syntax error 메시지와 함께 번역 과정을 중단한다.
> 4. 그렇지 않다면, 컴파일러는 소스 코드를 바이트 코드로 변환한다.
> 5. 바이트 코드는 파이썬 가상머신(PVM)으로 보내진다.
> 6. 바이트 코드와 함께 입력값과 라이브러리 모듈이 PVM에 입력된다.
> 7. PVM은 바이트 코드를 실행하며, 그 과정에서 에러가 발생하면 runtime error 메시지를 표시한다.
> 8. 그렇지 않다면, 결과 값을 출력한다.

![](/assets/img/posts/Python/More-About-Python/2023-12-09-01.png){: .w-75}
_ref: <https://www.c-sharpcorner.com/article/why-learn-python-an-introduction-to-python/>_

<br>

> 파이썬은 인터프리터 언어이지만, 실제로는 컴파일 과정을 수행한다! 즉,
> 
> 1. 바이트 코드가 만들어지는 과정까지는 **컴파일 방식**
> 2. 바이트 코드를 하나씩 실행하는 과정은 **인터프리터 방식**
> 
> 이라고 할 수 있다.
{: .prompt-tip}

<br>

### 2-1. 바이트 코드(Byte Code)

바이트 코드란, CPU가 아닌 **가상머신(virtual machine)**에서 실행되도록 컴파일 된 **중간 코드**이다.

> 컴파일러는 반드시 기계어로만 변환하는 것은 아니다. 바이트 코드와 같은 중간 형태로 변환하는 것 또한 컴파일이며, 파이썬의 컴파일러는 이러한 역할을 수행한다.
> 

바이트 코드는 보통 low-level 언어인 기계어보다는 더 추상적이며, high-level 언어보다는 덜 추상적이고 더 컴퓨터 중심적이다.

또한, 바이트 코드는 기계어와는 다르게 특정 플랫폼에 종속적이지 않고(= **platform independent**), 한 번 컴파일된 바이트 코드는 **어떤 환경에서나 동일하게 실행될 수 있다**는 특징이 있다.

<br>

그렇다면 파이썬은 **왜 소스 코드를 바이트 코드로 컴파일 하는 과정을 도입**했을까?

파이썬은 속도를 개선하기 위해 **소스 코드를 캐싱**하는 방법을 도입하였는데, 이때 인터프리터가 high-level 언어의 소스 코드를 직접 읽는 대신 그보다는 **읽기 쉬운 중간 코드**를 읽는 방식으로 채택하였기 때문이다.

이때 사용하는 중간 코드로써 바이트 코드를 사용한다.

> 확장자가 `.py`인 파이썬의 소스 코드 파일과는 다르게 바이트 코드로 작성된 캐시 파일의 확장자는 `.pyc`이다.
{: .prompt-info}

![](/assets/img/posts/Python/More-About-Python/2023-12-09-02.png)

<br>

### 2-2. 파이썬 인터프리터의 공식 구현체: CPython

파이썬 코드를 실행할 때 수행되는 동작은 파이썬의 여러 구현체(implementation)마다 약간 다르게 구현되어 있다. 그 중 파이썬의 공식 구현체로 사용되는 것은 CPython으로, C로 작성되어 있으며 다음의 그림과 같이 동작한다.

![](/assets/img/posts/Python/More-About-Python/2023-12-09-03.png){: .w-75}
_ref: <https://www.c-sharpcorner.com/article/why-learn-python-an-introduction-to-python/>_

<br>

## References

- <https://devocean.sk.com/blog/techBoardDetail.do?ID=164537>
- <https://blog.insightbook.co.kr/2022/09/19/cpython-파헤치기-따라-하면서-이해하는-파이썬-내부의-동/>
- <https://coding-factory.tistory.com/303>
- <https://noodabee.tistory.com/entry/컴파일러-언어와-인터프리터-언어>
- <https://dingrr.com/blog/post/compiled-vs-interpreted-언어>
- <https://www.c-sharpcorner.com/article/why-learn-python-an-introduction-to-python/>
- <https://blog.naver.com/pisibook/221711169180>
- <https://miintto.github.io/docs/python-pycache>
