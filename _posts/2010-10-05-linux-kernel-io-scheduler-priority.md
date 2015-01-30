---
layout: post
title: "리눅스 커널 I/O 스케줄링 우선순위"
tags: [ Kernel,  Linux,  Schedule ]
---

리눅스 커널 CPU 스케줄링과 마찬가지로 I/O 스케줄링에 적용되는 스케줄링 클래스와 우선순위도 [`ioprio_set()`](http://www.kernel.org/doc/man-pages/online/pages/man2/ioprio_set.2.html) 시스템콜을 이용해 사용자가 제어할 수 있습니다. 하지만 리눅스 커널이 제공하는 여러가지 I/O 스케줄러 중에서 CFQ(Completely Fair Queuing) 스케줄러에서만 사용할 수 있습니다. 물론 리눅스 커널은 블럭 장치마다 다른 I/O 스케줄러를 사용할 수 있도록 허용하므로 필요한 경우 적절하게 시스템을 구성하는 것도 가능합니다.[1]

I/O 스케줄링 클래스는 CPU 스케줄링과 비슷하게 나누어집니다. 첫번재 `IOPRIO_CLASS_RT(1)` 클래스는 실시간(real-time) I/O 클래스로서, 이 클래스에 속한 태스크는 다른 클래스에 속한 태스크보다 항상 디스크에 먼저 접근합니다. 클래스 내부 우선순위는 가장 높은 0부터 가장 낮은 7까지 지정할 수 있습니다. 두번째 `IOPRIO_CLASS_BE(2)` 클래스는 최선노력(best-effort) I/O 클래스로서, 특별히 I/O 우선순위를 지정하지 않은 대부분의 태스크가 이 클래스에 속합니다. 첫번째 클래스와 마찬가지로 0부터 7까지 내부 우선순위를 지정할 수 있으며, 이 우선순위에 따라 얼마나 많은 I/O 대역폭을 할당할지 결정합니다. 마지막 `IOPRIO_CLASS_IDLE(3)` 클래스에 속한 태스크는 위의 두 클래스에 속한 어떤 태스크도 해야할 I/O 작업이 없을때만 I/O 작업을 수행하고, 내부 우선순위는 무시됩니다.

I/O 스케줄링 클래스와 우선순위는 읽기 작업과 동기화 쓰기 작업(`O_DIRECT`, `O_SYNC`)에만 반영됩니다. 즉, 일반적인 비동기 쓰기 작업에는 적용되지 않습니다.

`ioprio_set()`, `ioprio_get()` 시스템콜은 리눅스 표준 C 라이브러리에 대응하는 함수가 없기 때문에 다음과 같이 직접 `syscall()` 함수를 이용해 호출해야 합니다.[2]

    #define _GNU_SOURCE      /* or _BSD_SOURCE or _SVID_SOURCE */
    #include <unistd.h>
    #include <sys/syscall.h> /* For SYS_xxx definitions */

    static inline int ioprio_set (int which, int who, int ioprio)
    {
      return syscall (__NR_ioprio_set, which, who, ioprio);
    }

    static inline int ioprio_get (int which, int who)
    {
      return syscall (__NR_ioprio_get, which, who);
    }

위와 같이 프로그램 소스 코드를 수정하지 않아도 [`ionice`](http://linux.die.net/man/1/ionice) 프로그램을 이용하면 셸(shell)이나 스크립트에서 직접 다른 태스크의 I/O 스케줄링 클래스와 우선순위를  변경할 수도 있습니다.

이와 더불어 리눅스 커널 cgroups 시스템의 blkio 서브시스템을 이용하면 태스크 / 디스크별 대역폭(bandwidth)을 할당하거나 그룹별로 더 다양한 I/O 정책을 세밀하게 적용할 수 있습니다.[3]

[1] [ioprio\_set(), Linux Programmer's Manual](http://www.kernel.org/doc/man-pages/online/pages/man2/ioprio_set.2.html)
 [2] [Block IO Priorities, Linux Kernel Source Documentation](http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=blob;f=Documentation/block/ioprio.txt)
 [3] [Block IO Controller, Linux Kernel Source Documentation](http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=blob;f=Documentation/cgroups/blkio-controller.txt)
