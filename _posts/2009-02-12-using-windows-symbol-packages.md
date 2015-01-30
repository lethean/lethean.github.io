---
layout: post
title: "윈도우 심볼 패키지 사용하기"
tags: [ Coding,  Windows ]
---

윈도우 비주얼 스튜디오 환경에서 디버깅을 하다보면 시스템에서 기본으로 제공하는 DLL 라이브러리에 대한 디버깅 심볼 정보가 없어 불편한 경우가 많습니다. 이 문제를 해결하려면 DLL 라이브러리에 대한 디버깅 심볼을 설치하면 됩니다.

[이 사이트](http://www.microsoft.com/whdc/DevTools/Debugging/debugstart.mspx)에서 자신의 운영체제와 서비스 팩에 맞는 윈도우 심볼 패키지(Windows Symbol Packages)를 다운로드해서 설치한 뒤 '호출 스택(Call Stack)'을 확인해 보면, 아래 그림과 같이, 전과 다르게 ntdll.dll, kernel32.dll, user32.dll 등의 함수 이름이 표시됩니다.

![](/figures/call-stacks-with-symbols.png)

참고로, 위 사이트에 보면 비주얼 스튜디오를 설치하지 않아도(되는지 확인은 안해보았지만) 실행 중인 프로그램을 디버깅할 수 있도록 도와주는 WinDbg 디버깅 도구도 다운로드할 수 있습니다.
