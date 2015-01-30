---
layout: post
title: "우분투 10.10 한글 글꼴 설정"
tags: [ FontConfig,  Hangul,  Ubuntu ]
---

Btrfs 파일 시스템을 사용해 보려고 새로 나온 우분투 10.10 버전을 설치했습니다. 그런데, 아니나 다를까 한글 글꼴 설정은 여전히 맘에 들지 않는군요. 예전에는 이것 저것 쉽게 찾아 고쳤는데, 시간이 점점 흘러서 그 과정을 자꾸 잊어버리게 되다 보니 오늘은 조금 더 많이 헤매게 되어 기록해 두려고 합니다.

먼저 메인 글꼴로 사용하는 나눔글꼴은 우분투 저장소에 패키지(ttf-nanum, ttf-nanum-coding)가 이미 올라와 있어서 그대로 설치해서 사용했습니다. 더불어 기본적인 MS 글꼴 패키지(ttf-mscorefonts-installer)를 설치하고, 터미널 글꼴로 사용하는 드로이드 글꼴(ttf-droid)을 설치하고 [cairo 라이브러리 패치 작업을 한 뒤](/2010/07/19/hinting-for-different-fonts/) '**시스템-기본 설정-모양**'은 다음 그림과 같이 설정했습니다.

![](/figures/gnome-appearance-properties.jpg)

이제 글꼴 설정 파일을 건드려야 하는데, 제일  먼저 `/etc/fonts/conf.d/` 디렉토리에서 <span style="text-decoration:line-through;">`10-hinting-slight.conf` 파일과</span> `29-language-selector-ko-kr.conf` 파일을 삭제합니다. 그래야 일반적인 영문 / 한글 글꼴에 대한 힌팅이 예쁘게 동작합니다. 그 다음에 같은 디렉토리의 `69-language-selector-ko-kr.conf` 파일을 다음과 같이 수정합니다.

    <?xml version="1.0"?>
    <!DOCTYPE fontconfig SYSTEM "fonts.dtd">
    <fontconfig>

    <!-- Set preferred Korean fonts -->
    <match target="pattern">
      <test qual="any" name="family">
        <string>sans-serif</string>
      </test>
      <edit name="family" mode="prepend" binding="strong">
        <string>DejaVu Sans</string>
        <string>나눔고딕</string>
        <string>UnDotum</string>
      </edit>
    </match>

    <match target="pattern">
      <test qual="any" name="family">
        <string>serif</string>
      </test>
      <edit name="family" mode="prepend" binding="strong">
        <string>DejaVu Serif</string>
        <string>나눔명조</string>
        <string>UnBatang</string>
      </edit>
    </match>

    <match target="pattern">
      <test qual="any" name="family">
        <string>monospace</string>
      </test>
      <edit name="family" mode="prepend" binding="strong">
        <string>Droid Sans Mono</string>
        <string>DejaVu Sans Mono</string>
        <string>나눔고딕코딩</string>
        <string>Guseul</string>
        <string>UnDotum</string>
      </edit>
    </match>

    <match target="font">
       <test name="family">
        <string>나눔고딕</string>
        <string>NanumGothic</string>
        <string>나눔고딕코딩</string>
        <string>NanumGothicCoding</string>
        <string>맑은 고딕</string>
        <string>Malgun Gothic</string>
        <string>UnDotum</string>
        <string>UnBatang</string>
      </test>
      <edit name="hintstyle" mode="assign">
        <const>hintmedium</const>
      </edit>
    </match>

    <match target="font">
      <test name="family">
        <string>나눔명조</string>
        <string>NanumMyeongjo</string>
      </test>
      <edit name="hintstyle" mode="assign">
        <const>hintslight</const>
      </edit>
    </match>

    <match target="font">
      <test name="family">
        <string>DejaVu Sans Mono</string>
        <string>Droid Sans Mono</string>
      </test>
      <edit name="hintstyle" mode="assign">
        <const>hintslight</const>
      </edit>
    </match>

    <match target="font">
     <test name="family">
     <string>Andale Mono</string>
     <string>Arial Black</string>
     <string>Arial</string>
     <string>Comic Sans MS</string>
     <string>Courier New</string>
     <string>Georgia</string>
     <string>Impact</string>
     <string>Tahoma</string>
     <string>Times New Roman</string>
     <string>Trebuchet MS</string>
     <string>Verdana</string>
     <string>Webdings</string>
     </test>
     <edit name="hintstyle" mode="assign">
     <const>hintmedium</const>
     </edit>
    </match>

    </fontconfig>

뭐, ﻿대충 이 정도만 설정해도 깔끔한 모양의 글꼴을 볼 수 있습니다.

사족) Btrfs 파일 시스템은 많은 디스크 I/O가 동시에 걸리면 시스템 전체가 느려지는 듯한 느낌이 여전히 듭니다. 뭐, 앞으로 조금씩 더 좋아지겠지요.

**[UPDATE-2010.12.23]** 이 포스트를 작성하는 시점에서는 아직 Ubuntu 글꼴이 패포판에 포함되지 않았던 시점이라서 이를 사용한 fontconfig 설정으로 업데이트 했습니다.

**[UPDATE-2011.01.04]** 한동안 크로미엄 브라우저만 사용하다가 최근 다시 파이어폭스를 사용하다보니 MS 글꼴이 이쁘게 나오지 않는 걸 확인하고 이를 반영했습니다.
