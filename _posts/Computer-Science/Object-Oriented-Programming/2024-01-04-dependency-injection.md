---
title: "[OOP] DI(Dependency Injection, 의존성 주입)"
date: 2024-01-04 12:50:00 +0900
categories: [Computer Science, Object-Oriented Programming]
tags: [oop, di, dependency injection]
math: true
mermaid: true
---

> 본문은 **“개발자가 반드시 정복해야 할 객체 지향과 디자인 패턴”**(최범균 저)을 읽고 정리한 내용입니다.

<br>

로버트 C 마틴은 소프트웨어를 다음의 두 영역으로 구분한다.

1. **어플리케이션 영역**: 고수준 정책 및 저수준 구현을 포함한다.
2. **메인 영역**: 어플리케이션이 동작하도록 각 객체들을 연결해준다.

이때, <span class="ulr">**메인 영역**</span>에서 객체를 연결하기 위해 사용되는 방법으로 <span class="shl">**DI(dependency injection, 의존성 주입)**</span>가 있다. 이에 대해 자세히 알아보자! 

<br>

## 1. 메인 영역과 DI(의존성 주입)

### 1-1. 비디오 포맷 변환기의 요구사항 분석

요구사항을 분석할 때 **변화가 발생하는 부분을 인터페이스로 추상화**함으로써 <span class="shlp">**개방 폐쇄 원칙**</span>을 따를 수 있고, 해당 부분에 **인터페이스를 상속받아 구현한 콘크리트 클래스를 제공**함으로써 <span class="shlp">**의존 역전 원칙**</span>을 따를 수 있다. 이러한 원칙을 따르며 ‘비디오 포맷 변환기’를 설계한다고 가정해보자.

`Worker`에서 변환 요청 정보를 저장하기 위한 `JobQueue` 인터페이스와 변환 처리를 하기 위한 `Transcoder` 인터페이스를 사용한다고 하자. 그리고 `JobQueue` 인터페이스를 상속받아 구현한 콘크리트 클래스로는 `FileJobQueue`, `DbJobQueue`가 있고, `Transcoder` 인터페이스를 상속받아 구현한 콘크리트 클래스로는 `FfmpegTranscoder`, `SolTranscoder`가 있다고 가정한다.

![class diagram](/assets/img/posts/Computer-Science/OOP/2024-01-04-01.png)

이러한 경우, `Worker` 클래스는 `JobQueue`와 `Transcoder`의 구현 객체를, `JobCLI` 클래스는 `JobQueue`의 구현 객체를 필요로 한다.

<br>

### 1-2. 구현 객체를 생성하는 방법

그렇다면 이러한 구현 객체를 어떻게 생성해야 할까?

**사용할 객체를 `Worker` 클래스나 `JobCLI` 클래스에서 직접 생성**한다면, 콘크리트 클래스에 대한 의존이 발생하여 의존 역전 원칙을 위반하게 되고 나아가 개방-폐쇄 원칙 또한 위반하게 된다.

```java
public class Worker {
    ...
    public void run() {
        // 직접 콘크리트 클래스를 사용
        JobQueue jobQueue = new FileJobQueue(); // DIP 위반!
    }
    ...
}
```

이때, <span class="shl">**DI(dependency injection, 의존성 주입)**</span>가 등장한다. 이는 <span class="ulr">**필요한 객체를 직접 생성하거나 찾지 않고 외부에서 넣어주는 방식**</span>으로, 사용할 객체를 전달받을 수 있는 방법을 제공함으로써 구현할 수 있다. 대표적으로는 생성자를 이용하여 객체를 주입하는 방식이 있다.

```java
public class Worker {
    private JobQueue jobQueue;
    private Transcoder transcoder;

    // 생성자를 이용한 의존 객체 주입
    public Worker(JobQueue jobQueue, Transcoder transcoder) {
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }
    ...
}
```

```java
public class JobCLI {
    private JobQueue jobQueue;

    // 생성자를 이용한 의존 객체 주입
    public JobCLI(JobQueue jobQueue) {
        this.jobQueue = jobQueue;
    }
    ...
}
```

<br>

### 1-3. 구현 객체를 주입하는 방법

이처럼 DI에서 필요한 객체를 생성하고 주입해주는 것은 <span class="shl">**메인 영역**</span>에서 담당한다. **메인 영역의 역할**은 다음과 같다.

1. 어플리케이션 영역에서 사용될 객체를 생성한다.
2. 각 각체 간의 의존 관계를 설정한다.
3. 어플리케이션을 실행한다.

```java
public class Main {
    public static void main(String[] args) {
        // 하위 수준 모듈 객체 생성
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();

        // 상위 수준 모듈 객체를 생성하면서 의존 객체 주입
        final Worker worer = new Worker(jobQueue, transcoder);
        ...
        JobCLI cli = new JobCLI(jobQueue);
        ...
    }
}
```

