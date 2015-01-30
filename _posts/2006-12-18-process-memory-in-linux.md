---
layout: post
title: Process Memory in Linux
tags: [ Linux ]
---

리눅스에서 일반적으로 `top`, `ps` 명령어를 통해 얻을 수 있는 프로세스의 메모리 정보는 다음과 같이 요약할 수 있다.

-   <span style="font-weight:bold;">SIZE(VSZ, VIRT)</span> : 프로세스의 주소 공간 크기. 즉, 프로세스가 맵핑한 메모리의 전체 크기를 나타낸다.
-   <span style="font-weight:bold;">RSS(RES, Resident Set Size)</span> : 실제 메모리에 올라와 있는 메모리 크기. 물론 스왑(swapped out)된 메모리는 제외된다. RSS는 프로세스에게 유일한 메모리 공간과 다른 프로세스와 공유된 공간도 포함한다. 리눅스 환경에서는 대부분 공유 라이브러리가 차지한다. libc 라이브러리가 대표적이다.
-   <span style="font-weight:bold;">SHARE(SHR)</span> : RSS에서 다른 프로세스가 공유된 메모리 크기.

따라서 어플리케이션이 실제 사용하고 있는 메모리는 RSS에서 SHARE를 뺀 크기다. 물론 공유 라이브러리를 하나의 프로세스가 사용하고 있다면 그 크기도 실제 사용량에 포함되겠지만.

참고:

1.  [Memory Usage with smaps](http://bmaurer.blogspot.com/2006/03/memory-usage-with-smaps.html)
2.  [Re: Can anything be done to reduce memory usage?](http://mail.gnome.org/archives/gnome-list/1999-September/msg00036.html)

