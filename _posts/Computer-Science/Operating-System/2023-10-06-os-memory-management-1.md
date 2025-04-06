---
title: "[OS] 6. 메모리 관리: ① 개요"
date: 2023-10-06 16:50:00 +0900
categories: [Computer Science, Operating System]
tags: [cs, os, 운영체제, os 총정리, 메모리 관리, 주소]     # TAG names should always be lowercase
math: true
mermaid: true
---

> 중기 스케줄러(medium term scheduler, CPU 스케줄러)의 동작에 대해 알아보자.

<br>

## 1. 주소 바인딩

### 컴퓨터의 주소 체계

컴퓨터 시스템은 주로 **32bit 혹은 64bit** 주소 체계를 사용한다.

> ex) **32bit 주소 체계**를 사용할 경우, 이진수이므로  <span class="hl">$$2^{32}$$가지의 메모리 위치</span>를 구분할 수 있다. 컴퓨터는 byte 단위로 메모리 주소를 부여하므로, 이 경우 <span class="hl">$$2^{32}$$byte 만큼의 메모리 공간</span>에 서로 다른 주소를 할당할 수 있다.
{: .prompt-info}

일반적으로 이러한 메모리 공간을 <span class="hl">4KB(= $$2^{12}$$byte)</span> 크기의 <span class="hl">페이지(page)</span>라는 단위로 계층적으로 구역을 나누어 관리한다.

이때, **페이지 내에서 byte 별 위치 구분**을 위해서는 **12bit**가 필요하다. 따라서 32bit 주소 중 하위 12bit를 페이지 내의 주소를 나타내는 데에 사용한다.

<br>

### 주소의 종류

- **논리적 주소(logical address); 가상 주소(virtual address)**
    - 프로그램이 실행을 위해 메모리에 적재되면 생성되는 프로세스의 독자적인 주소 공간이다.
    - CPU는 이에 근거하여 명령을 실행한다.
    - 각 프로세스마다 독립적으로 할당되며, **0번지부터** 시작된다.
- **물리적 주소(physical address)**
    - 물리적 메모리에 실제로 올라가는 위치이다.
    - 일반적으로 낮은 주소 영역에는 운영체제가, 높은 주소 영역에는 사용자 프로세스들이 적재된다.

<br>

### 주소 바인딩 (Address Binding)

> 프로세스의 **logical address**를 **physical address**로 연결시켜주는 작업

#### Logical Address → Physical Address로 바인딩해야 하는 이유
    
프로세스가 실행되기 위해서는 해당 프로그램이 physical memory에 위치해야 하나, **CPU는 logical address로 메모리를 참조**하므로 해당 logical address가 physical memory의 어디에 mapping 되는지 알아야 하기 때문이다.
    
#### 주소 바인딩의 세 가지 방식
    
> 프로그램이 적재되는 **physical address가 결정되는 시점**에 따라 분류
{: .prompt-info}

1. **compile time binding**
    - **프로그램을 compile 하는 시점**에 physical address가 결정된다.
    - 프로그램이 절대주소로 적재되므로 절대코드(absolute code)를 생성하는 바인딩 방식이라고도 한다.
    - 프로그램의 physical address를 변경하고 싶다면 compile을 다시 해야 하므로, 현대의 시분할 환경에서는 잘 사용하지 않는다.
2. **load time binding**
    - **프로그램의 실행이 시작될 때**에 physical address가 결정된다.
    - **loader**(사용자 프로그램을 메모리에 적재시키는 프로그램)의 책임하에 physical address가 부여되며, **프로그램이 종료될 때까지 physical address가 고정**된다.
    - compiler가 재배치 가능 코드를 생성한 경우에 가능하다.
3. **run(execution) time binding**
    - CPU가 **logical address를 참조**할 때마다 **address mapping table**을 통해 binding을 점검하는 방식이다.
    - 프로그램이 실행을 시작한 후에도 그 프로그램의 **physical address가 변경**될 수 있다.
    - **base(relocation) register**와 **limit register**를 포함하는 **MMU**의 지원이 필요하다.

<br>

### MMU Scheme (Memory Management Unit)

> 여기에서는 **프로그램의 주소공간**이 physical memory의 한 장소에 **연속적으로 적재**되는 것으로 가정한다. **페이징 기법**과 같이 **불연속적으로 적재**되는 경우, 페이지 테이블(page table)을 사용하는 등 다른 방법을 도입해야 한다.

#### MMU가 필요한 이유
- **MMU**는 logical address를 physical address로 mapping 해주는 하드웨어 장치이다.
    
    ![](/assets/img/posts/Computer-Science/Operating-System/2023-10-06-01.jpeg){: .w-50}
