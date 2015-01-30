---
layout: post
title: VMware /dev/parport0 in Ubuntu
tags: [ Ubuntu,  VMware ]
---

Ubuntu 리눅스에서 VMware를 돌리고, 그 안에서 JTAG 케이블을 이용해 부트로더를 올리는 작업을 하기 위해서는 무엇보다 VMware가 `/dev/parport0` 를 찾을 수 있도록 해주어야 한다. 매번 반복되면서 기록을 남기지 않아 오늘도 다시 고생... 여기에 그 방법을 남겨 놓는다.

먼저 ppdev 커널 모듈을 올린다.

    modprobe ppdev

아예 `/etc/modules` 에 등록시켜 두는 것이 좋다.
 일반 계정으로 접근이 가능하도록 `/etc/group` 에서 lp 부분을 다음과 같이 수정한다. (여기 lethean이 일반 계정이다)

    lp:x:7:cupsys,lethean

이게 전부다.
