---
title: "[CS231n] 02. Image Classification Pipeline"
date: 2021-08-02 19:50:00 +0900
categories: [Deep Learning, CS231n]
tags: [cs231n, deep-learning]     # TAG names should always be lowercase
math: true
mermaid: true
---
> 본문은 Stanford의 CS231n 강의와 자료를 바탕으로 작성된 글입니다.


# **Image Classification**
- Computer Vision의 core task
- **문제** - semantic gap (image는 컴퓨터 입장에서 `[0, 255]` 사이의 수를 가진 행렬일 뿐)
- **어려움**
    - viewpoint variation (카메라가 움직일 때 모든 pixel이 움직임)
    - illumination (조명의 방향, 세기)
    - deformation
    - occlusion (가려짐)
    - background clutter (배경과 비슷함)
    - intraclass variation (클래스 내의 다양성)

<br>

-------

# **Data-Driven Approach**
> **순서**
>    1. collect a dataset of images and labels
>    2. use machine learning to train a classifier
>    3. evaluate the classifier on new images

## **(1) Nearest Neighbor**

- NN algorithm = "the closest sample"
- **distance metric** to compare images
    - L1 distance (= Manhattan Distance)
        
        $$
        d_1(I_1, I_2) = \sum_p |I_1^p - I_2^p|
        $$
        
        ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7731ccee-1cdd-4dee-adf5-59485f67fd80/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T084421Z&X-Amz-Expires=86400&X-Amz-Signature=457b0ba2a363576ce41a1a0c01abea0218decea293a9a64e991c95819ca41c10&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject){: width="80%" height="80%"}
        
    
- Nearest Neighbor classfier의 구조
    - memorize all training data and labels (학습 데이터 학습만 함)
        
        ```python
      def train(self, X, y):
        self.Xtr = X
        self.ytr = y
        ```
        
    - predict the label of the most similar training image
        
        ```python
      def predict(self, X):
        num_test = X.shape[0]
        Ypred = np.zeros(num_test, dtype = self.ytr.dtype)
      
        # find the nearest training image to the i'th test image
        for i in xrange(num_test):
          # using the L1 distance (sum of absolute value differences)
          distances = np.sum(np.abs(self.Xtr - X[i,:]), axis = 1)
          # get the index with smallest distance (find closest train image)
          min_index = np.argmin(distances)
          # predict the label of the nearest example (image)
          Ypred[i] = self.ytr[min_index]
        
        return Ypred
        ```
        > **python에서의 `X[i, :]`**
        > <details>
        > <summary>assigning a vector to a slice of numpy 2D array (slice assignment)</summary>
        > <div markdown="1">
        > 
        > ```python
        >>> import numpy
        >>> a = numpy.array([[0,0,0],[1,1,1]])
        >>> a[0,:] = [3,4,5]
        >>> a
        > array([[3, 4, 5],
        >         [1, 1, 1]])
        > ```
        > 
        > </div>
        > </details>
        {: .prompt-note}
            
- with N examples, **train is O(1)** (data 기억만), **predict is O(N)** (N개의 학습 데이터 전부를 테스트 이미지와 비교)
    - **BAD** - 우리는 prediction이 fast한 것을 원함. training이 slow한 것은 ok
    - parametric models(ex. CNN)는 NN과 반대

<br>

## **(2) K-Nearest Neighbors**

- nearest neighbor의 copying label 대신, **take majority vote** from closest points (NN의 문제점 보완)
- **distance metric**
    - **L1(Manhattan) distance** - 어떤 좌표 시스템이냐에 따라 영향 받음 (좌표계 회전시키면 변함)  
        ⇒ vector의 요소들이 각각 개별적인 의미를 가지고 있다면
        
        $$
        d_1(I_1, I_2) = \sum_p |I_1^p - I_2^p|
        $$
        
    - **L2(Euclidean) distance** - 좌표계와 연관 X (일반적인 vector)  
        ⇒ decision boundary 더 자연스러움
        
        $$
        d_2(I_1, I_2) = \sqrt {\sum_p |I_1^p - I_2^p|^2}
        $$
        
