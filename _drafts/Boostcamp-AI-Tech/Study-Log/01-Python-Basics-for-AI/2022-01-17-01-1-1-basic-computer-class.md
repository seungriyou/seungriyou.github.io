---
title: "[Python] 1-1. Basic computer class for newbies"
date: 2022-01-17 18:50:00 +0900
categories: [부스트캠프 AI Tech 3기, 01 - Python Basics for AI]
tags: [부스트캠프AITech, study-log, week01, python]     # TAG names should always be lowercase
math: true
mermaid: true
image: 
  src: /assets/img/posts/Boostcamp-AI-Tech/boostcamp_ai_tech_title.png
---
> 본문은 네이버 부스트캠프 AI Tech 3기의 강의와 자료를 바탕으로 작성된 글입니다.

# **1. 컴퓨터 OS**
- Operating System, 운영체제
- 우리의 프로그램이 동작할 수 있는 구동 환경
- 응용 프로그램은 운영체제에 dependent 하다.

> 어떤 개발 환경에서 개발을 실행할 것인가에 대한 선택!

<br>

-------

# **2. 파일 시스템**
## **Overview**
- **file system** - OS에서 파일을 저장하는 트리구조 저장 체계
- **file** - 컴퓨터 등의 기기에서 의미 있는 정보를 담는 논리적인 단위
  ⇒ 모든 프로그램은 파일로 구성되어 있고, 파일을 사용한다.

<br>

## **파일과 디렉토리**
- **디렉토리 (directory)**
    - 폴더 또는 디렉토리로 불림
    - 파일과 다른 디렉토리를 포함할 수 있음
- **파일 (file)**
    - 컴퓨터에서 정보를 저장하는 논리적인 단위
    - 파일은 파일명과 확장자로 식별됨 (ex. `hello.py`)
    - 실행, 쓰기, 읽기 등을 할 수 있음
- 파일 시스템 root 디렉토리로부터 시작하는 트리구조로 되어 있음 (`Windows key + E`)
    
<br>

## **절대 경로와 상대 경로**
- **경로** - 컴퓨터 파일의 고유한 위치, 트리 구조 상 노드의 연결
- **절대 경로** - 루트 디렉토리부터 파일 위치까지의 경로
- **상대 경로** - 현재 있는 디렉토리부터 타깃 파일까지의 경로
    - `..` - 부모 디렉토리
    - `.` - 현재 디렉토리

<br>

-------
# **3. 터미널**
## **Overview**
- **터미널** - 마우스가 아닌 키보드로 명령을 입력하여 프로그램을 실행
- GUI (Graphical User Interface)
- CLI (Command Line Interface)

<br>

## **Command Line Interface**
- text를 사용하여 컴퓨터에 명령을 입력하는 인터페이스 체계
- **Windows** - CMD window, Windows Terminal  
    ⇒ cmder도 권장 (Linux 명령어 사용 가능) 
    > **cmder 관련 참고 자료**  
    > [윈도우즈 콘솔 에뮬레이터 cmder](https://webdir.tistory.com/548)
    {: .prompt-note }
- **Mac, Linux** - Terminal
- Console = Terminal = CMD 창
    - 어원) 디스플레이와 키보드가 조합된 장치
    - 현재) CLI로 입력하는 화면

<br>

## **Terminal 시작하기**
- **Windows** - 윈도우 키 + terminal 또는 윈도우 키 + R → CMD 입력
    > Ubuntu도 설치해보는 것을 권장한다.
    {: .prompt-note }
- **Mac** - 빠른 실행 terminal 입력

<br>

## **기본 명령어**
- 각 터미널에서는 프로그램을 작동하는 shell이 존재
- shell마다 다른 명령어를 사용 (ex. PowerShell, bash shell, etc.)
    - `copy ..\..\abc.txt .\` - abc.txt를 현재 디렉토리로 복사한다.
  ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/235c2ce7-1121-4df8-8ae0-601090efcc32/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220206%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220206T055324Z&X-Amz-Expires=86400&X-Amz-Signature=9e3e21582ffafaf598a03496619b0171dcdb82aa0a94ff6a2b460e50c2a46eba&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22&x-id=GetObject)

> Windows에서는 디렉토리를 구분할 때 `/`는 불가능, `\`만 가능하다.
{: .prompt-warning }
