---
title: "[Paper] Face, Body, Voice: Video Person-Clustering with Multiple Modalities"
date: 2022-04-26 19:50:00 +0900
categories: [Deep Learning, Paper Review]
tags: [paper review, clustering, face clustering, person clustering]
math: true
---

> 📎 Paper: <https://arxiv.org/abs/2105.09939>

<br>

## 0. Abstract

- person-clustering in videos (grouping characters according to their identity)
- previous methods
    - narrower task of face-clustering
    - ignore other cues such as voice, appearance, and the editing structure of the videos
- contributions
    - Multi-Modal High-Precision Clustering algorithm for person-clustering in videos using cues from several modalities (face, body, voice)
    - Video Person-Clustering dataset
        - body-tracks, face-tracks when visible, voice-tracks when speaking
- effectiveness of using multiple modalities for person-clustering

<br>

---

## 1. Introduction

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-01.png){: .w-75}

- 기존의 identity clustering 방법의 대부분은 **face-level 정보를 이용**하는 것으로 **한계**가 있다.
    1. 많은 **informative cues**(voice, appearance, editing structure)**를 무시**한다.
        
        > **editing structure** - neighboring shots를 생각하면 된다.
        > 
    2. story understanding과 같은 **downstream application에서의 사용성을 제한**한다.
        
        > 보이지 않는 모든 등장인물에 대해 알아야 story-line을 이해할 수 있다.
        > 
    
    **⇒ 따라서 multi-modal 형태의 person-level 정보를 사용하는 person-clustering 방법을 제안한다.**
    

- 한 사람에 대한 **multiple modalities가 사용될 수 있는 곳**은 다음과 같다.
    1. how to obtain **pure** clusters
    2. how to **merge** clusters w/o violating their purity
        
        > multiple modalities는 <span class="blue">**unmergeable clusters**</span>에 대해 <span class="blue">**bridge**</span> 역할을 한다.
        > 

- <span class="shl">**Multi-Modal High-Precision Clustering (MuHPC)**</span>
    - **multiple modalities**를 사용한다. (face, voice, body appearance)
    - **first nearest neighbour clustering algorithms**를 기반으로 설계했다.
    - **cannot-link constraints**와 **video editing structure**도 이용한다.

- <span class="shl">**Video Person-Clustering Dataset (VPCD)**</span>
    - 기존 face-clustering dataset의 경우, skin color가 다양하지 않다.
    - 기존 face dataset 대비 **개선한 사항**은 다음과 같다.
        - person-level multi-modal annotations를 추가했다. (person-tracks, voice utterances)
        - 다른 TV show와 films를 추가했다. (→ program sets라고 명명한다.)
    - 모든 annotated characters에 대해 **포함하는 정보**는 다음과 같다.
        - body-tracks
        - face-tracks (when visible)
        - voice utterances (when speaking)


- 새로운 architecture를 제안하거나 새로운 network를 학습시키는 것이 아니다.
    - **face / speaker recognition**은 pre-trained networks에서 얻은 feature를 사용한다.
    - **body Re-ID**에 대해서만 network를 학습시킨다.

<br>

---

## 2. Comparison to Related Work

- **Face-Clustering**
    - [previous] **“face” clustering**에 국한되어 있다.
    - [our work] <span class="blue">**multi-modal person-clustering**</span>을 통해 face가 visible 한지와는 상관 없이 clustering이 가능하다.
- **Face-Labelling**
    - [previous] **label 정보나 external sources**가 필요했다.
    - [our work] label이 아닌 cluster 작업이므로, character-classifiers, ID supervision, 혹은 <span class="blue">**extra annotation이 필요하지 않다**</span>. 그 대신 <span class="blue">**editing structure**</span>나 <span class="blue">**multi-modality**</span> 등의 이용 가능한 모든 cue를 사용한다.
- **Person Re-ID**
    - [our work] video에서 track-level에서의 모든 charater를 cluster하므로 <span class="blue">**search queries가 필요하지 않다.**</span>
