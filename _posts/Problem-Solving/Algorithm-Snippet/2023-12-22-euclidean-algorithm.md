---
title: "[Algorithm] 유클리드 호제법(Euclidean Algorithm): 최대공약수(GCD)와 최소공배수(LCM) 구하기"
date: 2023-12-22 12:50:00 +0900
categories: [Problem Solving, Algorithm Snippet]
tags: [algorithm, python, math, gcd, lcm]
math: true
image: 
  path: /assets/img/posts/Problem-Solving/Algorithm/thumbnail.png
---

## 유클리드 호제법(Euclidean Algorithm)

유클리드 호제법이란, **두 개의 양의 정수에 대해 최대공약수(GCD, greatest common divisor)**를 구하는 알고리즘이다.

> “호제법”이라는 것은 두 수가 서로 상대방 수를 나누어 원하는 수를 얻는 알고리즘을 의미한다.

<br>

유클리드 호제법의 내용을 간단히 정리하면 다음과 같다.

> 두 양의 정수 $a, b \space (a>b)$에 대하여 $a=b \cdot t+r$ $(0\le r<b)$, 즉 $r =a\mod b$ 라고 하자. 이때,
>
> - $r=0$이라면, $\gcd (a, b)=b$이다.
> 
> - 그렇지 않다면, $\gcd(a, b)=\gcd(b, r)$이다.
{: .prompt-tip}

<br>

이러한 유클리드 호제법을 이용하여 두 양의 정수의 **최대공약수**와 **최소공배수**를 구할 수 있다. 각 방법을 알아보기 전에, 어떻게 유클리드 호제법을 증명할 수 있는지 살펴보자.

<br>

### 증명

<span class="shlb">$r=a \mod b$</span>이고 <span class="shlb">$a=b\cdot t +r$</span>일때, <span class="shl">$\gcd(a, b)=\gcd(b, r)$</span> 임을 증명해보자.

1. 우선, $d=\gcd(a,b)$로 가정한다.
    
    기본 정의에 의해 $a$와 $b$는 모두 $d$로 나누어떨어지는데, $a =b\cdot t+r$이므로 $r$ 또한 $d$로 나누어떨어지게 된다.
    
    즉, $b$와 $r$이 모두 $d$로 나누어떨어지므로, $\gcd(b,r)$은 $\gcd(a, b)$(== $d$)로 나누어떨어진다.
    
2. 이어서 $c=\gcd(b, r)$로 가정한다.
    
    기본 정의에 의해 $b$와 $r$은 모두 $c$로 나누어떨어지는데, $a=b\cdot t+r$이므로 $a$ 또한 $c$로 나누어떨어지게 된다.
    
    즉, $a$와 $b$가 모두 $c$로 나누어떨어지므로, $\gcd(a,b)$는 $\gcd(b,r)$(== $c$)로 나누어떨어진다.
    
<br>

이를 모두 종합해보면 <span class="shlp">$\gcd(a,b)$와 $\gcd(b,r)$이 서로의 값으로 나누어떨어지기 때문</span>에, <span class="shl">$\gcd(a,b)=\gcd(b,r)$</span> 임을 확인할 수 있다.

<br>

## 최대공약수(GCD) 구하기 (in Python)

개념적으로 접근하면 다음과 같이 **재귀**적으로 함수를 작성할 수 있다.

```python
def gcd(a, b):
    if (r := a % b) == 0:
        return b
    else:
        return gcd(b, r)
```

<br>

이를 **반복문**(w/ 파이썬 다중 할당)으로 바꾸면 다음과 같다.

```python
def gcd(a, b):
    while b != 0:
        a, b = b, a % b
    return a
```

<br>

또한, 시간 복잡도는 <span class="hl">$O(\log(\min(a,b)))$</span>이다. 이에 대한 자세한 유도 과정은 **[링크1](https://www.baeldung.com/cs/euclid-time-complexity#ea-div-complexity)**, [**링크2**](https://www.geeksforgeeks.org/time-complexity-of-euclidean-algorithm/)를 참고한다.

<br>

> 이때, `gcd(a, b)` 함수에서 $a$와 $b$의 **입력 순서가 꼭 $a>b$일 필요는 없다**. 왜냐하면 $a <b$인 경우일지라도, 첫 나머지 $r$을 구하고 $b$와 $r$로 입력 값을 바꿈으로써 그 순서가 맞추어지게 되기 때문이다.
>
> 다음과 같이 입력 값의 순서를 바꾸어서 각 단계마다 값을 출력해보면 동작을 확인할 수 있다.
>
> ```python
> print(gcd(1071, 1029))
> # a=1071 / b=1029
> # a=1029 / b=42
> # a=42 / b=21
> # 21
> 
> print(gcd(1029, 1071))
> # a=1029 / b=1071
> # a=1071 / b=1029
> # a=1029 / b=42
> # a=42 / b=21
> # 21
> ```
{: .prompt-tip}

<br>

## 최소공배수(LCM) 구하기 (in Python)

두 양의 정수의 **최소공배수(LCM, least common multiple)**를 구하려면, **두 수를 곱한 값을 최대공약수로 나누면** 된다.

> **두 양의 정수의 곱**은 두 수의 **최대공약수와 최소공배수의 곱**으로 나타낼 수 있다.
>
> $$
> a\times b=\gcd(a, b) \times \text{lcm}(a,b)
> $$
{: .prompt-info}

<br>

즉, 다음과 같은 순서로 로직을 작성할 수 있다.

1. 유클리드 호제법을 통해 두 수의 최대공약수를 구한다.
2. 두 수의 곱을 1번에서 구한 최대공약수로 나눈다.

<br>

```python
def lcm(a, b):
    return a * b // gcd(a, b)
```

<br>

## References

- <https://ko.wikipedia.org/wiki/유클리드_호제법>
- <https://app.codility.com/programmers/lessons/12-euclidean_algorithm/>
- <https://tech.lonpeach.com/2017/11/12/Euclidean-algorithm/>
- <https://www.geeksforgeeks.org/time-complexity-of-euclidean-algorithm/>
- <https://www.baeldung.com/cs/euclid-time-complexity>
- <https://www.geeksforgeeks.org/mathematical-algorithms/mathematical-algorithms-gcd-lcm/>
