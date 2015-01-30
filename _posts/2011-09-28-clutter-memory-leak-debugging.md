---
layout: post
title: Clutter 메모리 누수 디버깅
tags: [ Clutter,  Coding ]
---

최근 [클러터를 이용한 프로그램을 개발](/2011/09/15/note-about-using-clutter/)하면서 메모리 누수 현상을 발견했습니다. 코드를 하나 하나 막아가면서 테스트를 한 결과 [`ClutterGstVideoSink`](http://developer.gnome.org/clutter-gst/1.3/ClutterGstVideoSink.html) 객체를 사용하지 않으면 메모리 누수가 발생하지 않았습니다. 하지만, 아무리 소스를 분석해도 원인을 찾아낼 수 없었고, 잘못된 부분도 없는 것 같았습니다. 물론 구글링을 해도, 검색 실력이 미천한지, 답을 찾을 수 없었습니다.

그래서 결국 예전에 소개한 적 있는 [구글 성능 도구(google-perftools)를 이용해 디버깅](/2009/06/18/debugging-memory-leaks-with-tcmalloc-google-perftools/)을 했습니다. 그런데 문제는, [아치 리눅스(Arch Linux)](http://www.archlinux.org/) x86\_64 환경으로 개발 환경을 바꾸면서 메모리 프로파일 기능이 제대로 동작하지 않는다는 사실인데, 특히 메모리 누수 발생 지점을 정확하게 알기 위해서 필요한 함수 호출 백트레이스(backtrace) 정보가 추출되지 않는 게 가장 큰 문제였습니다. 이 문제를 해결하기 위한 과정을 기록으로 남겨봅니다.

**구글 성능 도구 설치**

아치 리눅스(Arch Linux) x86\_64 환경에서 구글 성능 도구(google-perftools)가 정확한 메모리 프로파일 결과를 얻으려면 [libunwind](http://www.nongnu.org/libunwind/) 라이브러리를 설치해야 하는데, 아치리눅스 [AUR](https://wiki.archlinux.org/index.php/AUR) 패키지를 [yaourt](https://wiki.archlinux.org/index.php/Yaourt)를 이용해 다음과 같이 쉽게 설치했습니다.

    $ yaourt -S libunwind

그리고 다음과 같이 구글 성능 도구를 빌드하고 설치합니다.

    $ cd google-perftools
    $ ./configure --prefix=/usr --enable-frame-pointers
    $ make
    $ sudo make install

**라이브러리 패키지 재생성 및 재설치**

정확한 함수 호출 백트레이스(backtrace) 정보를 얻기 위해 프로그램에 사용되는 모든 라이브러리를 다시 컴파일해 패키지를 다시 설치해야 하는데, 그 과정은 다음과 같습니다. ([관련 위키 페이지](https://wiki.archlinux.org/index.php/Debug_-_Getting_Traces) 참고)

먼저 아치 리눅스 빌드 시스템(ABS) 정보를 동기화합니다.

    $ sudo abs

그러면 `/var/abs` 디렉토리 밑에 모든 공식 패키지의 빌드 정보가 다운로드됩니다.

라이브러리의 패키지 빌드 옵션을 수정하기 위해, `/etc/makepkg.conf` 파일에서 아래 부분을 찾아 디버그 심볼(`-g`)과 프레임 포인터 포함(`-fno-omit-frame-pointer`) 컴파일 옵션을 추가하고 빌드 옵션에서 `strip`을 제외합니다.

    CFLAGS="-g -fno-omit-frame-pointer -march=x86-64 -mtune=generic -O2 -pipe"
    CXXFLAGS="-g -fno-omit-frame-pointer -march=x86-64 -mtune=generic -O2 -pipe"
    OPTIONS=(!strip docs libtool emptydirs zipman purge)

`/var/abs/local` 디렉토리로 이동해서(없으면 새로 생성) 다음과 같이 사용되는 프로그램에 사용되는 모든 라이브러리 패키지를 다시 생성하고 설치합니다. 예를 들어 클러터 라이브러리는 다음과 같습니다.

    $ src=$(find /var/abs -name clutter | grep -v /var/abs/local)
    $ cp -r $src /var/abs/local
    $ cd /var/abs/local/clutter
    $ makepkg -f
    $ sudo pacman -U *.pkg.tar.xz

위와 같은 방식으로 clutter, cogl, glib2, glibc 패키지를 다시 만들고 설치합니다.

**메모리 프로파일링**

이제 다음 명령으로 디버깅할 프로그램(`eview-demo`)을 실행합니다.

    $ G_SLICE=always-malloc 
      HEAPPROFILE=/tmp/profile 
      HEAP_PROFILE_ALLOCATION_INTERVAL=10737418240 
      LD_PRELOAD=/usr/lib/libtcmalloc.so 
      ./eview-demo
    Starting tracking the heap
    Dumping heap profile to /tmp/profile.0001.heap (...)
    Dumping heap profile to /tmp/profile.0002.heap (...)
    Dumping heap profile to /tmp/profile.0003.heap (...)
    Dumping heap profile to /tmp/profile.0004.heap (...)

정상적으로 구글 성능 도구의 메모리 프로파일러가 동작하면 위와 같은 메시지가 출력됩니다. 이제 적당한 시점에서 프로그램을 멈추고, 다음과 같이 프로파일링 데이터를 분석합니다.

    $ pprof 
        --pdf 
        --lines 
        --base /tmp/profile.0001.heap 
        ./eview-demo 
        /tmp/profile.0004.heap 
        > profile-1.pdf

이렇게 생성된 그래프는 다음과 같습니다.

![](/figures/clutter-1-6-memory-leak-profile.jpg)

이 그래프를 분석해서 관련 코드를 분석해 보니, 결정적으로 두 군데에 문제가 있습니다. 첫번째는 `cogl_pipeline_fragend_arbfp_start()` 함수 내부에서 생성한 `arbfp_program_state` 객체를 해제하는 곳이 없다는 점이고, 두번째는 `cogl_pipeline_get_layers()` 함수에서 생성한 `deprecated_get_layers_list` 리스트를 해제하는 곳이 없다는 점입니다. 그런데 최근 클러터 1.8 버전 소스를 보면 두번째 문제는 해결이 된 것 같은데, 첫번째 문제가 있는 곳은 코드 수정이 많이 되어 해결 여부를 알 수가 없습니다.

그래서 결론은, 며칠 전에 릴리스된 클러터 1.8 안정 버전이 아치 리눅스 패키지로 올라오면 다시 메모리 누수 여부를 확인해볼 예정입니다. (GNOME 3 핵심 라이브러리를 직접 컴파일해서 사용하는게 귀찮기도 하고 두렵기도 해서입니다... :)

**[UPDATE 2011-10-04]** 클러터 1.8 버전에서 확인해 보니 메모리 누수 문제가 해결된 것 같습니다. 역시, 미루기를 잘 했습니다. ;)

**[UPDATE 2011-10-05]** 다시 확인해 보니, 이제는 다른 부분에서 메모리 누수가 발생합니다. 그래서 이번에는 당당히(?) 버그 리포팅([Bug 660985](https://bugzilla.gnome.org/show_bug.cgi?id=660985), [Bug 660986](https://bugzilla.gnome.org/show_bug.cgi?id=660986)) 했습니다.

**[UPDATE 2011-10-10]** CPU 사용량이 가장 많은 함수를 프로파일링하려면 다음과 같이 실행하면 됩니다.

    $ CPUPROFILE=./cpu.prof 
      LD_PRELOAD=/usr/lib/libtcmalloc_and_profiler.so 
      ./eview-demo

정상적으로 종료한뒤 다음과 같이 CPU 사용량을 함수별로 프로파일한 그래프를 얻을 수 있습니다.

    $ pprof 
        --pdf 
        --lines 
        ./eview-demo 
        ./cpu.prof
        > profile-1.pdf

**[UPDATE 2012-02-04]** 최신 아치 리눅스에 포함되어 있는 [glibc 2.15 버전의 버그](http://code.google.com/p/gperftools/issues/detail?id=396) 때문에 프로파일링이 제대로 동작하지 않을 경우 ~~[sscanf 관련 패치](http://permalink.gmane.org/gmane.comp.lib.glibc.alpha/17093)를 적용해 glibc 패키지를 다시 빌드하고 설치해야 합니다.~~ glibc 2.15-5 버전으로 업그레이드하면 됩니다. [버그 리포팅](https://bugs.archlinux.org/task/28246)이 바로 반영되어 버렸습니다. :)
