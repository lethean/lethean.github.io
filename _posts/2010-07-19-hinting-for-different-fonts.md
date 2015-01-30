---
layout: post
title: "글꼴마다 다른 힌팅 사용하기"
tags: [ Cairo,  FontConfig,  Ubuntu ]
---

구글 리더를 읽다가 어떤 분이 [터미널 글꼴로 'Droid Sans Mono' 사용한 포스트](http://blog.digital-scurf.org/2010/07/13#dev-fonts)를 보고 따라해 보았습니다. 그런데, 이상하게도 글꼴이 예쁘지 않아서 확인해 보니 폰트 설정에서 힌팅(hinting)을 살짝(slight)만 사용하도록 설정해야 했습니다. 사실, 대부분의 영문 폰트는 힌팅을 살짝 주어야 원래 의도대로 멋지게 표시되는 경우가 많습니다. (아래 그림에서 폰트 설정 화면 참조)

![](/figures/gnome-font-properties.png)

그런데 문제는, 이 설정을 이용하면 나눔글꼴과 같은 일부 한글 글꼴이 오히려 안 이쁘게 표시됩니다. 특히 나눔고딕은 힌팅을 중간(hintmedium)이나 충분히(hintfull) 사용해야 합니다. 그런데 위 그림을 보면 나눔고딕 역시 정상적으로 표시되고 있습니다. 이 포스트는 그 과정을 정리한 것입니다. 사용환경은 우분투 10.04 배포판이고, [나눔글꼴](http://packages.debian.org/sid/ttf-nanum)과 [나눔고딕코딩](http://packages.debian.org/sid/ttf-nanum-coding) 글꼴은 데비안 패키지를 직접 내려받아 설치했습니다. 우분투 저장소에도 조만간 반영되겠지요.

먼저 폰트별로 다른 힌팅 스타일을 사용하기 위해 폰트 설정에서, 나눔글꼴 계열 힌팅 스타일을 충분히(hintfull)로 변경합니다. `~/.fonts.conf` 파일을 편집해도 되지만, 제 경우 그냥 ﻿﻿`﻿/etc/fonts/conf.avail/69-language-selector-ko-kr.conf` 파일에 다음 내용을 추가했습니다. 그래야 루트 사용자를 포함한 모든 사용자가 사용할 수 있고, 다른 설정 항목도 모두 거기 있어서 나중에 다시 설치할때 파일 하나만 복사해서 사용하기 때문입니다.

    <match target="font">
      <test name="family">
        <string>나눔고딕</string>
        <string>NanumGothic</string>
        <string>나눔명조</string>
        <string>NanumMyeongjo</string>
        <string>나눔고딕코딩</string>
        <string>NanumGothicCoding</string>
      </test>
      <edit name="hintstyle" mode="assign">
        <const>hintfull</const>
      </edit>
    </match>

그런데 문제는, 이 설정이 먹혀들지 않는다는 점입니다. 이상하게도 파이어폭스나 크롬브라우저, KDE 등에서는 잘 적용되는데, 정작 그놈 터미널이나 모든 그놈 프로그램에서는 적용이 되지 않았습니다. 검색해보니, GTK+ 툴킷을 포함한 대부분 그놈 프로그램이 사용하는 카이로(cairo) 라이브러리에 [관련 버그](http://bugs.freedesktop.org/show_bug.cgi?id=11838)가 이미 보고되어 있는데 아직 패키가 반영되지 않은 상태였습니다. 카이로 라이브러리가 힌팅의 경우 FontConfig 설정을 따르지 않고 무조건 그놈 글꼴 설정, 더 정확히 말하면 X 리소스의 Xft.hintstyle 값만 사용하기 때문이었습니다.

그래서 다음과 같은 과정을 거쳐 직접 빌드한 패키지를 설치해서 사용하고 있습니다.

```sh
$ apt-get source libcairo2 # 패키지 소스 내려받기
$ sudo apt-get build-dep libcairo2 # 빌드를 위한 패키지 내려받기
$ cd cairo-*
$ vi src/cairo-ft-font.c # 위 버그질라에 등록된 패치 적용
$ vi debian/changelog # 패키지 버전 올림
$ dpkg-buildpackage -rfakeroot # 패키지 생성
$ cd ..; sudo dpkg -i libcairo*.deb # 패키지 설치
```

패키지 생성 방법을 익힌게 거의 10년 전 쯤 데비안 사용 시절이라 요즘은 어떻게 만드는지 잘 모르겠지만, 다행히도 위 방식도 제대로 동작하는 것 같아 그냥 사용하고 있습니다. 요즘은 우분투 PPA도 활성화되었고, 빌드 방식도 더 간단해진 것 같긴 한데, 게을러서... :)

결론은, Droid Sans Mono 글꼴을 터미널 글꼴로 잘 사용하고 있습니다. 참, 이 글꼴 역시 다음과 같이 쉽게 설치할 수 있습니다.

```sh
$ sudo apt-get install ttf-droid
```

그런데, 위 패치가 아직까지도 최근 cairo 소스에는 반영되지 않은 것 같아 약간 아쉽군요.
