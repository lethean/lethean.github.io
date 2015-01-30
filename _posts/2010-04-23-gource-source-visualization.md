---
layout: post
title: Gource 소스 저장소 시각화 프로그램
tags: [ Agile,  Git,  Ubuntu ]
---

LWN.net 기사 중에서 [소스 코드 작업 시각화 관련 기사](http://lwn.net/Articles/382468/)를 보고 재미있을 것 같아 [Gource](http://code.google.com/p/gource/) 프로그램을 이용해 회사에서 진행중인 프로젝트에 적용해 보았습니다.

<iframe width="480" height="360" src="http://www.youtube.com/embed/RUwDxM28EBA"></iframe>

만드는 방법은 우선 필요한 패키지를 설치하고(Ubuntu 기준)

```sh
$ sudo apt-get install gource ffmpeg
```

Git 저장소가 있는 디렉토리로 이동해서 다음과 같이 실행합니다.

```sh
$ gource 
    -s 0.01 
    --auto-skip-seconds 0.1 
    --file-idle-time 500 
    --disable-progress 
    --output-framerate 25 
    --highlight-all-users 
    -800x600 
    --stop-at-end 
    --output-ppm-stream - | 
  ffmpeg 
    -y 
    -b 1000K 
    -r 17 
    -f image2pipe 
    -vcodec ppm 
    -i - 
    -vcodec mpeg4 
    gource-edc-20100423.avi
```

프로젝트에 참여했던 사람들 이름이 나타났다 사라지는 걸 보면 기분이 약간 묘해지는 것 같습니다 ;)
