---
title: "[OOP] 객체지향의 장점과 특징"
date: 2023-11-28 12:50:00 +0900
categories: [Computer Science, Object-Oriented Programming]
tags: [oop, dependency, encapsulation, 캡슐화, 의존]
math: true
---

> 본문은 **“개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴”**(최범균 저)을 읽고 정리한 내용입니다.

<br>

소프트웨어의 가치는 사용자가 요구하는 기능을 올바르게 제공하는 데에 있다. 하지만 요구사항은 언제나 변하기 때문에 <span class="hl">**소프트웨어를 변화 가능한, 유연한 구조로 설계**</span>하는 것이 중요하다.

그리고 이를 달성하기 위한 핵심 기법 중 하나가 바로 <span class="hl">**객체지향(Object Oriented)**</span>이다!

객체지향이란 무엇인지, 어떠한 점을 통해 유연함을 제공하는지 등에 대해 관련 용어들을 살펴보며 알아보자 🚀

<br>

## 절차지향 vs. 객체지향

객체지향적인 코드를 작성하기 위해서는 절차지향적으로 작성하면 안 된다. 그 이유는 무엇일까?

### 절차지향 (Procedural-Oriented)

절차지향이란 프로시저(procedure)로 프로그램을 구성하는 기법으로, 다음과 같은 특징으로 인해 **“데이터를 중심으로”** 프로그램을 구현 하게 된다.

1. 각 프로시저가 **데이터를 사용**해서 기능을 구현한다.
2. 여러 프로시저가 **동일한 데이터**를 공유한다.

따라서 절차지향적으로 코드를 작성하는 것은 자연스러운 과정이기에 쉽지만, 프로그램의 규모가 커져 데이터의 종류 및 프로시저가 증가하게 되면 다음과 같은 문제들이 발생할 수 있다.

1. 데이터 타입이나 의미를 변경해야 할 때, **함께 수정해야 하는 프로시저가 증가**한다.
2. 같은 데이터를 프로시저들이 **서로 다른 의미로 사용**하는 경우가 발생한다.

즉, 절차지향적으로 프로그램을 구성하게 되면 **코드의 수정이 어려워지며**, 새로운 요구사항에 **대응하는 데에 더 많은 비용**이 필요하게 된다.

<br>

### 객체지향 (Object-Oriented)

객체지향적인 프로그램은 객체(object)들의 네트워크로 구성된다. 이때, <span class="hl">**객체(object)**</span>란 다음의 요소로 구성된다.

1. <span class="hl">데이터</span>
2. <span class="hl">데이터와 관련된 프로시저</span> (오퍼레이션, 메서드, 함수)

객체지향적인 코드에서는 데이터와 프로시저를 객체 별로 알맞게 정의하므로 **객체의 데이터를 변경하더라도 해당 객체로만 변화가 집중**되며 다른 객체에는 영향을 주지 않는다.

따라서 최초에는 객체지향적으로 설계하는 데에 더 많은 노력이 들어갈 수는 있으나, 요구사항이 변화하더라도 절차지향 방식보다 **프로그램을 더 쉽고 유연하게 수정**할 수 있게 된다.

<br>

## 객체의 기능, 책임, 그리고 의존

### 객체의 기능 (Operation)

> 객체의 핵심은 **“기능”을 제공하는 것**이다. 즉, 내부적으로 어떤 데이터를 가지고 어떻게 동작하는지는 중요하지 않다. 객체의 기능을 중심으로 객체지향의 특징을 살펴보기 위해 관련 용어를 정리하고 넘어가자!
{: .prompt-info}


- **오퍼레이션 (operation)**: 객체가 제공하는 기능

- **시그니처 (signature)**: 오퍼레이션의 사용법
    
    > 다음의 세 가지 요소로 구성된다.
    
    1. 기능 식별 이름
    2. 파라미터 및 파라미터 타입
    3. 기능 실행 결과 값
    
