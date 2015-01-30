---
layout: post
title: Linux Kernel 2.6.20 Release
tags: [ Kernel,  Linux ]
---

리눅스 커널 2.6.20 버전이 릴리스되었다. 관심있는 사항만 요약하면 다음과 같다.

<span style="font-weight:bold;">소니 플레이스테이션3(PS3) 지원</span>

아직 그래픽 장치처럼 모든 주변장치를 지원하는 것은 아니고 일반적인 PS3에서 부팅이 가능한 것도 아니지만, 소니 엔지니어에 의해 공식적으로 PS3가 지원되기 시작했다.

<span style="font-weight:bold;">KVM을 이용한 가상화(Virtualization) 지원</span>

나처럼 구식(?) PC를 사용하는 사람들한테는 그림의 떡이지만, 최신 인텔과 AMD CPU에서 지원하는 가상화 기능(Intel VT / AMD-V)을 이용해 VMWare처럼 가상의 머신을 실행시킬 수 있도록 도와주는 KVM이 공식적으로 지원된다. 물론 [QEMU](http://fan4326.blogspot.com/2007/01/qemu.html)와 함께 동작하며 아직 윈도우 운영체제는 APIC 문제로 잘 동작하지 않는 문제이지만, 조만간 모두 해결될 것으로 보인다.

<span style="font-weight:bold;">i386에서 병렬가상화(Paravirtualization) 지원</span>

KVM이 하드웨어의 도움을 받는 가상화솔루션이라면 이 기능은 이미 존재하는 가상화 솔루션에서 이용할 수 있는 공통 모듈이다. 게스트 운영체제에 대한 제어 기능이 커널에서 공식적으로 지원하게 됨에 따라 VMWare, Xen 등의 커널 모듈도 이 기능을 이용하도록 변경될 것으로 생각된다.

<span style="font-weight:bold;">X86에서 재배치가능(relocatable)한 커널 지원</span>

런타임 오버헤드없이 컴파일시 커널 주소 공간을 지정할 수 있도록 한다. 일반 사용자에게는 별로 중요하지 않지만 kexec 등을 이용해 커널 크래쉬 상태를 덤프하고 다시 로드할때 커널 주소 공간을 다르게 함으로서 유용하게 사용할 수 있다고 한다.

<span style="font-weight:bold;">결함 주입 (Fault Injection)</span>

실시간 시스템의 결함 허용 중 하나인 결함 주입 기능은 일부러 에러를 일으키는 값이나 환경을 만들어 커널이 이를 얼마나 잘 견디고 처리하는지를 확인하는 것으로 쉽게 찾을 수 없는 에러를 디버깅하는데 유용하다. 구체적으로 이 기능은 메모리 할당 오류와 디스크 I/O 실패를 고의로 발생시키는데, 파일시스템 개발자들은 이에 대한 예외처리를 얼마나 잘 하는지 테스트 가능하다.

<span style="font-weight:bold;">상대적 atime 지원</span>

마운트시 'noatime' 옵션을 주면 파일을 읽을때마다 접근 시간(access time)을 갱신하지 않게 되어 실제적으로 매우 많은 디스크 성능 향상을 체험하게 된다. 실제로 kernel.org 사이트도 이 옵션 하나만으로 평균 부하량이 반으로 줄었다고 하며, 현재 우리 회사에서 개발한 DVR 시스템에도 이 옵션이 적용되어 있다. 그런데 이 기능은 무조건 갱신하는 것이 아니라 생성시간(ctime)이나 수정시간(mtime)보다 접근시간(atime)이 더 오래되었을 경우에만 갱신하도록 한다. 이를 통해 정확하면서도 'noatime' 옵션과 비슷한 성능향상을 얻을 수 있다고 한다. 아직은 OCFS2 파일시스템에서만 지원된다고 하며 mount(8) 에도 'reltime' 옵션으로 적용되어 있다.

<span style="font-weight:bold;">X86-32에서 'regparm' 사용</span>

GCC 확장 기능을 이용하여 함수 인수 전달시 인수 갯수가 3개 이하일 경우 레지스터를 이용하여 전달하도록 하는 방식('-mregparm=3' )을 사용한다. 그런데 내가 알기로는 레드햇 커널에서는 이미 예전부터 기본적으로 사용하고 있는데...

<span style="font-weight:bold;">워크큐(Workqueue) 구조 변경</span>

워크큐(struct work\_struct) 구조가 변경되어 이를 사용하고 있는 외부 드라이버나 모듈이 있다면 [이 페이지](http://lwn.net/Articles/211279/)를 참고하여 수정하는 것이 좋다.

<span style="font-weight:bold;">GCC 버전 변경</span>

이제 커널 2.6.20 버전 이상을 컴파일하려면 최소 gcc 3.2 버전 이상이 필요하다.

<span style="font-weight:bold;">i386에서 300Hz 지원</span>

25FPS를 사용하는 PAL 방식과 달리 29.99FPS를 사용하는 NTSC 방식 비디오 프레임 처리에 유용하도록 300Hz 클럭을 지원한다. 이 클럭을 사용하면 25 / 30 FPS 모두를 처리하는데 유용하며 성능은 250Hz와 비슷하다고 한다.

물론 더 자세한 내용은 이미 잘 정리된 [KernelNewbies 2.6.20 변경사항](http://kernelnewbies.org/Linux_2_6_20) 페이지를 참고하면 된다.
