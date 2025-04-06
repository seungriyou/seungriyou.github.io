---
title: "[OS] 7. 가상 메모리"
date: 2023-10-09 16:50:00 +0900
categories: [Computer Science, Operating System]
tags: [cs, os, 운영체제, os 총정리, 가상 메모리, 요구 페이징, 페이지 교체, 스레싱, demand paging, page replacement, thrashing]     # TAG names should always be lowercase
math: true
mermaid: true
---

## 1. 가상 메모리 (Virtual Memory)

### 개요

여러 프로그램이 동시에 수행되는 **시분할 환경**에서는, 한정된 메모리 공간을 여러 프로그램이 조금씩 나누어서 사용한다. 프로그램이 CPU에서 실행되려면 **실행에 당장 필요한 부분이 메모리에** 올라와 있어야 하므로, 운영체제는 몇몇 프로그램들에게 집중적으로 메모리를 할당한 후, 시간이 흐르면 이들로부터 메모리를 회수하여 다른 프로그램들에게 다시 메모리를 할당하는 방식을 채택한다.

운영체제는 CPU에서 당장 수행해야 할 부분만을 **memory**에, 그렇지 않은 부분은 **disk의 swap area**에 내려놓는다. 이때, swap area에 존재하는 부분은 필요해지면 **memory에 적재된 부분과 교체하는 방식**을 사용한다.

<br>

### 가상 메모리란?

0번지부터 시작하는 **프로그램 자신만의 메모리 주소 공간**으로, 일부는 **physical memory**에, 일부는 **disk의 swap area**에 저장된다.

> 프로그램은 physical memory 크기에 대한 제약을 생각할 필요가 없기 때문에 자기 자신만이 메모리를 사용한다고 가정할 수 있다.
{: .prompt-tip}

**가상 메모리 기법**은 프로세스의 주소 공간을 메모리로 적재하는 단위에 따라 **요구 페이징(demand paging)** 방식과 **요구** **세그먼테이션(demand segmentation)** 방식으로 구현될 수 있다. 대부분의 경우에는 요구 페이징 방식을 사용한다.

<br>

---

## 2. 요구 페이징 (Demand Paging)

프로그램 실행 시 프로세스를 구성하는 <span class="hl">모든 page를 한꺼번에 메모리에 올리는 것이 아니라, **당장 사용될 page만**을 올리는 방식</span>이다. 따라서 특정 page에 대해 **CPU의 요청이 들어온 후에야** 해당 page를 메모리에 적재하게 된다.

<br>

**요구 페이징의 장점**에는 다음과 같은 것들이 있다.

- 메모리 사용량이 감소한다.
- 프로세스 전체를 메모리에 올리는 데에 소요되는 입출력 오버헤드가 감소한다.
- 응답시간을 단축시킬 수 있다.
- 시스템이 더 많은 프로세스를 수용할 수 있게 해준다.
- physical memory의 용량보다 큰 프로그램도 실행할 수 있게 한다.

<br>

### 유효 비트 (Valid Bit)

> 가상 메모리 기법에서는 프로세스가 실행되는 동안 일부 page만 memory에 적재되어 있고, 나머지 page는 disk의 swap area에 존재한다. 따라서 요구 페이징에서는 **페이지 테이블의 유효 비트**를 사용한다.
{: .prompt-info}

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-01.jpeg){: .w-75}

- 각 page가 **memory에 존재하는지 여부**를 표시한다.
- page table의 각 entry 마다, 즉 **모든 page에 대해 존재**한다.

#### Valid Bit의 설정
1. **프로세스가 실행되기 전**: 모든 page에 대해 `0`으로 초기화
2. **특정 page가 참조되어 메모리에 적재되는 경우**: 해당 page에 대해 `1`로 설정
3. **메모리에 적재되어 있던 page가 swap area로 swap-out 되는 경우**: 해당 page에 대해 `0`으로 설정

#### Valid Bit == `0`인 경우
1. 해당 page가 현재 physical memory에 없는 경우
  
    > → **페이지 부재(page fault)** 상황!
    {: .prompt-warning}
    
2. 해당 page가 속한 주소 영역을 프로세스가 사용하지 않는 경우

