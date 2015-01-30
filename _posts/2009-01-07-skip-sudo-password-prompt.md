---
layout: post
title: sudo 비밀번호 안물어보게 하기
tags: [ Linux,  Ubuntu ]
---

sudo를 이용해 루트 권한으로 프로그램을 실행하거나, 패키지 관리자처럼 루트 권한이 필요한 GUI 프로그램을 실행할때마다 비밀번호 물어보는게 귀찮다면 다음과 같이 안물어보게 할 수 있습니다.

먼저 /etc/sudoers 파일을 편집하게 위해 visudo 명령을 실행합니다.

    $ sudo visudo

맨 밑으로 가면 기본적으로 admin 그룹에 대한 설정이 있는데, 이 행을 고치거나 자신의 계정에 관한 설정을 새로 추가하면서 'NOPASSWD:' 항목을 명시하고 편집을 끝냅니다.  (여기서 계정은 lethean)

    %admin ALL=(ALL) ALL
    lethean ALL=(ALL) NOPASSWD: ALL

이후 모든 sudo 명령 실행시 더 이상 비밀번호를 물어보지 않습니다.

물론, 자리를 비웠을때 누군가 내 계정으로 루트 권한 작업을 마음대로 할 수 있는 보안상의 위험을 상관할 필요가 없을 경우에만 사용하시기 바랍니다.
