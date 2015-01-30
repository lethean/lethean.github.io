---
layout: post
title: "우분투에서 ccache 사용하기"
tags: [ GCC,  Ubuntu ]
---

[ccache](http://www.google.co.kr/search?hl=ko&newwindow=1&q=ccache&btnG=%EA%B2%80%EC%83%89&lr=lang_ko&aq=f&oq=)는 워낙 유명하고 오래된 개발 도구라서, 검색해보면 우리말로 된 글도 많이 찾을 수 있으므로 자세한 설명은 생략하고, 최근에 다른 개발 장비를 구축할 일이 있어서 검토해보다가, 페도라는 이미 기본으로 ccache를 사용하도록 되어 있고, 우분투 / 데비안 계열에서도 매우 쉽게 사용할 수 있도록 되어 있는 걸 알게 되었습니다. 그래서 다시 한 번 정리합니다.

먼저 패키지를 설치합니다.

    $ sudo apt-get install ccache

`/etc/environment` 파일을 열어 `PATH` 변수 맨 앞에 `'/usr/lib/ccache'` 를 추가합니다. 예를 들어 아래와 같다면

    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"

다음과 같이 변경합니다.

    PATH="/usr/lib/ccache:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"

다시 로그인하면 모든 gcc, g++ 명령은 자동으로 ccache를 이용하게 됩니다.

물론 본인 계정에서만 사용하고 싶다면, 홈디렉토리 밑의 `.bashrc` 또는 `.bash_profile`, `.profile` 파일에 다음과 같은 내용을 추가하면 됩니다.

    export PATH="/usr/lib/ccache:$PATH"

GCC가 느리다고 불평하는 일이 조금 줄어들까요?
