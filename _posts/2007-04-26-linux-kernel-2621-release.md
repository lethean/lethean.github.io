---
layout: post
title: Linux Kernel 2.6.21 Release
tags: [ Kernel,  Linux ]
---

어김없이 또 [리눅스 커널 2.6.21 버전이 릴리스](http://lkml.org/lkml/2007/4/25/561)되었다.

개인적으로 관심있는 부분만 정리하면 다음과 같다.

<span style="font-weight:bold;font-size:100%;">Dynticks과 클럭이벤트</span>

리누스 말로는 타이머 부분이 가장 많이 바뀌었다고 하는데, 멀티태스킹 시분할 시스템(time-sharing system)의 기본 원리라고 할 수 있는 타이머 인터럽트의 오버헤드를 최소화하기 위해, 필요한 경우에만 사용하겠다는(tickless) 아이디어 자체가 신선하다. 전원 관리에도 효과가 있고, 시스템 성능에도 미미하지만 영향을 끼칠 수 있을 뿐 아니라 고해상도 타이머 구현도 더 효율적으로 구현되었다고 한다. 이번 릴리스에서는 X86-32만 지원하지만 다른 아키텍쳐도 곧 지원된다고 한다.

<span style="font-weight:bold;">ASoC (ALSA 시스템온칩) 레이어</span>

ASoc는 오디오 시스템을 다음과 같이 세가지로 구성한다.

-   코덱 드라이버 : 플랫폼 독립, 오디오 제어, 코덱 IO 등
-   플랫폼 드라이버 : DMA, 인터페이스 드라이버(I2S, AC97, PCM)
-   머신 드라이버 : 장치 관련 제어나 오디오 이벤트 처리

이를 통해 임베디드 시스템처럼 직접 연결된 오디오칩에 대한 드라이버 개발을 더 쉽게 해준다. 더불어 동적 전원 관리 시스템이 더 적은 전원을 사용하도록 도와준다. 예를 들어 실제로 캡쳐나 재생 작업이 있을 경우에만 알아서 전원 스위치를 제어한다.

<span style="font-weight:bold;">GPIO API</span>

임베디드 시스템 CPU에서 거의 대부분 사용하는 GPIO 관련 API가 공식적으로 추가되었다. 이렇게 단순한 것도 API가 될 수 있구나 하면서도, 왜 이제야 정리되었을까 하는 생각도 든다.

<span style="font-weight:bold;">utrace</span>

ptrace() 시스템콜의 기능을 넘어 dtrace 기능을 준비하기 위한 기본 API로 보인다.

<span style="font-weight:bold;">ARM11 oprofile 지원</span>

가끔 유용하게 사용하는 oprofile이 ARM11 플랫폼도 지원한다.

그외 가상화 관련 VMI, KVM 도 많이 개선되었다고 하는데, 사실 아직은 별로 관심이 없다. 더 자세한 변경사항은 [커널뉴비 페이지](http://kernelnewbies.org/LinuxChanges)에서 확인할 수 있다. API 변경 사항은 [LWN 페이지](http://lwn.net/Articles/2.6-kernel-api/)에서 확인이 가능하다.
