---
layout: post
title: SSH 디렉토리 백업하기
tags: [ Network,  Shell ]
---

시간이 갈수록 자주 사용하는 명령어를 기억하지 못해 동일한 검색어로 구글링을 반복하는 경우가 점점 많아져서, 그런 수고를 덜기 위해 여기에 적어두는 글입니다.

SSH 원격 셸 명령어를 이용하여 디렉토리 전체를 복사하고자 할 때, 특히 파일의 수정일시, 권한, 심볼릭 링크 등을 그대로 이전하고자 할때 다음과 같이 실행하면 됩니다.

    $ tar czpf - . | ssh lethean@hades "tar xzpvf - -C /home/lethean/backup"

위 명령어는 현재 디렉토리의 모든 내용을 그대로 `hades` 서버에 `lethean` 계정으로 접속해서 `/home/lethean/backup` 디렉토리에 복사합니다.

**[UPDATED]** 네트웍 속도가 충분하고 CPU 점유율을 낮춘 상태로 빨리 복사하고 싶다면 다음과 같이 압축 옵션을 사용하지 않아도 됩니다.

    $ tar cpf - . | ssh lethean@hades "tar xpvf - -C /home/lethean/backup"

:)
