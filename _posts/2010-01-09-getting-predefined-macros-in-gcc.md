---
layout: post
title: GCC 미리 정의된 매크로 얻기
tags: [ GCC,  iPhone,  Linux,  MacOSX,  Windows ]
---

멀티플랫폼에서 동작하는 C/C++ 코드를 gcc를 이용해 컴파일할때 플랫폼이나 운영체제를 확인하는 방법 중 하나는 gcc 툴체인이 만들어질때 정의되는 매크로를 사용하는 것입니다. 그런데 이번에 MacOS X / iPhone 플랫폼에 기존 코드를 포팅하면서 이 방법을 이용하려 하는데, 너무 오래 전에 했던 작업이라 (역시나) 명령어를 기억할 수 없었습니다. 그래서 겨우 구글링해서 다시 알게된 내용을 기록해 둡니다.

    $ gcc -E -dM -x c /dev/null

그리고 이 방법을 이용해 사용한 최종 코드는 다음과 같습니다.

    #if defined(_WIN32)
    #include "lib-win32/config.h"
    #elif defined(_WIN64)
    #include "lib-win64/config.h"
    #elif defined(__ENVIRONMENT_IPHONE_OS_VERSION_MIN_REQUIRED__)
    #include "lib-iphone/config.h"
    #elif defined(__ENVIRONMENT_MAC_OS_X_VERSION_MIN_REQUIRED__) /* or __APPLE__ */
    #include "lib-macosx/config.h"
    #else /* linux */
    #include "config.h"
    #endif

사실은, 더 깔끔한 다른 방법이 있을지 궁금하기도 합니다.
