---
layout: post
title: linux-gate.so.1
tags: [ glibc,  Linux ]
---

데비안 패키지 glibc-2.3.5 이후부터 `ldd` 명령의 결과에 예전에 없던 'linux-gate.so.1' 공유 라이브러리가 포함되어 있어 궁금했다. 다음 예에서 첫번째 줄이다.

    # ldd /bin/sh
    linux-gate.so.1 =>  (0xffffe000)
    libdl.so.2 => /lib/libdl.so.2 (0xb7fb2000)
    libc.so.6 => /lib/libc.so.6 (0xb7e7c000)
    /lib/ld-linux.so.2 (0xb7fba000)

Google에서 찾아본 결과 "[What is linux-gate.so.1?](http://www.trilithium.com/johan/2005/08/linux-gate/)" 문서가 가장 잘 설명해주고 있었다. 2.5 커널부터 시스템콜 호출을 전통적인 `'int 0x80'`이 아닌 인텔 펜티엄 II 이후에 추가된 `sysenter` 인스트럭션을 이용하여 오버헤드를 줄이고, 64비트 머신과의 일관성을 유지하기 위해 고안된 방법이었다. 이 공유라이브러리는 커널이 마치 DSO처럼 제공하는 시스템콜 라이브러리인 셈이다. 그래서 파일시스템에는 실제 이 파일이 없다. '*linux-gate*' 라는 이름은 커널과 사용자 프로세스간의 통로라는 뜻에서 그렇게 붙인 것 같다.

몇년 동안 2.6 커널을 사용해왔는데 왜 모르고 있었을까. 이유를 조금 더 조사해 보니, 다음에 보는 것처럼 glibc 버전과 상관없이 커널은 이미 지원하고 있었다. 다음은 glibc-2.3.5가 아니고 커널만 2.6인 시스템에서 출력한 결과다. 여기서 가장 마지막의 '`[vdso]`' 부분이 바로 그것이다.

    [lethean@lethean ~]$ cat /proc/self/maps
    08048000-0804c000 r-xp 00000000 03:01 758909     /bin/cat
    0804c000-0804d000 rw-p 00003000 03:01 758909     /bin/cat
    0804d000-0806e000 rw-p 0804d000 00:00 0          [heap]
    ...
    b7f23000-b7f38000 r-xp 00000000 03:01 734436     /lib/ld-2.3.6.so
    b7f38000-b7f39000 rw-p 00014000 03:01 734436     /lib/ld-2.3.6.so
    bf923000-bf938000 rw-p bf923000 00:00 0          [stack]
    ffffe000-fffff000 ---p 00000000 00:00 0          [vdso]

단지 glibc-2.3.5 버전에 포함된 `ldd`가 이를 'linux-gate.so.1' 이라는 표시하기 시작한 것이다.