- **사용자 프로그램과 CPU**는 <span class="red">**logical address만**</span> 다룬다.
- 다중 프로그래밍 환경에서는 하나의 주소 체계를 가지는 **physical memory에 여러 프로세스가 적재**되어 있으므로, 다음의 조치가 필요하다.
    1. **문맥교환**으로 CPU에서 수행 중인 프로세스가 바뀔 때마다 **base register의 값**을 그 프로세스에 해당하는 값으로 **재설정**한다.
    2. **memory protection**을 위해 **limit register**를 사용하여 프로세스가 자신의 주소 공간을 넘어서는 메모리 참조를 하려고 하는지를 체크한다.
 
#### 주소 변환 과정
    
> if <span class="shlp">**logical address** < **limit register**</span> → <span class="shlp">**logical address** + **base register**</span> → <span class="shlp">**physical address**</span>
{: .prompt-info}

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-06-02.jpeg){: .w-75}

| --- | --- |
| **logical address** | CPU가 요청하는 주소값 (= 일종의 **offset**) |
| <span class="hl">**limit register**</span> | 현재 CPU에서 실행 중인 프로세스의 **logical address의 최댓값** (= 프로세스의 크기) |
| <span class="hl">**base register**</span> | 현재 CPU에서 실행 중인 프로세스의 **physical memory** 시작 주소 |

#### Memory Protection을 달성하는 방법
 - CPU가 요청한 프로세스의 **logical address**가 **limit register** 내에 저장된 값보다 작은지 확인한다.
 - 작다면, **logical address + base register → physical address**를 구한 다음, 해당 위치에 접근하도록 허락한다.
 - 크다면, **trap**을 발생시켜 해당 프로세스를 강제 종료한다.

<br>

---

## 2. 메모리 관리와 관련된 용어

### 동적 로딩 (Dynamic Loading)

- 프로세스가 시작될 때 해당 프로세스의 전체 주소 공간을 메모리에 적재하는 것이 아닌, **실행에 필요한 부분이 요청될 때마다 그 부분만 메모리에 적재**하는 방식이다.
    
    > 실제 프로그램 코드 중 상당 부분은 가끔씩만 사용 되는 방어용 코드이므로, 전부 메모리에 적재하게 되면 메모리가 낭비될 수 있다.
    > 
- 여러 프로그램이 동시에 메모리에 적재되어 수행되는 **다중 프로그래밍(multi-programming)** 환경에서 **메모리 효율성**을 높일 수 있다.
- 운영체제의 특별한 지원 없이 프로그램 자체에서 구현이 가능하며, 운영체제가 라이브러리를 통해 지원할 수도 있다.

<br>

### 동적 연결 (Dynamic Linking)

- **linking**: 소스 코드를 컴파일하여 생성된 **object file**과 이미 컴파일된 **library file**을 묶어 하나의 실행 파일을 생성하는 과정
- **static linking**: object file과 library file을 모두 합쳐 실행 파일을 생성하는 기법
    - 실행 파일의 크기가 상대적으로 크다.
    - 동일한 library를 각 프로세스의 주소 공간에 개별적으로 적재하므로 physical memory가 낭비된다.
- **dynamic linking**: object file과 library file 사이의 linking을 프로그램 실행 시점까지 지연시키는 기법
    - 실행 파일에 library 코드가 포함되지 않으며, 프로그램이 실행되어 library 함수를 호출할 때 linking이 이루어진다.
    - **실행 파일의 library 호출 부분**에, 해당 library 위치를 찾기 위한 **stub**이라는 작은 코드를 작성한다.
        
        > **stub**을 통해 **해당 library가 메모리에 이미 존재하는지** 살펴본다.
        > 
        > - 존재할 경우, 그 주소의 위치에서 직접 참조한다.
        > - 존재하지 않을 경우, 디스크에서 dynamic library file을 찾아 메모리에 적재하여 수행한다.
    - 다수의 프로그램이 **공통으로 사용하는 library를 메모리에 한 번만 적재**하므로 **메모리 사용의 효율성**을 높일 수 있다.
    - 운영체제의 지원을 필요로 한다.

<br>

### 중첩 (Overlays)

- **프로세스의 주소 공간을 분할**하여 **실제 필요한 부분만**을 메모리에 적재하는 기법이다.
- 운영체제의 지원 없이 프로그래머에 구현되어야 했으므로 manual overlays라고도 부른다.

<br>

