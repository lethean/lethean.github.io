---
layout: post
title: 리눅스에서 이더넷 프레임 보내고 받기
tags: [ Linux, Network ]
---

다음에 진행할 프로젝트를 위해 [이더넷 프레임](http://en.wikipedia.org/wiki/Ethernet_frame) 패킷을 리눅스에서 소켓 API를 이용해 직접 읽고 쓰는 방법이 필요해서 조사한 결과를 남겨봅니다.

언제나처럼 구글의 도움을 받아 발견한 "[Receiving raw packets in Linux without pcap](https://austinmarton.wordpress.com/2012/06/03/receiving-raw-packets-in-linux-without-pcap/)" 글의 코드를 참고해서 샘플로 구현한 소스는 [여기](https://gist.github.com/lethean/5fb0f493a1968939f2f7)에서 확인할 수 있습니다.

윈본 소스와 다른 점은, 알려져 있는 [이더넷 타입](http://en.wikipedia.org/wiki/EtherType) 대신 자신만의 고유 타입을 사용하고, 브로드캐스트 주소도 인식하면서 간단한 메시지를 전송하거나 수신합니다.

컴파일 방법은 다음과 같습니다.

```sh
$ gcc -Wall -o ethcom ethcom.c
```

사용법은 `-l` 옵션을 주면 수신 모드로 동작하고, `-i` 옵션으로 네트워크 인터페이스 이름(기본 `eth0`)을 지정할 수 있고, `-d` 옵션으로 전송할 대상 주소(기본 `ff:ff:ff:ff:ff:ff`)를 지정할 수 있습니다. 예를 들어 다음과 같이 입력하면 'Hello, World' 문자열을 이더넷 네트워크에 브로드캐스트합니다.

```sh
$ sudo ./ethcom "Hello, World"
```

다른 장비에서 수신 모드로 실행하면 다음과 같이 문자열을 수신합니다.

```sh
$ sudo ./ethcom -l
60:a4:4c:xx:xx:xx -> ff:ff:ff:ff:ff:ff Hello, World
```

참고로, 실행할 때 반드시 루트(root) 권한이어야 합니다.

