---
layout: post
title: Raspberry Pi + X11 + Clutter(+ Cogl)
tags: [ Linux, RaspberryPi, Clutter ]
---

결론부터 말하자면 실패한 해킹에 대한 기록입니다.

Raspberry Pi 2 장비를 하나 얻게 되어, 이삼일 정도 클러터(Clutter) 라이브러리가 제대로 동작하도록 삽질을 했습니다.

구글에는 X 서버, 즉 X 윈도 없이 리눅스 프레임 버퍼 위에서 EGL + GLES2 API를 이용하는 방법은 많이 나와 있는데, X11 + EGL + GLES2 조합은 없어서 여기저기 구글링을 통해 얻은 정보를 이용해 Cogl 예제 디렉터리에 있는 프로그램들이, 비록 전체화면 방식이기는 하지만, 문제없이 실행되게 하는 데까지는 성공했습니다. ([패치 파일](https://gist.github.com/lethean/ac21450495dddc597f79#file-cogl-1-8-raspberrypi-patch)과 [빌드 스크립트](https://gist.github.com/lethean/ac21450495dddc597f79#file-cogl-build-sh))

하지만 Clutter 예제 프로그램을 돌리면 여러 가지 경고를 내고 멈추거나 아무 메시지도 출력하지 않고 CPU 점유율만 차지하는 경우가 발생합니다. EGL + Raspberry Pi API가 전혀 생소한 것은 물론 Cogl + Clutter 연결 고리도 잘 모르지만, 다른 할 일도 많고, 내일모레부터는 설 연휴이기도 하고, 당장 급한 일도 아니라 일단 이 상태에서 작업을 멈추었습니다.

혹시 Clutter / Cogl 라이브러리를 Raspberry Pi 상에서 깔끔하게 돌아가게 하는 패치나 소스를 알고 계신 분 있나요? 아마도 회사에서 업무로 Raspberry Pi를 건드리는 분 중에는 분명 이미 작업한 분이 있을 것 같은데...

시간이 지날수록 구글링 실력도 점점 줄어드는 것 같고... :)
