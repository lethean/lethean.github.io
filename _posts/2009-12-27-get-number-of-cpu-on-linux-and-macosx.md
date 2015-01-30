---
layout: post
title: "리눅스 / 맥오에스에서 CPU 갯수 얻기"
tags: [ glibc,  Linux,  MacOSX ]
---

Linux / MacOS X 플랫폼에서 CPU 갯수를 얻어오는 함수입니다. 잊어버릴까봐 기록해 둡니다.

    #include <stdio.h>
    #ifdef __linux__
    #include <sys/sysinfo.h>
    #else
    #ifdef __APPLE__
    #include <sys/param.h>
    #include <sys/sysctl.h>
    #else
    #error "Only Linux or OSX is supported!"
    #endif /* __APPLE__ */
    #endif /* __linux__ */

    static int
    get_cpu_nr (void)
    {
    #ifdef __linux__
      return get_nprocs ();
    #endif
    #ifdef __APPLE__
      int i = 0;
      size_t s = sizeof (i);
      if (sysctlbyname ("hw.ncpu", &i, &s, NULL, 0))
        return 1;
      return i;
    #endif
    }

    int
    main (int argc, char **argv)
    {
      printf ("cpu_nr:%dn", get_cpu_nr ());

      return 0;
    }

출처는 [이 소스](http://github.com/chergert/dukes_of_hazard/raw/master/main.c)입니다. 참고로 리눅스의 `/usr/include/sys/sysinfo.h` 파일을 들여다보면 램 크기 / 스왑 크기 / 물리 페이지 크기 등과 같은 시스템 관련 정보를 얻을 수 있는 함수가 여럿 보이는군요. `/proc` 정보를 파싱하는 것보다 더 편리할 것 같습니다.
