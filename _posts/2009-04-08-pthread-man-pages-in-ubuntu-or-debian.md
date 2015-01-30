---
layout: post
title: "우분투에서 C 라이브러리 맨페이지(manpage) 설치하기"
tags: [ Coding,  glibc ]
---

리눅스에서 개발할때 'man' 명령을 이용해 매뉴얼 페이지를 많이 참고하는데, 자주 시스템을 다시 설치하다 보니 설치되지 않은 매뉴얼 때문에 매번 구글을 찾는라 귀찮은 적이 많아 적어둡니다. 우분투나 데비안에서만 유효합니다.

기본적인 C 라이브러리 관련 매뉴얼 패이지는 \`manpages-dev' 패키지를 설치하면 되고, pthread\_\*()  류의 함수는 \`glibc-doc' 패키지를 설치하면 됩니다. 즉, 터미널에서 다음과 같이 입력하면 됩니다.

    sudo apt-get install manpages-dev glibc-doc