- **Related Datsets**
    - [preivous] **demographic groups**를 under-represent하며, **face annotation** 만을 포함한다.
    - [our work] 6개의 다른 TV show와 movie에 대해 <span class="blue">**다양한 characters**의 **multi-modal**</span> annotations를 포함한다.
- **Story Understanding**
    - [our work] scene 안에 누가 존재하는지에 대해 집중한다.

<br>

---

## 3. Multi-Modal High-Precision Clustering (MuHPC)

### 3-1. Features

- single hierarchical agglomerative clustering (HAC) approach이다.
    - <span class="shlb">similarities of modality features</span>와 <span class="shlb">constraints from the video structure</span>를 이용하여 person-tracks를 묶는다.
- <span class="shlp">pre-computed features를 사용하므로, optimal hyper-parameter를 학습시킬 때 외에는 training이 필요하지 않다.</span>
- 여기에서는 face, voice, body appearance 총 3개의 modalities를 사용하지만, 쉽게 더 추가할 수 있다.

<br>

### 3-2. Notation

- dataset은 **person-tracks** $$x_i$$와 **$$C$$개의 characters**로 이루어져 있다.
    
    > 모든 $$x_i$$를 identity로 묶어 $$C$$개의 cluster로 만드는 것이 목표! ($$C$$는 unknown)
    > 
- 각 **person-track $$x_i$$**는 multi-modality를 포함하는 하나의 feature vector로 나타낼 수 있다.
    
    $$
    x=\{x_f, x_v, x_b\}
    $$
    
    - 각각 face, voice, body에 관한 feature이다.
    - face/body가 visible 한지, 혹은 speaking 인지에 따라 종속적이다.
- 하나의 modality에 대해 두 track features 간의 **distance**는 $$d(x_i, x_j)$$라고 한다.
- NN은 nearest neighbor로, $$n_{x_i}^1$$은 **track $$x_i$$의 첫 번째 NN track**이다.
- $$x_i$$가 존재하는 **video frame의 집합**을 $$T_i$$라고 한다.
- 각 **cluster partition**은 $$\Gamma$$라고 한다.

<br>

### 3-3. Three stages

1. <span class="shl">**single modality(= face)**를 이용해 **high-precision clusters**를 $$K_1$$개 생성한다.</span>
    - HAC을 여러 번 iterate 하여 **first nearest neighbour(NN)**를 공유하는 person-tracks 끼리 묶는다.
    - 다음의 **두 additional constaints**에 종속적이다. (= 다음을 만족해야 NN이 valid 하다.)
        - <span class="blue">**spatio-temporal cannot-link constraint**</span> for concurrent tracks
            
            > 일시적으로 겹치는 track은 같은 group으로 묶이면 안된다. 한 사람은 하나의 frame에서 단 한 번 등장하기 때문이다.
            > 
        - <span class="blue">**NN distance constraint**</span> (= conservative threshold in the maximum NN distances)
            
            > <span class="shlp">$$d_f<\tau_f^{tight}$$</span>
            > 
            > 
            > ⇒ track과 첫번째 NN 간의 distance $$d_f(x_i, n_{x_i}^1)$$는 **$$\tau_f^{tight}$$ 보다 작아야 한다.**
            > 
    - 각 <span class="pink">**cluster parition** $$\Gamma$$</span>에서, 다음의 조건 중 하나를 만족하는 tracks를 merge하여 <span class="pink">**$$K_\Gamma$$ 개의 clusters**</span>를 생성한다. 이는 **adjancency matrix**로 나타낸다.
        1. 서로, 혹은 일방적으로 first NN인 경우
        2. 공통의 NN $$n_{x_i}^1$$을 가지는 경우
            
            ![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-02.png){: .w-75}
            
    - **stopping criteria**는 다음과 같다. (둘 중 하나 만족 시 stop)
        1. 모든 clusters 간의 distance가 $$\tau_f^{tight}$$ 보다 큰 경우
        2. 모든 clusters가 cannot-link constraint로 분리된 경우
    - <span class="pink">**$$K_1$$ 개의 high-precision clusters**</span>($$K_1 \ge C$$)가 생성된다.

    <br>

