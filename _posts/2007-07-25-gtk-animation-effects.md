---
layout: post
title: GTK+ Animation Effects
tags: [ Clutter,  GTK+,  GUI ]
---

점점 화려해지는 GUI 추세를 이제서야 인식했는지, GTK+ / GNOME 에서도 애니메이션 효과에 대한 논의와 구현이 점점 활발해지고 있는 것 같다. 아직 GTK+ 메인 소스에 반영되려면 시간이 더 걸릴 것 같지만 [GtkTimeline](http://bugzilla.gnome.org/show_bug.cgi?id=444659) 이라는 기본적인 시간 관리 객체가 이미 논의 중이고, 이를 기반으로 여러 개발자들이 여기저기에 적용해보기도 하고 있다. ([GtkPathBar 스크롤 효과](http://blogs.gnome.org/carlosg/2007/06/06/animateinanimate/), [iPhone 방식 슬라이드 효과](http://micke.hallendal.net/archives/2007/07/bling_in_gtk.html), [iPhone 방식 가상키보드](http://codeposts.blogspot.com/2007/07/iphone-like-virtual-keyboard.html))

GtkTimeline API가 참고한 소스 중 하나라고 하는 [Clutter](http://clutter-project.org/) 라이브러리는 OpenGL을 렌더링 엔진으로 사용하는데, GObject 기반으로 GTK+와 친근한 방식의 API를 제공하여 이미 여러 프로젝트에서 사용하고 있는 것 같다. [Clutter를 이용한 간단한 프리젠테이션 도구](http://butterfeet.org/?p=38), [휴대폰 인터페이스](http://butterfeet.org/?p=39) 등과 같은 예제도 점점 늘어나고 있다. 하지만 OpenGL 기반이라 임베디드 시스템이나 그래픽칩셋이 3D 가속을 지원하지 않는 환경에서는 활용하기 어렵다는 점이 아쉽다. 아직[OpenGL API로 모든 2D 그래픽을 대체하는 건 시기상조일 수도 있다는 얘기](http://blogs.gnome.org/timj/2007/07/17/17072007-opengl-for-gdkgtk/)도 심심챦게 나오고 있고...

참고로 더 나열해보자면, QT 계열에서는 이미 [코딩하면서 바로 캔버스 API를 테스트할 수 있는 스크립트 엔진](http://zrusin.blogspot.com/2007/07/scripter.html)도 지원한다. [E17](http://www.enlightenment.org/)의 [EFL 라이브러리](http://www.enlightenment.org/p.php?p=about/libs&l=en)는 이미 오래전부터 Evas, Edje 등으로 조금 앞선 플랫폼을 제공하더니 이제는 [EFL 기반 솔루션을 제공하는 업체](http://www.fluffyspider.com/demos/live_videos/live_videos.html)도 생겨났다. 최근의 [미지리서치 Prizm 플랫폼](http://www.mizi.com/content/view/4/5/)도 휴대폰에서 화려한 UI 효과를 쉽게 구현하도록 도와주고 있다.(glib 기반이라 반가웠다)

물론 좋은 오픈소스 라이브러리와 플랫폼이 참 많지만, 이미 오랜 시간을 GTK+와 함께 해왔더니 쉽게 다른 플랫폼으로 바꾸기가 어려운 것 같다. 하지만, 언제나 그렇듯이 잘 설계되고 잘 구현된 오픈 소스 프로젝트를 들여다보는 일은 즐겁다. 가능하면 참여중인 프로젝트에 적용해보고도 싶고, 더 나아가 프로젝트에 직접 참여해보기도 싶지만, 언제나 그렇듯이...
