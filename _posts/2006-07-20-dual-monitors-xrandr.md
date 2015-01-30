---
layout: post
title: Dual Monitors +XRandR
tags: [ Xorg ]
---

프로젝트를 위해 새로운 모니터 2대로 교체하면서 듀얼 모니터를 사용하기 위해 비디오카드도 함께 교체했다. (LG Flatron L2012P x 2 + NVidia Geforce 7300 GT) Ubuntu 6.06 LTS 버전을 사용하고 있는데, 우분투 위키에 있는대로 nvidia-glx 패키지를 설치와 몇몇 설정을 마친 뒤 X 서버 설정까지 튜닝한 이후 다음과 같이 동작하는 환경을 얻게 되었다.

[![screenshot-dual\_monitor-2400x1600](/figures/193829822_f8fa5aee5e_b.jpg)](http://www.flickr.com/photos/72033444@N00/193829822/ "Photo Sharing")

[![photo-dual\_monitor-2400x1600](/figures/193839516_d49be3cb3c_b.jpg)](http://www.flickr.com/photos/72033444@N00/193839516/ "Photo Sharing")

원래는 각각 1600x1200 해상도로 셋팅한 것이지만, 이를 세로로 돌려(모니터가 돌아간다!) 길게 옆으로 붙인 것이다.

이에 대한 X 설정은 아래와 같다.

    Section "Files"
       FontPath    "/home/lethean/.fonts"
       FontPath    "/usr/share/X11/fonts/misc"
       FontPath    "/usr/share/X11/fonts/cyrillic"
       FontPath    "/usr/share/X11/fonts/100dpi/:unscaled"
       FontPath    "/usr/share/X11/fonts/75dpi/:unscaled"
       FontPath    "/usr/share/X11/fonts/Type1"
       FontPath    "/usr/share/X11/fonts/100dpi"
       FontPath    "/usr/share/X11/fonts/75dpi"
    EndSection

    Section "Module"
       Load    "bitmap"
       Load    "dbe"
       Load    "ddc"
       Load    "extmod"
       Load    "freetype"
       Load    "glx"
       Load    "int10"
       Load    "type1"
       Load    "vbe"
    EndSection

    Section "InputDevice"
       Identifier    "Generic Keyboard"
       Driver        "kbd"
       Option        "CoreKeyboard"
       Option        "XkbRules"    "xorg"
       Option        "XkbModel"    "pc104"
       Option        "XkbLayout"    "us"
    EndSection

    Section "InputDevice"
       Identifier    "Configured Mouse"
       Driver        "mouse"
       Option        "CorePointer"
       Option        "Device"        "/dev/input/mice"
       Option        "Protocol"        "ExplorerPS/2"
       Option        "ZAxisMapping"        "4 5"
    EndSection

    Section "Device"
       Identifier    "NVIDIA Corporation NVIDIA Default Card"
       Driver        "nvidia"
       Option        "NoLogo"    "true"
       BusID        "PCI:1:0:0"

       Option        "RandRRotation" "true"
       Option        "Rotate"    "left"
       Option        "UseEdidFreqs"    "true"

       # TwinView
       Option        "TwinView"    "true"
       Option        "SecondMonitorHorizSync"    "28-83"
       Option        "SecondMointorVertRefresh"    "56-85"
    #    Option        "TwinViewOrientation"        "RightOf"
       Option        "TwinViewOrientation"        "Above"
       Option        "MetaModes" "1600x1200,1600x1200"
    #    Option        "ConnectedMonitor"        "CRT, DFP"
    EndSection

    Section "Monitor"
       Identifier    "L2012P"
       Option        "DPMS"
       HorizSync    28-83
       VertRefresh    56-85
    EndSection

    Section "Screen"
       Identifier    "Default Screen"
       Device        "NVIDIA Corporation NVIDIA Default Card"
       Monitor        "L2012P"
       DefaultDepth    24
       SubSection "Display"
           Depth        16
           Modes        "1600x1200"
       EndSubSection
       SubSection "Display"
           Depth        24
           Modes        "1600x1200"
       EndSubSection
    EndSection

    Section "ServerLayout"
       Identifier    "Default Layout"
       Screen        "Default Screen"
       InputDevice    "Generic Keyboard"
       InputDevice    "Configured Mouse"
    EndSection

    Section "DRI"
       Mode    0666
    EndSection
