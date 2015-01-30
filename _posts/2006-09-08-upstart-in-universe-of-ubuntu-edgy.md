---
layout: post
title: Upstart in Universe of Ubuntu Edgy
tags: [ Linux,  Ubuntu ]
---

(http://www.netsplit.com/blog/work/canonical/upstart.html)

<span class="caps">UNIX</span> System V부터 현재 대부분의 리눅스 배포판에 지금까지 사용하는 sysvinit 시스템이 Ubuntu Edgy 버전에서 upstart라는 이벤트-작업(job) 기반 시스템으로 교체되고 있다. 현존하는 initng, launchd, SMF 등과 같은 sysvinit의 다른 대안을 선택하지 않고 우분투 팀에서 새로 만들어가고 있는 것 같다.

아마도 가장 큰 변화는 USB 메모리나 USB 네트웍 장치처럼 실행 중에 추가되고 삭제되는 환경을 고려한다는 점이고, 이벤트 기반으로 시스템 초기화 스크립트(/etc/rcS.d)가 재작성되고, 결국에는 모든 패키지의 데몬이 upstart 방식으로 변경될 것 같다.

과연 upstart 방식이 우분투 리눅스에서만 사용하게 될 것인가, 다른 배포판에도 영향을 끼칠 것인가는 아직 미지수다. 하지만 지금까지 대부분 Unix / Linux 사용자와 관리자에게 너무나 당연하게 여겨졌던 /etc/rc?.d 데몬 방식이 변경되면, 새로 공부할게 한 가지 더 늘어나게 되겠군..

dbus, udev, hal, ... 리눅스도 이제 충분히 (개발자에게는) 복잡한 시스템이 되어가고 있다.
