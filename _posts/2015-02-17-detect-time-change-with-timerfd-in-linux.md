---
layout: post
title: 리눅스에서 시간 변경 여부 감지하기
tags: [ Linux, glibc ]
---

`systemd` 데몬은 가끔 "Time has been changed"라는 로그 메시지를 출력합니다. 그래서 과연 어떤 방식으로 시간이 변경되는 이벤트를 감지하는지 궁금해서 [소스 코드](http://cgit.freedesktop.org/systemd/systemd/tree/src/core/manager.c#n1920)를 한 번 들여다보았습니다.

일반적으로 [`timerfd_create()`](http://man7.org/linux/man-pages/man2/timerfd_create.2.html) 함수는 지연되어 실행하거나 주기적으로 처리해야 작업을 실행할 때 사용합니다. 맨 페이지의 예제 소스는 이러한 용도로 완벽합니다. 그런데 `systemd`에서는 `timerfd_settime()` 함수를 호출할 때 두 번째 인수에 `TFD_TIMER_CANCEL_ON_SET` 값을 전달합니다. 이 값은 맨 페이지에도 없어서 구글 검색을 해보니 [glibc API 변경 문서](http://man7.org/tlpi/api_changes/#Linux-3.0)에서 다음과 같이 설명된 부분을 찾을 수 있었습니다.

> The timerfd_settime() system call adds a TFD_TIMER_CANCEL_ON_SET flag.
> If this flag is set for a CLOCK_REALTIME absolute (TFD_TIMER_ABSTIME) timer,
>  then the timer is expired if the clock is reset.

즉, 커널 내부 시계(clock)가 초기화(reset)되면 타이머가 만기되어(expired) 이벤트가 발생한다는 내용입니다. 그리고 눈으로 읽는 것만으로는 역시 잊어버릴 것 같아서, [예제 프로그램을 간단하게 수정](https://gist.github.com/lethean/446cea944b7441228298#file-timechange-c)해서 시간이 변경되는 여러 경우를 실험해보니, 직접 시간을 변경하거나 NTP 시간 동기화로 인한 시간 변경은 감지하는데, 시간대(time zone) 변경은 감지하지 못하는군요. 게다가 사람이 보기에는 같지만 아주 조금만 달라져도, 추측하기에는 시간 변경 함수가 호출만 되어도, 이벤트가 발생합니다. 게다가 시간이 앞으로 이동했는지 뒤로 이동했는지도 알 방법이 없고.

아무튼, 오늘도 역시 여전히 아는 것보다 모르는 게 더 많다는 사실을 새삼 깨닫게 된 하루입니다.

