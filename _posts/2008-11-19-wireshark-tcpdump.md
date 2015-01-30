---
layout: post
title: Wireshark & tcpdump
tags: [ Linux,  Wireshark ]
---

[Wireshark](http://www.wireshark.org/) 프로그램을 이용할때, 문제의 현상이 발생하는 패턴을 모르는 경우 무작정 발생할때까지 패킷을 캡쳐해야 하는 경우가 있습니다. 이때 Wireshark 프로그램으로 무조건 캡쳐를 하면 금방 메모리가 부족해서 프로그램이 죽어버리게 됩니다. 이런 경우 tcpdump 프로그램을 이용하여 패킷을 캡쳐하여 파일에 저장하고, 현상이 발생했을때 멈추고 난뒤 캡쳐한 파일을 다시 Wireshark 프로그램에서 볼 수 있는 방법이 있습니다.

먼저 tcpdump를 설치합니다.

    $ sudo apt-get install tcpdump

그리고 다음과 같이 패킷 캡쳐를 시작합니다. (물론 모두 한 줄에 적어도 됩니다)

    $ sudo tcpdump 
      -i eth0 
      -s 1500 
      -C 5 
      -W 3 
      -w capture.pcap 
      'host 192.168.0.100'

여기서 '-i' 옵션은 네트웍 장치 이름, '-s' 옵션은 패킷 크기, '-C' 옵션은  캡쳐할 파일을 구분할 크기(MB), '-W' 옵션은 순환할 파일 갯수,  '-w' 옵션은 파일 이름 앞부분, 마지막 필터 조건에서 'host'는 캡쳐할 캡쳐의 IP 주소를 의미합니다.

이렇게 실행하면 'capture.pcap0', 'capture.pcap1', 'capture.pcap2' 식으로 5MB 단위로 캡쳐 파일을 생성합니다. 그리고 항상 마지막 3개 파일만 남깁니다. 캡쳐 도중 현상이 발생했다면 CTRL-C 키를 눌러 캡쳐를 멈추고, 마지막 파일을 Wireshark 프로그램 메뉴에서 'File' -\> 'Open...' 기능을 이용해 읽어오면 됩니다.

더 자세한 내용은 [Wireshark 매뉴얼](http://www.wireshark.org/docs/wsug_html_chunked/AppToolstcpdump.html)과 'man tcpdump'를 참고하시길... :)
