---
layout: post
title: GObject - Glib object system
tags: [ GLib,  GTK+ ]
---

오랜만에 Glib 객체 시스템인 GObject에 대한 글들을 다시 정독해 보았다. 아무 것도 모르고 처음 읽었을때와 몇년동안 GTK+와 친숙해진뒤 다시 읽어볼때는 역시 차이가 있는 법, 이제 GObject 객체를 프로젝트에 하나씩 적용해볼 생각이다.

참고 사이트:

1.  [GObject 공식 리퍼런스와 튜토리얼](http://developer.gnome.org/doc/API/2.0/gobject/index.html)
2.  [GNOME 개발자 설명서 모음](http://developer.gnome.org/doc/tutorials/)
3.  [위키피디아에 있는 간략한 GObject 소개](http://en.wikipedia.org/wiki/Gobject)
4.  [GObject Tutorial Copyright Ryan McDougall (2004)](http://docs.programmers.ch/index.php/HOWTO_gobject) - C 언어로 객체지향을 구현하기 위해 GObject를 어떤 이유와 근거로 설계했는지를 단계별로 하나씩 설명해주는 튜토리얼
5.  [The Glib Object system](http://le-hacker.org/papers/gobject/) - 공식(?) GObject 튜토리얼
6.  [GOB(GObject Builder)](http://www.jirka.org/gob.html) - 쉽게 GObject C 코드를 생성해주는 전처리기
7.  [Beginning GTK+ Programming](http://hellocity.net/%7Eiolo/files/gnome/BeginningGTK2Programming/BeginningGTK2Programming.pdf) - 훌륭한 GTK+, GObject 프로그래밍 관련 컨퍼런스 자료

