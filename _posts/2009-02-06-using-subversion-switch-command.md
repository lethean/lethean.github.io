---
layout: post
title: "서브버전 스위치 명령어 사용하기"
tags: [ Subversion ]
---

서브버전(Subversion) 기반으로 개발할때 브랜치(branches) 작업은 빈번하게 발생합니다. 서브버전은 브랜치 작업을 효율적으로 수행할 수 있도록 스위치(switch) 명령어를 지원하는데, 생각보다 이 명령어를 활용하는 사람이 별로 없는 것 같아 간단하게 사용법을 소개하려 합니다. 여기서 소개하는 방법은 커맨드 라인 명령어를 사용하고 있지만 TortoiseSVN 등과 같은 GUI 프로그램을 이용해도 동일한 작업을 처리할 수 있습니다.

**현재 작업 중인 소스 트리를 그대로 이용하기**

디스크 공간이 부족하거나 다른 브랜치 작업 내용을 잠시만 확인하고 수정할 필요가 있을 경우, 굳이 다른 브랜치를 새로 체크아웃(checkout) 할 필요 없이 현재 소스 트리를 변경만 하면 됩니다. 특히 방대한 전체 소스가 아닌 특정 디렉토리만 해당 브랜치의 디렉토리로 변경하면 시간도 절약됩니다.

예를 들어 '`$SVN/dooly/trunk`' 소스 트리에서 작업하다가 '`$SVN/dooly/branches/cms-2.1-0-remote`' 브랜치 소스를 작업하고 싶은데, 변경할 부분이 '`$SVN/dooly/branches/cms-2.1-0-remote/cms/src`' 디렉토리 뿐이라면 다음과 같이 작업하면 됩니다. (현재 디렉토리가 소스 시작 디렉토리라고 가정합니다)

    cd cms/src
    svn switch $SVN/dooly/branches/cms-2.1-0-remote/cms/src

이렇게 하면 다른 디렉토리는 그대로 '`trunk`'를 이용하고 '`cms/src`' 디렉토리만 '`cms-2.1-0-remote`' 브랜치 소스로 변경됩니다. 이후 이 디렉토리에 작업하는 내용을 커밋하면 모두 해당 브랜치로 커밋됩니다. 물론 다시 원래 소스 트리로 되돌아오고 싶은 경우 다음과 같이 작업하면 됩니다.

    cd cms/src
    svn switch $SVN/dooly/trunk

물론 혼란을 막기 위해 가능한 수정 중인 파일이 없는 게 좋겠지요?

**새로운 브랜치 빨리 내려 받기**

한 번 빌드하는데 많은 시간이 걸리는 소스 트리를 새로 받는 작업은 지루하고 재미없습니다. 만일 브랜치 작업 내용이 소스 디렉토리 전체가 아닌 특정 디렉토리에서만 이루어진다면 다음과 같이 빨리 새로운 브랜치를 구성할 수 있습니다. (현재 디렉토리 밑에 '`dooly-trunk`' 소스 트리가 이미 존재한다고 가정합니다)

    cp -a dooly-trunk dooly-remote
    cd dooly-remote/cms/src
    svn switch $SVN/dooly/branches/cms-2.1-0-remote/cms/src

이렇게 하면 쉽고 빠르게 새로운 브랜치에 대한 소스 트리가 준비됩니다. 물론 특정 디렉토리가 아닌 전체 디렉토리에 걸쳐 있는 작업이라도 다음과 같이 하면 새로운 소스를 내려받아 다시 모든 라이브러리를 빌드하는 수고는 줄어듭니다.

    cp -a dooly-trunk dooly-remote
    cd dooly-remote
    svn switch $SVN/dooly/branches/cms-2.1-0-remote

처음에도 언급한 것처럼 위의 모든 작업은 어느 운영체제 하에 어떤 서브버전 도구를 이용해도 동일합니다.