> **동적 로딩 vs. 중첩**
>
> | 구분  | 환경  | 목적  |
> | ---- | ---- | ---- |
> | **동적 로딩**  | 다중 프로그래밍 환경 | 메모리에 더 많은 프로세스를 동시에 올려놓고 실행하기 위함<br>(메모리 이용률 ⬆️) |
> | **중첩** | 단일 프로세스 환경   | physical memory 용량보다 큰 프로세스를 실행하기 위함<br>(physical memory 크기가 제한적이었던 초창기 컴퓨터 시스템)|


<br>

### 스와핑 (Swapping)

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-06-03.jpeg){: .w-50}

- 프로세스가 종료되어 그 주소 공간을 디스크로 내쫓는 것이 아닌, 특정한 이유로 **수행 중인 프로세스의 주소 공간을 일시적으로 디스크의 swap area에 일시적으로 내려놓는 것**을 의미한다.
    
    > **swap area (backing store)**
    > 
    > - 디스크 내에 **file system과 별도**로 존재하는 영역이다.
    > - **file system**: 비휘발성 저장공간
    > - **swap area**: 프로세스 수행 중, 디스크에 일시적으로 저장하는 공간
    {: .prompt-tip}
    
- [디스크 → 메모리]를 **swap in**, [메모리 → 디스크]를 **swap out**이라고 한다.

#### Swapping이 일어나는 과정
1. **중기 스케줄러**(medium-term scheduler)에 의해 swap out 시킬 프로세스를 선정한다.
2. 선정된 프로세스에 대해서, 현재 메모리에 적재되어 있는 **주소 공간 전체**를 **디스크의 swap area**에 swap out 시킨다.

#### Swapping의 특징
- swapping을 통해 **메모리에 존재하는 프로세스의 수**, 즉, **다중 프로그래밍의 정도**(degree of multi-programming)를 조절한다.
    - 너무 많은 프로그램이 메모리에 동시에 적재되면, **프로세스당 할당되는 메모리의 크기**가 지나치게 작아져 시스템 전체 성능이 떨어진다.
    - swapping을 통해 메모리에 적재되어 있는 프로그램들에게 **실행에 필요한 적절한 메모리 공간을 보장**한다.
- **swapping에 소요되는 시간**의 대부분은 디스크 섹터에서 실제 데이터를 read/write 하는 **전송시간(transfer time)**이 차지한다.
    
    > 보통 디스크 swap area에 프로세스의 주소 공간이 **순차적으로 저장**되므로, 디스크의 탐색시간(seek time)이나 회전지연시간(rotational latency)은 크지 않다.
    > 

#### 주소 바인딩 방식에 따른 Swapping

| 주소 바인딩 방식                           | swapping 동작                                                             |
| ------------------------------------------ | ------------------------------------------------------------------------- |
| compile time binding,<br>load time binding | swap-out 된 프로세스는 원래 존재하던 메모리 위치로 다시 swap-in 되어야 함 |
| run time binding                           | 추후 빈 메모리 영역 아무 곳에나 프로세스를 올릴 수 있음                   |

<br>

---

## 3. 물리적 메모리의 할당 방식

> **physical memory**는 다음과 같이 구성된다. 
> 
> 이번 장에서는 physical memory 중 **사용자 프로세스 영역의 관리 방법**에 대해 알아보자!
> 
> | 구분 | 위치 |
> | --- | --- |
> | 운영체제 상주 영역 | 낮은 주소 영역 |
> | 사용자 프로세스 영역 | 높은 주소 영역 |

<br>

### [1] 연속 할당 방식 (Contiguous Allocation)

> 각각의 프로세스를 physical memory의 연속적인 공간에 올리는 방식
>
> → <span class="red">**physical memory는 분할**</span>하고, <span class="red">**프로세스는 통째로(연속적으로)**</span> 적재한다.
{: .prompt-info}

#### ① 고정 분할 방식 (Fixed Partition Allocation)

> physical memory를 고정된 크기의 partition으로 미리 나누어두는 방식

- 이 방식에서는 partition의 크기는 모두 동일할 수도, 서로 다를 수도 있다.
- 하나의 partition에는 하나의 프로그램만을 적재할 수 있다.
- **단점**
    1. 동시에 메모리에 올릴 수 있는 프로그램의 수가 고정되어 있다.
    2. 수행 가능한 프로그램의 최대 크기가 제한된다.
    3. external fragmentation과 internal fragmentation이 모두 발생할 수 있다.
        
        
    | fragment 종류                               | 크기 비교           | 설명                                                                                                                                                   |
    | ------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | **외부 단편화**<br>(external fragmentation) | program > partition | - 어떠한 프로그램에게도 배정되지 않는 공간<br>- partition 크기보다 작은 크기의 프로그램이 도착한다면 적재 가능                                         |
    | **내부 단편화**<br>(internal fragmentation) | program < partition | - 하나의 partition 내부에서 남는 공간<br>- 특정 프로그램에게 이미 배당된 공간<br>- fragmentation 크기보다 작은 프로그램이 있어도 공간을 활용할 수 없음 |

