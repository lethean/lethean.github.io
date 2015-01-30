---
layout: post
title: "리눅스 커널 실시간 스케줄링 우선순위"
tags: [ Kernel,  Linux,  Schedule ]
---

`SCHED_OTHER`, `SCHED_IDLE`, `SCHED_BATCH` 스케줄링 정책(policy)에 속하는 일반 태스크는 스케줄링 우선순위(priority)는 항상 0입니다. 하지만 `SCHED_FIFO`, `SCHED_RR` 등과 같은 실시간 스케줄링 정책에 속하는 태스크는 가장 낮은 1부터 가장 높은 99까지의 우선순위가 부여됩니다.

리눅스 커널 스케줄러는 태스크 우선순위별로 실행 가능한 태스크 목록을 유지하고,  가장 높은 우선순위부터 차례대로 각 우선순위별 태스크 목록이 비어있는지 검사해서 태스크가 있을 경우 목록의 첫번째 태스크를 다음에 실행할 태스크로 선택합니다. 따라서, 항상 가장 높은 우선순위가 부여된 실시간 태스크가 실행되고, 결과적으로 일반 태스크는 항상 실시간 태스크보다 나중에 스케줄링됩니다.

`SCHED_FIFO`, `SCHED_RR` 정책은 우선순위가 같을 경우 태스크를 목록 어디에 삽입할 지를 결정하는데 사용합니다. `SCHED_FIFO` 정책에 속하는 태스크는 실행 가능 상태가 되면 우선순위 태스크 목록의 마지막에 삽입됩니다. 이 태스크는 I/O 요청을 기다리거나, 우선순위가 더 높은 태스크에 의해 선점(preempted)되는 경우, 직접 [`sched_yield(2)`](http://www.kernel.org/doc/man-pages/online/pages/man2/sched_yield.2.html) 시스템콜을 호출하는 경우가 아니면 계속 실행됩니다. 반면에 `SCHED_RR` 정책은 `SCHED_FIFO` 정책과 비슷하지만 각 태스크에게 할당된 최대 실행 시간(maximum time quantum)이 지나면 자동으로 우선순위 태스크 목록 마지막으로 이동됩니다.[1]

만일 프로세스(process)가 아닌 쓰레드(thread)의 스케줄링 속성을 변경하려면, `gettid()` 시스템콜을 호출해서 얻은 tid 값을 프로세스 pid 대신 넘겨주면 됩니다. 하지만 `gettid()` 시스템콜은 C 라이브러리에 대응하는 함수가 없기 때문에 다음과 같이 직접 `syscall()` 함수를 이용해 호출해야 합니다.[2]

    #define _GNU_SOURCE      /* or _BSD_SOURCE or _SVID_SOURCE */
    #include <unistd.h>
    #include <sys/syscall.h> /* For SYS_xxx definitions */
    #include <sys/types.h>   /* For pid_t */

    static inline pid_t gettid (void)
    {
      return (pid_t) syscall (SYS_gettid); /* or __NR_gettid */
    }

위와 같이 직접 프로그래밍하지 않아도 [`chrt`](http://linux.die.net/man/1/chrt) 유틸리티를 사용하면 셸(shell)이나 스크립트에서 직접 다른 태스크의 실시간 스케줄링 속성을 변경할 수도 있습니다.

하지만 리눅스 커널은 실시간 태스크 스케줄링에 필수적인 스케쥴링 지연시간(scheduling latency), 인터럽트 지연시간(interrupt latency)을 보장하지(guarantee) 않기 때문에 이를 개선하기 위해서는 별도의 패치를 적용한 커널을 사용해야 합니다.[3][4]

[1] [sched\_setscheduler(), Linux Programmer's Manual](http://www.kernel.org/doc/man-pages/online/pages/man2/sched_setscheduler.2.html)
 [2] [gettid(), Linux man-pages](http://www.kernel.org/doc/man-pages/online/pages/man2/gettid.2.html)
 [3] [Real-time Linux Wiki](https://rt.wiki.kernel.org/)
 [4] [OSADL Project: Real-time Linux](https://www.osadl.org/Realtime-Linux.projects-realtime-linux.0.html)
