---
layout: post
title: "구글 프로토콜 버퍼(Protocol Buffers)"
tags: [ Coding,  Eclipse,  Network,  Wireshark ]
---

구글 [프로토콜 버퍼](http://code.google.com/p/protobuf/) 라이브러리에 대해서는 몇 번 [언급](/2010/01/29/0mq-a-new-approach-to-messaging/)을 한 적이 있는데, 관심만 있고 아직 프로젝트에 사용은 못해보고 있었습니다. 그 이유는 프로젝트 대부분이 C 언어를 사용하는데, **공식적으로** 지원하는 언어에는 없었기 때문입니다.

그런데 오늘 우분투 패키지를 업데이트하다가 [protobuf-c](http://code.google.com/p/protobuf-c/) [우분투 패키지](http://packages.ubuntu.com/lucid/protobuf-c-compiler)가 있어서 확인해보니, 프로토콜 버퍼 데이터를 읽고 쓸 수 있는 런타임 라이브러리와 \*.[hc] 파일을 생성하는 컴파일러가 들어있군요. 그래서 아예 구글 코드 사이트에 있는 [protobuf 관련 프로젝트 목록](http://code.google.com/hosting/search?q=label:protobuf)을 훑어보니, C 언어 뿐 아니라 다양한 언어에 대한 바인딩을 지원할 뿐 아니라 이클립스 플러그인, 와이어샤크(wireshark) 플러그인 등처럼 유용한 도구도 많이 찾을 수 있었습니다.

이런 경우 언제나 느끼는 거지만, 요즘은 노하우(know-how)보다 노훼어(know-where)가 더 필요한 세상인 것 같습니다.