<br>

### Page Fault 처리 과정

> CPU가 invalid page, 즉 현재 physical memory에 존재하지 않는 page에 접근하는 경우의 동작 과정
> 

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-02.jpeg){: .w-75}

1. 참조하려는 page에 해당하는 page table entry의 valid bit가 `0`이라면, **MMU**가 **page fault trap**을 발생시킨다.
2. CPU의 제어권이 운영체제에게 넘어가 **kernel mode**로 전환되고, **page fault handler**가 호출된다.
3. page fault handler는 다음의 동작을 통해 **page fault를 처리**한다.
    1. 운영체제가 **해당 page에 대한 접근이 적법한지**를 확인한다.
        
        > (1) 사용되지 않는 주소 영역에 속한 page이거나 (2) protection violation인 경우, 해당 프로세스를 종료시킨다.
        > 
    2. physical memory에서 **free frame**을 할당받아 그 공간에 **disk로부터 해당 page를 읽어온다**(<span class="red">**swap-in**</span>).
        
        > free frame이 없다면, **memory에 적재되어 있는 page** 중 하나를 disk로 <span class="red">**swap-out**</span> 시킨다.
        > 
    3. page fault를 일으킨 프로세스는 CPU를 빼앗기고 **`blocked` 상태**가 된다. 이때, **PCB에 CPU 레지스터 상태 및 PC 값 등을 저장**한다.
        
        > 요청된 page를 disk → memory로 적재하는 데에는 상당한 시간이 소요되기 때문이다.
        > 
    4. **disk I/O가 완료**되어 <span class="red">**interrupt**</span>가 발생한다.
    5. page table에서 **해당 page의 valid bit를 `1`로 설정**하고, `blocked` 상태였던 프로세스를 **준비 큐**(= **`ready` 상태**)**로 이동**시킨다.
4. 해당 프로세스가 다시 CPU를 할당받으면, **PCB에 저장했던 값을 복원**하여 **중단 위치부터 실행을 재개**한다. (<span class="red">**문맥교환**</span>)

<br>

### 요구 페이징의 성능

요구 페이징 기법의 성능에 가장 큰 영향을 미치는 요소는 <span class="hl">**“page fault의 발생 빈도”**</span>이다. 이는 요청된 page를 disk → memory로 읽어오는 오버헤드가 발생하기 때문이다.

> page fault ⬇️ : 유효 접근 시간 ⬇️ : 요구 페이징의 성능 ⬆️
{: .prompt-info}

<br>

이때, <span class="hl">**유효 접근 시간 (Effective Access Time)**</span>은 다음의 식으로 구할 수 있다.

> $$P$$ = page fault rate ($$0\le P \le1$$)
> 

```
(1 - P) × 메모리 접근 시간 +
 P × (page fault 처리 오버헤드 
        + free frame이 없는 경우 swap-out 오버헤드 
        + 요청된 page의 swap-in 오버헤드 
        + 프로세스의 재시작 오버헤드)
```

<br>

---

## 3. 페이지 교체 (Page Replacement)
- **페이지 교체 (page replacement)**: 메모리에 올라와 있는 page 중 하나를 disk로 쫓아내(= swap out) 빈 공간을 확보하는 작업
    - page fault가 발생하면 요청된 page를 <disk → memory>로 읽어와야 하는데, 이때 physical memory에 free frame이 존재하지 않는 경우도 있기 때문이다.
- **페이지 교체 알고리즘 (page replacement algorithm)**: 어떠한 frame에 있는 page를 쫓아낼 것인지 결정하는 알고리즘
    - <span class="hl">page fault rate을 최소화</span> 하는 것이 목표이므로, 가까운 미래에 참조될 가능성이 가장 작은 page를 선택해야 한다.
    - 성능을 계산할 때는 주어진 page reference string에 대해 page fault rate을 계산한다.

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-03.jpeg){: .w-75}


<br>

### [1] 최적 알고리즘 (Belady’s Optimal Algorithm; MIN, OPT)

> page replacement 시 physical memory에 존재하는 page 중 **가장 먼 미래에 참조될 page**를 선택
> 

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-04.jpeg){: .w-75}

