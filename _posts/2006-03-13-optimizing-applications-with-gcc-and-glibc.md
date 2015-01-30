---
layout: post
title: GCC와 GLIBC로 최적화하기
tags: [ GCC,  glibc ]
---

gcc와 glibc로 어플리케이션 최적화하기 ([Optimizing Applications with gcc & glibc](http://people.redhat.com/drepper/optimtut1.ps.gz))

glibc 개발자인 [Ulrich Drepper](http://people.redhat.com/drepper/)가 1999년에 작성한 40페이지 분량의 글이다. 대략 관심있는 내용을 정리해본다.

<span style="font-weight:bold;">
 사용 안되는 코드 컴파일시 제거 </span>

다음 코드는 주어진 타입에 따라 다른 작업을 하는데, `int`형과 `long int`형이 같은 플랫폼일 경우 좋지 않은 성능을 보인다.

    long intadd (long int a, void *ptr, int type)
    {
      if (type == 0)
        return a + *(int *)ptr;
      else
        return a + *(long int *)ptr;
    }

위 코드는 다음과 같이 최적화할 수 있다. 이는 컴파일러가 알아서 최적화해준다.

    long intadd (long int a, void *ptr, int type)
    {
      if (sizeof(int) == sizeof(long int) || type == 0)
        return a + *(int *)ptr;
      else
        return a + *(long int *)ptr;
    }

다음과 같은 방법을 이용하면 전처리기(preprocessor)가 알아서 최적화해준다.

    long intadd (long int a, void *ptr, int type)
    {
    #if LONG_MAX != INT_MAX
      if (type == 0)
        return a + *(int *)ptr;
      else
    #endif
      return a + *(long int *)ptr;
    }

<span style="font-weight:bold;">
 컴파일러 내부 함수(intrinsics) 이용하기</span>

gcc 2.96 이후부터 많은 `__builtin_*` 함수가 존재한다. 이 함수들은 컴파일시에 컴파일러가 판단하여 인수가 상수이거나 정해진 크기 등일 경우 최적화된 코드를 생성해 준다. 이 함수들은 매크로인 경우도 있고 컴파일시 정적으로 링크되는 라이브러리일 수도 있다. 어떤 함수들은 컴파일러 내부 함수를 이용해 최적화되어 있다. 따라서 가능한 이러한 함수들은 별도 랩핑을 두지 않고 헤더파일을 포함하여 그대로 사용하면 더 좋은 성능의 코드를 얻을 수 있다. 내부 함수를 이용하는 함수 중에 대표적인 것들은 다음과 같은 것들이 있다.

    alloca()
    memcpy(), memcmp(), memset()
    strcmp(), strcpy(), strlen()

더 많은 함수는 원문을 참고하면 된다.

<span style="font-weight:bold;">
 strcpy()와 memcpy()</span>

문자열의 크기를 이미 알고 있다면 `str*()` 류의 함수보다 `mem*()` 류의 함수를 이용하는 것이 좋다. 우선 매 바이트마다 문자열 끝 코드인 0x00을 검사하는 코드가 없어 빠르고, 무엇보다도 `memcpy()`는 플랫폼 기본 단위(예를 들어 32비트 플랫폼에서는 4바이트) 연산을 하기 때문에, 항상 바이트 단위로 작업하는 `strcpy()` 또는 `strncpy()`보다 빠를 수 밖에 없다.

<span style="font-weight:bold;">
 strcat(), strncat()</span>

절대로 `strcat()`, `strncat()` 함수는 사용하지 말아야 한다. 문자열을 합치는데 이 함수를 이용하는 것은 성능에 치명적이다. 내부적으로 `strlen()`을 호출하는 것 뿐 아니라 문자열을 처리하기 위한 여러 조건을 검사하기 때문에 매우 느리다. 이보다는 직접 `strlen()`으로 문자열 길이를 구한 뒤 `memcpy()`를 이용해 해당 위치에 복사하거나 필요한 작업을 하는 것이 더 현명한 방법이다.

다음은 이에 대한 샘플 코드이다.

    {
      char *buf = ...;
      size_t bufmax = ...;

      /* Add 's' to the string in buffer 'buf'. */
      if (strlen(buf) + strlen(s) + 1 > bufmax)
        buf = (char *)realloc(buf, (bufmax *= 2));
      strcat(buf, s);
    }

프로그래머 관점에서 이 코드는 괜찮아 보이지만, 성능은 최악이다. 다음과 같이 고치면 더 좋은 성능을 낼 수 있다.

    {
      char *buf = ...;
      size_t bufmax = ...;
      size_t slen, buflen;

      /* Add 's' to the string in buffer 'buf'. */
      slen = strlen(s) + 1;
      buflen = strlen(buf);

      if (buflen + slen > bufmax)
        buf = (char *)realloc(buf, (bufmax *= 2));

      memcpy(buf + buflen, s, slen);
    }

<span style="font-weight:bold;">
 메모리 할당 최적화</span>

`malloc()`과 `calloc()`의 차이점을 모르는 프로그래머는 없겠지만, 그래서 `calloc()`보다 직접 `malloc()` 호출 이후 `memset()` 등을 이용해 0으로 초기화하는 방법을 많이 이용하기도 한다. 하지만 `calloc()`은 커널 페이지에서 이미 0으로 초기화되어 있는 영역이 있을 경우 불필요한 초기화 과정을 건너뛰기 때문에 더 좋은 성능을 낼 수 있다.

`alloca()` 함수를 모르는 사람도 많은데, 이 함수는 지정한 크기의 영역을 현재 스택 영역에 할당한다. 메모리 관리자를 거치지 않고 단순히 스택 포인터 레지스터 조작만으로 할당 작업이 이루어지기 때문에 매우 성능이 좋을 뿐 아니라, 따로 해제하지 않아도 함수가 끝나는 시점에서 자동으로 해제된다. 따라서 함수 내부에서 임시로 할당하여 사용하는 많은 구현에서 `malloc()` 보다 `alloca()` 를 사용한 코드의 성능은 매우 향상된다. glibc에서 확장으로 제공하는 `strdupa()`, `strndupa()` 등의 API는 모두 이 방법을 이용한 최적화된 성능을 보장한다.

<span style="font-weight:bold;">그외...</span>

기타 이 글에서는 gprof, sprof 등을 이용한 코드 프로파일링 방법과 GCC 컴파일러 확장을 이용한 다양한 최적화 방법을 제시하고 있다. 더불어 저자는 같은 내용에 조금 더 내용이 보강된 [Application Optimization on Linux](http://people.redhat.com/drepper/optimtut2.ps.gz) 글에서 프로필과 관련된 더 자세한 정보를 제공하고 있다.

이 글에서도 언급한 내용이지만, 많은 프로그래머는 자신이 작성한 프로그램이 정말 잘 쓰여지고 성능이 좋다고 생각하지만, 불행하게도 아닌 경우가 대부분이다. 최적화된 프로그램을 작성하는 것은 항상 진행중인 배우는 과정(learning process)이다. 항상 새로운 테크닉을 배우고, 자신의 코드를 검토하고, 상호작용하는 라이브러리와 프로세서에 대해 생각해야 한다.
