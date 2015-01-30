---
layout: post
title: Ubuntu 7.04 (feisty) 느린 SSH 접속
tags: [ Linux,  Ubuntu ]
---

Ubuntu 7.04에서 ssh 접속 등을 시도할때 다른 시스템보다 초기 접속이 느린 이유가 [avahi-daemon 관련 설정 때문](https://bugs.launchpad.net/ubuntu/+source/avahi/+bug/94940)이라고 한다. 그래서 /etc/nsswitch.conf 파일에서 'hosts:' 부분을 다음과 같이 수정해보았더니, 역시 빨리 접속된다.

    hosts:          files dns

정식 릴리스에 반영되기에는 시간이 촉박한 것 같다...
