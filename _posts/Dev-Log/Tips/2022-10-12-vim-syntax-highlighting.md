---
title: "[Tips] Vim 에디터에 Syntax Highlighting 적용하기"
date: 2022-10-12 20:50:00 +0900
categories: [Dev-Log, Tips]
tags: [shell, vim, tips]     # TAG names should always be lowercase
---

`vim` 에디터 내부 텍스트 파일에 syntax highlighting이 적용되지 않는 문제점을 해결했다.

![](/assets/img/posts/Dev-Log/Tips/2022-10-12-vim-1.png)


다음의 명령어를 실행한 후 터미널을 재시작 한다.

```shell
echo $MYVIMRC 
# $MYVIMRC가 있다면 해당 파일 열고, 없다면 .vimrc 파일 생성
vim ~/.vimrc 

# >>> 다음을 작성 후 저장 >>>
syntax on
# <<<<<<<<<<<<<<<<<<<<<<
```


![](/assets/img/posts/Dev-Log/Tips/2022-10-12-vim-2.png)


> 참고: [https://stackoverflow.com/questions/37081223/why-colors-are-not-displaying-in-vim-oh-my-zsh](https://stackoverflow.com/questions/37081223/why-colors-are-not-displaying-in-vim-oh-my-zsh)

