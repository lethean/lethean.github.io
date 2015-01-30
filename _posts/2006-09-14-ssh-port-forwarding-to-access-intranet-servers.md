---
layout: post
title: SSH 포트 포워딩(port forwarding) + 내부 네트웍 서버 접근하기
tags: [ Linux,  Network ]
---

외부에서 접속할 수 있는 서버가 master.emstone.com 이고, 접근하려고 하는 내부 네트웍 서버가 svn.emstone.com(192.168.0.143), 서비스가 Subversion 이라고 하면 다음과 같은 작업을 통해 외부에서 직접 접근이 가능하다.

1.  우선 master.emstone.com 서버에 SSH 접속이 가능한 계정이 있어야 한다. (여기서는 lethean)
2.  현재 컴퓨터의 /etc/hosts 파일을 다음 내용을 추가한다.

        127.0.0.1 svn svn.emstone.com

3.  내부 네트웍 서버 IP가 192.168.0.143 일 경우 다음과 같은 옵션으로 master.emstone.com 서버에 접속한다.

        ssh lethean@master.emstone.com 
            -L 3690:192.168.0.143:3690

4.  Subversion 작업을 하면 동작한다.

원리는 이렇다. SSH 접속시 '-L' 옵션은, 로컬 장비(localhost:3690)의 특정 포트에 대한 연결을 접속한 서버(master.emstone.com)에서 접근 가능한 특정 IP 포트(192.168.0.143:3690)로 무조건 송신하고 그 결과를 다시 돌려받는다. 따라서 실제로는 로컬호스트에 대해 작업을 하는 것처럼 보이지만 SSH 연결을 통해 원격으로 동작하는 셈이다.

SSH 연결을 끊으면 당연히 더이상 동작하지 않게 된다.