- **장점**: 어떠한 알고리즘보다도 가장 작은 page fault rate를 보장하므로, 다른 알고리즘의 성능에 대한 upper bound를 제공한다.
- **단점**: 실제 시스템에서 온라인으로 사용할 수 있는 알고리즘이 아니다. (→ 오프라인 알고리즘)

<br>

### [2] 선입선출 알고리즘 (First In First Out; FIFO)

> page replacement 시 **physical memory에 가장 먼저 올라온 page**를 선택
> 

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-05.jpeg){: .w-75}

- page의 향후 참조 가능성을 고려하지 않기 때문에 비효율적인 상황이 발생할 수 있다.
- **FIFO anomaly**
    - 위의 그림처럼 memory 크기를 증가시켰음에도 불구하고 page fault가 오히려 늘어나는 상황이다.
    - 가장 먼저 physical memory에 들어온 page가 계속해서 많은 참조가 이루어진다고 하더라도 내쫓기 때문이다.
    - LRU 알고리즘에서는 이러한 이상 현상이 발생하지 않는다.

<br>

### [3] LRU 알고리즘 (Least Recently Used)

> page replacement 시 **가장 오래전에 참조가 이루어진(= 마지막 참조 시점이 가장 오래된) page**를 선택
> 

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-06.jpeg){: .w-75}

- **시간 지역성**(temporal locality)을 이용하여 향후 사용될 가능성이 낮은 페이지를 우선적으로 메모리에서 쫓아내어 페이지 교체 성능 향상을 위함이다.
    
    > **시간 지역성**: 최근에 참조된 페이지가 가까운 미래에 다시 참조될 가능성이 높은 성질
    > 

<br>

### [4] LFU 알고리즘 (Least Frequently Used)

> page replacement 시 **reference count가 가장 적었던 page**를 선택
> 
> 
> → 최소 reference count를 가진 page가 여러 개라면, 임의로 하나를 선정하거나, 상대적으로 더 오래전에 참조된 page를 선정

- **page의 reference count를 계산하는 방식에 따른 구현 방식**
    
    | 구현 방식   | reference count 계산 방법                                                                                                        |
    | ----------- | -------------------------------------------------------------------------------------------------------------------------------- |
    | incache-LFU | physical memory에 올라온 후부터의 reference count<br>(memory에서 쫓겨났다가 다시 들어온 경우, reference count는 다시 1부터 시작) |
    | perfect-LFU | 해당 page의 과거 총 reference count                                                                                              |

- **장점**: reference count를 통해 LRU 알고리즘보다 장기적인 시간 규모에서의 참조 기록을 반영할 수 있다.
- **단점**: 시간에 따른 페이지 참조의 변화를 반영하지 못하고, LRU보다 구현이 복잡하다.

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-07.jpeg){: .w-75}

<br>

### [5] 클럭 알고리즘 (Clock; Not Used Recently, NUR; Not Recently Used, NRU)

> page replacement 시 **오랫동안 참조되지 않은 page 중 하나**를 선택

- 소프트웨어적 오버헤드를 가지고 있는 LRU, LFU 등의 알고리즘에서 발생하는 운영 오버헤드를 **하드웨어적인 지원**을 통해 줄인 방식이다.
- **LRU를 approximation** 시킨 알고리즘으로, **최근에 참조되지 않은 page를 대상으로 선정**한다는 점은 LRU와 유사하나 **하드웨어의 지원**을 통해 동작한다는 점이 다르다.
- page 관리가 훨씬 빠르고 효율적이므로 **대부분의 시스템에서** page replacement algorithm으로 채택한다.
- **replace 할 page를 선택하는 방법**: page frame들의 **reference bit**를 순차적으로 조사한다.
    - reference bit는 **각 frame 마다 하나씩** 존재하며, **해당 page가 참조될 때 하드웨어에 의해 `1`로 자동 설정**된다.
    - reference bit에 따른 클럭 알고리즘의 동작은 다음과 같다.
        
        
        | reference bit | 클럭 알고리즘의 동작                                |
        | ------------- | --------------------------------------------------- |
        | `1`           | 해당 page의 reference bit를 0으로 바꾼 후 지나간다. |
        | `0`           | 해당 page를 교체한다.                               |

    - 즉, **시곗바늘이 한 바퀴 도는 동안 다시 참조되지 않은 page를 교체**하는 것이다.
