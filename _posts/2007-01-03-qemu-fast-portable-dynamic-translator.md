---
layout: post
title: "빠르고 포터블한 동적 변환기, QEMU"
tags: [ Linux ]
---

요즘 리눅스 2.6.20부터 추가된 가상화 기술 [KVM](http://kvm.sourceforge.net/)과 더불어 이슈가 되고 있는 [QEMU](http://fabrice.bellard.free.fr/qemu/) 에뮬레이터의 내부 구현 원리를 밝힌 [기사](http://www.usenix.org/publications/library/proceedings/usenix05/tech/freenix/bellard.html)를 보면 흥미로운 기법을 소개하고 있다. 제목 그대로 빠르고 포터블한 동작 변환기(a fast and portable dynamic translator)를 말하는데, 이 기법의 동작 원리는 다음과 같다.

QEMU 는 인터프리터 방식처럼 하나씩 번역해서 가상 상태 머신을 동작시키는 대신, 변환기(translator)가 실행중에 대상 CPU 명령어를 호스트 CPU 명령어로 바꾸어 실행한다. 호스트 명령어로 변환하기 전에, 대상 명령어는 일종의 중간코드인 마이크로작업(micro operations)으로 변환된다. 이 마이크로 작업 각각이 실제 호스트 CPU 명령어로 대치되어 실행된다.

그런데 이 마이크로 작업에 대한 호스트 CPU 명령어 코드는 어셈블리로 작성된 것이 아니라, 하나의 마이크로 작업에 대해 하나의 C 함수로 구현되어 있다. 그리고, 이를 GCC 컴파일러가 생성한 바이너리를 가져다 그대로 이용한다. 물론 GCC 옵션을 조율해 불필요한 함수 앞/뒤 부분 파싱이나 최적화를 수행한다. 따라서 이론적으로는 GCC가 지원하는 모든 CPU를 쉽게 지원할 수가 있다.

당연한 얘기지만 플랫폼마다 다른 여러 특성을 모두 구현해야 하기 때문에 새로운 CPU나 플랫폼을 지원하는 일이 그리 쉽지는 않다. 하지만 이미 Linux, Windows, MacOS X 등의 호스트 운영체제를 지원하고 있으며 X86, PowerPC, ARM, Sparc 등의 CPU를 에뮬레이트하고, x86, PowerPC, ARM, Sparc, Alpha, MIPS 등의 실행 호스트 CPU를 지원한다.

[FFMpeg](http://ffmpeg.mplayerhq.hu/) 부터 알게된 [Fabrice Bellard](http://fabrice.bellard.free.fr/)라는 프랑스 사람(?)의 프로그래밍 + 성능 최적화 능력은 항상 내게 존경심을 불러일으키는 동시에, 자괴감에 빠지게 한다...

참고:

1.  [Finally user-friendly virtualization for Linux](http://linux.inet.hr/finally-user-friendly-virtualization-for-linux.html)
2.  [KVM: Kernel-based Virtual Machine for Linux](http://kvm.sourceforge.net/)
3.  [QEMU: open source processor emulator](http://fabrice.bellard.free.fr/qemu/)
4.  [QEMU, a Fast and Portable Dynamic Translator](http://www.usenix.org/publications/library/proceedings/usenix05/tech/freenix/bellard.html)

