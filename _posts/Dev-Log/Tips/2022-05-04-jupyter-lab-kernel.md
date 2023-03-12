---
title: "[Tips] Jupyter Lab에 가상 환경 추가하기"
date: 2022-05-04 22:50:00 +0900
categories: [Dev-Log, Tips]
tags: [jupyter lab, tips]     # TAG names should always be lowercase
math: true
mermaid: true
---

<br>

부스트캠프 P Stage의 semantic segmentation 대회 기간 동안 공식 `mmsegmentation`에서 제공하지 않는 `SeMask-FPN`을 수정하여 학습시켜 보았다.

이 과정에서 `jupyter lab`의 커널에 내가 만든 `conda` 가상 환경을 추가하는 방법을 기록하려 한다.


<br>

우선, 추가할 가상 환경을 `activate` 한다.

```bash
conda activate [virtual-env-name]
```

<br>

`ipykernel` 라이브러리가 설치되어 있지 않다면 설치한다.

```bash
pip install ipykernel
```

<br>

다음의 명령어를 통해 새로운 `kernel`로 가상 환경을 추가한다.

```bash
python -m ipykernel install --user --display-name [display-kernel-name] --name [virtual-env-name] 
# 같은 이름을 주면 편리하다.
```

<br>

`jupyter lab`에서 다음과 같이 `kernel`을 선택 및 변경할 수 있다.

![](/assets/img/posts/Dev-Log/Tips/2022-05-04-jupyter-lab-kernel.png){: width="50%" height="50%"}
_`open_mmlab` kernel 추가 예시_

<br>

`jupyter lab`의 `kernel`을 삭제하려면 다음의 명령어를 이용한다.

```bash
jupyter kernelspec uninstall [virtual-env-name] 
```
