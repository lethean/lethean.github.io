---
layout: post
title: 'VMWare rtc: lost some interrupts'
tags: [ Linux,  VMware ]
---

64비트 장비에 우분투 서버를 설치하고 VMware를 가동하면 다음과 같은 에러가 발생한다.

    [16825.988196] printk: 246 messages suppressed.
    [16825.988201] rtc: lost some interrupts at 2048Hz.

구글링을 통해 알게된 [사이트](http://chxo.com/be2/20060821_3333.html)에서 `/etc/vmware/config` 파일에 `host.useFastClock = FALSE` 항목을 추가하면 된다는 걸 알았는데, 문제는 시각 동기화가 잘 안되어 VMware로 실행하는 이미지 내부에서 ntp 데몬 등을 이용하여 시각동기화를 해주어야 한다.
