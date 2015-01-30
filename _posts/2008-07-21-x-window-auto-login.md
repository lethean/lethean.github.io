---
layout: post
title: X 윈도우 자동 로그인하기
tags: [ Linux,  Ubuntu,  Xorg ]
---

GDM 등과 같이 덩치가 큰 디스플레이 관리자를 사용하지 않고, 시스템 부팅후 자동으로 X 서버를 실행하고 특정 계정으로 로그인한뒤 자동으로 특정 프로그램을 실행하는 기능은 의외로 많이 사용합니다. 이 글에서는 여러가지 방법 중에 제가 알고 있는 몇가지 방법을 정리해 보았습니다. 적용 가능한 배포판은 데비안(Debian) 혹은 우분투(Ubuntu) 리눅스 기반입니다.

### 첫번째 방법 - init 데몬 이용하기

"[How to autologin X without a display manager](http://www.enricozini.org/2008/tips/lightweight-autologin.html)" 글에서 설명하는 방법입니다.

먼저 init 데몬이 자동으로 실행할 수 있도록 `/etc/inittab` 파일에 다음 항목을 추가합니다.

    6:23:respawn:/sbin/getty -L -n -l /usr/local/sbin/autologin

위 항목은 시스템 시작시 자동으로 `/usr/local/sbin/autologin` 프로그램을 실행합니다. 또한 프로그램이 종료해도 다시 자동으로 재시작합니다. 이제 사용자 로그인 과정을 자동으로 수행하도록 하려면 `/usr/local/sbin/autologin` 프로그램을 다음과 같이 작성합니다.

    #!/bin/sh
    /bin/login -f root

여기서 `-f` 뒤에 로그인할 계정을 적어줍니다. 이제 계정 홈 디렉토리에 있는 셸 스크립트 시작 파일(`~/.bash_profile`)을 수정해서 마지막에 다음 항목을 넣어줍니다.

    startx
    logout

이 스크립트는 X 서버를 시작하고 종료시 자동으로 로그아웃을 합니다. 마지막으로 X 서버가 실행하면서 자동으로 수행될 스크립트를 만들어야 합니다. 계정 홈 디렉토리에 있는 X 서버 시작 파일(`~/.xsession` 또는 `~/.xinitrc`)을 다음과 같이 작성합니다.

    #!/bin/sh
    my-window-manager &

    # If the touch screen is not calibrated, run the calibration
    while [ ! -f /etc/touchscreen-calibration ]
    do
      calibrate-touchscreen
    done

    # Run the main application: if it ends, the session ends
    main-application

제일 먼저 창 관리자(여기서는 'my-window-manager')를 백그라운드로 실행합니다. 그리고 필요한 선행작업(여기서는 'calibrate-touchscreen')을 처리한 뒤 실제 어플리케이션(main-application)을 실행합니다.

### 두번째 방법 - upstart 데몬 이용하기

우분투 리눅스는 init 데몬 대신 [Upstart](http://upstart.ubuntu.com/) 데몬을 이용하여 시스템 초기화 작업을 처리합니다. 따라서 첫번째 방법에서 `/etc/inittab` 파일을 수정하는 대신 `/etc/event.d/` 디렉토리에 시작 파일을 등록해야 합니다. 예를 들면 `/etc/event.d/autostart` 파일을 다음과 같이 작성합니다.

    start on runlevel 2
    start on runlevel 6
    respawn
    exec /sbin/getty -L -n -l /usr/local/sbin/autologin

위 내용은 런레벨 2,6 에서 해당 프로그램을 실행하고 종료시 자동으로 재시작하도록 합니다. 나머지는 첫번째 방법과 동일합니다.

### 세번째 방법 - 런레벨(run-level) 이용하기

init 데몬이든 Upstart 데몬이든 상관없이 동작하는 방법입니다. 먼저 다음과 같은 스크립트를 `/etc/init.d/autologin` 파일로 만들어 줍니다.

    #!/bin/sh
    /usr/local/sbin/my-startx &
    exit 0

그리고 런레벨 2로 동작한다는 가정하에 스크립트가 자동 실행할 수 있도록 다음 명령을 실행합니다.

    # chmod +x /etc/init.d/autologin
    # update-rc.d autologin defaults 05

여기서 마지막 '05'는 런레벨에서 다른 데몬보다 먼저 실행하도록 결정해주는 우선순위입니다. 이제 `/usr/local/sbin/my-startx` 스크립트를 작성합니다.

    #!/bin/sh
    while true; do
      sleep 1
      echo "xinit /root/.xinitrc -- /etc/X11/xinit/xserverrc" 
      | su - root
    done

이 스크립트는 루트(root) 계정으로 X를 시작하면서 /root/.xinitrc 파일을 시작 스크립트 파일로 지정합니다. 따라서, 이 방법은 위 두가지와 다르게 사용자 셸(bash)을 거치지 않고 직접 X 서버를 실행합니다. 그리고, 다른 방법과 마찬가지로 종료시 자동으로 X를 재시작합니다.  X 실행 이후 시작하는 스크립트는 다른 방법과 동일합니다.
