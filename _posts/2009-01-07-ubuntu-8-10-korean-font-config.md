---
layout: post
title: Ubuntu 한글 글꼴 설정
tags: [ Hangul,  Ubuntu ]
---

Ubuntu 설치시 한국어를 선택하면 한글을 사용하는데 별 문제가 없는 것 같지만, 글꼴이나 입력기 등이 조금 문제가 있기 때문에 '시스템' -\> '관리' -\> '언어' 설정에서 '지원되는 언어'에 '한국어'를 선택하고, 기본 언어로 '한국어'를 선택해주면 한글 관련 패키지와 설정이 자동으로 설치됩니다.

하지만 이렇게 해도 기본 한글 글꼴 설정이 별로 미려하지 않은데, 제 경우 기본 글꼴 설정을 다음과 같이 변경해서 사용하고 있습니다.

먼저 `/etc/fonts/conf.d/29-language-selector-ko-kr.conf` 파일은 다음과 같습니다.

    <fontconfig>

    <!-- Turn on antialias and hinting with hintmedium for ttf-Unfonts -->
    <match target="font">
            <test name="family" compare="contains">
                    <string>Un</string>
            </test>
            <edit name="antialias" mode="assign">
            <bool>true</bool>
        </edit>
            <edit name="hinting" mode="assign">
            <bool>true</bool>
        </edit>
        <edit name="hintsytle" mode="assign">
            <const>hintmedium</const>
        </edit>
    </match>

    <!-- Turn off antialias and autohint for ttf-alee depending on pixelsize -->
    <match target="font">
            <test name="family">
                    <string>Guseul</string>
            </test>
            <edit name="autohint" mode="assign">
            <bool>true</bool>
        </edit>
    </match>
    <match target="font">
            <test name="family">
                    <string>Guseul</string>
                    <string>Guseul Mono</string>
            </test>
        <test name="pixelsize" compare="more">
            <int>11</int>
        </test>
        <test name="pixelsize" compare="less">
            <int>16</int>
        </test>
        <edit name="antialias" mode="assign">
            <bool>false</bool>
        </edit>
            <edit name="autohint" mode="assign">
            <bool>false</bool>
        </edit>
    </match>

    </fontconfig>

두번째로 `/etc/fonts/conf.d/69-language-selector-ko-kr.conf` 파일은 다음과 같습니다.

    <fontconfig>

    <!-- Set preferred Korean fonts -->
        <match target="pattern">
            <test qual="any" name="family">
                <string>serif</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
                <string>DejaVu Serif</string>
                <string>NanumMyeongjo</string>
                <string>UnBatang</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
                <string>sans-serif</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
                <string>DejaVu Sans</string>
                <string>NanumGothic</string>
                <string>UnDotum</string>
                <string>Guseul</string>
            </edit>
        </match>
        <match target="pattern">
            <test qual="any" name="family">
                <string>monospace</string>
            </test>
            <edit name="family" mode="prepend" binding="strong">
                <string>DejaVu Sans Mono</string>
                <string>Guseul</string>
                <string>UnDotum</string>
            </edit>
        </match>

    <!-- Bind EunGuseul Mono with Bitstream Vera Sans Mono -->
    <match target="pattern">
        <test name="family">
            <string>Guseul</string>
        </test>
        <edit mode="append" binding="strong" name="family">
            <string>DejaVu Sans Mono</string>
        </edit>
    </match> 

    </fontconfig>

위에서는 기본 글꼴로 영문은 DejaVu Sans 글꼴 계열, 한국어는 네이버 나눔 글꼴 계열을 사용하고 있습니다. 참고로, Adobe 플래시에서 한글이 깨지지 않게 하려면 한글 글꼴을 영문 글꼴보다 위에 두면 됩니다.
