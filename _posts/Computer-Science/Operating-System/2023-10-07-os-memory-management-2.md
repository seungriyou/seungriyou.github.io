---
title: "[OS] 6. 메모리 관리: ② 페이징 기법 (Paging)"
date: 2023-10-07 16:50:00 +0900
categories: [Computer Science, Operating System]
tags: [cs, os, 운영체제, os 총정리, 메모리 관리, 주소, 페이징, paging]     # TAG names should always be lowercase
math: true
mermaid: true
---

> 불연속 할당 방식의 세 가지 기법 중 **페이징 기법**에 대해 알아보자.

<br>

## 1. 페이징 기법이란?

페이징(paging) 기법이란 프로세스에게 물리 메모리를 할당하는 방법 중 <span class="hl">불연속 할당 방식</span>에 해당하는 기법으로, 프로세스를 <span class="hl">동일한 크기의 page 단위</span>로 나누어 <span class="hlp">물리 메모리</span> 또는 <span class="hlp">스왑 영역(swap area, backing store)</span>의 서로 다른 위치에 저장하는 방식이다.

> **물리 메모리 관리 방법**에 대해서 알고 싶다면? [**→ [OS] 6. 메모리 관리: ① 개요**](/posts/os-memory-management-1/)
{: .prompt-tip}

<br>

### 특징

- **프로세스**와 **물리 메모리**를 각각 다음의 단위로 분할한다. 이때, <span class="hl">**두 단위의 크기는 동일**</span>하다.
    
    | 분할 대상 | 단위 |
    | ----- | ----- |
    | 프로세스 (process)  | **페이지** (page)  |
    | 물리 메모리 (physical memory) | **프레임** (frame) |
- 물리 메모리에 **빈 frame이 있으면 어떤 위치이든 사용**될 수 있으므로, 연속 할당 방식에서 발생했던 동적 메모리 할당 문제가 발생하지 않는다.
    
    > 주소 변환 절차가 page 단위로 이루어져야하므로!
    > 
- **external fragmentation**은 발생하지 않으나, 프로세스의 크기가 항상 page 크기의 배수는 아니므로 마지막 page에서는 **internal fragmentation**이 발생할 수 있다.
- 프로세스가 분할되어 위치하므로, CPU가 사용하는 logical address를 physical address로 변환하는 과정에서 **페이지 테이블(page table)**을 사용한다.
    
    > 프로세스가 통째로 위치하는 **연속 할당 방식**의 경우, logical address를 physical address로 변환하는 과정에서 **base / limit register**를 이용하여 범위 확인만 하면 된다.
    {: .prompt-tip}
    

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-01.jpeg){: .w-50}

<br>

---

## 2. 주소 변환 기법 (Address Translation)

> CPU가 사용하는 logical address를 physical address로 변환하는 방법은?
> 

### Logical Address의 구조

CPU가 사용하는 **logical address(virtual address)**는 페이지 번호($p$)와 페이지 오프셋($d$)으로 구성된다.

| 구성 | 용도 | 특징 |
| --- | --- | --- |
| $p$ (page **번호**) | - **page table** 접근 시 **index로** 사용<br>- **TLB** 접근 시 **참조 값**으로 사용 | page table의 entry에는 해당 page의 physical memory 상<br>**base address(= 시작 위치)**가 들어있다. |
| $d$ (page **오프셋**) | 하나의 **page** 내에서의 **변위**로 사용 | base address 값에 **더하면** 대응되는 physical address를<br>구할 수 있다. |

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-02.jpeg){: .w-75}

<br>

### 페이지 테이블 (Page Table)

- **각 프로세스 마다** 주소 변환(virutal address → physical address)을 위해 가지고 있는 자료구조이다.
    
    > 물리 메모리에 위치한다.
    > 
- 해당 <span class="hl">프로세스의 몇 번째 page</span>가 <span class="hl">물리 메모리의 몇 번째 frame</span>에 위치하는지에 대한 정보를 담고 있다.
    
    > 따라서 **page의 개수**만큼 entry를 가진다.
    > 
- **운영체제**는 현재 CPU에서 **실행 중인 프로세스의 page table**에 접근하기 위해 다음의 두 레지스터를 사용한다.
    
    
    | 레지스터                          | 저장하는 값                            |
    | --------------------------------- | -------------------------------------- |
    | page table base register (PTBR)   | 메모리 내에서의 page table의 시작 위치 |
    | page table length register (PTLR) | page table의 크기                      |
