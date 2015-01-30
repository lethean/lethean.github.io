---
layout: post
title: MacOS X의 한글 이름 파일을 리눅스에 복사하면 깨지는 현상 고치기
tags: [ Hangul,  Linux,  MacOSX ]
---

요즘 맥북과 리눅스를 같이 사용하면서 여러가지 방법으로 파일을 공유하고 있는데, SSH 등으로 직접 복사할때 파일 이름에 한글이 포함되어 있으면 깨지는 현상이 발생합니다. 리눅스에서 맥으로 복사할때는 문제가 없는데, 다시 맥에서 리눅스로 복사하면 파일 이름의 한글이 요상하게 보입니다. 동일한 UTF-8 환경이라 문제가 없을 줄 알았는데, 이 때문에 Unison 같은 프로그램도 오동작을 합니다.

대략 검색해보니 UTF-8을 인코딩할때 리눅스 계열의 운영체제는 NFC(normalization form C) 방식을 사용하는데 맥의 다윈 커널에서는  NFD(normalization form D) 방식을 사용하기 때문이랍니다. 아무튼, 해결하는 방법은 convmv 프로그램을 이용하면 됩니다.

우선 다음과 같이 convmv 프로그램을 설치합니다.

    $ sudo apt-get install convmv

한글이 깨진 파일이나 디렉토리에서 다음 명령을 실행합니다.

    $ convmv -f utf8 -t utf8 -r --nfc --notest *

더 자세한 사용법은 \``man convmv`'를 입력하시길~
