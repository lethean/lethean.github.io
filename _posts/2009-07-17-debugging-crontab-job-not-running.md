---
layout: post
title: "실행 안되는 crontab 작업 디버깅하기"
tags: [ Agile,  Linux,  Shell ]
---

리눅스를 비롯한 유닉스 방식 운영체제에서 주기적인 작업을 수행하기 위해 cron 데몬을 많이 사용합니다. 루트 권한으로 시스템 관리에 필요한 작업을 예약할 수도 있지만, 일반 사용자 계정도 '`crontab -e`' 명령으로 작업 예약이 가능합니다. 현재 활발하게 개발 중인 프로젝트의 일일 빌드(daily build)를 위해 crontab 작업을 다음과 같이 예약했는데, 문제는 동일한 설정이 한 장비에서만 동작을 안한다는 점입니다. (dooly 계정으로 작업한다고 가정합니다)

    # m h  dom mon dow   command
    0 23 * * * /home/dooly/build-cms.sh

빌드 스크립트(`/home/dooly/build-cms.sh`) 내용은 다음과 같습니다.

    #!/bin/sh
    cd /home/dooly/svn/dooly
    svn update && 
    sudo make install-depends && 
    make clean && 
    make && 
    make packages && 
    make upload

인터넷을 찾아보니 다음과 같이 수정하여 로그 파일을 분석하라고 해서 따라해 보았습니다.

    # m h  dom mon dow   command
    * * * * * /home/dooly/build-cms.sh >> /home/dooly/cron.log 2>&1
    * * * * * env > /home/dooly/env.log 2>&1

로그 파일을 분석하니, 환경 변수 LANG이 ko\_KR.UTF-8 로 설정되지 않아서 서브버전 갱신(update) 도중 에러가 발생하고 있었습니다. 그래서 스크립트를 다음과 같이 수정해서 일단 문제는 해결했습니다.

    #!/bin/sh
    export LANG=ko_KR.UTF-8
    cd /home/dooly/svn/dooly
    svn update && 
    sudo make install-depends && 
    make clean && 
    make && 
    make packages && 
    make upload

하지만 향후 문제 발생시 디버깅을 위해 다음과 같이 crontab 항목도 아예 변경해 두었습니다.

    # m h  dom mon dow   command
    0 23 * * * /home/dooly/build-cms.sh > /home/dooly/cron.log 2>&1
    #* * * * * env > /home/dooly/env.log 2>&1

물론, 자동으로 메일을 전송하도록 하거나 하는 다른 추가 기능도 가능하겠지만, 일단 이 정도 수준에서 만족하고 현재는 잘 동작하고 있습니다.