#### ② 가변 분할 방식 (Variable Partition Allocation)

> physical memory에 적재되는 프로그램의 크기에 따라 partition의 크기 및 개수가 동적으로 변하는 방식

- internal fragmentation은 발생하지 않으나, external fragmentation은 발생할 수 있다.
    
    > 이미 메모리에 존재하는 프로그램이 종료되면 중간에 빈 공간이 발생하게 되고, 그 공간의 크기가 새롭게 시작되는 프로그램의 크기보다 작을 경우 external fragmentation이 된다.
    {: .prompt-tip}
    
- <span class="hl">**동적 메모리 할당 문제 (dynamic storage-allocation problem)**</span>
    - 프로세스를 **physical memory 내 가용 공간 중 어디에 올릴 것인지 결정**하는 문제이다.
    - 프로세스의 **주소 공간 전체(= 크기 n)**를 적재할 수 있는 가용 공간을 찾아야 한다.
        - 가용 공간은 사용되지 않은 메모리 공간으로, 여러 곳에 분산되어 존재할 수 있다.
        - 운영체제는 이미 사용 중인 메모리 공간과 사용하고 있지 않은 가용 공간에 대한 정보를 유지하고 있다.
    - **대표적인 해결 방법**
        
        > 실험 결과, **최초 적합**과 **최적 적합** 방식이 속도 및 공간 이용률 측면에서 효과적인 것으로 알려져 있다.
        
        1. <span class="hlp">**최초 적합 (first-fit)**</span>
            - 크기가 n 이상인 가용 공간 중 가장 먼저 찾은 곳에 프로세스를 할당한다.
            - **장점** - 시간적인 측면에서 효율적이다.
        2. <span class="hlp">**최적 적합 (best-fit)**</span>
            - 크기가 n 이상인 가장 작은 가용 공간을 찾아 그곳에 프로세스를 할당한다.
            - **장점** - 공간적인 측면에서 효율적이다.
            - **단점** - 가용 공간 리스트가 크기순으로 정렬되어 있지 않다면 전체를 탐색해야 하므로 시간적 overhead가 발생하고, 다수의 매우 작은 가용 공간들이 생성될 수 있다.
        3. <span class="hlp">**최악 적합 (worst-fit)**</span>
            - 가용 공간 중 가장 크기가 큰 곳에 프로세스를 할당한다.
            - **단점** - 모든 가용 공간 리스트를 탐색해야 하는 overhead가 발생하고, 상대적으로 더 큰 프로그램을 담을 수 있는 가용 공간을 빨리 소진한다.
- <span class="hl">**compaction**</span>: external fragmentation 문제를 해결하는 방법
    - physical memory에서 사용 중인 영역과 가용 공간을 각각 한쪽으로 모아서 하나의 큰 가용 공간을 만드는 방법이다.
    - 현재 수행 중인 프로세스의 메모리상 위치를 이동시켜야 하므로 비용이 많이 드는 작업이고, run time binding 방식이 지원되는 환경에서만 수행될 수 있다.

<br>

### [2] 불연속 할당 방식 (Noncontiguous Allocation)

> 하나의 프로세스를 physical memory의 여러 영역에 분산해 적재하는 방식
>
> → <span class="red">**프로세스를 분할**</span>하여 적재한다.
{: .prompt-info}

#### ① 페이징 기법 (Paging)

프로세스의 주소 공간을 **동일한 크기**의 **page** 단위로 잘라 메모리에 적재하는 방법


#### ② 세그멘테이션 기법 (Segmentation)

프로세스의 주소 공간을 코드, 데이터, 스택 등 의미 있는 **segment** 단위로 나누어 메모리에 적재하는 방법 (**크기가 일정하지 X**)


#### ③ 페이지드 세그멘테이션 기법 (Paged Segmentation)

하나의 **segment**를 **다수의 동일 크기 page**로 나누어 메모리에 적재하는 방법


<br>

> 이어지는 포스팅에서 불연속 할당 방식의 세 가지 기법에 대해 다룬다.
{: .prompt-tip}

<br>

---

## References
- “운영체제와 정보기술의 원리(반효경 저)”, 7장 메모리 관리
