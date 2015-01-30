---
layout: post
title: I810 + Compiz in Ubuntu
tags: [ Ubuntu,  Xorg ]
---

많은 사람들이 3D 데스크탑 효과를 얻기 위해 [Beryl 프로젝트](http://www.beryl-project.org/) 결과물을 이용하는데, 내 경우 Beryl 보다 심플한 설정 인터페이스를 지닌 오리지널 [Compiz 윈도우 매니저](http://www.go-compiz.org/)를 더 선호한다. 특히 노트북처럼 1.4Hz CPU지만 전원절약을 위해 대부분 시간을 600MHz로 동작하고, 비디오카드도 인텔 계열(I810)이라서, 무분별하게(?) 화려한 Beryl 프로젝트보다 Compiz 프로젝트가 더 적합한 것 같다.

회사에서 작업용으로 사용한 장비의 경우도 NVidia 비디오카드이고 CPU도 넉넉하지만 굳이 Compiz를 사용하는데, Beryl을 이용하면 발생하는 가끔 윈도우가 까맣게 보이는 현상이 Compiz의 경우 발생하지 않기 때문이다.

우분투의 경우 XGL을 설치하지 않아도 '[Ubuntu Compiz Repository](http://gandalfn.wordpress.com/compiz-repository/)' 페이지의 설명을 따라 몇 개 패키지만 설치하면 자동으로 설치한다. 특히 여기서 <span style="font-weight:bold;">개발 버전(development branch)</span>를 설치하면 Beryl 프로젝트에 있는 대부분의 플러그인도 비슷하게 사용할 수 있다.
