---
title: "[DEVIEW 2023] WebtoonMe 개발기 (NAVER WEBTOON) 정리"
date: 2023-03-25 13:50:00 +0900
categories: [Free Notes, Memo]
tags: [deview, conference]     # TAG names should always be lowercase
math: true
mermaid: true
---

> 발표 자료 및 영상: <https://deview.kr/2023/sessions/565>

<br>

## 1. WebtoonMe란?

> **WebtoonMe** = 주어진 사진 또는 영상을 (실시간으로) 웹툰 화풍으로 바꿀 수 있는 기술

![](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-01.png)

<br>

----

## 2. 모델 개발

### 생성 모델: Model-Centric → Data-Centric

기존에는 성능이 좋은 모델을 만드는 것이 중요했다면, 최근에는 **생성 모델**을 활용하여 

1. task에 적합한 <span class="hl">고품질의 데이터를 생성</span>한 후, 
    
    > 모델을 수정하는 것보다 데이터를 다루는 것이 더 쉽다.
    > 
2. <span class="hl">가볍고 학습이 쉬운 별도의 network</span>를 학습시키는 것
    
    > 모델이 가벼울수록 빠르게 추론이 가능하기 때문에 product화가 편리하다. (ex. U-Net 계열)
    > 

이 트렌드이다.

<br>

### 고품질의 데이터를 어떻게 만들까?

1. **독자적인 “데이터 생성” 프로세스 구축** 
    
    > WebtoonMe: A Data-Centric Approach for Full-Body Portrait Stylization [Back et al. 2022]
    > 
    
    ![](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-02.png)
    
2. **“웹툰” 도메인에 알맞은 얼굴 변환 모델, 배경 변환 모델을 자체적으로 연구/개발**
    - 얼굴 변환 모델- CDSM: Cross-Domain Style Mixing for Face Cartoonization [Kim et al. 2022]
    - 배경 변환 모델 - Interactive Cartoonization with Controllable Perceptual Factors [Ahn et al. 2023]
3. **SoTA 모델의 적극적 활용 및 웹툰 도메인에 최적화 연구**
    
    **<u>얼굴 변환</u>**
    
    - 웹툰 캐릭터의 경우, 사람과 캐릭터 간의 기하학적이 특징이 상이하여 **데포르메**가 심하기 때문에 얼굴 변환이 잘 되지 않는다.
    - 데포르메가 심한 캐릭터이더라도 사람과 얼굴 형태가 유사해질 수 있도록 하는 **warping 알고리즘**을 연구/개발했다.
        
        ![(a) JoJoGAN 변환 결과 (데포르메로 인해 변환이 잘 되지 않음)](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-03.png){: width="80%"}
        _(a) JoJoGAN 변환 결과 (데포르메로 인해 변환이 잘 되지 않음)_
        
        ![(b) warping 알고리즘을 적용한 이미지로 학습하여 변환한 결과](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-04.png){: width="80%"}
        _(b) warping 알고리즘을 적용한 이미지로 학습하여 변환한 결과_
        
    
    **<u>Stable Diffusion</u>**
    
    - **머릿결 표현**이 부족하다는 문제점과, **전경과 배경 간 스타일**이 상이한 문제점을 해결할 수 있었다.

    - 디테일을 개선하는 목적으로 사용한다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-05.png)
    
4. **다양한 최신 데이터셋 활용** (FFHQ, LSUN, SHHQ 등)

<br>

### 현업의 니즈를 바로바로 적용하다.

1. **다양한 피부톤 반영** (피부 영역의 분포를 가져옴)
    
2. **다양한 조명 조건 반영**
    
3. **얼굴 외의 피부 영역에서도 캐릭터 고유 피부 반영** (피부 영역을 segmentation)
    
4. **추론 속도 개선**
    - preprocessing과 inference를 병렬로 처리한다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-06.png){: width="80%"}
        
    - 7개의 TensorRT가 메모리에 상주하며 빠른 추론 및 경량화의 효과를 낸다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-07.png){: width="80%"}
        
    - 가벼운 장비 구성으로 언제 어디서나 쉽고 빠르게 시연이 가능하다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-08.png){: width="80%"}
        
