---
layout: post
title: tcpdump를 이용한 네트워크 사용량 측정하기
tags: [ Linux,  MacOSX,  Network,  Shell ]
---

업무상 실제 패킷 사용량을 측정할 필요때문에 여러가지 도구를 찾던 중 마땅한 걸 찾지 못해 직접 측정한 방식을 정리해 봅니다. 물론 이보다 더 좋은 방법들이 당연히 있을테지만, tcpdump 프로그램만 겨우 사용할 수 있는 환경에서 측정하는 법을 정리한 문서를 찾지 못해 남겨둡니다.

우선 어떤 방식으로든 해당 장비에 tcpdump 프로그램을 설치합니다.

그리고 측정하려는 과정이나 단계가 시작하는 동시에 다음과 같이 tcpdump 프로그램을 실행합니다.

    $ tcpdump -qvtttt dst xxx.xxx.xxx.xxx > packet-dump.txt

여기서 `xxx.xxx.xxx.xxx`는 측정에 사용할 대상 장비입니다. 즉, 위 예제는 특정 IP로 전송하는 패킷량만 캡쳐하여 `packet-dump.txt` 파일에 저장합니다. 중요한 점은 앞의 옵션인데, 이 옵셥을 사용해야 아래에서 사용하는 스크립트가 분석할 수 있는 형태의 결과물로 저장됩니다. 그리고, 필요하다면, 저장한 파일을 리눅스 또는 맥 장비로 복사합니다.

저장한 파일을 `conv2csv.sh` 스크립트를 이용해 엑셀이나 오픈오피스에서 읽어들일 수 있는 CSV 파일 형태로 변환합니다.

    $ ./conv2csv.sh packet-dump.txt

변환된 `packet-dump.csv` 파일은 한 행에 '**TIMESTAMP BYTES Kbps**' 형태로 각 초당 데이터가 저장되어 있습니다. 따라서 이 파일을 액셀이나 오픈오피스에서 공백(space)을 구분자로 해서 읽어들인 후 3번째 컬럼을 사용하면 됩니다. 참고로 여기서 측정한 크기는 IP/TCP/UDP 헤더까지 포함한 크기입니다.

다음은 이렇게 변환한 데이터를 구글 스프레드시트를 이용해 만든 차트입니다.

![](/figures/packet-traffic-analysis.png)

위에서 언급한 `conv2csv.sh` 스크립트는 다음과 같습니다.

    #!/bin/sh

    CSVFILE="$(dirname $1)/$(basename $1 .txt).csv"

    awk '{ print $2, $18 }' $1 | 
      tr '.)' '  ' | 
      awk 'BEGIN { last = ""; sum = 0; } 
           { if (last == $1) 
               { sum += $3 } 
             else 
               { print last, sum, sum * 8 / 1000; 
                 last = $1; 
                 sum = $3; } 
           }' > $CSVFILE

;)
