---
layout: post
title: FontConfig 2.4 in Ubuntu Feisty
tags: [ Ubuntu,  Xorg ]
---

그렇다. 한동안 잠잠하더니 fontconfig 패키지가 2.4.1 버전으로 올라가면서 폰트 관련 설정이 대폭 변경되었다.

`/etc/fonts/language-selector.conf` 파일은 아예 참조도 안하고, 설정 파일도 철저하게(?) 분리되고 위치도 `/etc/fonts/conf.d`와 `/etc/fonts/conf.avail`로 나뉘어 설치와 사용을 구분하고 있다. 폰트 캐시 파일은 폰트가 위치한 각 디렉토리에 `fonts.cache-1`로 유지하더니 이제는 `/var/cache/fontconfig`로 통합되었다. 변동사항을 보니 이렇게 한군데로 모은뒤 메모리맵핑된 영역을 모든 프로세스가 공유하는 방식으로 바뀌면서 전체적인 어플리케이션 시작 속도 향상을 꾀한 것도 같다.
