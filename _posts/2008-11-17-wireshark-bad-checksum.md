---
layout: post
title: Wireshark Bad Checksum
tags: [ Linux,  Wireshark ]
---

네트웍 패킷을 검사하기 위해 자주 사용하는 [Wireshark](http://www.wireshark.org/) 프로그램을 리눅스에서 실행할때 이더넷 장치에 따라 'Bad Checksum'을 표시하며 패킷이 붉은 색으로 표시되어 눈에 거슬리는 경우가 종종 있습니다.

![](/figures/wireshark-bad-checksum.png)

이는 패킷 전송시 이더넷 하드웨어가 패킷 체크섬을 계산하도록 하는 기능이 자동으로 켜져있는데, 이 부분에 오류가 있는 경우입니다. 대부분의 경우 문제를 야기하지 않지만, 때로는 IP가 제대로 할당되었는데 실제 동작을 안하는 원인이 되기도 합니다. 따라서 이 기능을 끄고 싶을 경우가 발생하는데, 다음과 같은 명령어만 실행하면 됩니다.

    $ sudo ethtool -K eth0 tx off

여기서 'eth0'는 장치 이름입니다. 더 궁금하시면 'man ethtool'을 통해 확인하시기 바랍니다.