- **인터페이스 (interface)**: 객체가 제공하는 모든 오퍼레이션 집합
    
    > 객체가 제공하는 기능에 대한 명세서라고 생각하면 된다.
    
- **타입 (type)**: 서로 다른 인터페이스를 구분할 때 사용되는 명칭

- **클래스 (class)**: 실제 객체의 구현 정의 (데이터 및 오퍼레이션 구현)

- **인스턴스 (instance)**: 클래스를 통해 메모리에 생성된 객체

- **메시지 (message)**: 오퍼레이션의 실행을 요청하는 것
    
    > 메서드를 호출하는 것이 해당된다.
    

<br>

### 객체의 책임 (Responsibility)

객체는 **객체가 제공하는 기능으로 정의**가 되므로, 다시 말하면 <span class="hl">**객체마다 자신만의 책임(responsibility)**</span>가 있는 것이다. 또한, 한 **객체가 갖는 책임을 정의한 것**이 바로 **타입** 혹은 **인터페이스**가 된다.

필요한 기능들을 **객체들에게 어떻게 할당하느냐**에 따라 객체의 구성이 달라지게 되는데, 이때 **지켜야 할 규칙**은

> 객체가 갖는 <span class="hl">**책임의 크기는 작을수록 좋다**</span>. 즉, **하나의 객체가 제공하는 기능의 개수가 적을수록** 좋다.
{: .prompt-tip}

는 것이다. 기능을 어떻게 분배해야 할지 헷갈릴 때는 이 규칙을 떠올리자! 🤓

한 객체에 많은 기능이 포함된다면 **객체에 정의된 많은 오퍼레이션들이 데이터를 공유하는 방식**으로 프로그래밍되기 때문에, 곧 **절차지향 방식**과 동일한 구조가 되기 때문이다.

객체가 갖는 책임의 크기가 작아질수록 객체지향의 장점인 <span class="hl">**“변경의 유연함”**</span>을 얻을 수 있으며, 이는 <span class="hl">**“단일 책임 원칙(Single Responsibility Principle, SRP)”**</span>과 관련이 있다. 이 원칙에 따르면 객체는 단 한 개의 책임만을 가져야 하며, 이를 통해 변경해야 할 부분이 한 곳으로 집중되게 된다.

<br>

### 의존 (Dependency)

객체지향적인 프로그램에서는 다른 객체가 제공하는 기능을 이용해서 자신의 기능을 완성하는 객체가 출현할 수 있다. 이러한 현상은 실제 구현에서는 다음과 같은 형태로 나타나며, 이를 **그 객체에** <span class="hl">**의존(dependency)**</span> **한다**고 표현한다.

1. 한 객체가 다른 객체를 **생성**한다.
2. 다른 객체의 **메서드를 호출**한다.
3. 다른 객체를 **파라미터로** 전달받는다.

이처럼 다른 타입에 의존한다는 것은 <span class="hl">**의존하는 타입에 변경이 발생할 때 나도 함께 변경될 가능성이 높다**</span>는 것을 의미한다.

> 변경은 객체들의 **의존 관계를 따라**서 **전이**된다.
{: .prompt-tip}

의존이 상호간에 미치는 영향을 정리하면 다음과 같다.

1. 내가 변경되면 나에게 의존하고 있는 코드에 영향을 준다.
2. 나의 요구가 변경되면 내가 의존하고 있는 타입에 영향을 준다.

특히, 의존이 순환해서 발생할 경우에는 변경의 여파가 나 자신에게 다시 영향을 줄 수 있으므로 다른 방법이 없는지 고민해야 한다. **순환 의존이 발생하지 않도록** 하는 것과 관련이 있는 원칙은 <span class="hl">**“의존 역전 원칙(Dependency Inversion Principle, DIP)”**</span>이다.

<br>

## 캡슐화 (Encapsulation)

> 객체지향의 장점은 <span class="hl">**한 곳의 변화가 다른 곳에 미치는 영향을 최소화**</span>한다는 데에 있다. 그리고, 이러한 동작은 **캡슐화**를 통해서 이루어진다.
{: .prompt-tip}

