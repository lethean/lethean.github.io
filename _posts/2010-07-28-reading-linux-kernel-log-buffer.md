---
layout: post
title: "리눅스 커널 로그 버퍼 읽기"
tags: [ Kernel,  Linux ]
---

우분투 최신 버전을 설치하면 [kerneloops-daemon](http://packages.ubuntu.com/lucid/kerneloops-daemon) 패키지가 자동으로 설치됩니다. 이 프로그램은 커널 로그 메시지를 주기적으로(10초 간격으로) 가져와서 파싱(parsing)한 뒤 커널 패닉(OOPS) 메시지를 추출해 이를 [D-Bus](http://www.freedesktop.org/wiki/Software/dbus)를 통해 전달하는 역할을 하는데. [kerneloops-applet](http://packages.ubuntu.com/lucid/kerneloops-applet) 패키지를 설치하면 로그인시 자동으로 애플릿 하나가 실행되면서 D-Bus에서 메시지를 받아 사용자 확인을 거쳐 [kerneloops.org](http://kerneloops.org/) 사이트 등으로 보고서를 자동으로 전송합니다. 참고로, 우분투에서 데몬을 동작하게 하려면 `/etc/default/kerneloops` 파일 안에서 `enabled` 항목을 1로 변경해야 하고, 세부 동작 옵션은 `/etc/kerneloops.conf` 설정 파일을 수정하면 됩니다.

그런데 이 kernelooops 소스를 검토하던 중 커널 로그 버퍼(보통 dmesg 명령 결과)를 가져오기 위해 다음과 같은 시스템콜을 직접 호출하는 것을 발견했습니다. (kerneloops 패키지 소스 안에 `dmesg.c:423`)

    syscall(__NR_syslog, 3, buffer, getpagesize());

이 시스템 콜 사용법이 궁금해서 dmesg 소스를 확인해 보니 여기서는 다음과 같은 C 라이브러리 함수를 사용합니다. (util-linux 패키지 소스 안에 `sys-utils/dmesg.c:120`)

    n = klogctl(3, buf, sz); /* read only */

그래서 매뉴얼을 찾아보니(`man klogctl`) 둘 모두 같은 동작을 하는 것은 물론, 지금껏 모르고 있었던 몇가지 기능도 알 수 있었습니다.

![](/figures/man-klogctl.png)

예를 들어, 매뉴얼에도 나와 있듯이, 지금까지는 syslogd 데몬과 통신하는 syslog(3) 함수만 알고 있었는데,  이 함수는 커널 syslog 시스템콜과 아무 관계가 없다는 점 등입니다. 참고로, 리눅스 커널 소스는 `kernel/printk.c` 파일에 있는 `do_syslog()` 함수가 실제로 syslog 시스템콜을 처리하고 있습니다.
