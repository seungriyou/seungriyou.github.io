---
title: "[Python] 1-3. 파이썬 코딩 환경"
date: 2022-01-17 18:50:00 +0900
categories: [부스트캠프 AI Tech 3기, 01 - Python Basics for AI]
tags: [부스트캠프AITech, study-log, week01, python, jupyter, colab]     # TAG names should always be lowercase
math: true
mermaid: true
image: 
  src: /assets/img/posts/Boostcamp-AI-Tech/boostcamp_ai_tech_title.png
---
> 본문은 네이버 부스트캠프 AI Tech 3기의 강의와 자료를 바탕으로 작성된 글입니다.

## **for Windows**
1. “python miniconda” 검색하여 python 3.8 버전 다운로드
   
    [Miniconda - Conda documentation](https://docs.conda.io/en/latest/miniconda.html)

    - just me에 체크
    - cmder를 사용하는 경우, PATH 설정에도 체크
2. 앞으로 miniconda에 접속하여 파이썬 개발하는 것을 권장한다.
    -  cmder를 사용하더라도 `conda activate base` 명령어를 실행한 후, 앞에 `(base)`가 붙어있는지 확인하고 `python` 명령어를 통해 개발한다.
3. vscode 설치
4. miniconda나 cmder를 사용하여 디렉토리 이동 후 `code .`를 실행하면 vscode가 실행된다.

<br>

## **for Mac**
(영상 참고)

<br>

---

## **Jupyter & Colab**
> cmder를 사용하는 사람들은 무조건 `conda activate base` 실행 후 시작하기!
{: .prompt-warning}

### **Jupyter 개요**
> Python Shell + 코드 편집 도구

- IPython 커널을 기반으로 한 대화형 파이썬 셸
- 일반적인 터미널 셸 + 웹 기반 데이터 분석 Notebook 제공
- 미디어, 텍스트, 코드, 수식 등을 하나의 문서로 표현 가능
- 사실상의 데이터 분석 Interactive Shell의 표준

<br>

### **Jupyter 실행**
- jupyter 설치 - `conda install jupyter`
- jupyter 실행 - `jupyter notebook` ⇒ 실행 폴더 기준으로
- cell 단위로 실행 ⇒ 실행 시점에 해당 코드가 memory에 올라감
- 실행 명령어
    - `ctrl` + `enter` - 해당 셀 코드 실행
    - `alt` + `enter` - 셀 추가
    - `shift` + `tab` - 툴 팁 보기
    - `ctrl` + `[` / `]` - 들여쓰기
    - `ctrl` + `shift` + `-` - 셀 나누기
    - `shift` + `M` - 아래 셀이랑 합치기
    - `esc` - edit mode → command mode
    - `dd` - (command mode) 셀 지우기
    - `x` - 셀 오려두기
    - `c` - 셀 복사하기
    - `v` / `shift` + `v` - 셀 붙여넣기
    - `d`, `d` - 셀 지우기
    - `z` - 셀 지우기 취소
    - `m`, `m` - markdown 변환
    - `y`, `y` - code로 변환

<br>

### **Colab**
- 구글이 개발한 클라우드 기반의 jupyter notebook
- 구글 드라이브 + GCP + jupyter 등이 합쳐져서 사용자가 손쉽게 접근
- 초반 여러가지 모듈 설치의 장점을 가짐
- 구글 드라이브의 파일을 업로드하여 사용가능한 장점 가짐
- vscode등과 연결해서 사용 가능
- V100 이상의 GPU를 무료로 쓸 수 있다는 장점을 가짐
- colab pro를 사용할 경우 안정적인 colab의 활용이 가능해짐
- colab과 jupyter는 비슷한 듯 다른 단축키를 가짐
    
    ![Untitled](/assets/img/posts/Boostcamp-AI-Tech/Study-Log/01-Python-Basics-for-AI/1-3-01.png){: width="70%" height="70%"}