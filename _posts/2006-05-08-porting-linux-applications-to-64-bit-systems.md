---
layout: post
title: Porting Linux applications to 64-bit systems
tags: [ Linux ]
---

[Porting Linux applications to 64-bit systems](http://www-128.ibm.com/developerworks/linux/library/l-port64.html) 를 보면 64비트 환경에서 기존 리눅스 어플리케이션을 동작시키기 위해 C/C++ 프로그래머가 알아야 할 내용을 정리해주고 있다. 다음은 그 중에서 일부분을 정리한 내용이다.

<span style="font-weight:bold;">64비트 표준</span>

리눅스는 <span style="font-weight:bold;">LP64</span> 표준을 따른다. <span style="font-style:italic;">LP64</span>는 long / long long / pointer 형만 64비트이고 나머지는 32비트 환경과 동일한다. 참고로 <span style="font-style:italic;">LLP64</span>는 long long / pointer 형만 64비트, <span style="font-style:italic;">ILP64</span>는 int / long / long long / pointer 가 모두 64비트인 환경을 의미한다. 따라서 기존 32비트 프로그램을 포팅할때 가장 유의할 점은 long 형으로 선언된 자료구조에 대한 처리이다.

<span style="font-weight:bold;">자료구조 정렬(align) 방식</span>

64비트형(long / double / pointer 등)은 항상 64비트 경계에 정렬된다. 나머지 공간은 모두 패딩된다. 따라서 자료구조 정의시 64비트형일 경우 적절하게 배치하는 것이 좋다.

<span style="font-weight:bold;">변수/상수 정의</span>

32/64비트 환경에서 32비트 크기를 원하면 int 형을 사용한다. 32비트 환경에서는 32비트, 64비트 환경에서는 64비트로 동작하는 크기를 원하면 long 형을 사용한다. 숫자를 다룰때 바이트를 아끼기 위해 char, short 등을 성능에 그리 좋지 않다.

32비트 환경에서 모든 비트를 켜기 위해 대개 0xFFFFFFFFL 상수가 사용된다. 하지만 64비트 시스템에서 이 값은 하위 32비트만 켜진 0x00000000FFFFFFFF 값을 의미한다. 따라서 이식성이 좋게 만드려면 다음과 같이 사용하는 것이 좋다.

    long x = -1L;

또다른 예로 들 수 있는 것은, 최상위 비트를 켤때 많이 사용하는 0x80000000 값이다. 이 값은 다음과 같이 사용하는 것이 좋다.

    1L << ((sizeof(long) * 8) - 1);

쉬프트(shift) 연산에서 'L'을 붙여주지 않으면 기본 int(32비트) 형으로 인식하여 원하는 결과를 얻을 수 없음에 주의해야 한다.

<span style="font-weight:bold;">타입 정의</span>

가능한 표준 C 라이브러리에 정의된 이식성있는 형을 사용하는 경우가 좋다.

-   ptrdiff\_t : 두 포인터의 차이를 저장할 수 있는 부호있는 정수형이다.
-   size\_t : 부호없는 정수형으로 sizeof 연산자의 결과 뿐 아니라 많은 표준 C 라이브러리가 크기를 표현하는데 사용한다.
-   int32\_t, uint32\_t, ... : 정확한 크기의 정수형을 정의하는데 사용한다.
-   intptr\_t, uintptr\_t : 정수형으로 변환해도 위험하지 않은 포인터를 담기위한 정수형이다.

<span style="font-weight:bold;">포맷팅하기</span>

32비트 시스템에서 printf() 함수의 "%d"는 int / long 형 모두 적절하게 동작하지만, 64비트 시스템에서 long을 인수로 넘기면 하위 32비트만 출력된다. 따라서 long 형을 출력할 경우 정확하게 "%ld" 라고 적어주는 것이 좋다. 비슷하게 포인터를 출력하기 위해 많이 사용하는 "%x" 대신 "%p" 를 사용하면 더 정확한 포인터 값을 출력할 수 있다.