2. <span class="shl">**multi-modality(= face and voice)**를 이용해 single face modality로는 unmergeable 했던 cluster들을 합쳐준다.</span>
    - **bridge** 역할을 하는 것이다.
    - face와 voice distance에 대한 새로운 threshold를 지정한다.
        
        > <span class="shlp">$$d_f<\tau_f^{loose}$$</span> 이고 <span class="shlp">$$d_v<\tau_v^{loose}$$</span> 일 때 merge clusters!
        > 
    - <span class="pink">**$$K_2$$ 개의 high-precision clusters**</span>($$K_2 \le K_1$$)가 생성된다.
        
        ![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-03.png){: .w-75}
    
    <br>

3. <span class="shl">**visible faces가 없어 앞 단계에서 clustered 되지 못한 tracks**를 기존의 clusters와 연결해준다.</span>
    - **body-appearance**를 이용하여 **face-less person-tracks**를 앞 단계의 **high-precision clusters**에 이어준다.
    - 다음의 **두 constraints**를 사용한다.
        - <span class="blue">**editing structure**</span> (neighboring shots인지)
        - <span class="blue">**conservative threshold**</span> on **body features** (same person w/ same clothing)
    - **ratio-test**를 수행한다.
        1. 각 body-track에 대해 first & second NN distance를 구한다.
        2. <span class="shlp">$$d_{b,x_i}^2/d_{b, x_i}^1 > \rho$$</span> 인 body-track은 ignore 한다. ($$\rho$$ = threshold)
            
            > **back(뒷모습)**의 경우, cluster 되기 어려운 경우가 많다. 
            따라서 새로운 threshold를 설정해 <span class="shlp">$$d_b>\tau_b^{back}$$</span>인 것들은 ignore 하는 방법을 사용한다.
            > 
    - **Stage 3에서는 <span class="pink">cluster의 개수가 $$K_2$$</span>에서 변하지 않는다!**

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-04.png){: .w-75}
_Buffy_

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-05.png){: .w-75}
_Sherlock_

<br>

### 3-4. Number of Clusters

- <span class="pink">**최종 cluster 개수인 $$K_2$$**</span>를 $$C$$에 가깝게 만드는 것이 목표이다.
- **[Idea]** largest cluster 간에는 identity overlap이 잘 발생하지 않는다. 하지만 small cluster와 large cluster 간에는 overlap이 존재하기 쉽다.
    - large cluster는 해당 identity에 대해 충분한 정보를 담고 있다.
    - 만약 두 large clusters가 같은 identity를 포함한다면, merge 되었을 것이다.
    - 즉, 반복적으로 smallest와 largest cluster를 merge 하여 $$K_2$$가 $$C$$에 가까워지도록 하자!

<br>

### 3-5. Pre-trained

- [previous] video dataset에 대해 character features를 **fine-tune** 한다.
- [our work] **pre-trained features**를 사용하므로 다음의 장점이 있다.
    1. **reducing the <span class="blue">computational burden</span>**
    2. **leading to incread <span class="blue">generalisation</span> capabilities**

<br>

### 3-6. Hyper-parameters

> VPCD의 validation parition을 통해 학습한다. 여기에서는 constant 값으로 사용한다.
> 

- <span class="shl">**$$\tau_f^{loose}$$ (face modality와 관련)**</span>

    - face feature가 대용량의 dataset에서 이미 학습되어 있기 때문에 discriminative 하고 universal 하다.
    
        > 일반화 good!

- <span class="shl">**$$\tau_v^{loose}$$ (voice modality와 관련)**</span>
    - less universal 하므로, 하나의 값이 다른 program sets에 대해 잘 generalise 되지 않는다.

        > 따라서 각 program set에 대해 automatically 하게 학습한다.
        
    - $$\tau_v^{loose}$$ 가 다른 사람의 **voices 간의 minimum distance보다 작은 값**을 가지도록 한다.

        > <span class="blue">**cannot-link speaking face-tracks**</span>와 <span class="blue">**Stage1의 clusters**</span>를 사용하여 example을 더 만들어서 distances를 구하고, 이 분포를 이용하여 $$\tau_v^{loose}$$를 구한다. 2개의 face-tracks가 same shot에서 말하는 경우는 거의 없기 때문이다.
        >
        > 비슷한 목소리를 가진 character가 많은 program set의 경우, $$\tau_v^{loose}$$는 낮은 값을 가질 것이다.