추가로, 각 객체들을 의존 관계에 따라 연결해주는 <span class="shl">**조립 기능을 별도로 분리**</span>하면 변경의 유연함을 얻을 수도 있다.

> 백엔드 개발에서 널리 사용되는 스프링 프레임워크는 이러한 객체 생성 및 조립 기능을 제공하는 DI 프레임워크이다.


```java
public class Assembler {
    public void createAndWire() {
        // 어플리케이션 영역에서 사용될 사위 수준 모듈 객체 생성
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();

        // 상위 수준 모듈 객체를 생성하면서 의존 객체 주입
        this.worker = new Worker(jobQueue, transcoder);
        this.jobCLI = new JobCLI(jobQueue);
    }

    // 실행 대상이 되는 객체를 제공하는 메서드
    public Worker getWorker() {
        return this.worker;
    }
    public JobCLI getJobCLI() {
        return this.jobCLI;
    }
    ...
}
```

```java
public class Main {
    public static void main(String[] args) {
        // 조립기 생성 및 조립 수행
        Assembler assembler = new Assembler();
        assembler.createAndWire();

        // 실행 대상이 되는 객체 확보
        final Worker worker = assembler.getWorker();
        JobCLI jobCli = assembler.getJobCLI();
        ...
    }
}
```

<br>

## 2. DI의 대표적인 방식

DI를 적용하기 위해 의존 객체를 전달받는 대표적인 방법으로는 다음의 두 가지가 있다.

1. **생성자 방식**
    
    생성자를 통해서 의존 객체를 전달받는 방식으로, 1-2에서 살펴봤던 예제와 동일하다. 생성자를 통해 전달받은 객체를 필드에 보관한 뒤, 메서드에서 사용하는 형식이다.
    
2. **설정 메서드 방식**
    
    설정 메서드(ex. `worker.setJobQueue(…)`)를 통해 의존 객체를 전달받는 방법이다.
    

<br>

이러한 두 가지 방식 중에서 <span class="shl">**생성자 방식을 권장**</span>한다. 그 이유는 생성자 방식은 객체를 생성하는 시점에 필요한 모든 의존 객체를 준비할 수 있기 때문이다. 따라서 **객체를 생성하는 시점에서 의존 객체가 정상인지 확인**할 수 있으며, <span class="ulr">**한 번 객체가 생성되면 객체가 정상적으로 동작함을 보장**</span>할 수 있다.

하지만 생성자 방식은 <span class="ulr">**의존 객체가 먼저 생성**</span>되어 있어야 하기 때문에, 의존 객체를 먼저 생성할 수 없다면 생성자 방식을 사용할 수 없게 된다.

<br>

반면, <span class="shl">**설정 메서드 방식**</span>은 객체를 생성한 뒤에 의존 객체를 주입하게 된다. 따라서 의존 객체를 설정하지 못한 상태에서 객체를 사용할 수 있게 되기 때문에, <span class="ulr">**객체의 메서드를 실행하는 과정에서 NPE가 발생**</span>할 수 있다.

그러나 어떠한 이유로 인해 <span class="ulr">**의존할 객체가 나중에 생성된다면**</span> 설정 메서드 방식을 사용해야 한다. 또한, 의존할 객체가 많은 경우, 설정 메서드 방식은 메서드 이름을 통해 어떤 의존 객체가 설정되는지 생성자 방식에 비해 보다 쉽게 알 수 있으므로 **코드 가독성**이 좋아진다.

<br>

## 3. DI와 테스트

단위 테스트는 한 클래스의 기능을 테스트하는 데에 초점을 맞춘다. 그렇다면, 아직 `FileJobQueue` 클래스나 `FfmpegTranscoder` 클래스의 구현이 완료되지 않은 상태에서 `Worker` 클래스의 동작을 테스트하려면 어떻게 해야 할까?

이는 **`Mock` 객체**를 이용하여 해결할 수 있으며, <span class="ulr">**`Worker` 클래스가 DI 패턴을 따른다면 생성자나 설정 메서드를 이용해서 `Mock` 객체를 쉽게 전달**</span>할 수 있다.

```java
@Test
public void shouldRunSuccessfully() {
    // Mockito 등을 이용해서 Mock 객체 생성
    JobQueue mockJobQueue = ...;
    Transcoder mockTranscoder = ...;

    Worker worker = new Worker();
    worker.setJobQueue(mockJobQueue);
    worker.setTranscoder(mockTranscoder);
    worker.run(); // Mock 객체를 이용한 실행
}
```

만약 DI를 사용하지 않는 경우라면, 아직 구현이 완료되지 않은 클래스에서 `Mock` 객체를 리턴하도록 코드를 변경해주어야 하는 상황이 발생하게 된다.

즉, DI를 적용한다면 **필요한 클래스의 구현 완료 여부와 상관없이 `Mock` 객체를 이용**해서 `Worker` 클래스를 테스트할 수 있으며, **`Mock` 객체를 생성하기 위해 기존의 다른 코드를 변경할 필요가 없게** 된다.