- paging 기법의 경우, 매번 **메모리에 두 번 접근**해야 하는 오버헤드가 존재한다.
    1. 주소 변환을 위해 <span class="hl">page table</span>에 접근
    2. 변환된 주소를 이용하여 <span class="hl">실제 데이터</span>에 접근
    
    > TLB를 통해 오버헤드를 줄일 수 있다!
    {: .prompt-info}
    

<br>

---

## 3. TLB (Translation Look-aside Buffer)

### TLB란?

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-03.jpeg){: .w-75}

- 메모리 접근 속도를 향상시키기 위한 **고속의 주소 변환용 하드웨어 캐시**이다.
- page table의 전체 내용을 담을 수는 없으며, 빈번히 참조되는 page에 대한 주소 변환 정보만을 담게 되므로 요청되는 **page에 대한 정보가 TLB 내에 존재하지 않을 수도** 있다.
    1. TLB에 존재한다면 **곧바로** 대응되는 물리 메모리의 frame 번호를 얻을 수 있다.
    2. TLB에 존재하지 않는다면 **main memory**에 있는 **page table**로부터 frame 번호를 알아내야 한다.
- 주소 변환 정보는 프로세스마다 다르므로, **문맥 교환** 시 이전 프로세스의 정보를 담고 있던 **TLB 내용을 모두 지워야** 한다.
- **TLB**에는 <span class="hl">**page 번호**</span>와 <span class="hl">**frame 번호**</span>를 쌍으로 저장해야 한다.
    
    > **page table**에서는 page 번호를 인덱스로 사용하여 해당하는 항목에 곧바로 접근할 수 있으므로 **page 번호**가 저장될 필요가 없었다.
    {: .prompt-tip}
    
- TLB의 모든 entry를 전부 찾아보는 것은 오버헤드이므로, **parallel search**(= 모든 항목을 동시에 탐색)가 가능한 **associative register**를 사용한다.
    
    > **associative memory register**
    > 
    > - key & value 로 이루어져 있으며, 각 **항목마다 비교 회로**를 가지고 있으므로 **모든 key는 동시에 비교**될 수 있다. (→ **parallel search**)
    > - ex) TLB, Inverted Page Table
    {: .prompt-info}
    
- **평균적인 메모리 접근 시간 (Effective Access Time, EAT)**
    
    $$
    EAT=(1+\epsilon)\alpha + (2+\epsilon)(1-\alpha)=2+\epsilon-\alpha
    $$
    
    | ----- | ----- |
    | 메모리 접근 시간 | 1 |
    | associative register 접근 시간 | $\epsilon$ |
    | 요청된 page에 대한 정보가 associative register에 존재할 확률 | $\alpha$   |

<br>

### <span class="hl">MMU, Page Table, TLB, Cache 총정리</span>

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-04.png){: .w-75}
_ref: <http://www.cs.rpi.edu/academics/courses/fall04/os/c12/index.html>_

#### `PA`(physical address)를 얻기 위한 과정
1. `VA`를 가지고 `TLB` 확인 
2. TLB에 없으면 `Page Table` 확인 (찾은 경우, TLB 업데이트)
3. Page Table에도 없으면 `Page Fault`
    
    > Page Fault 발생 시, disk에서 main memory로 읽어들여야 한다.
    > 

#### TLB 혹은 Page Table을 통해 `PA`를 얻은 후
1. `PA`를 가지고 `Cache`에 caching 된 데이터가 있는지 확인 (찾은 경우, 해당 데이터 가져오기)
2. Cache에 없으면 `Main Memory`에서 불러와서 Cache 업데이트
 
#### TLB vs. Cache
    
    
| 구분  | 캐싱 된 값 | 참조하는 값 | 찾는 값 |
| ----- | ----- | ----- | ----- |
| **TLB**   | <span class="pink">**address translation 정보**</span>가 caching 된 것 | page 번호   | frame 번호 (= **`PA`**) |
| **Cache** | <span class="blue">**실제 data**</span>가 caching 된 것 | `PA` | 실제 **data** |

#### TLB, Page Table vs. Cache
    