5. **빠른 ad-hoc 기능 반영** (ex. 동일 영상 내 다중 캐릭터 변환을 위한 tracking 알고리즘)
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-09.png){: width="80%"}
    

<br>

----

## 3. 빠르게 개발하고 널리 알리기

> **Bottom-up 스타일의 AI 프로덕트**
> 
> - 초기에는 추상화 높은 방법론으로 접근 (최대한 빠른 결과 & 작지만 유의미한 성공)
> - 전문가를 섭외해서 추상화 낮추기
{: .prompt-info }

### 빠른 프로덕트 프로토타이핑

- 초기
    
![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-10.png){: width="80%"}
    
- 후기
    
![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-01-11.png){: width="80%"}
    

<br>

### 홍보의 효과

실제 사용자들로부터 유의미한 피드백(ex. 피부색, inference 속도, 선호 캐릭터 등)을 들을 수 있다.

<br>

----

## 4. 활용

- 쇼핑 라이브 기술 지원
- 사내 실시간 체험 존 구축
- 다양한 적용 시나리오 발굴 (ex. 인터랙티브 웹툰, 웹툰 AI 아바타, 네 컷 사진 등)

<br>

----

## 5. 다음은?

Stable Diffusion을 이용한 높은 개인화, 실시간 변환, 다양한 웹툰 캐릭터 지원…

<br>

----

## 6. Q&A

1. **Snow AI와의 차별점**
    - Snow AI는 개인에게 특화된 결과물을 제공하지만 비디오 변환은 지원하지 않는다.
    - WebtoonMe는 특정 웹툰과 비슷한 화풍의 결과물을 제공하지만 경량화와 가속화 기술을 통해 비디오 변환도 지원한다.
2. **FastAI 활용 과정**
    - FastAI는 추상화가 높은 라이브러리로, API화가 되어 있기 때문에 단 한 줄만으로도 파라미터에 대한 고민 없이 baseline 정도의 성능을 얻을 수 있다.
    - 프로토타입으로 사용하기 적합하다.
3. **웹툰마다 새로 학습하는 것인지?**
    - 웹툰 캐릭터마다 학습을 수행한다.
    - 이미지 수십 장만으로도 충분히 학습 가능하다.
4. **GAN → SD 전환**
    - GAN은 control이 쉽지만 quality가 좋지 않고 개인화가 어렵다.
    - SD는 capacity & diversity가 좋지만 control이 어렵다. (ex. eye mapping) 하지만 control을 정교하게 할 수 있어지면서 사용하게 되었다.
5. **접근**
    - 다른 모델들이 왜 웹툰 도메인에서는 잘 안 되는지에 의문을 가지고 접근했다.
    - 처음에는 FastAI를 통해 빠르게 프로토타입을 내보고, 모델을 재구현하고 수정 작업을 진행하면서 결과를 잘 만드는 데에 집중했다.
6. **경량화**
    - TensorRT, Ray 정도로 했으며 추가 연구는 진행하지 않았다.
    - 모바일에도 포팅하려면 필요할 것으로 보인다.
7. **자동화**
    - 데이터 생성 프로세스의 전 과정이 자동화 되어 있다.
    - warping 알고리즘, MLOps 관련 툴 등
8. **프레임 간 연결을 자연스럽게 하기 위해 추가 기술을 사용했는지?**
    - 데이터셋 고도화 방법론이 메인이기 때문에 진행하지 않았다.

<br>

----

<br>

> **메모**
> - 생성 모델을 통해 고품질의 데이터를 생성하고, 그 데이터로 더 가벼운 모델을 학습시키는 방법론
> - bottom-up 스타일의 AI 프로덕트 개발 과정 (빠른 프로토타입 → 결과 개선) (추상화 줄여나가기)
> - 데이터 생성 프로세스의 자동화
{: .prompt-tip}
