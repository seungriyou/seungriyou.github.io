---
title: "[Paper] Crowdsourcing Annotations for Visual Object Detection"
date: 2022-09-20 19:50:00 +0900
categories: [Deep Learning, Paper Review]
tags: [paper review, annotation]
math: true
---

> 📎 Paper: <http://vision.stanford.edu/pdf/bbox_submission.pdf>

<br>

crowdsourcing을 이용한 bounding-box annotation system에 대해 다루는 논문이다. (스탠포드 Fei-Fei 교수님 연구실 논문!) 회사에서 crowdsourcing 사업을 진행했어서 읽었던 논문이다.

<br>

## 1. Introduction

> bounding box annotation을 crowd-source 하기 위한 **fully automated, highly accurate, cost-effective** 한 시스템

### Requirements

- **quality** - 각 bbox는 tight 해야 한다.
- **coverage** - 모든 object는 bbox를 가져야 한다.

> **cost-effective(품질은 보장하면서 비용을 최소화)** 한 방법으로 high quality와 complete coverage를 달성하는 방법은?
{: .prompt-tip}

<br>

### Three sub-tasks

> 한 작업자가 bbox를 그리고, 다른 작업자가 bbox의 품질을 확인하고, 또 다른 작업자가 모든 object가 bbox로 표시되었는지 확인하는 방법

- **drawing**: 이미지에서 하나의 물체에 대응하는 하나의 bbox를 그린다.
- **quality verification**: bbox가 올바르게 그려졌는지 확인한다.
- **coverage verification**: 모든 object에 bbox가 그려졌는지 확인한다.
    
    > verification 과정은 binary answer가 필요하므로 이미 잘 알려진 majority voting과 같은 방법을 적용할 수 있다.
    {: .prompt-info}
    
<br>

---

## 2. Approach

### Work Flow

![Untitled](/assets/img/posts/Deep-Learning/Paper-Review/2022-09-20-1.png){: style="max-width: 70%"}

1. **drawing task**: 하나의 object에 대응하는 하나의 bbox를 그린다. (ex. raccoon)
    - **worker training (rules)**
        1. all visible part & as tightly as possible
        2. include only one
        3. new instance
        4. check the check box when completed
    - **qualification test**
        - rule을 제대로 숙지했는지 테스트 이미지 셋을 통해서 확인 후 instant feedback을 전송한다.
        - 충분히 tight 하지 않음 / solicited object가 아님 / 이미 bbox가 있는 object 임
    - 그 후 실제 이미지에서 작업할 수 있다.
2. **quality verification task**: 새롭게 그려진 bbox의 quality를 측정하고 good bbox는 DB에, bad bbox는 버린다.
    - 작업자가 집중할 수 있도록 이미지 당 하나의 bbox만을 보여준다.
    - **worker training (rules)**
        1. include an instance of the required object
        2. all visible part & as tightly as possible
        3. include only one
    - **qualification test**
    - **quality control (gold standard)**
        - good bbox를 bad로, bad bbox를 good로 평가하는 경우를 방지하기 위함이다.
        - batch에 일부 포함되는 validation images를 작업자가 올바르게 평가해야 작업 내역이 accept 된다.
            
            > **validation image를 위한 good & bad bbox를 얻는 방법**
            > 
            > - bad bbox는 good bbox를 변형해서 생성할 수 있다.
            > - good bbox는 **majority voting**을 통해 모을 수 있다.
            >     1. 특정 object를 포함하는 image에서 일부를 샘플링하고 bbox를 얻는다.
            >     2. 여러 작업자들이 bbox에 평점을 매기고, strong consensus(at least 3 workers)가 있는 것들을 “gold standard”로 선정한다.
            {: .prompt-info}

3. **coverage verification task**: raccoon에 해당하지만 아직 bbox로 표시가 되지 않은 object가 있는지 확인하고, 모두 표시가 되었으면 완료한다.
    - 모든 instance가 bbox를 가지고 있는지 확인한다.
    - 같은 object를 포함하는 이미지들이 한 명의 annotator에게 배정된다.
    - 마찬가지로 **worker training, qualification test**를 수행한다.
    - **quality control**
        - 두 종류의 validation images를 사용한다.
            - completely covered는 majority voting을 통해 생성할 수 있다.
            - 그렇지 않은 것은 bbox의 subset을 지우는 방법으로 생성할 수 있다.
4. 모든 raccoon이 bbox로 표시될 때까지 반복한다.

<br>

### Two principles

- simple (draw only one bbox)
- have a fixed and predictable amount of work
    
<br>

---

## 3. Experiments

20,000개의 카테고리를 가진 ImageNet 데이터셋에서 10개의 카테고리를 선정하였고, 각 카테고리 당 200개의 이미지를 randomly sample 하였다.

### Overall quality

- **image level**: 97.9% images가 completed covered
- **bbox level**: 99.2% bboxes가 accurate
- 해당 시스템을 통해 highly accurate bbox가 생성되었다.

<br>

### Overall cost

> cost = 작업자가 소비한 시간
{: .prompt-info}

- drawing task가 quality/coverage verification task 보다 2배 이상 오래 걸린다.

    > verification task의 경우, binary answer만 필요로 하기 때문이다.

- 해당 시스템과 consensus based 방법의 cost 비교
    
    |  | our approach | consensus based approach | how expensive |
    | --- | --- | --- | --- |
    | mean | 88.0 sec | 116.9 sec | <span style="color: crimson">32.8%</span> |
    | median | 42.4 sec | 58.8 sec | <span style="color: crimson">38.9%</span> |

![Untitled](/assets/img/posts/Deep-Learning/Paper-Review/2022-09-20-2.png){: style="max-width: 70%"}

<br>

### Quality control

- **drawing task**: quality verification task를 통해 control
    - acceptance ratio = 62.2%
- **quality verification task**: majority voting 방법을 이용한 “gold standard”를 통해 control
    - validation images를 대상으로 performance를 평가한다.
    - acceptance ratio = 89.9%
- **coverage verification task**: quality verification task와 비슷
    - validation images를 대상으로 performance를 평가한다.
    - accpetance ratio = 95.0%

> drawing task는 더 time consuming 할 뿐만 아니라 더 difficult 하다.

<br>

### Effectiveness of worker training

drawing task에서 worker training 과정을 삭제하였을 때에 비해 worker training 과정을 진행할 때의 quality verification acceptance가 4.2% 높았다.

![Untitled](/assets/img/posts/Deep-Learning/Paper-Review/2022-09-20-3.png){: style="max-width: 70%"}
