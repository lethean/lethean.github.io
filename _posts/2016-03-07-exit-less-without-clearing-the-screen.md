---
layout: post
title: less 명령어가 끝나도 내용을 지우지 않게 하기
tags: [ Git, Less, Zsh ]
---

`git diff`, `git log` 같은 명령어는 내부적으로 다시 `less` 프로그램을
사용합니다. 그런데 언젠가부터 갑자기 명령어가 끝나면 내용이 지워지면서 원래
화면이 표시되어 여간 불편했는데, 그 이유를 조사해 보니 Zsh 셸을 사용하면서
설치한 [Oh My Zsh][oh-my-zsh] 때문이었습니다.

`~/.oh-my-zsh/lib/misc.zsh` 파일에 다음과 같이 되어 있는데,

```sh
export PAGER="less"
export LESS="-R"
```

아래와 같이 수정하면 됩니다.

```sh
export PAGER="less"
export LESS="-R -F -X"
```

구글링으로 참조한 링크는 아래와 같습니다.

* ['less' command clearing screen upon exit - how to switch it off?](http://superuser.com/questions/106637/less-command-clearing-screen-upon-exit-how-to-switch-it-off)
* [Is there any way to exit “less” without clearing the screen?](http://unix.stackexchange.com/questions/38634/is-there-any-way-to-exit-less-without-clearing-the-screen)

:)

[oh-my-zsh]: https://github.com/robbyrussell/oh-my-zsh

