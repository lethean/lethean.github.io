---
layout: post
title: ThinkPad X40 + Ubuntu + Power Saving
tags: [ Linux,  Ubuntu ]
---

[PowerTop](http://www.linuxpowertop.org/) 유틸리티를 실행하면 전원을 절약할 수 있는 여러가지 방법도 친절하게 설명해주는데, 이 정보를 바탕으로 ThinkPad X40 노트북에 맞게 켤때마다 자동으로 설정하는 스크립트를 만들어봤다. Ubuntu Gutsy 배포판에서 사용하면 약간 과장해서 50% 이상 배터리 수명이 연장되는 걸 체감할 수 있다.

    #!/bin/sh
    #
    # Power Save Tunings for ThinkPad X40
    #

    # enable wireless power saving mode
    iwpriv eth1 set_power 5

    # enable AC97 powersave mode
    echo 1 > /sys/module/snd_ac97_codec/parameters/power_save

    # enable USB autosuspend
    echo 1 > /sys/module/usbcore/parameters/autosuspend
    for dev in /sys/bus/usb/devices/*; do
      file=$dev/power/autosuspend
      [ -f $file ] && echo 1 > $file
    done

    # increase the VM dirty writeback time from 5.00 to 15 seconds
    echo 1500 > /proc/sys/vm/dirty_writeback_centisecs

    # enable laptop-mode
    echo 5 > /proc/sys/vm/laptop_mode

이 스크립트를 자동으로 실행하게 하는 방법은 다음과 같다. 먼저 다음과 같이 에디터를 열어 위 내용을 입력한다.

    # sudo gedit /etc/init.d/thinkpad-x40-powersave

실행권한을 준다.

    # sudo chmod +x /etc/init.d/thinkpad-x40-powersave

부팅시 자동으로 실행하도록 한다.

    # sudo update-rc.d thinkpad-x40-powersave defaults

바로 적용하려면 `sudo /etc/init.d/thinkpad-x40-powersave`와 같이 직접 실행해도 된다.
