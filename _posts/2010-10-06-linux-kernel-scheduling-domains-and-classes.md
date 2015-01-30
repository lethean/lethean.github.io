---
layout: post
title: "리눅스 커널 스케줄링 영역과 클래스"
tags: [ Kernel,  Linux,  Schedule ]
---
**스케줄링 영역 (Scheduling Domain)**

멀티 프로세서 시스템에서 스케줄러의 중요한 역할 중 하나는 모든 CPU의 부하를 균등하게 맞추는 일입니다. 이를 위해 스케줄러는 한 CPU에서 동작하던 태스크를 다른 CPU로 이동해야 하는 경우, 각 아키텍쳐의 특성을 고려해야 합니다.  왜냐하면, 한 CPU에서 동작하던 태스크를 다른 CPU로 옮기면(migration) 캐시 불일치 등으로 인한 오버헤드가 발생하는 것은 물론 아키텍쳐에 따라 심각한 성능 저하를 일으킬 수도 있기 때문입니다.[1]

예를 들어, 하이퍼 쓰레드 프로세서는 하나의 프로세서 안에 논리적인 CPU가 여러 개 포함된 구조이기 때문에 여러 논리 CPU는 모두 캐시와 메모리를 모두 공유합니다. 따라서 이 논리 CPU간에는 태스크를 이동하는데 오버헤드가 없습니다. SMP(Symmectric Multi Processor) 시스템이나 멀티 코어(Multi Core) 프로세서에는 하나의 물리 CPU 안에 한 개 이상의 CPU 코어가 장착되어 있습니다. 이 CPU 코어는 각각 캐시(cache)를 가지고 있지만, 메모리는 모두 공유합니다. 따라서 프로세서간 태스크 이동(migration)이 가능한 적을수록 좋은 성능을 얻을 수 있습니다. NUMA 아키텍쳐는 여러 노드로 구성되는데, 한 노드는 하나의 CPU와 메모리, 캐시로 구성됩니다. 그리고 한 CPU가 같은 노드의 메모리를 접근할 때는 속도가 다른 노드의 메모리를 접근할 때보다 훨씬 빠릅니다. 그러므로 CPU간 태스크 이동은 정책적으로 반드시 필요한 경우에만 일어나야 합니다. 최근 출시되는 프로세서는 대부분 멀티 코어이면서 각 프로세서 코어가 다시 하이퍼 쓰레드를 통해 논리 CPU를 지원하고, NUMA 아키텍쳐에서 한 노드의 프로세서가 하이퍼 쓰레드를 지원하기도 합니다.

이처럼 다양한 구조의 시스템을 효율적으로 지원하기 위해 리눅스 커널 2.6 버전부터 스케줄링 영역(scheduling domains) 기능이 추가되었습니다. 스케줄링 영역은 스케줄링 그룹 자료구조와 함께 구성되며 계층적인 방식으로 전체 시스템의 영역을 구성합니다. 스케줄링 영역(`struct sched_domain`)은 속성(properties)과 정책(policies)을 공유하는 CPU 집합이기 때문에, 해당 CPU 간에 부하가 균등하게 조절됩니다. 스케줄링 그룹(`struct sched_group`)은 CPU별로 할당되면서 동시에 영역의 여러 CPU를 묶어서 구성되기도 합니다. 스케줄링 영역은 스케줄링 그룹을 한 개 이상 포함하기 때문에 스케줄러가 한 영역 내에서 균형을 유지하려할 때 그룹 안에서 발생하는 상황을 걱정하지 않고 그룹의 부하(load)를 다른 그룹으로 이동합니다.

**스케줄링 클래스 (Scheduling Class)**

리눅스 커널 2.6.23 버전에서 기존 O(1) 스케줄러를 대체한 CFS(Completely Fair Scheduler) 스케줄러를 구현하면서 함께 도입된 스케줄링 클래스(Scheduling Class)는 리눅스 커널의 다양한 스케줄링 정책을 캡슐화해서 기본 스케줄러를 확장하기 쉽도록 도와줍니다. 하지만 스케줄러 메인 코드가 모듈 방식으로 구현된 건 아니기 때문에 새 스케줄러를 일반 커널 모듈처럼 쉽게 넣거나 뺄 수 있는 건 아닙니다. 다만 POSIX 표준에서 요구하는 `SCHED_RR`, `SCHED_FIFO`, `SCHED_NORMAL`, `SCHED_BATCH`, `SCHED_IDLE` 등과 같은 정책을 분리해서 구현하기 위해 도입된 구조입니다.[2] 하지만, 새 스케줄러 모듈을 추가하는 작업이 예전에 비해 쉬어진 것은 사실입니다.