- 적어도 시곗바늘이 한 바퀴를 도는 데에 소요되는 시간만큼 page를 메모리에 유지시킴으로써 **page fault rate를 줄이도록 설계**되었다.
    
    > → **second chance algorithm**이라고도 부른다.
    > 

<br>

---

## 4. 페이지 프레임의 할당

각 프로세스에 얼마만큼의 메모리 공간을 할당할지는 다음과 같은 **할당 알고리즘 (allocation algorithm)**으로 결정한다.

1. **균등 할당(equal allocation)**: 모든 프로세스에게 page frame을 균일하게 할당
2. **비례 할당(proportional allocation)**: 프로세스의 크기에 비례하여 page frame을 할당
3. **우선순위 할당(priority allocation)**: 프로세스의 우선순위에 따라 page frame을 다르게 할당 (당장 CPU에서 실행될 프로세스에 더 많은 page frame 할당)

<br>

하지만, 다음과 같은 이유로 **할당 알고리즘만으로는 프로세스의 page 참조 특성을 제대로 반영하지 못할 수도** 있다.

1. 현재 수행 중인 프로세스의 수가 지나치게 많을 경우, 프로세스 당 할당되는 메모리 양이 과도하게 적어질 수 있다. 
2. 반복문(loop)을 실행 중인 프로세스의 경우, 반복문을 구성하는 page들을 한꺼번에 메모리에 올려 놓는 것이 유리하다.
3. 프로세스에게 필요한 최소 메모리 양은 시간에 따라 다를 수 있다.

<br>

---

## 5. 전역 교체와 지역 교체

> 교체할 page를 선정할 때, 교체 대상이 될 frame의 범위를 어떻게 정할 것인가

- **전역교체 (global replacement)**
    - 모든 page frame이 교체 대상이 될 수 있다.
    - 각 프로세스에 할당되는 메모리 양(= 프로세스의 frame 할당량)이 가변적으로 변한다.
    - 교체할 page가 어떤 프로세스에 속한 것인지는 고려하지 않아 다른 프로세스에 할당된 frame을 뺏어올 수 있는 방식이다.
    - physical memory 내에 존재하는 모든 page frame을 대상으로 한다.
    - LRU, LFU, Clock, Working Set, PFF 알고리즘 등 여러 알고리즘이 전역교체 방법으로 사용될 수 있다.
- **지역교체 (local replacement)**
    - 현재 수행 중인 프로세스에게 할당된 frame 내에서만 교체 대상을 선정할 수 있다.
    - 프로세스마다 page frame을 미리 할당하는 것을 전제로 하는 방법이다.
    - LRU, LFU 등의 알고리즘을 프로세스별로 독자적으로 운영할 때에 해당한다.

<br>

---

## 6. 스레싱 (Thrashing)

> 집중적으로 참조되는 page의 집합을 메모리에 한꺼번에 적재하지 못하면 **page fault rate가 증가**하여 **CPU 이용률이 감소**하는 현상
> 

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-08.jpeg){: .w-75}

<br>

### 스레싱이 발생하는 시나리오

1. **CPU 이용률**이 낮으면 운영체제는 메모리에 동시에 올라가 있는 프로세스의 수(= **MPD**)를 늘린다.
    - CPU 이용률이 낮다는 것은 준비 큐가 비는 경우가 있다는 것이므로, 운영체제는 메모리에 올라와 있는 프로세스의 수가 적기 때문이라고 판단한다.
    - **MPD** = Multi-Programming Degree = 다중 프로그래밍의 정도
2. **MPD**가 과도하게 높아지면 **각 프로세스에게 할당되는 메모리의 양이 지나치게 감소**하여 각 프로세스는 정상적으로 수행되기 위한 최소한의 page frame도 할당받지 못하게 된다.
3. **page fault rate**이 증가한다.
    - page fault는 **disk I/O 작업**을 수반하므로 **문맥교환**을 통해 다른 프로세스에게 CPU가 이양된다.
