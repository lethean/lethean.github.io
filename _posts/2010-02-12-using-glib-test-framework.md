---
layout: post
title: GLib 테스트 프레임워크 사용하기
tags: [ Agile,  Clutter,  Coding,  GLib,  GTK+ ]
---

GLib 라이브러리 2.16 버전부터 지원하는 [테스트 프레임워크](http://library.gnome.org/devel/glib/stable/glib-Testing.html)는 C 언어용 [유닛테스트](http://en.wikipedia.org/wiki/Unit_testing) 도구입니다. 물론 많은 유닛 테스트 도구가 이미 존재하지만, GLib 라이브러리 기반 C 언어 프로그램이라면 굳이 다른 라이브러리를 사용하는 것보다는 이미 지원하는 훌륭한 도구를 사용하는게 더 좋겠지요. 참고로,[GTK+](http://www.gtk.org/), [Clutter](http://clutter-project.org/) 등 같은 프로젝트도 이미 이 기능을 이용해 테스트 코드를 작성하고 있으므로 알아두면 도움이 됩니다. 모든게 그렇지만, 알고나면 별게 아니므로 기본 개념과 API 사용법만 충실히 이해하면 됩니다.

**기본 개념 및 사용법
**

유닛테스트 개념은 스몰토크, 자바, C++처럼 언어적으로 객체지향 개념을 지원하는 언어에서 시작했기 때문에 C 언어에 그대로 적용하기에는 조금 까다로운 점이 많습니다. 그래서 GLib 테스트 프레임워크는 유닛테스트에서 기본 개념과 테스트 실행 방식만 빌려옵니다. 우선 알아야하는 기본 개념은 다음과 같습니다.

-   테스트 케이스 (Test Case) : 가장 기본이 되는 하나의 테스트 단위입니다. GLib에서는 하나의 테스트 함수(function)가 이 역할을 합니다.
-   픽스쳐 (Fixture) : 고정 설치된 물건이라는 뜻처럼, 테스트 케이스 실행 전후에 항상 실행하는 함수를 의미합니다. 실제로는, 테스트 함수를 실행하기 위해 필요한 환경을 미리 구축하거나(setup) 실행 후 리소스를 정리하는(teardown) 함수, 그리고 이와 함께 사용되는 사용자 데이터(data)로 구성됩니다. 참고로, GLib에서는 각 테스트간 의존성을 피하기 위해 모든 테스트 케이스를 실행할때마다 매번 픽스쳐를 새로 구성하는 방식(fresh fixture)을 사용합니다.
-   테스트 슈트 (Test Suite) : 여러 테스트 케이스를 묶은 그룹입니다. 트리 구조처럼 테스트 슈트 여러개를 묶어 더 큰 테스트 슈트를 구성할 수도 있습니다. GLib에서는 테스트 경로(path)라는 개념으로 사용합니다.

개념은 조금 복잡한 것 같지만, 복잡하고 다양한 테스트 케이스를 그룹화하면 나중에 테스트 슈트별로 테스트를 진행할 수도 있는 등 많은 장점이 있습니다. 그리고 GLib이 제공하는 커맨드라인 도구를 이용하면 테스트 결과를 XML로 출력할 수도 있고, HTML 문서로 자동 변환할 수도 있는데 이 경우에도 테스트 슈트를 구성해 두면 많은 도움이 됩니다.

물론 GLib은 정교하게 테스트 슈트와 테스트 케이스, 픽스쳐를 구성할 수 있는 많은 API를 제공하지만, 복잡한 과정을 API 호출 하나로 처리할 수 있는 기능도 제공합니다.

```c
g_test_add_func ("/onvif/nvc-connections", test_onvif_nvc_connections);
```

위 예제에서 [`g_test_add_func()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-test-add-func) 함수는 "onvif" 테스트 슈트 밑에 "nvc-connections" 이름의 테스트 케이스를 추가합니다. 테스트시 실행할 함수는 사용자가 직접 구현한 `test_onvif_nvc_connections()` 함수입니다. `g_test_add_func()` 함수가 테스트 슈트를 자동으로 생성해 주기 때문에 별도의 추가 작업이 불필요합니다. 비슷한 기능의 [`g_test_add_data_func()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-test-add-data-func) 함수는 테스트 함수에 데이터를 전달할 수 있어서, 한 함수로 데이터만 바꿔서 테스트하고자 할때 유용합니다. 하지만, 두 API는 픽스쳐를 지정할 수 없으므로, 픽스쳐를 사용하려면 [`g_test_add()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-test-add) 함수를 이용해야 합니다.일단, 간단한 예제 코드를 보여드리면 다음과 같습니다. ("[Writing Unit Tests with GLib](http://blogs.gnome.org/timj/2008/06/24/23062008-writing-unit-tests-with-glib/)" 글에서 발췌했습니다)

```c
#include <glib.h>

static void
simple_test_case (void)
{
  /* a suitable test */
  g_assert (g_bit_storage (1) == 1);

  /* a test with verbose error message */
  g_assert_cmpint (g_bit_storage (1), ==, 1);
}

int
main (int argc, char **argv)
{
  /* initialize test program */
  g_test_init (&argc, &argv, NULL);

  /* hook up your test functions */
  g_test_add_func ("/Simple Test Case", simple_test_case);

  /* run tests from the suite */
  return g_test_run ();
}
```

이 코드를 `g-test-sample1.c` 파일로 저장하고 컴파일 후 실행하면 다음과 같은 결과를 볼 수 있습니다.

```sh
$ gcc -o g-test-sample1 g-test-sample1.c `pkg-config --cflags --libs glib-2.0`
$ ./g-test-sample1
/Simple Test Case: OK
```

이 결과를 재활용하기 위해 XML 형식으로 저장하거나, HTML 문서로 만들고 싶다면 [gtester](http://library.gnome.org/devel/glib/stable/gtester.html) / [gtester-report](http://library.gnome.org/devel/glib/stable/gtester-report.html) 프로그램을 사용하면 됩니다.

```sh
$ gtester -o sample-log.xml g-test-sample1
TEST: g-test-sample1... (pid=2771)
PASS: g-test-sample1
$ gtester-report sample-log.xml > sample-log.html
```

위와 같이 실행하여 생성한 HTML 문서 결과는 다음과 같습니다.

![](/figures/gtester-report-screenshot.png "gtester-report screenshot")

참고로, gtester 프로그램의 인수로 여러 테스트 실행 파일을 한꺼번에 전달하면 모든 테스트 실행 파일의 테스트 슈트가 하나의 결과로 통합됩니다.

위 코드에서 사용한 테스트 코드를 보면 제일 먼저 [`g_test_init()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-test-init) 함수가 나타납니다. 이 함수는 테스트 기능을 초기화하는데, 리퍼런스 매뉴얼을 보시면 프로그램 실행 인수를 통해 사용자가 여러 테스트 옵션을 지정할 수 있는 걸 알 수 있습니다. 물론 특정 테스트 슈트만 실행하게 하는 옵션도 인수로 지정할 수 있습니다.

테스트 함수를 보면 [`g_assert_cmpint()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-assert-cmpint)라는 다소 생소한 API가 보이는데, GLib은 테스트 코드를 위해 이와 비슷한 매크로를 더 제공합니다.

```c
#define g_assert             (expr)
#define g_assert_not_reached ()
#define g_assert_cmpstr      (s1, cmp, s2)
#define g_assert_cmpint      (n1, cmp, n2)
#define g_assert_cmpuint     (n1, cmp, n2)
#define g_assert_cmphex      (n1, cmp, n2)
#define g_assert_cmpfloat    (n1,cmp,n2)
#define g_assert_no_error    (err)
#define g_assert_error       (err, dom, c)
```

위 매크로를 사용하여 테스트 코드를 작성하면 더 친절하고 자세한 에러 메시지를 출력합니다. 예를 들어 다음 코드는,

```c
gchar *string = "foo"; g_assert_cmpstr (string, ==, "bar");
```

이런 메시지를 출력합니다.

```c
ERROR: assertion failed (string == "bar"): ("foo" ==  "bar")
```

물론 기본적으로 실패한 경우에만 메시지를 보여줍니다.

**그 외 더 많은...**

지금까지 설명한 기본 기능 외에도 표준출력 / 표준에러 메시지를 표시하지 않도록 한 뒤 이 메시지에서 특정 문자열을 확인한다든가, 항상 동일한 패턴의 난수를 생성하여 이를 테스트에 이용하거나,  테스트에 시간이 얼마나 더 걸리는지 측정할 수도 있습니다. 프로그램을 종료시키는 치명적인 에러가 발생하는 경우도 테스트할 수 있고, 여러가지 테스트 모드(quick / slow / performace 등)를 두어 프로그램 인자를 이용해 원하는 테스트 코드만 실행할 수도 있습니다.

더 많은 활용 예제가 GLib 자체 테스트 코드에([glib/tests/testing.c](http://git.gnome.org/browse/glib/tree/glib/tests/testing.c)) 있으므로, 별로 길지 않으니, 직접 확인해 보시기 바랍니다.

**프로젝트에 활용하기**

MVP 개발 모델과 TDD + 유닛테스트 도구를 이용하여 응용 프로그램을 개발하면([Presenter First 개발](/2008/12/17/presenter-first-development/)) 더 빠르고 쉽게 튼튼한 코드를 만들 수 있으니, 한 번 검토해 보시기 바랍니다. 개발자가 TDD 방법론을 주저하는 이유 중 하나가 테스트 코드까지 만들다 보니 늘어나는 코드량과 늘어나는 개발 시간 때문인데, 테스트 코드를 그대로 실제 코드로 재활용할 수 있다면 얘기가 달라지겠죠.

프로젝트 일일빌드시 테스트 루틴도 동작하도록 한뒤 자동으로 테스트 결과를 웹사이트에 게재하는 것도 좋은 개발 습관입니다. 아예 코드 수정 후 저장소에 커밋하면 반드시 모든 테스트 케이스를 통과해야만 커밋되도록 저장소를 설정할 수도 있지만, 엄청난 서버 부하를 야기할 수 있으므로, 테스트 케이스를 통과한 코드만 커밋할 수 있도록 가이드라인을 규정하는 것도 좋습니다.

유닛테스트는 특정 객체나 모듈의 모든 API가 항상 정상적으로 동작하는지를 검사하기 위해 사용합니다. 그래서 가장 기본적인 사용법은 공개 함수를 다양한 인수로 호출한 뒤 그 결과값을 확인하는 방식입니다. 하지만 실무에서는 그렇게 단순한(?) 버그만 존재하는게 아니라서, 특정 시나리오나 특정 조건을 만족할 경우에만 버그 현상이 재현되는 경우도 많습니다. 이러한 경우, 버그에 대한 테스트 케이스를 추가하고 이 케이스에 대한 테스트가 통과할때까지 디버깅을 합니다. 이렇게 해두면 동일한 버그가 나중에 재발하는 걸 방지할 수 있습니다. 대부분 회사에서는 버그(이슈)관리시스템을 사용하므로 [`g_test_bug()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-test-bug) API를 사용하면 편리합니다.

참고로, GTK+ 라이브러리는 GLib 테스트 프레임워크를 기반으로 마우스 버튼 동작이나 키보드 입력을 에뮬레이션하는 기능처럼 [GUI 프로그램 테스트용 API](http://library.gnome.org/devel/gtk/stable/gtk-Testing.html)를 제공합니다. 더불어 [Xvfb](http://en.wikipedia.org/wiki/Xvfb) 같은 더미 X서버를 이용하면 원격 터미널이나 cron 작업처럼 실제 X서버가 없는 환경에서도 GUI 프로그램 테스트 진행이 가능합니다. 꼭 GTK+ 프로그램이 아니더라도, 폰트 렌더링 루틴이 정확한 그래픽 비트맵을 생성하는지, 특정 항목을 선택하고 특정 행동을 취했을때 정상적으로 문자열이 표시되는지 등도 테스트 케이스로 작성할 수 있습니다.

**테스트 케이스 실행 방식 및 테스트 코드 위치**

위 예제처럼 테스트 케이스를 특정 주제별로 나누어 각각의 실행파일로 만들어도 되지만, 테스트 케이스를 초기화하는 부분을 잘 정리하여 테스트 케이스를 여러 모듈로 분리한 뒤,  모든 테스트 케이스를 통째로 하나의 실행파일로 만들어도 됩니다. 이렇게 하면 추가적인 스크립트나 도구의 도움없이도 명령어 한번 실행으로 모든 테스트 케이스를 실행할 수 있기 때문에 더 편리할 수 있습니다. 또는 Clutter 프로젝트처럼 테스트 모듈을 각각 공유라이브러리로 만들어 플러그인처럼 로드해서 실행하는 방법도 있습니다.

하지만, 위 방식은 모두 실제 코드와 테스트 코드가 서로 다른 파일에 존재하는 방식입니다. 테스트 코드가 실제 코드와 하나의 파일에 존재한다면 테스트 코드 작성이 더 일상화되고 자연스러워질 수 있습니다. 그러므로, 프로그램 실행 파일 크기가 별로 문제가 되지 않는다면, 또는 릴리스 / 디버그 모드를 분리하여 컴파일하도록 구성된 프로젝트라면,  프로그램에 특정 옵션을 주었을 경우에만 테스트 케이스 실행 모드로 동작하게 하면 됩니다. 물론 특정 테스트 프로그램은 예제로 사용하기 위해 분리할 수도 있겠지만, 모듈이나 객체의 고유 기능만 테스트하는 코드라면 같은 파일에 있는게 더 자연스러울 수 있습니다. 예를 들어 GObject 객체라면, 속성(properties) / 시그널(signal) 이름이 갑자기 변경되었을때 이를 참조하는 모듈이 문제를 일으키지 않도록 하기 위해, 'validate-properties', 'validate-signals' 등의 테스트 케이스를 추가한뒤 통과하지 못했을 경우 [`g_test_message()`](http://library.gnome.org/devel/glib/stable/glib-Testing.html#g-test-message) 등을 이용해 이를 참조하는 모듈을 찾아 수정하라는 강조 메시지를 표시하는 것도 가능합니다. 또한 특정 시그널이 정상적으로 발생하는지, 순서대로 발생하는지 확인할 수 있습니다. 그리고 무엇보다도, 같은 파일에 있으면 내부 자료구조에도 접근할 수 있으므로 내부 로직에 대한 테스트 코드를 작성하는 것도 가능해집니다.

따라서 무조건 한 가지 방식만 고집하기보다, 적절하게 필요에 따라 알맞는 방식을 선택하는 것이 중요합니다.

**결론**

뭐 다른 결론이 있을리 없을만큼 유닛 테스트와 리그레션 테스트(regression test) 등은 이미 소프트웨어 개발 분야 전반에 광범위하게 사용하고 있습니다. 다만, C 언어를 이용해 개발하는 경우 리거시(legacy) 코드가 너무 많거나, 마땅한 테스트 도구를 찾지 못했거나, 여러가지 이유로 도입하지 못하는 경우가 많은데, 함께 잘 극복하고 익숙해져서 더 좋은 방향으로 나아가야 하지 않을까... 생각해 봅니다.
