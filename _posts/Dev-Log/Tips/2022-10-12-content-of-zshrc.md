---
title: "[Tips] 내가 사용하는 Shell 설정 모음 (.zshrc)"
date: 2022-10-12 19:50:00 +0900
categories: [Dev-Log, Tips]
tags: [shell, zsh, linux, tips]     # TAG names should always be lowercase
---

> 추후 업데이트 예정

<br>

내가 사용하는 쉘(`zsh`)의 설정 파일인 `~/.zshrc`의 내용을 기록하려 한다.

<br>

### `ll` 명령어 등록
```shell
# ll command
alias ll='ls -alvh'
```

### Starting Directory 지정
```shell
# starting directory
export START="/starting/directory"
if [[ $PWD == $HOME ]]; then
    cd $START
fi
```

### Terminal에서 User Name 숨기기
```shell
# to hide user name at terminal
DEFAULT_USER="user-name"
```

### Terminal에서 MacBook Name 숨기기
{% raw %}
```shell
# to hide macbook name at terminal
prompt_context() {
        if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
        prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
    fi
}
```
{% endraw %}
