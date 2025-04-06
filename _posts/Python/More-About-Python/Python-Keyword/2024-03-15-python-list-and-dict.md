---
title: "[Python] list와 dict의 내부 구현과 시간 복잡도 (+ 동적 배열, 해시 충돌)"
date: 2024-03-15 16:50:00 +0900
categories: [Python, More About Python]
tags: [python, list, dict, cpython, array, doubling, complexity, amortized, hash, hash table, rehashing, collision, open addressing]
math: true
---

## 1. 파이썬의 `list`

### 1-1. 배열(Array)

자료구조에는 크게 두 가지 방식이 있다.

1. **연속 방식(contiguous)**: 메모리 공간 기반 (ex. 배열)
2. **연결 방식(link)**: 포인터 기반 (ex. 연결 리스트)
    

<br>

**연속 방식**에 해당하는 **배열**의 경우, <span class="shl">**고정된 크기만큼의 연속된 메모리를 할당하는 방식**</span>으로 구현된다. 이때 메모리에 대한 접근은 바이트 단위로 이루어지며, 1바이트마다 주소가 1씩 증가하게 된다.

> ex) `int` 형 배열을 선언하면 각 원소는 4바이트가 된다. 즉, 배열의 주소도 4씩 증가하게 된다.
> 

이러한 특성으로 인해, <span class="shl">**어느 위치에서나 해당 메모리 주소에 있는 값을 $$O(1)$$에 조회**</span>할 수 있다.

<br>

### 1-2. 동적 배열(Dynamic Array)

배열을 구현하는 방식에는 두 가지가 있다.

1. **정적 배열(static array)**: 선언할 때 크기를 지정하며, 그 크기가 고정되어 있는 배열
2. **동적 배열(dynamic array)**: 크기가 변경될 수 있는 배열

<br>

대부분의 **동적 프로그래밍 언어에서는 런타임에 배열의 크기를 변경**할 수 있는데, 이는 해당 언어에서는 배열을 **동적 배열**로 구현했기 때문인 것이다.

> Java는 `ArrayList`, C++은 `std::vector`, Python은 `list`라는 자료형으로 동적 배열을 지원한다.
> 

<br>

그렇다면 동적 배열은 어떻게 크기가 변경될 수 있는 것일까? **기본적인 원리**는 다음과 같다.

1. **배열을 처음 생성**할 때, 주어진 데이터보다 조금 더 큰 크기의 메모리를 할당한다. (우선 작은 메모리 할당)
2. **배열에 원소를 삽입**할 때, 남은 메모리에 원소가 추가될 수 있는지 여부에 따라 다르게 동작한다.
    - **메모리가 여유있는 상황**: 원소를 남은 공간에 삽입한다.
    - **메모리가 부족한 상황**: 더 큰 크기의 메모리를 할당한 새로운 배열을 만들고, 기존 데이터를 새로운 배열에 옮기고, 그 뒤에 원소를 삽입한다.
        
        > 이때, 새로운 배열의 메모리 크기는 보통 2배씩 늘려주게 되며, 이를 <span class="shl">**더블링(doubling)**</span>이라 한다. 재할당 비율(growth factor)은 언어마다 다르다.
        {: .prompt-info}

![](/assets/img/posts/Python/More-About-Python/2024-03-15-01.png){: .w-75}
_ref: <https://medium.com/@yasufumy/data-structure-dynamic-array-3370cd7088ec>_

<br>

이러한 원리로 동작하는 **동적 배열에서 발생하는 시간 복잡도**는 다음과 같다.

| 상황                                                                                 | 시간 복잡도 |
| ------------------------------------------------------------------------------------ | ----------- |
| 특정 위치(인덱스)에 존재하는 **원소를 조회**하는 경우                                | $$O(1)$$    |
| **원소 삽입** 시, 메모리가 여유있어 남는 공간에 데이터를 추가하는 경우               | $$O(1)$$    |
| **원소 삽입** 시, 메모리가 부족하여 새로운 배열을 만들고 기존 데이터를 복사하는 경우 | $$O(n)$$    |