4. 다른 프로세스 역시 할당받은 메모리 양이 지나치게 적으면 page fault가 발생하여, **준비 큐에 있는 모든 프로세스에게 CPU가 한 번씩 할당**되더라도 모든 프로세스가 **page fault**를 발생시킬 수 있다.
5. 따라서 **CPU 이용률**은 더 떨어지게 되는데, 이때 운영체제는 또 **MPD를 높이기** 위해 메모리에 다른 프로세스를 추가한다.
6. 프로세스당 할당된 **frame 수가 더욱 감소**하고 **page fault는 더욱 빈번히** 발생하게 된다.

<br>

> **thrashing**이 발생하지 않도록 하면서 **CPU 이용률**을 최대한 높일 수 있도록 **MPD를 조절**해야 한다.
{: .prompt-tip}

<br>

### [1] 워킹셋 알고리즘 (Working-Set)

- 일정 시간 동안 집중적으로 참조되는 주소 영역인 **지역성 집합(locality set)**이 메모리에 **동시에** 올라갈 수 있도록 보장하는 메모리 관리 알고리즘이다.
    
    > **워킹셋(working-set)**: 프로세스가 일정 시간 동안 원활히 수행되기 위해 한꺼번에 메모리에 적재되어야 하는 page의 집합
    
    1. 프로세스의 **working-set이 한꺼번에 메모리에 올라갈 수 있는** 경우, 그 프로세스에게 메모리를 할당한다.
    2. 그렇지 않다면, 프로세스에게 할당된 page frame을 모두 회수하고, **프로세스 주소 공간 전체를 swap out** 시킨다.    
- **working-set에 포함된 page들은 메모리에 유지**, 그렇지 않은 page들은 swap out 된다.
    
    > 즉, page가 참조된 시점부터 시간 $$\Delta$$ 동안은 메모리에 유지하고, 그 시점이 지나면 메모리에서 지운다.
    > 

#### Working-set을 구하는 방법
    
> working-set window 크기가 $$\Delta$$일 때, 시각 $$t_i$$에서의 working-set $$WS(t_i)$$
>
> = 시간 간격 $$[t_i-\Delta, t_i]$$ 사이에 참조된 서로 다른 page의 집합
{: .prompt-info}

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-09.jpeg){: .w-75}

#### 시나리오
1. **(메모리에 올라와 있는 프로세스들의 working-set 크기의 합) > (frame의 수) 인 경우**
    - 일부 프로세스를 swap out 시켜 남은 프로세스의 working-set이 메모리에 모두 올라가는 것을 보장한다.
    - 즉, MPD를 줄이는 효과를 발생시킨다.
2. **프로세스들의 working-set을 모두 할당한 후에도 frame이 남을 경우**
    - swap out 되었던 프로세스를 다시 메모리에 적재하여 working-set을 할당한다.
    - 즉, MPD를 증가시키는 효과를 발생시킨다.
    
<br>


### [2] 페이지 부재 빈도 알고리즘 (Page Fault Frequency; PFF)

![](/assets/img/posts/Computer-Science/Operating-System/2023-10-09-10.jpeg){: .w-75}

- 프로세스의 **page fault rate를 주기적으로 조사**하고 이 값에 근거하여 **각 프로세스에 할당할 메모리 양을 동적으로 조절**하는 알고리즘이다.
- **어떤 프로세스의 page fault가 upper bound를 넘는 경우**: 해당 프로세스에 할당된 frame의 수가 부족하다고 판단하여, frame을 추가로 더 할당한다.
- **어떤 프로세스의 page fault가 lower bound 이하인 경우**: 해당 프로세스에게 필요 이상으로 많은 frame이 할당된 것으로 간주하여, 할당된 frame의 수를 줄인다.
- **모든 프로세스에게 frame 할당한 후에도 frame이 남는 경우**: swap out 되었던 프로세스에게 frame을 할당하여 MPD를 높인다.

<br>

---

## References

- “운영체제와 정보기술의 원리(반효경 저)”, 8장 가상메모리
  