<br>

---

## 4. Video Person-Clustering Dataset (VPCD)

### 4-1. Contents

- 다양한 TV show와 movie 속의 다양한 등장인물들에 대해서 full multi-modal annotations을 담고 있다.
- val. set과 test set을 포함한다.
- **voice-track의 특징**
    - **laughter**도 포함한다. (ex. TBBT/Friends의 live studio audience, character의 laughter)
    - 여러 개의 voice-track이 겹치는 경우나, 너무 짧은 경우는 ignore 한다.

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-06.png){: .w-75}

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-07.png)

<br>

### 4-2. Annotation Process

> automatic & manual annotation methods를 모두 사용한다.

- <span class="shlp">**Face**</span>
    - previous works에서 제공하고 있는 original datasets의 face bbox, track annotations, ID labels를 이용하므로 <span class="blue">automatic</span> 이다.

- <span class="shlp">**Body**</span>
    - **annotation 방법**
        1. MovieNet 데이터셋으로 train 된 Cascade R-CNN을 이용하여 body를 detect 한다.
        2. IoU tracker를 사용하여 tracks를 생성한다.
    - **두 가지 경우**
        - body-track이 face-track과 명확하게 대응되는 경우 (ex. 다른 face-track과 유의미한 IoU 값을 가지지 않는 경우)
            - 해당 face-track의 정보로 <span class="blue">automatically</span> annotated 된다.
            - Hungarian Algorithm을 사용한다.
        - 그 외의 경우 (ex. face-less)
            - <span class="blue">manually</span> annotate 한다.
            - 평균적으로 전체 body-track의 10~15% 정도에 해당했다.
            
- <span class="shlp">**Voice**</span>
    - 모든 character에 대해 speaking parts를 <span class="blue">manually</span> segment 한다.

<br>

### 4-3. Feature Extraction

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-08.png){: .w-50}

<br>

---

## 5. Experiments

### 5-1. Detail Settings

- feature distance $$d_f, d_b, d_v$$는 모두 `1 - cosine similarity`로 계산한다.
- VPCD val. set에서 학습한 hyperparameters는 다음과 같다.
    
    $$
    \tau_f^{tight}=0.48, \text{  }\delta=0.025, \text{  }\rho=0.9, \text{  }\tau_b^{back}=0.4
    $$
    
    ⇒ features가 바뀔 때에만 재학습하면 된다.
    

> <span class="shl">**[Appendix] 학습된 parameter는 different program sets에서 잘 generalise 한다.**</span>
> 
> - face features가 universal 하며, VPCD 내의 program sets가 visually disparate 하기 때문이다.
> - MuHPC는 VPCD에 없는 다양한 program sets에서도 잘 동작할 것이다!
{: .prompt-info}

<br>

### 5-2. Metrics

- <span class="shlp">**Weighted Cluster Purity (WCP)**</span>
    - 해당 cluster 내부의 track 개수를 통해 <span class="blue">**해당 cluster의 purity**</span>를 나타낸다.
    - 해당 cluster의 WCP = 1일 때, 모든 sample이 같은 class에서 왔다는 뜻이다.

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-09.png){: .w-25}

- <span class="shlp">**Normalized Mutual Information (NMI)**</span>
    - clustering quality와 cluster 개수 간의 <span class="blue">**trade-off**</span>를 측정한다.

- <span class="shlp">**Character Precision & Recall (CP, CR)**</span>
    - WCP, NMI와 다르게 ground truth identities의 개수를 사용한다.
    - **CP** - 각 character에 assign 된 cluster 내에서의 tracks의 비율을 나타낸다.
    - **CR** - 해당 character의 total tracks 중 cluster에 나타나는 track의 비율을 나타낸다.
    - 모든 character에 대해 평균으로 구한다.

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-10.png){: .w-50}
_Stage 1에서의 결과 → Stage 1에서는 face를 이용한다._

