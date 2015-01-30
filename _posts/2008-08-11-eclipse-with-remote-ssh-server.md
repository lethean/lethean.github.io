---
layout: post
title: "이클립스(Eclipse) + 원격 SSH 서버 작업하기"
tags: [ Eclipse,  Linux ]
---

프로젝트를 빌드할때마다 매번 ssh 프로그램으로 로그인해서 emacs / vi 등의 에디터로 컴파일해서 다시 이를 타겟 장비에 scp 명령어로 복사하고... 조금 복잡하고 번거롭죠. 또한 개발환경으로 터미널 에디터 밖에 사용이 불가능합니다. 이 글은 이러한 개발 환경을 조금 탈피해서, 실제 소스 파일은 빌드 호스트에 두고 빌드 작업도 빌드 호스트에서 실행하면서, 이클립스(Eclipse) 개발 환경을 이용해 개발하는 방법을 간단하게 설명합니다.

이 글은 개인 장비에 우분투 리눅스 + Eclipse 개발 환경(3.4 Ganymede 기준)이 구축되어 있다는 가정하에 설명합니다. 원격 빌드 호스트 장비(build-dvr24)는 SSH 서버가 설치되어 있고 개인 계정도 이미 만들어져 있다고 가정합니다.(여기서는 lethean)

### SSH 로그인 비밀번호 안물어보게 하기

SSH 로그인시 비밀번호 확인 과정을 넘어가기 위해 개인공개키를 빌드 호스트에 복사합니다. 그러면 이후 모든 SSH 작업시 비밀번호를 물어보지 않게 되어 편리합니다. 만일 개인공개키가 만들어져 있지 않다면 다음과 같이 생성합니다.

    $ ssh-keygen -t dsa
    Enter file in which to save the key (/home/lethean/.ssh/id_dsa): [Enter]
    Enter passphrase (empty for no passphrase): [Enter]
    Enter same passphrase again: [Enter]

이 경우 개인 공개키 파일은 `~/.ssh/id_dsa.pub` 파일입니다. 이 파일을 원격 빌드 호스트 계정의 인증키 목록에 다음과 깉이 추가합니다.

    $ ssh-copy-id -i ~/.ssh/id_dsa.pub lethean@build-dvr24

여기서 'lethean@build-dvr24' 부분은 '접속계정@호스트이름' 형식입니다.

### 원격 파일시스템 연결하기

제일 먼저 마운트할 디렉토리를 미리 생성합니다.

    $ mkdir -p ~/build-dvr24

터미널에서 다음과 같이 'sshfs' 프로그램을 설치합니다.

    $ sudo apt-get install sshfs

프로그램을 설치한 뒤 sshfs / fusermount 명령어를 이용해 원격 SSH 서버의 디렉토리를 로컬 파일 시스템에 연결하거나 해제할 수 있습니다. 예를 들어 연결(mount)하려는 원격 디렉토리가 '/home/lethean'이고, 로컬 홈 디렉토리 밑의 'build-dvr24' 디렉토리에 연결할 경우 다음과 같이 실행합니다. (원격 디렉토리는 절대경로 방식으로 지정해야 하며 반드시 홈디렉토리일 필요는 없습니다)

    $ sshfs lethean@build-dvr24:/home/lethean ~/build-dvr24

사용이 다 끝났으면 다음과 같이 연결을 해제할 수 있습니다

    $ fusermount -u ~/build-dvr24

이를 부팅시 자동으로 연결하는 방법은 여러가지 방법이 있지만, 쉬운 방법 중 하나는 '/etc/rc.local' 파일에 다음과 같은 내용을 마지막 'exit 0' 전에 추가하는 것입니다.

    su lethean -c 'sshfs lethean@build-dvr24:/home/lethean ~/build-dvr24'
    exit 0

여기까지 하면 원격 파일 시스템을 마치 로컬 파일 시스템처럼 사용이 가능하므로 이클립스 뿐 아니라 VI, Emacs 등의 에디터를 이용해 쉽게 편집이 가능합니다.

([/etc/fstab 파일을 수정하는 방법](http://fuse.sourceforge.net/wiki/index.php/SshfsFaq#Exporting_via_NFS)도 있는데 조금 복잡하군요. 관심이 있으시다면 직접 해보시기 바랍니다)

### 이클립스에서 빌드 명령어 실행하기

먼저 연결(mount)한 소스를 기반으로 새로운 프로젝트를 생성해야 합니다. C 언어 기반 프로젝트일 경우를 가정할때 순서는 다음과 같습니다.

1.  'File -\> New -\> C Project...'를 선택하여 새로운 프로젝트를 시작합니다.
2.  프로젝트 이름(Project name)을 입력합니다.
3.  '기본 위치 사용(Use default location)' 선택을 해제한 뒤,
4.  위치(Location)를 직접 선택하여(Browse...) '~/build-dvr24' 디렉토리 밑의 해당 소스 디렉토리를 지정합니다.
5.  프로젝트 종류(Project types)는 'Makefile project' / 'Linux GCC'를 선택합니다.
6.  그리고 언어 설정 등을 선택한 뒤 마침(Finish) 버튼을 눌러 새로운 프로젝트를 생성합니다.

이제 프로젝트 탐색기(Project Explorer)에서 생성한 프로젝트를 선택하고, 마우스 오른쪽 버튼을 눌러 'Properties' 메뉴 항목을 선택합니다. 이제 다음 순서대로 빌드 명령어를 변경합니다.

1.  'C/C++ Build' 항목을 선택합니다.
2.  '기본 빌드 명령어 사용(Use default build command)' 선택을 해제한 뒤,
3.  빌드 명령어(Build command)를 다음과 같이 입력합니다.
     `ssh lethean@build-dvr24 'make -C project-dir'`

여기서 'project-dir'은 홈디렉토리 기준 원격 디렉토리를 의미합니다. 파일 시스템이 연결된 로컬 파일 시스템과는 상관없이 ssh 로 직접 연결해서 해당 디렉토리를 빌드하도록 하는게 이 방법의 핵심입니다. 물론 이 방법은 이클립스 뿐 아니라 다른 개발 환경에서도 응용이 가능합니다.
