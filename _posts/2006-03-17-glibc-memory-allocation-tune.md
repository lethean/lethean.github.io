---
layout: post
title: glibc 메모리 할당 방식 튜닝
tags: [ glibc,  Linux ]
---

'Malloc 연구 : 과도한 마이너 폴트 사례 ([A Study in Malloc: A Case of Excessive Minor Faults](http://www.usenix.org/publications/library/proceedings/als01/ezolt.html)'라는 논문은 개발자가 흔히 부딪힐 수 있는 문제에 대하여 원인 추적 및 해결 과정을 흥미롭게 보여준다. 더불어 리눅스에서 GNU libc의 메모리 관리자를 튜닝하는 방법을 이용해 코드 재작성 없이 어플리케이션 성능을 향상시키는 방법에 대해 논하고 있다.

컴팩(Compaq)의 CXML이라는 확장 수학 라이브러리를 이용해 어플리케이션을 개발하는 사용자로부터 다음과 같은 문의가 들어왔다. 똑같은 코드와 똑같은 사양의 하드웨어를 기반으로 할때 기존 유닉스(Tru64 UNIX) 환경보다 리눅스(Linux/Alpha)에서 어플리케이션 실행 시간이 너무 많이 차이가 난다는 것이다.

이 글의 저자는 여러가지 도구를 이용하여 원인이 리눅스 커널의 메모리 관리와 연관이 있다는 것을 밝혀낸다. 그리고, 더 나아가 커널의 메모리 관리가 아닌 GNU libc의 `malloc()`/`free()` 구현이 기존 유닉스나 FreeBSD 등과도 다르다는 점을 밝혀낸다.

일반적으로 `brk()` 시스템콜을 이용해 힙(heap) 영역을 확장하는 다른 유닉스 시스템과 달리 리눅스가 사용하는 glibc는 일정 크기 이상의 메모리 영역은 `mmap()`을 이용해 할당한다. 이렇게 할당한 메모리는 사용자가 해제했을때 바로 시스템으로 돌아가고, 전체적으로 시스템은 효율적으로 메모리를 사용할 수 있다는 장점이 있지만, `mmap()` 시스템콜 자체가 커널 영역 페이지 폴트 메카니즘을 사용하기 때문에 이 부분에서 많은 성능 저하가 발생하게 된다.

이 논문의 제목인 마이너 폴트(minor faults)란, 아예 할당할 메모리가 없어 스왑(swap)이 필요한 경우가 아닌(major faults), COW(copy-on-write) 기법이나 일반적인 디바이스 메모리 맵핑 등과 같이 물리적인 메모리를 가상 메모리에 맵핑할때 발생하는 페이지 폴트를 의미한다.

저자는 매우 자세하고 친절하게 몇가지 증상에서 원인을 찾아가는 과정을 설명하고 있다. 여기서 든 예제의 경우 128K 이상의 메모리를 할당하고 해제하는 과정을 반복할 경우 발생하는 이러한 마이너폴트를 줄이기 위해 결론적으로 다음과 같은 방법을 제시하고 있다.

첫번째 방법은 소스 코드를 수정해 `mmap()`을 쓰지 않도록 GNU libc의 메모리 관리자에게 알려주는 것이다.

    mallopt (M_MMAP_MAX, 0);
    mallopt (M_TRIM_THRESHOLD, -1);

두번째 방법은 어플리케이션을 실행하기 전에 환경변수에 이 값을 설정하는 방법이다.

    export MALLOC_MMAP_MAX_=0
    export MALLOC_TRIM_THRESHOLD_=-1

더 자세한 내용은 [mallopt()](http://www.gnu.org/software/libc/manual/html_node/Malloc-Tunable-Parameters.html) 함수를 참고하면 된다.

그런데, 요즘 커널과 요즘 glibc에도 그대로 적용이 가능한지는 확인해봐야 할 것 같다.
