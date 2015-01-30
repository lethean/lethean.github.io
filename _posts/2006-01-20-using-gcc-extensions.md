---
layout: post
title: GCC 확장 기능 이용하기
tags: [ GCC ]
---

Robert Love의 블로그에 올라온 '[With a little help from your compiler](http://rlove.org/log/2005102601)' 라는 글을 보면 GCC 3 이후에 추가된 여러가지 확장 기능에 대해 설명하고 있다. 이 기능들을 이용하면 GCC 컴파일러가 코드를 최적화하는데 더 많은 정보를 줄 수 있고, 프로그래머의 작은 실수를 컴파일 과정에서 미리 잡아낼 수도 있다. 이 중에서 관심이 가는 몇가지만 간추려보자.

우선 GCC가 아닌 컴파일러와의 호환을 위해 다음과 같은 헤더 파일을 하나 만들어 사용한다.

    #if __GNUC__ >= 3
    # define inline inline __attribute__ ((always_inline))
    # define __pure __attribute__ ((pure))
    # define __const __attribute__ ((const))
    # define __noreturn __attribute__ ((noreturn))
    # define __malloc __attribute__ ((malloc))
    # define __must_check __attribute__ ((warn_unused_result))
    # define __deprecated __attribute__ ((deprecated))
    # define __used __attribute__ ((used))
    # define __unused __attribute__ ((unused))
    # define __packed __attribute__ ((packed))
    # define likely(x) __builtin_expect (!!(x), 1)
    # define unlikely(x) __builtin_expect (!!(x), 0)
    #else
    # define inline /* no inline */
    # define __pure /* no pure */
    # define __const /* no const */
    # define __noreturn /* no noreturn */
    # define __malloc /* no malloc */
    # define __must_check /* no warn_unused_result */
    # define __deprecated /* no deprecated */
    # define __used /* no used */
    # define __unused /* no unused */
    # define __packed /* no packed */
    # define likely(x) (x)
    # define unlikely(x) (x)
    #endif

참고로 VisualC++은 inline 키워드를 C++ 컴파일러는 정상적으로 인식하나 C 컴파일러는 인식하지 못한다. 따라서 \_\_inline 로 해주면 문제없이 컴파일할 수 있다.

    __const int f (int x) { ... }

\_\_const 같은 경우 함수 내부에서 절대로 전역 변수에 접근할 수 없고, 포인터 인수를 받을 수도 없도록 한다. abs() 같은 수학 함수에 적합하다.

    __must_check int f (void) { ... }

리눅스 커널 API 중에서 copy\_{tofrom}\_user() 에서 사용되어 친숙한 기능으로, 호출하는 코드에서 이 함수의 리턴값을 반드시 검사하도록 컴파일 과정에서 알려준다.

    __deprecated void f (void) { ... }

함수를 호출하게 되면 컴파일 과정에서 경고를 준다. 새로운 API로 대체되었거나, 앞으로 없어질 API를 명시하는데 유용하다.

    static __used void f (void) { ... }

어셈블리 코드에서만 호출하거나 다른 이유로 최적화 과정에서 없어지면 안되는 경우를 명시한다. 사용하지 않은 변수나 함수라는 컴파일러 경고도 없애준다.

    void f (int x __unused) { ... }

지정한 인수를 사용하지 않는 경우를 이미 프로그래머가 알고 있음을 컴파일러에게 알려주어 경고 메시지를 뿌리지 말도록 한다. 권하는 경우는 아니지만, 이벤트 방식 GUI 프로그래밍에서 많이 발견할 수 있는 패턴이다.

    struct __packed s { ... }

구조체 필드를 정렬(align)에 상관없이 압축한다. 구조체 크기는 줄어들 수 있지만, 지금까지 경험으로 볼때 별로 권장하고 싶지 않은 기능이다. ARM과 같은 아키텍쳐에서는 정렬되어 있지 않을 경우 바로 오동작을 일으킨다. 가장 좋은 방법은 구조체의 필드 순서 및 크기를 프로그래머가 정확하게 배치하는 것이다.

    if (unlikely (foo) && likely (bar)) { ... }

리눅스 커널 소스에서 많이 볼 수 있는 문법이다. 특정 조건이 일어날 확률이 훨씬 많을 경우 GCC에게 이를 알려주어 블럭 재배치 최적화에 이용하도록 할 수 있다.