스케줄링 클래스는 코어 스케줄러를 돕는 여러 모듈을 연결해 놓은 고리처럼 볼 수 있습니다. 가장 앞에 있는 스케줄러 모듈이 먼저 실행되고, 실행할 태스크가 없으면 다음 스케줄러 모듈을 실행하는 방식으로 동작합니다. 실제로, 스케줄링 클래스에는 실시간 스케줄러(`sched_rt.c`), CFS 스케줄러(`sched_fair.c`), 유휴 스케줄러(`sched_idletask.c`) 순으로 단일 연결 목록에 등록되어 있어 순서대로 스케줄러 모듈을 실행합니다.

각 스케줄러 모듈은 스케줄링 클래스 구조체(`struct sched_class`)가 제안하는 기능을 콜백 함수처럼 구현합니다. 다음은 리눅스 커널 2.6.23 버전에서 정의된 스케줄링 클래스 구조입니다.

    struct sched_class { /* Defined in 2.6.23:/usr/include/linux/sched.h */
      struct sched_class *next;
      void (*enqueue_task) (struct rq *rq, struct task_struct *p, int wakeup);
      void (*dequeue_task) (struct rq *rq, struct task_struct *p, int sleep);
      void (*yield_task) (struct rq *rq, struct task_struct *p);
      void (*check_preempt_curr) (struct rq *rq, struct task_struct *p);
      struct task_struct * (*pick_next_task) (struct rq *rq);
      void (*put_prev_task) (struct rq *rq, struct task_struct *p);
      unsigned long (*load_balance) (struct rq *this_rq, int this_cpu,
                     struct rq *busiest,
                     unsigned long max_nr_move, unsigned long max_load_move,
                     struct sched_domain *sd, enum cpu_idle_type idle,
                     int *all_pinned, int *this_best_prio);
      void (*set_curr_task) (struct rq *rq);
      void (*task_tick) (struct rq *rq, struct task_struct *p);
      void (*task_new) (struct rq *rq, struct task_struct *p);
    };

위 구조체에서 중요한 함수를 살펴보면 다음과 같습니다.

-   `enqueue_task()` : 태스크가 실행 가능한 상태로 진입할 때 호출됩니다.
-   `dequeue_task()` : 태스크가 더 이상 실행 가능한 상태가 아닐때 호출됩니다.
-   `yield_task()` : 태스크가 스스로 `yield()` 시스템콜을 실행했을 때 호출됩니다.
-   `check_preempt_curr()` : 현재 실행 중인 태스크를 선점(preempt)할 수 있는지 검사합니다.
-   `pick_next_task()` : 실행할 다음 태스크를 선택합니다.
-   `put_prev_task()` : 실행중인 태스크를 다시 내부 자료구조에 넣을때 호출됩니다.
-   `load_balance()` : 코어 스케줄러가 태스크 부하를 분산하고자 할때 호출됩니다.
-   `set_curr_task()` : 태스크의 스케줄링 클래스나 태스크 그룹을 바꿀때 호출됩니다.
-   `task_tick()` : 타이머 틱 함수가 호출합니다.
-   `task_new()` : 새 태스크가 생성되었을때 그룹 스케줄링을 위해 호출됩니다.

스케줄링 클래스 구조는 기본 CFS 스케줄러에서 사용하는 내부 자료구조와 밀접하게 연관되어 있습니다. 예를 들어 각 CPU별로 유지하면서 콜백 함수의 인수로 넘겨지는 실행 큐(`struct rq`)는 CFS 스케줄러를 위해 채택한 레드-블랙 트리(red-black tree) 자료구조를 사용하기  때문에 새 스케줄러 모듈을 구현하려면 반드시 기존 CFS 스케줄러 동작 방식과 구조를 어느정도 자세히 알고 있어야 합니다.[3]

[1] [Jonathan Corbet, "Scheduling domains", LWN.net, 2004
](http://lwn.net/Articles/80911/)[2] [Avinesh Kumar, Multiprocessing with the Completely Fair Scheduler, IBM developerWorks, 2008
](http://www.ibm.com/developerworks/linux/library/l-cfs/)[3] [M. Tim Jones, Inside the Linux 2.6 Completely Fair Scheduler, IBM developerWorks, 2009](http://www.ibm.com/developerworks/linux/library/l-completely-fair-scheduler/)


