---
layout: post
title: SCIM in Ubuntu
tags: [ Ubuntu ]
---

Ubuntu(dapper) 리눅스에서 지금까지 사용하던 [나비(nabi)](http://nabi.kldp.net/) 입력기를 [SCIM](http://www.scim-im.org/)으로 대체하기로 했다.
 한국어 뿐 아니라 일본어, 중국어 입력이 필요해지기 시작했고, 현재 개발중인 제품의 기본 입력기로 장착하기 위해 미리 사용해 보기 위해서이기도 하다.

설치하는 순서하는 다음과 같다.

우선 먼저 SCIM 관련 패키지와 일본어 한자 입력을 위한 Canna 한자 서버 패키지를 설치한다.

    # sudo apt-get remove nabi
    # sudo apt-get install scim scim-gtk-immodule
    # sudo apt-get install scim-hangul scim-canna scim-pinyin
    # sudo apt-get install canna # for japanese dict server

X가 기동될때 자동으로 시작할 수 있도록 다음과 같은 작업을 해준다. (im-switch, update-alternatives 등의 데비안 도구를 사용할 수도 있지만 이 방법이 내겐 더 명확하다)

    # cd /etc/X11/xinit/xinput.d
    # sudo cat > scim
    XIM=SCIM
    XIM_PROGRAM=/usr/bin/scim
    XIM_ARGS="-d"
    GTK_IM_MODULE=scim
    DEPENDS="scim,scim-gtk2-immodule"
    [CTRL-D]
    # ln -sf scim ko_KR

이제 X를 재시작하거나, 시스템을 재시작하면 특별한 설정없이 자동으로 SCIM 입력기가 실행된다.

여기까지만 해도 문제는 없지만, 입력 전환 키가 [CTRL-SPACE]라서, 익숙한 [SHIFT-SPACE]를 사용하기 위해 SCIM 설정에서 키를 추가해주기만 하면 된다.
