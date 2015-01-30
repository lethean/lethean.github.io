---
layout: post
title: GCC Symbol Visibility Patch
tags: [ GCC ]
---

GCC 4.0부터 포함되어 있는 [GCC Symbol Visibility Patch](http://www.nedprod.com/programs/gccvisibility.html)는 GCC 컴파일러를 조금 더 유용하게 만들어 준다. 홈페이지에서 언급한 이 기능의 장점은 다음과 같다.

1.  공유 라이브러리와 같은 DSO(Dynamic Shared Object, 동적 공유 객체)를 로드하는데 걸리는 시간을 획기적으로 향상시켜준다.
2.  컴파일러 최적화기(optimiser)가 더 좋은 코드를 만들수 있다.
3.  DSO 크기를 5~20% 정도 줄여준다.
4.  심볼(symbol)이 충돌할 기회를 낮춰준다.

사용법은 다음과 같다. 물론 GCC 4.x 버전이 필요하다.

모듈에서 제공하는 인터페이스나 API를 정의하는 헤더파일에서 외부로 공개되는 모든 구조체, 클래스, 함수 선언 앞에 `__attribute__ ((visibility("default")))` 키워드를 붙여준다. (매크로를 사용하면 편리하다) 이렇게 한다음 Makefile과 같은 빌드 환경에서 컴파일 옵션에 `-fvisibility=hidden` 인수를 추가하면 된다. 이렇게 생성된 DSO 파일과 원래 방식으로 생성한 파일을 `nm -C -D `명령으로 확인해 보면 차이를 금방 알 수 있다. 다음은 윈도우즈 환경과 호환되는 예제 매크로 파일이다.

    #ifdef _MSC_VER
    #ifdef BUILDING_DLL
    #define DLLEXPORT __declspec(dllexport)
    #else
    #define DLLEXPORT __declspec(dllimport)
    #endif
    #define DLLLOCAL
    #else
    #ifdef HAVE_GCCVISIBILITYPATCH
    #define DLLEXPORT __attribute__ ((visibility("default")))
    #define DLLLOCAL __attribute__ ((visibility("hidden")))
    #else
    #define DLLEXPORT
    #define DLLLOCAL
    #endif
    #endif

    extern "C" DLLEXPORT void function(int a);

    class DLLEXPORT SomeClass{
      int c;
      DLLLOCAL void privateMethod();  // Only for use within this
      DSOpublic:Person(int _c) : c(_c) { }
      static void foo(int a);
    };

실제로 [TnFOX](http://www.nedprod.com/TnFOX/index.html) 프로젝트에서 사용하는 헤더파일을 참고해도 도움이 될 것이다.

    // Shared library support
    #ifdef WIN32
    #define FXIMPORT __declspec(dllimport)
    #define FXEXPORT __declspec(dllexport)
    #define FXDLLLOCAL
    #define FXDLLPUBLIC
    #else
    #define FXIMPORT
    #ifdef GCC_HASCLASSVISIBILITY
    #define FXEXPORT __attribute__ ((visibility("default")))
    #define FXDLLLOCAL __attribute__ ((visibility("hidden")))
    #define FXDLLPUBLIC __attribute__ ((visibility("default")))
    #else
    #define FXEXPORT
    #define FXDLLLOCAL
    #define FXDLLPUBLIC
    #endif
    #endif

    // Define FXAPI for DLL builds
    #ifdef FOXDLL
    #ifdef FOXDLL_EXPORTS
    #define FXAPI FXEXPORT
    #else
    #define FXAPI  FXIMPORT
    #endif // FOXDLL_EXPORTS
    #else
    #define FXAPI
    #endif // FOXDLL

    // Throwable classes must always be visible on GCC in all binaries
    #ifdef WIN32
    #define FXEXCEPTIONAPI(api) api
    #elif defined(GCC_HASCLASSVISIBILITY)
    #define FXEXCEPTIONAPI(api) FXEXPORT
    #else
    #define FXEXCEPTIONAPI(api)
    #endif