따라서 <span class="shl">**동적 배열의 원소 조회**는 $$O(1)$$</span>의 복잡도를 가진다고 할 수 있다.

그렇다면 **동적 배열에 원소를 삽입**하는 동작의 복잡도는 얼마라고 할 수 있을까? 동적 배열에 원소를 삽입하는 상황을 가정했을 때, 일반적으로는 <span class="shlp">**메모리가 부족한 상황(→ $$O(n)$$)**</span>보다 <span class="shlp">**메모리가 여유있는 상황(→ $$O(1)$$)**</span>의 빈도가 훨씬 더 높을 것이다.

이러한 경우에는 **분할 상환 분석**을 적용하여, 어쩌다 한 번 발생하는 $$O(n)$$ 상황을 주로 발생하는 $$O(1)$$ 상황에 나누어줌으로써 **amortized $$O(1)$$**의 복잡도를 가진다고 할 수 있다.

> **분할 상환 분석(Amortized Analysis)**
>
>최악의 경우를 여러 번에 걸쳐 골고루 나눠주는 형태로 복잡도를 계산하는 방식
{: .prompt-info}

<br>

간단하게, **연산의 평균 횟수**를 구하듯이 접근해보자.

현재 할당되어 있는 **메모리가 수용 가능한 원소의 개수가 $$n$$일 때, $$n + 1$$개의 원소를 추가하는 상황**을 가정한다. 이때, 각 삽입 연산에서 발생하는 복잡도는 다음과 같다.

1. 처음 <span class="blue">$$n$$</span>개의 삽입 연산은 <span class="blue">$$O(1)$$</span>
2. 마지막 <span class="blue">$$1$$</span>개의 삽입 연산은 <span class="blue">$$O(n)$$</span> (resizing)

즉, <span class="shlb">**$$n + 1$$개의 원소를 추가**</span>하는 데에 총 <span class="shlb">**$$O(2n)$$의 연산**</span>이 소요되었으므로 평균을 다음과 같이 구할 수 있다.

$$
\frac{O(2n)}{n+1}=\frac{O(n)}{n+1}\approx O(1) 
$$

