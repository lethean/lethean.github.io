---
layout: post
title: Glade 카탈로그 작성 / 설치하기
tags: [ GTK+ ]
---

[글레이드(Glade) 매뉴얼](http://glade.gnome.org/docs/index.html)에서 [카탈로그(Catalog)](http://glade.gnome.org/docs/catalog.html) 소개 부분을 날림 번역한 내용입니다.

### 소개 (Introduction)

사용자 위젯은 여러 방법으로 지원할 수 있는데, 패키지를 만들어 이를 시스템 디렉토리에 설치하거나, 사용자 디렉토리 또는 프로젝트 디렉토리에서 직접 추가적인 카탈로그를 읽어들일 수 있습니다. 라이브러리에 보이는 객체 종류(object type)를 나타내기 위해 필요한 아이콘(icons)을 지원하는 코드를 만들수도 있고, 다음에 설명할 부모(parent) 속성(property)을 이용하여 처리할 수도 있습니다. 인스펙터(inspector)와 팔레트(pallette)가 사용할 아이콘을 제공하지 않으면 글레이드는 단순하게 경고 메시지를 출력하고 기본 아이콘을 사용합니다. 카탈로그 파일은 XML 형식으로 작성되며, DTD 파일은 글레이드 압축파일에서 plugins/ 디렉토리에서 찾을 수 있습니다.
 대부분의 경우 GTK+ 파생 위젯은 적은 노력으로 추가할 수 있으며 간단하게 위젯 종류를 명시하는 것으로 충분합니다. 글레이드는 속성(properties)과 시그널(signals)을 조사합니다(introspection). 하지만 위젯 툴킷 구조상의 본질로 인해 예외는 있습니다. 이 문서는 몇가지 기본 예제와 UI 편집을 강화하고 예외를 처리하는데 사용할 수 있는 풍부한 옵션을 제공할 것입니다.

카탈로그 파일은 카탈로그 이름과 사용할 플러그인 라이브러리를 지정하면서 시작합니다. 다음 예제에서는 "Foo"라는 이름공간(namespace)을 가지면서 "Frobnicator" 객체를 통합한다고 가정합니다.

    <glade-catalog name="foo" library="foo" depends="gtk+">
      <init-function>my_catalog_init</init-function>
      <glade-widget-classes>
        <glade-widget-class name="FooFrobnicator"
                            generic-name="frobnicator"
                            title="Frobnicator"/>
        ... widget classes go here
      </glade-widget-classes>
      <glade-widget-group name="foo" title="Foo">
        <glade-widget-class-ref name="FooFrobnicator"/>
        ... widget class references go here
      </glade-widget-group>
      ... widget groups go here
    </glade-catalog>

### 최상위 카탈로그 속성과 태그 (Toplevel catalog properties and tags)

카탈로그를 정의할때 'glade-catalog' 태그의 'name', 'library' 속성은 반드시 정의되어야 합니다. 'icon-prefix', 'depends', 'domain' 속성은 선택사항입니다.

|-----------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <span class="term">name</span>          | 카탈로그 문자열 식별자, 카탈로그를 식별하는데 사용하며 카탈로그간 의존성을 관리하는데 필요합니다.                                                                                                                                                                                                                                                           |
| <span class="term">version</span>       | 'major.minor' 형식의 버전 설명. ([예] `version="1.0"`) 동작하기 위한 버전을 검사하는데 필요하지만, 선택사항입니다.                                                                                                                                                                                                                                          |
| <span class="term">targetable</span>    | 쉼표(comma)로 구분하는 'major.minor' 형식 버전 목록. 현재 버전 이전 중에서 지원하는 버전을 명시합니다.                                                                                                                                                                                                                                                      |
| <span class="term">icon-prefix</span>   | 위젯의 아이콘 이름을 구성하는데 사용. 이 속성의 기본값은 'name' 속성 값을 사용합니다.                                                                                                                                                                                                                                                                       |
| <span class="term">library</span>       | 형식(type)을 읽어들이고 속성(properties)을 조사하는데 사용. 글레이드가 이 라이브러리를 로드하는데 사용하는데, 위젯을 포함하는 라이브러리 이름일 수도 있고 위젯 라이브러리에 암묵적으로 링크되는 플러그인 라이브러리 이름일 수도 있습니다. 이 라이브러리는 사용자가 지정한 경로나 시스템 플러그인 디렉토리(`$prefix/lib/glade-3/modules/)`에서 읽어들입니다. |
| <span class="term">depends</span>       | 지 원 코드 상속이 제대로 동작하는데 사용. 즉, 객체가 GTK+ 라이브러리 객체에서 파생한다면 위젯을 가능하도록 하기 위해 gladegtk 플러그인의 기본 지원 코드를 사용합니다. 이 속성은 설치된 다른 글레이드 플러그인의 'name' 속성입니다. 대개는 'depends="gtk+"'를 사용할 것입니다.                                                                               |
| <span class="term">domain</span>        | 번역 문자열을 검색하는데 사용할 도메인. 카탈로그의 모든 문자열은 이 도메인을 이용해 번역되어 표시됩니다. 지정하지 않으면 'library' 속성을 사용합니다.                                                                                                                                                                                                       |
| <span class="term">book</span>          | devhelp 문서 라이브러리를 검색하는데 사용할 이름공간을 지정. (특히, gtk-doc Makefile.am 파일에서 $(DOC\_MODULE) 항목으로 지정된 이름).                                                                                                                                                                                                                      |
| <span class="term">init-function</span> | 플러그인 전역 시작점을 가져오는데 사용. 백엔드(backend) 초기화 등에 사용하며, 위젯 클래스 인스턴스가 생성되기 전에 호출됩니다.                                                                                                                                                                                                                              |

### 확인하기/ 설치하기 (Validating and installing)

글레이드와 함께 설치되는 DTD 파일은 카탈로그 파일을 검사하는데 사용할 수 있습니다. 유의할 점은 속성(properties)이 DTD에 정의된 것과 같은 순서로 입력되어야합니다.

파일을 검사하려면 다음과 같이 합니다:

    xmllint --dtdvalid glade-catalog.dtd --noout my-catalog.xml

위젯 플러그인을 설치하기 위해서는 먼저 카탈로그 XML 파일을 카탈로그 디렉토리에 복사해야 합니다. 이 디렉토리는 다음과 같이 얻을 수 있습니다:

    pkg-config --variable=catalogdir gladeui-1.0

팔레트에 사용하는 아이콘은 pixmap 디렉토리에 들어갑니다:

    pkg-config --variable=pixmapdir gladeui-1.0

플러그인 라이브러리는 modules 디렉토리에 설치되어야 합니다:

    pkg-config --variable=moduledir gladeui-1.0

환경변수에 추가 경로를 지정함으로써 사용자 디렉토리에서 카탈로그를 읽어들 수도 있습니다. 예를 들면:

    GLADE_CATALOG_PATH=~/mycatalogs:~/work/foo/glade

같은 방식으로 플러그인 라이브러리 경로도 지정할 수 있습니다::

    GLADE_MODULE_PATH=~/work/foo/src

설치하지 않은 아이콘 읽어들이기는 아직 지원하지 않습니다.