| 구분 | 목적 |
| ----- | ----- |
| **TLB, Page Table** | <span class="pink">**PA**</span>를 찾기 위함 (= **page 번호**에 대응되는 **frame 번호**를 찾기 위함) |
| **Cache**  | 실제 <span class="blue">**data**</span>를 찾기 위함 (= **PA**에 대응되는 **caching된 data**를 찾기 위함) |

#### MMU의 위치

> CPU — MMU — Cache
    
![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-05.png){: .w-75}

> **CPU로부터 전달 받는 값:**
> - `VA`
> - 현재 프로세스의 Page Table(= Memory에 위치)을 가리키는 register 값
{: .prompt-info}

#### 참조 흐름도
- **TLB Hit**
  
  ![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-06.png){: .w-50}
  
- **TLB Miss**
  
  ![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-07.png){: .w-50}
        

<br>

---

## 4. 계층적 페이징 (Multi-level Paging)

### 배경

32bit 주소 체계를 사용하는 컴퓨터를 예로 들어보자.

- **$2^{32}$byte(= 4GB)**의 주소 공간을 갖는 프로그램을 지원할 수 있다.
- **페이지의 크기**가 **4KB**라고 가정하면, **1M개**의 **page table entry**가 필요하다. (= 4GB/4KB)
- 각 **page table entry**가 **4byte**(= 32bit, page 시작 주소)라면, **한 process 당 page table**을 위해 **4MB** 크기의 메모리가 필요하다. (= 1M*4byte)

이처럼 수행 중인 프로세스의 수가 증가하게 되면, <span class="hl">전체 메모리의 상당 부분이 page table에 할애되어 실제로 사용 가능한 메모리 공간이 크게 줄어들기 때문</span>에 multi-level paging이 고안되었다.

<br>

### 특징

- **장점**
    1. **필요한 page table만 memory에 적재**하므로 메모리 공간의 소모를 줄일 수 있다.
    2. page table 확장이 용이하다.
- **단점**
    1. 접근해야 하는 page table의 수가 증가하여 **메모리 접근 횟수가 많아지므로** 접근시간이 크게 늘어난다.
        
        > → **TLB**의 중요성이 커진다!
        {: .prompt-warning}
        
        > **n 단계 page table**을 이용한다면, 실제 데이터에 접근하기 위한 것까지 **총 (n+1) 번의 메모리 접근**을 하게 된다. 이것에 유의하여 **EAT**를 계산해보면, n 단계 page table을 사용했을 때의 시간적 오버헤드를 산출해 볼 수 있다.
        {: .prompt-info}
        
    2. 구현이 복잡하다.

<br>

### 동작

- CPU 내부에는 **top-level의 PTBR(page table base register)**만 있으므로, 순차적으로 타고 내려가면서 찾는다.
- 이때, top-level page table entry의 **valid bit가 `0`**(= 하위 page table의 모든 entry의 valid bit가 `0`)이라면, **바로 page fault라고 판단**한다.

<br>

### 2단계 페이징 기법

#### Logical Address (ex. 32bit 주소 체계)
    
![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-08.png){: .w-50}

| 구성  | 크기  | 용도  | 찾는 정보  |
| ----- | ----- | ----- | ----- |
| $p_1$ | 10bit | top-level page table의 **index** | second-level page table의 address   |
| $p_2$ | 10bit | second-level page table의 **index** | 요청된 page가 존재하는 frame 위치   |
| $d$   | 12bit | 하나의 page 내 **변위** 값   | 실제 원하는 위치의 physical address |

- $d$: page 크기가 4KB(=$2^{12}$byte)이므로 12bit가 필요하다.
- $p_2$: second-level page table이 하나의 frame에 저장되어야 하므로 크기는 4KB여야 한다. 따라서 **1K(=$2^{10}$=4KB/4btye) 개의 entry**를 가지므로 이를 구분하기 위해 10bit가 필요하다.
- $p_1$: 32bit 주소 중 $d$와 $p_2$가 사용하고 남은 10bit를 사용한다.

#### Address Translation 과정
    
![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-09.png)
    

<br>

---

## 5. 역페이지 테이블 (Inverted Page Table)

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-10.jpeg){: .w-75}