이러한 이유로 <span class="shl">**동적 배열의 원소 삽입**은 **amortized $$O(1)$$**</span>의 복잡도를 가진다고 할 수 있다. 보다 자세한 증명 과정은 다음의 이미지 혹은 **[링크](https://www.cs.utexas.edu/~slaberge/docs/topics/amortized/dynamic_arrays/)**를 참고한다.

![](/assets/img/posts/Python/More-About-Python/2024-03-15-02.png){: .w-75}
_ref: <https://www.cs.utexas.edu/~slaberge/docs/topics/amortized/dynamic_arrays/>_

<br>

위의 내용을 바탕으로 다시 <span class="shl">**동적 배열에서의 시간 복잡도**</span>를 정리하면 다음과 같다.

| 상황                                                  | 시간 복잡도        |
| ----------------------------------------------------- | ------------------ |
| 특정 위치(인덱스)에 존재하는 **원소를 조회**하는 경우 | $$O(1)$$           |
| 배열의 맨 뒤에 **원소를 삽입**하는 경우               | amortized $$O(1)$$ |

<br>

### 1-3. CPython 구현

1. 파이썬의 `list`는 다음의 그림과 같이, 할당된 메모리에 <span class="shl">**각 원소 객체를 가리키는 포인터를 동적 배열에 저장**</span>하는 형태이다. 즉, 배열의 장점과 연결 리스트의 장점을 모두 취한다.
    
    ![](/assets/img/posts/Python/More-About-Python/2024-03-15-03.png){: .w-75}
    _ref: <https://jakevdp.github.io/PythonDataScienceHandbook/02.01-understanding-data-types.html>_
    
    - CPython에서 모든 것은 `PyObject`로 구현된다. `PyObejct_HEAD`에는 가비지 컬렉션을 위한 참조 카운트, 타입 코드 등이 들어있다.
    - 원소 객체를 가리키는 포인터를 저장하므로 각 원소마다 각기 다른 타입을 가질 수 있어 유연성이 뛰어나지만, 모든 원소의 타입이 같은 경우 각 원소마다 중복된 정보가 저장되므로 비효율적이다.
        
        ![](/assets/img/posts/Python/More-About-Python/2024-03-15-04.png){: .w-75}
        _ref: <https://realpython.com/python-hash-table/>_
        
        
    <br>

2. CPython에서 **resizing**을 수행할 때 사용하는 <span class="shl">**재할당 비율은 약 1.125**</span>이다.

    - CPython의 `Objects/listobject.c` ([링크](https://github.com/python/cpython/blob/41e844a4acbd5070f675e034e31c988b4849dec9/Objects/listobject.c#L114))
        
        ```c
      /* This over-allocates proportional to the list size, making room
        * for additional growth.  The over-allocation is mild, but is
        * enough to give linear-time amortized behavior over a long
        * sequence of appends() in the presence of a poorly-performing
        * system realloc().
        * Add padding to make the allocated size multiple of 4.
        * The growth pattern is:  0, 4, 8, 16, 24, 32, 40, 52, 64, 76, ...
        * Note: new_allocated won't overflow because the largest possible value
        *       is PY_SSIZE_T_MAX * (9 / 8) + 6 which always fits in a size_t.
        */
      new_allocated = ((size_t)newsize + (newsize >> 3) + 6) & ~(size_t)3;
        ```
        
    - 재할당 비율 관련 참고 자료: <https://stackoverflow.com/questions/52195579/what-is-the-resizing-factor-of-lists-in-python>
    

<br>

### 1-4. 주요 연산의 복잡도

| 연산                  | 시간 복잡도    | 설명                                                                                                                                                                                           |
| --------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `len(a)`              | $$O(1)$$       | 전체 요소의 개수를 리턴한다.                                                                                                                                                                   |
| `a[i]`                | $$O(1)$$       | 인덱스 `i`의 요소를 가져온다.                                                                                                                                                                  |
| `a[i:j]`              | $$O(k)$$       | `i`부터 `j`까지 슬라이스의 길이만큼인 `k`개의 요소를 가져온다. 이 경우 객체<br>`k`개에 대한 조회가 필요하므로 $$O(k)$$이다.                                                                    |
| `elem in a`           | $$O(n)$$       | `elem` 요소가 존재하는지 확인한다. 처음부터 순차 탐색하므로 `n`만큼<br>시간이 소요된다.                                                                                                        |
| `a.count(elem)`       | $$O(n)$$       | `elem` 요소의 개수를 리턴한다.                                                                                                                                                                 |
| `a.index(elem)`       | $$O(n)$$       | `elem` 요소의 인덱스를 리턴한다.                                                                                                                                                               |
| `a.append(elem)`      | $$O(1)$$       | 리스트 마지막에 `elem` 요소를 추가한다.                                                                                                                                                        |
| `a.pop()`             | $$O(1)$$       | 리스트 마지막 요소를 추출한다. **스택**의 연산이다.                                                                                                                                            |
| `a.pop(0)`            | $$O(n)$$       | 리스트 첫번째 요소를 추출한다. **큐**의 연산이다. 이 경우 전체 복사가<br>필요하므로 $$O(n)$$이다. 따라서 큐의 연산을 주로 사용한다면 리스트보다는<br>$$O(1)$$에 가능한 **`deque를` 권장**한다. |
| `del a[i]`            | $$O(n)$$       | `i`에 따라 다르다. 최악의 경우 $$O(n)$$이다.                                                                                                                                                   |
| `a.sort()`            | $$O(n\log n)$$ | 정렬한다. Timsort를 사용하며, 최선의 경우 $$O(n)$$에도 실행될 수 있다.                                                                                                                         |
| `min(a)`, `max(a)`    | $$O(n)$$       | 최솟값/최댓값을 계산하기 위해서는 전체를 선형 탐색해야 한다.                                                                                                                                   |
| `a.reverse()`         | $$O(n)$$       | 뒤집는다. 리스트는 입력 순서가 유지되므로 뒤집게 되면 입력 순서가 반대로<br>된다.                                                                                                              |
| `a.insert(idx, elem)` | $$O(n)$$       | 특정 인덱스 위치에 원소를 삽입한다. (삽입 후 원소 위치 조정)                                                                                                                                   |
| `a.remove(elem)`      | $$O(n)$$       | 특정 값을 가진 원소를 제거하는데, 해당 값의 원소가 여러 개면 하나만<br>제거한다. (삭제 후 원소 위치 조정)                                                                                      |

<br>

## 2. 파이썬의 `dict`

### 2-1. 해시 테이블(Hash Table)

**해시 테이블**(혹은 **해시 맵**)은 **<span class="shl">키를 값에 매핑</span>할 수 있는 구조인 연관 배열 추상 자료형(ADT)**을 구현하는 자료구조이다. 조회, 삽입 등 대부분의 연산이 <span class="shl">**분할 상환 분석에 따른 시간 복잡도가 $$O(1)$$**</span>이므로, 데이터의 양에 관계 없이 빠른 성능을 기대할 수 있다.

<br>

### 2-2. 해시(Hash)

> 해시와 관련된 용어를 정리해보자.

1. <span class="shlp">**해시 함수**</span>
    - **임의 크기 데이터**를 **고정 크기 값**으로 매핑하는 데 사용할 수 있는 함수이다.
    - 다음의 특징을 가질 때, 해시 함수의 성능이 좋다고 판단한다.
        - 해시 함수 값 충돌의 최소화
        - 쉽고 빠른 연산
        - 해시 테이블 전체에 해시 값이 균일하게 분포
        - 사용할 키의 모든 정보를 이용하여 해싱
        - 해시 테이블 사용 효율이 높을 것
    - 체크섬(checksum), 손실 압축, 무작위화 함수(randomization function), 암호 등과 관련이 있다.
    - 최상의 분포를 제공하는 해시 함수는 데이터에 따라 따르다.
    - 대표적인 해시 함수로는 **나눗셈 방식(modulo-division method)**이 있다.
        
        $$
        h(x)=x\mod m
        $$
        
        | 기호     | 설명                                                               |
        | -------- | ------------------------------------------------------------------ |
        | $$h(x)$$ | 입력값 $$x$$의 해시 함수를 통해 생성된 결과 (모듈로 연산의 결과)   |
        | $$m$$    | 해시 테이블의 크기 (일반적으로 2의 멱수에 가깝지 않은 소수를 택함) |
        | $$x$$    | 충분히 랜덤한 상태의 키 값                                         |
        
    <br>

2. <span class="shlp">**해싱(hashing)**</span>
    - 해시 테이블을 인덱싱하기 위해 해시 함수를 사용하는 것이다.
    - 정보를 가능한 한 빠르게 저장하고 검색하기 위해 사용하는 중요한 기법으로, 최적의 검색이 필요한 분야에 사용된다.
        
    <br>

3. <span class="shlp">**비둘기집 원리(pigeonhole principle)**</span>
    - $$n$$개 아이템을 $$m$$개 컨테이너에 넣을 때, $$n > m$$이라면 적어도 하나의 컨테이너에는 반드시 **2개 이상의 아이템**이 들어있다는 원리이다.
        
        > “하나의 컨테이너에 2개 이상의 아이템이 들어있다”는 것은 <span class="shl">**충돌(collision)**</span>을 의미한다! 충돌을 해결하는 방법에 대해서는 다음 섹션에서 살펴본다.
        {: .prompt-tip}
        
    - 여러 번 충돌한다는 것은 그만큼 추가 연산을 필요로 하기 때문에 가급적 충돌을 최소화하는 것이 좋다.
        
    <br>

4. <span class="shlp">**생일 문제(birthday problem)**</span>
    - 비둘기집 원리에 의해 366명 이상이 모여야 생일이 같은 2명이 존재할 것 같지만, 실제로는 23명만 모여도 확률이 50%를 넘고, 57명이 모이면 99%를 넘어선다는 것이다.
    - 충돌은 생각보다 쉽게, 자주 일어난다는 것을 보여주는 예시이다.
      
    <br>

5. <span class="shlp">**로드 팩터(load factor)**</span>
    - 해시 테이블에 **저장된 데이터 개수 $$n$$**을 **버킷의 개수 $$k$$**로 나눈 것이다.
        
        $$
        \text{load factor} = \frac {n}{k}
        $$
        
    - 이 값에 따라 해시 함수를 재작성해야 할지 또는 해시 테이블의 크기를 조정해야 할지를 결정할 수 있다.
    - 해시 함수가 키들을 잘 분산해주는지를 말하는 효율성 측정에 사용된다.
    - 일반적으로 로드 팩터가 증가할수록 해시 테이블의 성능은 감소한다.

<br>

### 2-3. 충돌(Collision)

**해시 충돌**이란, <span class="shl">**해시 함수가 서로 다른 두 개의 입력값에 대해 동일한 출력값을 내는 상황**</span>을 의미한다. 해시 함수가 무한한 가짓수의 입력값을 받아 유한한 가짓수의 출력값을 생성하는 경우에는 비둘기집 원리에 의해 해시 충돌이 항상 존재하게 된다.

이러한 해시 충돌은 **해시 함수를 이용한 자료구조 및 알고리즘의 효율성**을 떨어뜨리므로 해시 함수는 **해시 충돌이 자주 발생하지 않도록** 구성되어야 한다.

<br>

그렇다면, **충돌이 발생할 때 해시 테이블은 어떻게 원소를 저장**할까? 크게 두 가지 방법이 있다.

1. <span class="shlp">**개별 체이닝(Separate Chaining)**</span>: 충돌이 발생하면 연결 리스트로 연결한다.
    - **전체 과정**
        1. 키의 해시 값을 계산한다.
        2. 해시 값을 이용해 배열의 인덱스를 구한다.
        3. 같은 인덱스가 있다면 연결 리스트로 연결한다.
    - 대부분의 탐색은 $$O(1)$$이지만 **최악의 경우에는 $$O(n)$$**이다.
    
    ![](/assets/img/posts/Python/More-About-Python/2024-03-15-05.png){: .w-50}
    _ref: <https://realpython.com/python-hash-table/>_
    
    <br>

2. <span class="shlp">**오픈 어드레싱(Open Addressing)**</span>: 충돌이 발생하면 탐사(probing)를 통해 빈 공간을 찾아나선다.
    - **특징**
        1. 무한정 저장할 수 있는 체이닝 방식과 달리, 오픈 어드레싱 방식은 전체 슬롯의 개수 이상은 저장 불가능하다.
        2. 개별 체이닝 방식과 달리, 모든 원소가 반드시 자신의 해시값과 일치하는 주소에 저장된다는 보장이 없다.
    - 가장 간단한 방식은 <span class="shl">**선형 탐사(linear probing)**</span>이다.
        - 충돌이 발생할 경우, 해당 위치부터 순차적으로 탐사를 하나씩 진행하는 방법이다.
        - 가장 가까운 다음 빈 위치를 탐사하는 방법이므로 구현 방법이 간단하고 전체적인 성능이 좋은 편이다.
        - 하지만 <span class="shl">**클러스터링(clustering)**</span>이라는 문제점이 존재한다.
            - 해시 테이블에 저장되는 데이터들이 고르게 분포되지 않고 여기저기에 연속된 데이터 그룹이 생기는 현상이다.
            - 클러스터들이 점점 커지게 되면 인근 클러스터들과 서로 합쳐지게 되며, 해시 테이블의 특정 위치에 데이터가 몰려 탐사 시간을 오래 걸리게 할 수 있다. 따라서 해싱 효율이 저하된다.
    - **버킷 사이즈보다 큰 경우에는 삽입 불가능**하므로, <span class="shl">**rehashing**</span>이라는 방법을 사용한다.
        - 일정 이상 채워지면(= 기준 **로드 팩터** 비율을 넘어서게 되면) **그로스 팩터(growth factor)**의 비율에 따라 더 큰 크기의 또 다른 버킷을 생성한 후, 새롭게 복사한다.
        - **동적 배열**에서 공간이 가득 찰 경우, **더블링**으로 새롭게 복사해서 옮겨가는 과정과 유사하다.
        
        > **load factor, rehashing, complexity에 관한 참고 자료**: <https://www.geeksforgeeks.org/load-factor-and-rehashing/>
        {: .prompt-info}
        
    
    ![](/assets/img/posts/Python/More-About-Python/2024-03-15-06.png){: .w-75}
    _ref: <https://www.algolist.net/Data_structures/Hash_table/Open_addressing>_
    

<br>

### 2-4. CPython 구현

1. 파이썬의 `dict`는 **해시 테이블**로 구현되어 있으며, 충돌 발생 시 <span class="shl">**오픈 어드레싱**</span> 중 <span class="shl">**선형 탐사**</span> 방식을 사용한다.
    
    > 체이닝 시 `malloc`으로 메모리를 할당하는 오버헤드 때문에, 일반적으로 선형 탐사 방식이 체이닝에 비해 성능이 좋다.
    > 
    
    ![](/assets/img/posts/Python/More-About-Python/2024-03-15-07.png){: .w-75}
    _ref: <https://www.laurentluce.com/posts/python-dictionary-implementation/>_
        
    <br>

2. 그러나 **슬롯의 80% 이상**이 차게 되면 **급격한 성능 저하**가 일어나며, 체이닝과 달리 **전체 슬롯의 개수 이상(로드 팩터 1 이상)은 저장할 수 없다**는 문제점이 있다. 선형 탐사 방식은 공간이 찰수록 탐사에 걸리는 시간이 증가하고, 가득 차게 될 경우 더 이상 빈 공간을 찾을 수 없기 때문이다.
    
    따라서 오픈 어드레싱 방식을 택한 언어(ex. 파이썬, 루비)들은 <span class="shl">**로드 팩터를 작게 잡아 rehashing**</span>을 수행함으로써 성능 저하 문제를 해결한다.
    
    > ex) 파이썬의 로드 팩터 = 0.66, 루비의 로드 팩터 = 0.5
    > 
      
    <br>

3. **파이썬 3.7 이상**에서는 **데이터 입력 순서도 유지**된다.
      
    <br>

    
4. **조회 연산의 시간 복잡도가 $$O(1)$$**로 동일하다는 점에서 `list`와 비교되기도 한다.

    - `list`: 인덱스는 0부터 시작하는 정수형 숫자이다.
    - `dict`: 다양한 불변(immutable) 객체 타입을 키로 사용할 수 있다.

<br>

### 2-5. 주요 연산의 복잡도

| 연산             | 시간 복잡도 | 설명                                 |
| ---------------- | ----------- | ------------------------------------ |
| `len(a) `        | $$O(1)$$    | 요소의 개수를 리턴한다.              |
| `a[key]`         | $$O(1)$$    | 키를 조회하여 값을 리턴한다.         |
| `a[key] = value` | $$O(1)$$    | 키/값을 삽입한다.                    |
| `key in a`       | $$O(1)$$    | 딕셔너리에 키가 존재하는지 확인한다. |

<br>

## References

- “파이썬 알고리즘 인터뷰” - 7장. 배열, 11장. 해시 테이블
- <https://www.cs.utexas.edu/~slaberge/docs/topics/amortized/dynamic_arrays/>
- <https://stackoverflow.com/questions/4853979/amortized-time-of-dynamic-array>
- <https://jakevdp.github.io/PythonDataScienceHandbook/02.01-understanding-data-types.html>
- <https://medium.com/@yasufumy/data-structure-dynamic-array-3370cd7088ec>
- <https://stackoverflow.com/questions/52195579/what-is-the-resizing-factor-of-lists-in-python>
- <https://engineerinsight.tistory.com/311>
- <https://velog.io/@newdana01/CS자료구조-Dynamin-Array란>
- <https://stackoverflow.com/questions/114830/is-a-python-dictionary-an-example-of-a-hash-table>
- <https://ko.wikipedia.org/wiki/해시_충돌>
- <https://www.laurentluce.com/posts/python-dictionary-implementation/>
- <https://kadensungbincho.tistory.com/23>
- <https://realpython.com/python-hash-table/>
- <https://www.algolist.net/Data_structures/Hash_table/Open_addressing>
- <https://www.geeksforgeeks.org/load-factor-and-rehashing/>