<br>

### 5-3. Test Protocol

- **Automatic Termination (AT)** - cluster 개수가 알려지지 않은 realistic 한 상황
- **Oracle Cluster (OC)** - cluster 개수가 알려진 상황 (SOTA와의 비교를 위해서)

<br>

### 5-4. Person-Clustering

> realistic 한 상황에서의 비교를 위해 AT protocol 하에서 실험하였다.
> 

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-11.png)

- MuHPC가 기존의 가장 성능이 좋은 face-clustering algorithms보다 **significantly outperform** 하였다.
- MuHPC에 voice, body를 추가하는 경우, 대부분의 program set에서 성능이 향상되었다.
- 하지만 **Hidden Figures**의 경우, body를 추가한다고 해서 성능이 향상되지 않았다.
    - 많은 **dark scenes**와 **non-distinctive clothing**이 나오기 때문이다.
    - 이런 경우에는 face-less bodies를 cluster 하기 위해서 temporal-NN 등의 방법을 사용해 볼 수 있다.

<br>

### 5-5. Face-Clustering

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-12.png){: .w-75}

- 오직 face-tracks만 사용하여 previous works와 비교하였다.
- AT와 OC protocol 모두에서 MuHPC가 기존의 face-clustering algorithms보다 **significantly outperform** 하였다.
- dataset의 난이도가 더 어려울수록, multi-modality의 효과가 더 크게 나타났다.

<br>

### 5-6. Enabling Story Understanding

- story understanding을 위해서는 누가 해당 scene에 존재하는지를 알아야 하는데, 이는 person-clustering으로 해결 된다.
- 이를 통해 **character co-occurrences**를 예측할 수 있다. 이는 곧 **character 간의 interaction** 정도라고 할 수 있다.

![](/assets/img/posts/Deep-Learning/Paper-Review/2022-04-26-13.png)
_TBBT의 character co-occurrences_

- **person-level GT**를 보면, Sheldon과 Leonard는 전체 frame의 24%에서, 연인 관계인 Penny와 Leonard는 전체 frame의 9%에서 함께 등장한다는 것을 알 수 있다.
- **face-level GT**의 경우, GT 임에도 불구하고 Penny와 Leonard의 관계가 3%로, 다른 관계와 다를 바 없이 나타난다.
- **MuHPC**의 결과, **person-level GT**와 매우 비슷하게 나온다.
    - 본 방법을 통해 co-occurrence, 즉 interaction을 자동으로 정확하게 찾을 수 있다.

<br>

---

## 6. Conclusion

1. 새로운 multi-modal person-clustering 방법인 **MuHPC**를 제안하였다.
2. 해당 분야에서 가장 크고 다양한 dataset인 **VPCD**를 제작하였다.
3. MuHPC와 VPCD를 통해 **큰 성능 향상**을 이끌어냈다.
4. **각 character의 appearance**와 **co-occurrence**를 자동으로 정확하게 예측할 수 있다.

<br>

---

## 7. Additional Materials
> 네이버 부스트캠프 AI Tech 최종 프로젝트에 적용하기 위해 찾아봤었던 여러 자료들