|    | 개수  | entry | 특징 |
| ---- | ---- | ---- | ---- |
| **page table** | **process 마다** 하나씩 존재 | <span class="red">**page**</span> 마다 entry 하나  | 모든 process의 모든 page에 대해 page table entry를<br>구성해야 하므로 메모리 공간의 낭비 발생 |
| **inverted page table** | **전체 system**에 하나 존재  | <span class="red">**frame**</span> 마다 entry 하나 | page table에 사용하는 메모리 공간을 줄일 수 있음 |

- **각 frame**마다 어느 process의 어느 page가 저장되었는지에 관한 정보를 보관한다.
- inverted page table의 entry는 **process 번호** (`pid`)와 **process 내 page 번호** (`p`)로 구성된다.
- page table **전체를 linear search** 해야하므로, **page table**을 메모리 대신 **associative register에 보관**하여 전체 항목에 대한 **parallel search**를 가능하게 함으로써 시간적 효율성을 제고한다.
    
    > TLB도 associative register 이다!
    > 

<br>

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-11.jpeg)

<br>

---

## 6. 공유 페이지

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-12.jpeg){: .w-75}

- **공유 코드(shared code)**: 메모리 공간을 효율적으로 사용하기 위한 목적으로 여러 프로세스에 의해 공통으로 사용될 수 있는 코드로, 다음의 특징을 가진다. (re-entrant code, pure code 라고 불리기도 한다.)
    - **read-only** 이다.
    - 모든 process의 **logical address space에서 동일한 위치**에 존재해야, 즉, **동일한 page 번호**를 가져야 한다.
- **공유 페이지(shared page)**: 공유 코드를 담고 있는 page로, 여러 프로세스에 의해 공유되므로 **physical memory에 한 번만 적재**된다.
- **사유 페이지(private page)**: 해당 프로세스의 logical address space 중 어떠한 위치에 있어도 무방하다.

<br>

---

## 7. 메모리 보호

> page table의 각 entry에는 **주소 변환 정보** 뿐만 아니라 **메모리 보호를 위한 정보**도 존재한다.

- **보호 비트 (protection bit)**: 각 page에 대한 접근 권한(ex. `RW-`, `R--` 등)
- **유효-무효 비트 (valid-invalid bit)**: 해당 page의 내용이 유효한지 여부
    
    
    | valid bit | 의미 | 접근 허용 여부 |
    | --- | --- | --- |
    | valid (`1`) | 해당 frame에 그 page가 존재함 | O |
    | invalid (`0`) | process가 해당 frame을 사용하지 않거나, 해당 page가 backing store에 존재함 | X |

<br>

### Page Table Entry

> page table entry를 구성하는 요소에 대해 알아보자.
> 

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-13.png)
_ref: <https://www.geeksforgeeks.org/page-table-entries-in-page-table/>_

- **Frame Number**: page 번호에 대응되는 frame 번호 (필요한 bit 수는 frame 개수에 종속적임)
    
    > *Number of bits for frame = Size of physical memory / Frame Size*
    > 
- **Valid/Invalid Bit (Present/Absent)**: 해당 page가 memory에 존재하는지 여부 (page fault 관련해서 사용됨)
- **Protection Bit(s)**: 해당 page에 대한 접근 권한
- **Referenced Bit**: 해당 page가 최근 clock cycle에서 accessed 되었는지 여부
- **Caching Enabled/Disabled**: 해당 page에 대해 caching을 허용할지 여부 (fresh data가 필요한 경우에는 disable caching)
- **Dirty Bit (Modified)**: 해당 page가 수정되었는지 여부 (page replacement 시, 수정된 정보가 있다면 disk에 write 해야 하므로)

<br>

### Page Table의 Valid Bit vs. TLB의 Valid Bit

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-14.png){: .w-75}
_ref: <http://thebeardsage.com/virtual-memory-translation-lookaside-buffer-tlb/>_

- **Page Table**: 해당 frame이 process에 의해 할당되었는지, 즉 page가 존재하는지 여부
- **TLB**: TLB entry가 valid translation을 가지고 있는지 여부
    
    > context switching 시 `0`으로 초기화 → 이전 process의 정보를 이용하여 address translation 하지 않도록 함
    >


<br>

---

## References
- <http://cloudrain21.com/memory-management-unit-tlb-virtual-address-translation>
- <https://rond-o.tistory.com/266#d27902d6-bb68-4c2b-96a9-3ee193681153>
- “운영체제와 정보기술의 원리(반효경 저)”, 7장 메모리 관리
