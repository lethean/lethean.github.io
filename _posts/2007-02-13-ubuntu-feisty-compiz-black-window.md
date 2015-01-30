---
layout: post
title: Ubuntu Feisty Compiz 사용시 윈도우가 빈 상태로 나타나는 현상
tags: [ Xorg ]
---

Ubuntu Feisty에서 공식 compiz 패키지와 함께 gnome-compiz-manager를 사용하는데 가끔씩 윈도우가 빈 상태로 나타나는 현상이 발생했다. 곧 고쳐지겠거니 하고 기다리다 지쳐 검색해보니, 역시 [해결방법](https://bugs.launchpad.net/ubuntu/+source/compiz/+bug/82999)이 있었다.

다음과 같은 옵션 항목을 X서버 설정 파일에(`/etc/X11/xorg.conf`) 넣어주면 된다.

    Section "Device"
          Identifier      "Intel 82852/855GM"
          Driver          "i810"
          BusID           "PCI:0:2:0"
          Option          "XAANoOffscreenPixmaps" "true"
    EndSection

물론 약간의 성능 손실은 있겠지만, 일단 정상적으로 동작한다.

;)