- **저자의 설명 영상**: [Face, Body, Voice: Video Person-Clustering with Multiple Modalities - Spotlight](https://youtu.be/ho58dTFO9kg)
    
- **ICCV 2021 open access**: [ICCV 2021 Open Access Repository](https://openaccess.thecvf.com/content/ICCV2021W/CVEU/html/Brown_Face_Body_Voice_Video_Person-Clustering_With_Multiple_Modalities_ICCVW_2021_paper.html)

- **Video Person-Clustering Dataset**: [VPCD](https://www.robots.ox.ac.uk/~vgg/data/Video_Person_Clustering/)

- **GitHub**: [Video_Person_Clustering](https://github.com/Andrew-Brown1/Video_Person_Clustering) (코드는 커밍쑨이라 한다...)

-   <details>
    <summary>VPCD README.txt</summary>
    <div markdown="1">

    ```
    This README details the contents of the VPCD dataset tar.gz

    For any questions, email Andrew Brown: abrown@robots.ox.ac.uk

    If you find this dataset useful, please consider citing: 

    @misc{Brown21c,
        author       = "Andrew Brown and Vicky Kalogeiton and Andrew Zisserman ",
        title        = "Face, Body, Voice: Video Person-Clustering with Multiple Modalities",
        year={2021},
        archivePrefix={arXiv},
        primaryClass={cs.CV}
    }

    VPCD consists of person-tracks, and voice-tracks. Each person-track consists of either a face-track, or a body-track, or both. A face-track consists of linked face-detections in time. A body-track consists of linked body-detections in time. The entire audio-track of each program set is manually diarised into individual voice-tracks, which are all identity labelled. 

    Assigning voice-tracks to person-tracks: Voice-tracks can be linked to person-tracks by looking at temporal intersection between voice-tracks and person-tracks that are labelled with the same identity. Most of the time, for each voice-track there will be a co-occuring person-track with the same-ground truth label. Occasionally a voice-track corresponds to someone who is off camera. In these cases there will be no co-occurring person-track. 

    VPCD.zip contains a pickle file for each of the program sets included in VPCD. Each pickle file stores a python dictionary data structure. Each key corresponds to each episode of the program set. Each value stores another dictionary with the following keys:

    1) ‘face’

    This is a python dictionary, where each key is a person-track identifier, and the values are the face-track coordinates of the face-track for this person-track. The face-track coordinates are a numpy array where each row details a face detection in the track and is structured as [frame number, x1, y1, x2, y2].

    2) ‘face_feats’

    This is a python dictionary, where each key is a person-track identifier, and the values are the 256D identity-discriminating features for the face-tracks, extracted from a SE-NET50 trained on VGGFace2 and MS1M (https://arxiv.org/abs/1710.08092). The features are stored in a numpy array where each row corresponds to the feature for that given face-detection in the face-track.

    3) ‘GT’

    This is a python dictionary, where each key is a person-track identifier, and the values are the ground-truth labels.

    4) 'body'

    This is a python dictionary, where each key is a person-track identifier, and the values are the body-track coordinates of the body-track for this person-track. The body-track coordinates are a numpy array where each row details a body detection in the track and is structured as [frame number, x1, y1, x2, y2].

    5) 'body_feats'

    This is a python dictionary, where each key is a person-track identifier, and the values are the 256D identity-discriminating features for the body-tracks, extracted from a ResNet50 trained on Cast-Search in Movies (https://arxiv.org/abs/1807.10510). The features are stored in a numpy array where each row corresponds to the feature for that given body-detection in the body-track.

    6) 'back'

    This is a python dictionary, where each key is a person-track identifier, and the values are the body-track coordinates of the body-track for this person-track. The person-tracks here are those where the face is not visible, hence there is no matching face-track, and hence the 'back' label. The body-track coordinates are a numpy array where each row details a body detection in the track and is structured as [frame number, x1, y1, x2, y2].

    7) 'back_feats'

    Same as 'body_feats', but for the person-tracks with no matching faces.

    8) 'back_GT'

    same as 'GT', but for the person-tracks with no matching faces.

    9) 'Voice'

    This is a python dictionary containing the annotations for the audio track diarisation. Each key is a ground truth label (the same as those in ‘GT’). Each value is a dictionary which details all of the voice-tracks in the episode that are labelled as being spoken by this ground truth label (identity), with the following keys:

        9i) 'starts' - This is a list, detailing the start times (in seconds) of each of the voice-tracks.
        9ii) 'ends' - This is a list, detailing the end times (in seconds) of each of the voice-tracks.
        9iii) 'frames' - This is a list, detailing the frames that (in seconds) each of the voice-tracks are spoken in.
        9iv) 'features' - The 512D identity-discriminating features extracted from a Thick-ResNet34 trained on VoxCeleb2 (https://arxiv.org/pdf/1806.05622.pdf).

    In some program sets, there is a key in the 'voice' dictionary labelled as “laughter”. This label is given to voice-tracks corresponding to laughter (either live studio audience, or a character’s laughter).
    ```

    </div>
    </details>
    