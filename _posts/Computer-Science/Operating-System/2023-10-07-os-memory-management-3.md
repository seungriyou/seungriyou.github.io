---
title: "[OS] 6. 메모리 관리: ③ 세그멘테이션 기법 (Segmentation), 페이지드 세그멘테이션 기법 (Paged Segmentation)"
date: 2023-10-07 18:50:00 +0900
categories: [Computer Science, Operating System]
tags: [cs, os, 운영체제, os 총정리, 스케줄링, 세그멘테이션]     # TAG names should always be lowercase
math: true
mermaid: true
image: 
  path: /assets/img/posts/Computer-Science/Operating-System/thumbnail.png
---

> 불연속 할당 방식의 세 가지 기법 중 **세그멘테이션 기법**과 **페이지드 세그멘테이션 기법**에 대해 알아보자.
> 

<br>

> **물리 메모리 관리 방법**에 대해서 알고 싶다면? [**→ [OS] 6. 메모리 관리: ① 개요**](/posts/os-memory-management-1/)
{: .prompt-tip}

<br>

## 1. 세그멘테이션 (Segmentation)

> 프로세스의 주소 공간을 **의미 단위의 segment**로 나누어 물리적 메모리에 적재하는 기법
> 

### 세그멘트 (Segment)

- 프로세스의 주소 공간을 **기능 단위 or 의미 단위**로 나눈 것이다.
    
    > ex) 주소 공간 전체, 코드/데이터/스택 등의 기능 단위, 프로그램을 구성하는 함수 하나하나 등
    > 
- 특정 크기 단위로 나눈 것이 아닌 **logical unit으로 나눈 것**이므로 크기가 균일하지 않아 관리 오버헤드가 발생한다.

<br>

### Address Translation

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-15.jpeg){: .w-75}

#### Logical Address의 구조

> <segment 번호, offset>
> 

| 구성 요소 | 의미 |
| -------- | ------ |
| segment 번호 $s$ | 프로세스 주소 공간 내에서 **몇 번째 segment**에 속하는지 |
| offset $d$       | 그 segment 내에서 얼마만큼 떨어져 있는지 (**변위**)      |

#### 세그멘트 테이블 (Segment Table)
- **각 entry의 구성 요소**
    
    | 구성 요소 | 의미 |
    | --- | --- |
    | base | physical memory에서 그 segment의 **시작 위치** |
    | limit | 그 segment의 **길이**<br>(→ 페이징 기법과 다르게, segment의 길이가 균일하지 않으므로 길이 정보도 필요함) |
    | protection bit | 해당 segment의 접근 권한 정보 (ex. R, W, X) |
    | valid bit | 각 segment의 주소 변환 정보가 유효한지 여부<br>= 해당 segment가 현재 physical memory에 적재되어 있는지 여부 |

- **관련 register**
    
    | register 종류 | 저장하는 값 |
    | --- | --- |
    | segment table base register (STBR) | 현재 CPU에서 실행 중인 프로세스의 segment table의 memory 상 시작 주소 |
    | segment table length register (STLR) | 프로세스의 주소 공간이 총 몇 개의 segment로 구성되는지 (= segment의 개수) |

#### <Logical Address → Physical Address> 변환 전 확인하는 사항
    
> 각 단계에서 그렇지 않다면, exception을 발생시켜 해당 memory 위치에 대한 접근을 봉쇄한다.

1. logical address의 <span class="hl">**segment 번호**</span>가 **STLR**에 저장된 값보다 작은 값인지
2. logical address의 <span class="hl">**offset 값**</span>이 해당 **segment의 길이**(= **segment table entry의 limit**)보다 작은 값인지

#### 공유 세그멘트 (Shared Segment)
해당 segment를 공유하는 모든 프로세스의 주소 공간에서 **동일한 logical address에 위치**해야 한다.
    
![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-16.jpeg){: .w-75}
    

<br>

### Pros and Cons

- **장점**: segment는 의미 단위로 나뉘어져 있기 때문에, 의미 단위로 수행되는 **공유 / 보안 측면**에서 페이징 기법에 비해 효과적이다.
    
    > 페이징 기법에서는 공유 데이터 영역과 사유 데이터 영역이 동일 페이지에 공존하는 경우가 발생할 수 있다.
    > 
- **단점**: segment의 길이가 균일하지 않다.
    - external fragmentation이 발생할 수 있다.
    - segment를 어느 가용 공간에 할당할 것인지 결정하는 문제가 발생한다.
        
        > → first fit, best fit 등
        > 

<br>

---

## 2. 페이지드 세그멘테이션 (Paged Segmentation)

> 프로그램을 의미 단위의 **segment**로 나누고, 이 segment는 **동일한 크기의 page들의 집합**으로 구성한다.

- physical memory에 적재하는 단위는 page 단위가 된다.
- 하나의 segment 크기를 page 크기의 배수가 되도록 설정한다.
- **장점**
    1. external fragmentation 문제점을 해결한다. (→ segmentation 기법 보완)
    2. segment 단위로 프로세스 간 공유 / 프로세스 내 접근 권한 보호가 이루어지도록 한다. (→ paging 기법 보완)

<br>

### Address Translation

- 각 segment가 여러 page로 구성되므로, 다음과 같은 **2단계의 테이블**을 이용한다.
    
    
    | table 종류    | 위치  |
    | ------------- | ----- |
    | segment table | outer |
    | page table    | inner |

#### Logical Address의 구조
    
> <segment 번호, segment 내 page 번호, page 내 offset>
> 

![Untitled](/assets/img/posts/Computer-Science/Operating-System/2023-10-07-17.png)
_ref: <https://www.gatevidyalay.com/segmentation-and-paging-segmented-paging/>_

#### <Logical Address → Physical Address> 변환 과정
1. <span class="hl">**segment 번호**</span>를 통해 segment table의 항목에 접근하여 **segment 길이**와 **해당 segment의 page table 시작 주소**를 얻는다.
2. **해당 segment의 길이**와 **logical address에서 segment 번호를 제외한 offset**을 비교하여, offset 값이 더 크다면 trap을 발생시킨다.
    
    > → physical memory 기준에서의 크기 기준!

3. **해당 segment의 page table 시작주소**에 <span class="hl">**segment 내 page 번호**</span>를 더해 page table entry에 접근하고, physical memory의 **frame 번호**를 얻는다.
4. **frame 번호**에 <span class="hl">**page 내 offset**</span>을 더해 **physical address**를 얻는다.

<br>

---

## References
- “운영체제와 정보기술의 원리(반효경 저)”, 7장 메모리 관리
