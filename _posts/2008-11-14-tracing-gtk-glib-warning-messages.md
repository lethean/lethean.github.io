---
layout: post
title: GTK+ / GLib 경고 메시지 추적하기
tags: [ GLib,  GTK+ ]
---

GTK+ 또는 GLib 기반 프로그래밍을 할때 g\_warning() / g\_return\_if\_fail() 등과 같은 API를 사용한 경고 메시지가 어디서 발생했는지 찾기 어려운 경우가 있습니다. 더 나아가 해당 함수를 호출하는 부분을 알아야 하는데, 사실 메시지만으로는 찾기가 매우 어려운 경우가 많습니다.

예를 들어 수만라인짜리 프로그램이 실행 도중 다음과 같은 메시지를 출력합니다.

    Gtk-CRITICAL **: gtk_widget_set_sensitive: 
    assertion `GTK_IS_WIDGET (widget)' failed

아, gtk\_widget\_set\_sensitive() 함수를 호출할때 첫번째 인수를 잘못 넘겨준 건 알겠는데, 문제는 gtk\_widget\_set\_sensitve() 함수를 호출하는 부분이 수십군데입니다. 마땅히 실행 도중이라 어떤 모듈에서 호출하는지도 애매합니다. 다음에 설명할 방법을 모른다면, 리누스 토발즈의 말 그대로, 모든 소스 코드를 직접 검토할 수 밖에 없죠...

만일 이 메시지가 발생하는 시점에서 gdb의 backtrace 명령으로 함수 호출 스택을 알아낼 수 있다면 인생은 편해집니다. 그리고 이를 위해 GLib에는 당연하다는듯이 [g\_log\_set\_always\_fatal()](http://library.gnome.org/devel/glib/stable/glib-Message-Logging.html#g-log-set-always-fatal) 이라는 API가 존재합니다. 이 API는 설정하는 레벨의 로그 메시지가 발생하면 강제로 코어 덤프를 발생하고 프로그램 실행을 중지합니다. gdb에서 동작할 경우 해당 지점에서 정확하게 멈춥니다.

위 예에서 우리가 원하는 레벨은 CRITICAL 레벨인 경우이므로 다음 코드를 프로그램 시작 부분에 넣어주면 위 메시지가 출력되는 시점에서 정확하게 프로그램이 멈추게 됩니다.

    g_log_set_always_fatal (G_LOG_LEVEL_ERROR |
                            G_LOG_LEVEL_CRITICAL);

그리고 gdb에서 backtrace 명령을 실행하면, 정확하게 문제를 일으키는 코드를 찾아낼 수 있겠죠?
