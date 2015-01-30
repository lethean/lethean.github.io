---
layout: post
title: 'reactable : 테이블 표면 실감형 다중 접촉 인터페이스를 통한 협업 전자 음악 악기'
tags: [ GUI ]
---

어설픈 영한번역을 거치게 된 [reactable](http://mtg.upf.edu/reactable/) 프로젝트의 영어 설명은 이렇다.

> The *reactable* is a **collaborative electronic music instrument with a tabletop tangible multi-touch interface**. Several simultaneous performers share complete control over the instrument by moving and rotating physical objects on a luminous round table surface. By moving and relating these objects, representing components of a classic modular synthesizer, users can create complex and dynamic sonic topologies, with generators, filters and modulators, in a kind of tangible modular synthesizer or graspable flow-controlled programming language.

여러 연주자가 함께 야광의 둥근 테이블 위에 있는 물리적인 물체를 움직이고 돌리면서 악기를 완전하게 제어한다. 이처럼 고전적인 모듈러 신디사이저 요소를 나타내는 물체를 움직이고 연관을 지으면서, 사용자는 복합적이고 생동적인 음파 형상을 만들 수 있으며, 실감형 모듈러 신디사이저나 잡을 수 있는 흐름 제어 프로그래밍 언어의 한 종류인 발진기, 필터, 변조기를 이용한다.

기본적인 원리는 투명한 유리판(?) 밑에서 카메라를 이용해 사용자 반응을 감지하고 처리하며, 프로젝터를 이용해 다시 표시한다.

![](/figures/reactivision03.png)

또 눈여겨볼 만한 건 [대부분 소프트웨어 소스가 공개](http://mtg.upf.edu/reactable/?software)되어 있으며 많은 관련 논문과 자료도 잘 정리되어 있다는 점이다. 특히 Windows, MacOSX, Linux를 동시에 지원하는 크로스 플랫폼 비디오 캡쳐 라이브러리인 [PortVideo](http://www.iua.upf.es/mtg/reacTable/?portvideo) 라이브러리는 눈여겨볼만 하다. ([PortAudio](http://www.portaudio.com/) 프로젝트 이름을 흉내낸 걸까?)

물론 많은 [데모 동영상과 그림](http://www.iua.upf.es/mtg/reacTable/?media)도 볼 수 있다.
