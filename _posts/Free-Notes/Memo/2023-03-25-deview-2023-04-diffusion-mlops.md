---
title: "[DEVIEW 2023] 값비싼 Diffusion Model을 받드는 저비용 MLOps (Symbiote AI) 정리"
date: 2023-03-25 16:50:00 +0900
categories: [Free Notes, Memo]
tags: [deview, conference]     # TAG names should always be lowercase
math: true
mermaid: true
---

> 발표 자료 및 영상: <https://deview.kr/2023/sessions/533>

<br>

## 1. Diffusion Model

> <u>Multimodal</u> <u>image generation</u> model with <u>diffusion process</u>
 
- **image generation** - BigGAN → StyleGAN2 → DALL-E 2
- **multimodal** - pair data가 필요하다.
- **diffusion process** = denoising (clearer image 만드는 과정)
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-01.png){: width="80%"}
    
<br>

### 단점

1. 생성이 느리다. (= GPU 비용이 많이 든다.)
    
    > 생성 시 50회 정도 반복하므로 GAN보다 느리다.
    > 
2. 학습이 느리고 리소스가 많이 든다. (= GPU 비용이 많이 든다.)
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-02.png){: width="70%"}
    
    
<br>

### <span class="hlo">최근 6개월 간 Diffusion Model의 발전 과정</span>

![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-03.png){: width="30%"}

1. **Stable Diffusion**
    - 150,000 A100 GPU Hours = $600,000
    - 생성 시 50 step = V100 기준 10초
2. **DreamBooth & LoRA**
    - 4억 개의 이미지 없이도 새로운 모델을 학습할 수 있을까?
    - LENSA
3. **xformers**
    - **FlashAttention** - GPU 계산에 사용되는 메모리(SRAM) ↔ 크지만 I/O가 느린 메모리(HBM) 간의 다수의 I/O 과정을 없애는 방법이다.
        
        ![GPU 구조](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-04.png){: width="50%"}
        _GPU 구조_
        
4. **Prompt-to-Prompt**
    - slight한 수정 사항을 적용할 수 있다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-05.png){: width="70%"}
        
    - 기존의 diffusion model로 학습 데이터를 만들어서 새로운 모델을 학습하는 데에 사용할 수 있다.
5. **InstructPix2Pix**
    - **Prompt-to-Prompt로 만든 데이터**와 **GPT-3로 만든 데이터**를 통해 기존의 diffusion model을 재학습한다.
    - 다음과 같은 대화형 수정 AI를 만들 수 있다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-06.png){: width="70%"}
        
6. **ControlNet**
    - 기존의 방식과는 다르게 텍스트가 아닌 **이미지**로 생성과 수정을 할 수 있는 방식이다.
    - reference image를 입력으로 넣는다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-07.png){: width="80%"}
        
<br>

![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-08.png){: width="60%"}
_전체 정리_

<br>

---

## 2. 저비용: 어떻게 최적화 했는지

> generation이 메인인 서비스 ⇒ **latency**와 **scalability**가 중요하다.
{: .prompt-warning}
    
### Distillation

> 생성 시, 50 step에 V100 기준으로 10초가 걸린다. 추론 횟수를 줄이려면?

- **knowledge distillation**
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-09.png){: width="80%"}
    
- distillation으로 **모델 크기**도 줄일 수 있으나, **추론 step 수**를 절반씩 줄일 수 있다. ($2^n$씩)
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-10.png){: width="80%"}
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-11.png){: width="80%"}
    
- Image2Image 모델로 기존 모델이 학습한 데이터셋에서 학습한 결과, 1번의 distillation은 평균 1.5일 정도 소요되었으며 전체 학습 기간은 6일이었다.
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-12.png){: width="80%"}
    
- **Free 버전**에서는 distillation 모델을 사용하여 latency를 크게 줄여 사용자 경험을 향상시킬 수 있고, **Pro 버전**에서는 기존의 diffusion 모델을 사용하여 완성도를 향상시킬 수 있다.
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-13.png){: width="80%"}
    
    
<br>

### <span class="hlo">TensorRT</span>

- **GPU 하드웨어 구조**
    - **SRAM**과 **HBM** 간 I/O는 느리다.
    - **GPU가 더 빠른 연산**: ex) `torch.matmul(A, B)` (병렬 처리 가능)
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-14.png){: width="80%"}
        
    - **CPU가 더 빠른 연산**: ex) `A.cos()` → `A.cos()` (두 번의 연산을 한 번에 처리하면 I/O가 줄어듦)
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-15.png){: width="80%"}
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-16.png){: width="80%"}
        
- **TensorRT의 역할**은 다음과 같다.
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-17.png){: width="70%"}
    
- **사용 방법**
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-18.png){: width="80%"}
    
    
<br>

### Triton (OpenAI)

- Triton이란 OpenAI가 만든 언어/컴파일러로, I/O 최적화된 CUDA 코드를 최적화한다. (ex. swish, gelu, groupnorm, layernorm, cast 등)
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-19.png){: width="80%"}
    
- TensorRT를 통해 static graph를 최적화할 수는 있으나, static graph만으로는 모든 것을 알 수 없다.
    - TensorRT가 최적화 하지 못하는 코드를 Triton으로 최적화 할 수 있다.
    - ex) `gelu` - 7번의 I/O를 1번으로 줄일 수 있다.
        
        ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-20.png){: width="80%"}
        
<br>

### 최종 Latency 개선 결과
![최종 latency 개선 결과](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-21.png){: width="70%"}

<br>

---

## 3. MLOps: 어떻게 Serving 했는지

### 인프라 구조

- 클라우드가 아닌 bare metal 서버를 사용했다.
- kubespray로 self-hosted 클러스터를 운영했다.
- 2개의 서비스와 8개의 GPU 서버로 구성했다.

![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-22.png){: width="80%"}
    
<br>

### <span class="hlo">Triton Inference Server (NVIDIA)</span>

> OpenAI의 컴파일러인 Triton과는 다른 것이다.

- 장점
    - ensemble model, scheduler, dlpack 지원
    - TensorRT
    - gRPC
    - dynamic batching
- 전체 과정을 간단히 나타내면 다음과 같다.
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-23.png){: width="80%"}
    
- Triton을 활용하면 inference time을 다음과 같이 최적화 할 수 있다.
    - 하나의 request를 처리할 때에도 다른 request와 함께 동시에 처리할 수 있다.
    - ensemble 시 유리하다.
    
    ![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-24.png){: width="80%"}
    
- diffusion model의 앞 뒤로 한 두 개의 pipeline만 있다면 필요하지는 않다.
- 참고: <https://openai.com/research/techniques-for-training-large-neural-networks>

<br>

![Untitled](/assets/img/posts/Free-Notes/Memo/2023-03-25-04-25.png)
_latency와 scalability를 모두 잡았다!_

<br>

---

<br>

> **메모**
> - 학습 데이터를 만들 때 diffusion model을 사용하는 방법?
> - GPU 구조에 대한 이해
> - TensorRT, Triton
> - free / pro 버전
{: .prompt-tip}
