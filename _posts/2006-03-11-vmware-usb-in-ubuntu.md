---
layout: post
title: VMWare USB in Ubuntu
tags: [ Ubuntu,  VMware ]
---

우분투 리눅스는 항상 최신 아키텍쳐를 반영하기 때문에 기존 API나 아키텍쳐를 기반으로 동작하는 써드파티 어플리케이션이 동작 안하는 경우가 가끔 있는데, VMWare도 그 중 하나인 것 같다.

언젠가부터 USB 장치를 VMWare 윈도우즈에서 인식을 못한다고 생각했는데, 오늘 검색해 보니 VMWare가 근거로 하는 `/proc/bus/usb` 디렉토리 내용이 없어서였다. 다음 줄을 `/etc/fstab` 파일에 추가해 주고 재부팅하거나, `mount -a` 명령으로 마운트해준 다음 VMWare를 재시작하면 정상적으로 USB 장치를 인식하게 된다.

    none        /proc/bus/usb    usbfs  defaults      0    0
