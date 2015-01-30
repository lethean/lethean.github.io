---
layout: post
title: "우분투에서 루트 파일시스템 마운트 옵션 변경하기"
tags: [ Linux,  Ubuntu ]
---

커널트랩에 올라온 [마운트 옵션을 이용한 성능 최적화](http://kerneltrap.org/node/14148) 글을 보고 우분투 리눅스에도 적용시켜보기로 했다. 다른 파티션은 모두 `/etc/fstab` 파일에서 직접 'noatime,data=writeback' 옵션만 추가하면 되는데 루트 파일 시스템은 조금 손질이 더 갔다.

데비안 기반 시스템은 처음에는 루트 파일 시스템을 읽기전용(read-only)으로 마운트한 뒤 initrd 기반 초기화 과정을 수행하고, 나중에 다시 `/etc/fstab` 정보를 기반으로 루트파일 시스템을 다시 정상적인 쓰기 가능하도록 마운트한다.(remount) 그런데 이때 'noatime' 등과 같은 옵션은 정상적으로 동작하지만 'data=writeback' 등과 같은 옵션은 재마운트시 불가능하다는 메시지를 내면서 마운트에 실패하고 읽기전용 상태로 남아버린다.

이 문제의 해결 방법은 여러가지가 있겠지만, 내가 선택한 방법은 먼저 `/etc/fstab` 에는 'noatime' 옵션만 추가하고, `/boot/grub/menu.lst` 파일에서 defoptions 항목에 'rootflags=data=writeback' 을 추가하고, `update-grub` 명령을 실행하고 재부팅하면 적용된다.

성능이 좋은 PC의 경우 이 옵션이 있을 때와 없을 경우 차이점을 별로 못 느끼지만 X40 노트북에서는 어느 정도 체감 속도가 향상된 것을 느낄 수 있다. 더욱이[tracker](http://tracker-project.org/) 데몬 때문에 디스크가 혹사당하기 시작한 다음부터는 더욱더...
