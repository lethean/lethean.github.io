---
layout: post
title: Ninja 빌드 도구 소개
tags: [ Ninja, Build ]
---

'[2015년 주목받은 신인 오픈소스 SW 11선](http://www.bloter.net/archives/252083)' 기사와 Dropbox 클라이언트의 다음 버전을 [Rust](https://www.rust-lang.org/) 언어와 [Bazel](http://bazel.io/)로 만든다는 [글](https://www.reddit.com/r/rust/comments/4adabk/the_epic_story_of_dropboxs_exodus_from_the_amazon/)을 보고 궁금증이 생겨 잠시 Bazel 빌드 도구를 살펴보았다. 하지만 아직 윈도 플랫폼을 지원하지 않고 ~~Java 언어로 구현되어 있어서~~ 당분간 사용을 보류했다.

그러다가 문득 요즘에는 어떤 빌드 시스템이 인기 있는지 궁금해서 조사해 보니, 지난 몇 년 사이에 [Ninja](https://ninja-build.org)라는 빌드 도구가 계속 언급된다. [CMake](https://cmake.org/), [GYP](https://code.google.com/p/gyp/) 같은 전통의 빌드 시스템이 Make 빌드 파일뿐 아니라 Ninja 빌드 파일도 생성한다. 심지어 [Chromium 브라우저 빌드](https://www.chromium.org/developers/how-tos/build-instructions-chromeos)도 Ninja를 사용하는 방식으로 변경되었다.

반나절 정도 걸려 한 프로젝트에 대한 Ninja 빌드 파일을 만들어 실행해 본 소감은 기대 반 실망 반인 것 같다. 항상 [ccache](https://ccache.samba.org/)를 사용하기 때문에 이와 맞물려 더한 감도 있지만, 체감 빌드 속도는 대만족이다. 하지만 단순한 기능을 보완하려면 별도의 도구가 더 필요한 점은 아쉽다.

Ninja의 장점은 빠른 빌드 속도와 리눅스 / 윈도 / 맥 운영체제를 포함하는 멀티 플랫폼 지원이다. 빠른 속도를 얻기 위해 시스템에 장착된 모든 CPU를 모두 사용해서 병렬로 작업을 처리한다. `make -jN` 명령에서 `N`에 시스템 CPU 갯수를 지정한 것과 같은 셈이다. 하지만 조금 다뤄보니 병렬 작업에 대한 고려를 많이 했다는 느낌을 받는다. 병렬 처리되는 작업의 결과 내용을 섞어서 표시하지 않고 버퍼링 했다가 따로따로 출력한다거나 Make처럼 세심하게 신경 쓰지 않아도 대부분 작업을 병렬로 잘 처리한다.

쉬운 문법과 사용법도 장점이다. 1~2시간 정도면 매뉴얼을 다 읽을 수 있을 정도로 문법이 단순하고 양도 적다. [Make의 암묵적인 규칙](https://www.gnu.org/software/make/manual/html_node/Implicit-Rules.html) 같은 게 전혀 없어서 외우거나 혼란스러운 것도 없다. 하지만 이 단순함은 반대로 치명적인 단점이기도 하다.

예를 들어 Make에서는 `A = $(shell pkg-config --cflags libuv)`처럼 명령어 실행 결과를 변수에 담거나, `A += $(B)`, `A ?= src` 같이 다양한 조건을 이용해 변수를 조작할 수 있는데 Ninja에서는 `a = $b` 단순 대입 문법이 전부다. 심지어 환경 변수도 사용할 수 없다. 조건문 같은 건 상상도 할 수 없다. 다양한 운영체제를 지원하기 위해 유닉스 셸 실행 기능이 빠진 건 이해할 수 있지만, 환경 변수나 조건문이 없다는 건 Make처럼 혼자서 모든 다양한 경우에 대한 작업을 다 할 수 없다는 뜻이기도 하다. 그래서 홈페이지에도 Ninja를 직접 사용하지 말고 [Ninja 빌드 파일을 자동으로 만드는 다른 도구](https://github.com/ninja-build/ninja/wiki/List-of-generators-producing-ninja-build-files)를 추천하는 것 같다.

참고로 Sublime Text 같은 텍스트 에디터에서 Ninja 문법을 지원하거나 [빌드 명령어 실행](https://www.chromium.org/developers/sublime-text#TOC-Building-with-ninja)을 위한 설정 방법은 대부분 구글에서 검색하면 쉽게 발견된다. 다음은 리눅스에서 Sublime Text를 사용할 때 필요한 Ninja 빌드 시스템 내용인데, 따로 적어두는 이유는 Chromium 사이트의 내용과 조금 다르기 때문이다.

```python
{
 "shell_cmd": "ninja",
 // ${folder} or ${file_path}
 "working_dir": "${folder}",
 // (file_path):(line_number):(column):(error_message)
 "file_regex": "^([^:\n]*):([0-9]+):?([0-9]+)?:? (.*)$"  
}
```