<span class="hl">**캡슐화(encapsulation)**</span>란, 객체가 내부적으로 기능을 어떻게 구현하는지를 감추는 것으로, 내부의 기능 구현이 변경되더라도 **그 기능을 사용하는 코드는 영향을 받지 않아** <span class="hl">**내부 구현 변경의 유연함을 달성**</span>할 수 있다.

**절차지향적인 코드**에서는 데이터를 직접적으로 사용하여 데이터의 변화에 직접적인 영향을 받으므로, 데이터를 사용하는 코드들이 연쇄적으로 수정되어야 한다. 이에 해당하는 코드들이 많을수록 일부는 수정하지 못하는 실수를 할 가능성이 높아진다.

이러한 절차지향적인 코드를 방지하고 캡슐화를 잘 하기 위해 기억해야 할 규칙에 대해 알아보자.

<br>

### [규칙 #1] Tell, Don’t Ask

> 데이터를 물어보지 말고, 기능을 실행해달라고 말하라!
{: .prompt-info}

다른 객체에서 특정 데이터를 가져와서 직접 동작을 수행하기보다, 다른 객체에게 기능을 실행해달라고 요청하자. 이러한 방식으로 코드를 작성하면 자연스럽게 해당 기능을 어떻게 구현했는지가 감춰져 **기능 구현이 캡슐화** 된다.

<br>

### [규칙 #2] 디미터 법칙 (Law of Demeter)

> 어떤 객체에 대해서 단 한 번의 메서드 호출만을 사용하자!
{: .prompt-info}

다음과 같은 간단한 규칙들을 통해 “Tell, Don’t Ask” 규칙을 따를 수 있도록 만들어주는 법칙이다.

1. 메서드에서 **생성**한 객체의 메서드만 호출
2. **파라미터**로 받은 객체의 메서드만 호출
3. **필드로 참조**하는 객체의 메서드만 호출

쉽게 이야기하면, <span class="hl">**어떤 객체에 대해 단 한 번의 메서드 호출만을 사용**</span>할 수 있다는 것이다. 이를 통해 데이터 중심이 아닌 기능 중심으로 코드를 작성할 수 있게 되어 캡슐화가 향상된다.

> 이해하기 쉬운 예시로, 신문 배달부가 고객에게 신문을 판매 후 요금을 받아가는 상황을 생각해볼 수 있다. 신문 배달부가 고객의 지갑을 뒤진 후(`customer.getWallet()`) 돈이 있는지 확인하고(`wallet.getTotalMoney()`) 직접 돈을 꺼내가는 것보다 고객이 돈을 지불하도록 요청(`customer.getPayment()`)하는 것이 현실적이다!
>


**디미터의 법칙을 지키고 있지 않을 때 나타나는 전형적인 증상**은 다음과 같다. 이러한 증상이 발견되면 관련 기능을 캡슐화하도록 노력해야 한다.

1. 연속된 `get` 메서드 호출
2. 임시 변수의 `get` 호출이 많음

<br>

## 객체지향 설계 과정

> 객체지향 설계란 다음의 작업을 반복하는 과정이다.

1. 제공해야 할 기능을 찾거나 세분화하고, 그 기능을 알맞은 객체에 할당한다.
    - 기능을 구현하는 데에 필요한 데이터를 객체에 추가한다.
    - 기능은 최대한 캡슐화해서 구현한다.
    - 한 클래스에 여러 책임이 섞여 있다는 것을 알게되면, 객체를 새로 만들어서 책임을 분리한다.
2. 객체 간에 어떻게 메시지를 주고받을지 결정한다.
    
    > 객체가 기능을 제공할 때 사용할 인터페이스가 도출된다.
    
3. 1번과 2번을 지속적으로 반복한다.
    
    > 구현을 진행해나가면서 점진적으로 완성되기 때문에, 계속해서 설계가 변경되게 된다. 따라서 유연한 구조를 갖도록 노력해야 한다.