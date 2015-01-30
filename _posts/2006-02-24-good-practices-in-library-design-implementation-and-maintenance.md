---
layout: post
title: "라이브러리 설계, 구현, 유지에 좋은 습관"
tags: [ Coding,  glibc ]
---

[glibc(GNU libc)](http://www.gnu.org/software/libc/) 관리자이며 개발자인 [Ulrich Drepper](http://people.redhat.com/drepper/)의 글 중에 '[Good Practices in Library Design, Implementation, and Maintenance'](http://people.redhat.com/drepper/goodpractice.pdf) 는 비단 라이브러리 개발 뿐 아니라 일반 프로젝트를 진행할때도 유용한 여러 가이드라인을 제시한다. 간략하게 정리해보면 다음과 같다.

<span style="font-weight:bold;">인터페이스 설계하기</span>

1.  가능한 API에 변수를 포함하지 말라. 대신 내부 변수를 처리하는 get-/set-함수를 구현하라.
2.  라이브러리가 제공하는 모든 인터페이스, 변수, 함수, 자료구조에 접두사(prefix)를 붙여라. 모든 객체에 반드시 같은 접두사를 사용할 필요는 없으며, 한 라이브러리가 여러 접두사를 사용해도 된다.
3.  C/C++ 라이브러리 설치시 함께 제공하는 헤더파일은 인터페이스를 정의하는데 필요한 정의(definition)와 선언(declaration)만 포함해야 한다.
4.  사용자가 직접 객체를 할당하지 않는 불완전한 형식(incomplete type)을 사용한다면 선정의(forward declaration)를 이용하는 것이 맞다.
5.  컴파일 설정(configuration)이 변경되더라도 라이브러리 헤더 파일은 변하면 안된다.
6.  어쩔 수 없이 불완전하게 정의한 데이터 타입을 제공해야 한다면, 나중에 커질 부분을 고려하여 최소한의 패딩을 만들어야 한다.

<span style="font-weight:bold;">라이브러리 구현하기</span>

1.  가능한 많은 함수와 변수를 'static'을 이용하여 오브젝트 파일에 지역적으로(local) 정의하라.
2.  외부로 보여지는(export) 심볼은 최대한 줄여라. 가장 좋은 경우는 문서화된 인터페이스만 보여지는 것이다.

<span style="font-weight:bold;">라이브러리 유지하기</span>

1.  이전 버전에 없었던 새로운 인터페이스는 따로 표시해야 새로운 인터페이스를 사용하는 어플리케이션이 아예 동작하지 못하도록 할 수 있다.
2.  오류를 수정하는게 아닌 다른 이유로 인터페이스가 변경되더라도 이전 인터페이스는 그대로 존재해야 한다.
3.  문서화된 라이브러리 인터페이스의 모든 면은 문서화해야 한다. 인터페이스가 변경되어야 한다면, 이전 동작이 그대로 유지된다는 것을 보장하기 위해 최소한 새 테스트가 추가되어야 한다.

키워드만 정리한 것이지만, 10쪽 분량밖에 안되는 글이므로 가능한 원본을 읽어보는 것이 좋을 것이다.