- setting **hyperparameters**
    > what is the best value of **k** to use? what is the best **distance metric** to use?  
    > ⇒ problem-dependent (모두 시도해보고 제일 좋은 것 고르기)
    {: .prompt-note}

    ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/37ac4a08-f605-49ef-88ca-575438a00f04/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T092206Z&X-Amz-Expires=86400&X-Amz-Signature=fd36b49f0dc4f535785dd622403d19051e0e33bc3718738085b213a0907fc8cb&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject){: width="80%" height="80%"}
    
    ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7890898b-8ec0-45d5-8f0e-db4bf1c22b7c/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T092212Z&X-Amz-Expires=86400&X-Amz-Signature=3021bda82c5b57e75d1c024b8b07da8f55f59e66ed4757388a72857cb9e0a1c8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject){: width="75%" height="75%"}
    
    - **idea #1** - data에서 work best인 hyperparameter 선택
        - **BAD** - training data에만 perfectly work하는 k는 X
    - **idea #2** - data를 train과 test로 나누고, test data에서 work best인 hyperparameter 선택
        - **BAD** - new data에서 algorithm이 어떻게 수행할지 모름
        - ML의 궁극적인 목적은 한 번도 보지 못한 데이터에서 잘 동작하는 것
    - **idea #3** - data를 train, val, test로 나누고, **val**에서 hyperparameter 선택, **test**에서 evaluate
        - **BETTER**
            1. `train` - 다양한 hyperparameter로 학습
            2. `val` - 검증, 가장 좋았던 hyperparameter 선택
            3. `test` - validation set에서 가장 좋았던 classifier 가지고 '한 번만' 수행
    - **idea #4 : cross-validation** - data를 **folds**로 나누고, 각 fold를 번갈아가며 validation으로 하여 평균
        - small datasets에서 유용하지만, DL에서는 자주 쓰이지 X (계산량 많으므로)
        - `training set` - label 기억하고 있는 img (→ 알고리즘은 이 set 자체를 기억)
        - `validation set` - label을 볼 수 X, 알고리즘이 얼마나 잘 동작하는지 확인할 때만 사용
        - `test set` - data를 지속적으로 모으지 X, 일괄적으로 모으고 무작위로 나누어 한 번도 보지 못한 데이터를 대표함
    
- k-NN은 images에 사용되지 X
    - test time에 매우 느림
    - distance metrics(L1, L2) on pixels가 informative하지 X
        - img 간의 지각적 유사도 측정에 적합 X
    - curse of dimensionality
        - training data 이용하여 공간 분할 ⇒ 전체 공간을 조밀하게 커버할 만큼 충분한 training sample 필요 ⇒ 차원이 증가함에 따라 기하급수적 증가

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/708f6bd7-8530-48e8-a23b-90f5c999971a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T092218Z&X-Amz-Expires=86400&X-Amz-Signature=315ef684dd75e47142beb1943b7ac5d5e85fdb12a91e00f0ed3b429e491f63f6&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject){: width="80%" height="80%"}

<br>

---
# **Linear Classification**
> **Neural Network의 구성요소 = Linear classifiers**  
> 이미지 인식(CNN) + 언어 인식(RNN) ⇒ 레고 블럭(linear classifier)처럼 붙히고 학습

## **Parametric Approach**
- linear classifier = parametric model의 가장 단순한 형태
  > - **K-NN**은 parameter가 아님. 모든 training set을 test time에 이용함.
  > - **parametric approach**에서는 training data의 정보를 요약 → W에 모음 → test time에 더이상 training data가 필요하지 X → W만 있으면 되므로, 작은 device에서 모델 동작시킬 때 efficient
  {: .prompt-note}


![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/495e315c-484e-4fc1-ab5b-805b58a5edca/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T093222Z&X-Amz-Expires=86400&X-Amz-Signature=abb3c65fb792d054b152d87e8eba1fc67da04cc40814fbe1759dc85a25f35b9f&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject){: width="70%" height="70%"}

- $f(x,W) = Wx$는 **가중치 W**와 **데이터 X**를 조합하는 가장 쉬운 방법 (inner product)
    - **가중치 W** - 각 row는 각 class의 img에 대한 템플릿. img의 모든 pixel에 대응하는 entry(⇒ 이 pixel이 class 결정하는 데 얼마나 중요한 역할을 하는지에 관련된 가중치)
    - **데이터 X** - input img를 column vector로 변환
- **bias**는 data와 무관하게 특정 class에 우선권 부여 (scaling offset)  
    - **"data-independent preferences"** (ex. data set의 불균형)
    - data와 독립적으로 각 카테고리에 연결
      ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/951458bc-8a65-4604-9477-919f50a83600/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T093246Z&X-Amz-Expires=86400&X-Amz-Signature=8b96d6cf3f1bf8444db7cd6ec5989b2eb17cdd050927c7e31686c0640b8bf22d&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject){: width="80%" height="80%"}
    
- **linear score function ⇒** $f(x, W) = Wx + b$은 **template matching**
    - 하지만 linear classifier는 각 class(category)에 대하여 **단 하나의 template만**을 학습 → 다양한 특징들이 있을 수 있지만 평균화 시킴

<br>

## **Interpreting a Linear Classifier**

- linear decision boundary
- single linear line으로 분류되지 않는 경우가 존재함

> **linear classification**  
> = learning **"templates"** per class   
> = learning **"linear decision boundaries"** between pixels & high-dimensional space  
> (dimension = values of pixels)
{: .prompt-note}