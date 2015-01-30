---
layout: post
title: "클러터(Clutter) 사용기"
tags: [ Clutter ]
---

[클러터(Clutter)](http://www.clutter-project.org/) 라이브러리를 이용하면서 부딪친 대부분의 문제는 성능과 관련된 것입니다. 클러터 라이브러리 자체가 느리다는 얘기가 아니라, 주로 개발하는 분야에서 요구되는 16채널 이상 다채널 라이브 / 녹화 동영상 재생을 구현할 때, 고사양 장비는 문제가 되지 않지만 저사양 임베디드 보드에서는 성능 저하가 발생하기 때문입니다. 하지만, 효율적인 2D 그래픽을 위한 3D 그래픽 라이브러리로서 클러터는 아직까지 만족스럽습니다. OpenGL 기반 라이브러리는 기존 2D 그래픽 라이브러리와 여러가지 기본 개념이 달라서, 저처럼 이쪽 세상에 입문한지 얼마 안되는 개발자는 많은 시행 착오를 겪을 수 밖에 없는데,  이미 사용해 본  [GTK+](http://www.gtk.org/) / [GObject](http://developer.gnome.org/gobject/stable/) 방식에 익숙한 점도 유리하게 작용했지만, 2D 그래픽 + 효과를 위한 약간의 3차원 API 조합은 복잡하고 어려운(...) 3D 라이브러리를 직접 사용하는 것보다 훨씬 수월했습니다.

아무튼 그래서, 지금까지 겪은 경험 중 몇 가지를 정리해 보았습니다. 당연하지만, 아직 OpenGL에 대한 이해가 부족해 틀린 내용이 있을 수도 있으니, 감안해 주시기 바랍니다.

1.
 클러터 라이브러리는 계속 버전업 되는데 예전에 작성된 튜토리얼이나 예제는 갱신되지 않아 잘못되거나 사용을 권장하지 않는(deprecated) API를 사용하는 경우가 많이 있습니다. 가능한 클러터 개발자들이 라이브러리와 함께 직접 업데이트하는 [클러터 해설서(The Clutter Cookbook)](http://docs.clutter-project.org/docs/clutter-cookbook/1.0/)를 참고하는게 가장 정확했습니다.

2.
 OpenGL 기반 클러터 라이브러리 동작 방식은 일반적인 2D GUI 프로그래밍과 달리 화면, 즉 스테이지(stage)에 조그만 변화라도 있으면 그때마다 스테이지를 다시 그립니다. 즉, 클러터의 기본 단위인 액터(actor) 하나가 다시 그려져야 하면 액터가 속한 스테이지의 모든 액터를 다시 그립니다. 그리고 이로 인해 스테이지에 보이는 모든 액터의 [`paint()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수가 매번 호출되기 때문에 이 함수를 최적화하는 게 매우 중요합니다.

3.
 내부적으로 캐싱(caching)이 이용되긴 하지만, 한 액터의 좌표(크기 + 위치)가 변경되면 스테이지의 모든 액터의 크기를 다시 계산하기 위해 모든 액터의 [`allocate()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수가 호출됩니다. 예를 들어 텍스트(text) 액터 구현을 보면, 문자열이나 폰트, 크기 등이 변경되었을때 텍스트 액터를 포함하는 부모 컨테이너 액터가 변경된 크기를 감지하고 자신의 크기를 조정할 수 있도록 [`clutter_actor_queue_relayout()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-queue-relayout) 함수가 호출됩니다. 그리고, 이 함수가 호출되면 결국 스테이지 단계까지 호출이 계속된 다음, 다시 스테이지에 속한 액터의 [`allocate()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수가 재귀적으로 호출됩니다. 따라서 액터의 [`allocate()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수 역시 내부적으로 최적화되는 것이 좋습니다. 참고로 [`clutter_actor_queue_relayout()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-queue-relayout) 함수가 호출되면 자동으로 [`clutter_actor_queue_redraw()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-queue-redraw) 함수가 호출되어 스테이지의 모든 객체를 다시 그립니다.

4.
 클러터에서 기본으로 제공하는 [박스(ClutterBox)](http://docs.clutter-project.org/docs/clutter/stable/ClutterBox.html), [그룹(CutterGroup)](http://docs.clutter-project.org/docs/clutter/stable/ClutterGroup.html) 등과 같은 컨테이너 액터를 사용하지 않고 직접 컨테이너 액터를 구현해 자식 액터를 배치하고 싶거나 혹은 기존 컨테이너 액터를 상속받아 새 컨테이너 액터를 구현할 때가 있습니다. 그런데, 컨테이너 액터 좌표(크기 + 위치) 변경에 따라 자식 액터의 좌표를 자동으로 변경할 필요가 있으면 대개 [`"allocation-changed"`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActor-allocation-changed) 시그널을 이용해 감지한 뒤 [`clutter_actor_set_size()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-size), [`clutter_actor_set_position()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-position) 함수 등을 이용해 자식 액터의 좌표를 조정하거나, [제약(ClutterConstraints)](http://docs.clutter-project.org/docs/clutter/stable/ClutterConstraint.html) 기능을 이용하는데, 위에서 설명한 것처럼 좌표가 변환되면 자동으로 [`clutter_actor_queue_relayout()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-queue-relayout) 함수가 호출되면서 *"The actor 'xxx' is currenty inside an allocation cycle; calling clutter\_actor\_queue\_relayout() is not recommended"* 디버그 경고 메시지가 계속 출력됩니다. 메시지니를 무시할 수도 있지만, 문제는 한 액터의 좌표 변경으로 인해 매번 화면 전체가 다시 좌표를 다시 계산하기 때문에 결국 모든 액터의 [`allocate()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수가 계속 호출되면서 CPU 점유율이 매우 높아진다는 점입니다. 여기에 좌표를 이용한 애니메이션까지 사용하면 CPU 점유율은 상상을 초월할 정도로 올라갑니다. 이 문제를 해결하려면 반드시 [`allocate()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수에서 자식 액터의 좌표를 지정할때 [`clutter_actor_allocate*()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-allocate) 종류의 함수만 이용해야 하고, 어쩔 수 없을 경우 [`g_idle_add_full()`](http://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html#g-idle-add-full) 함수를 이용해 자식 액터 좌표 지정 루틴의 실행을 뒤로 조금 미룬 뒤에 좌표 중복 검사 등을 통해 가능한 화면 재구성(relayout) 작업이 덜 일어나게 해야 합니다. 메인루프 우선 순위는 [`CLUTTER_PRIORITY_EVENTS`](http://docs.clutter-project.org/docs/clutter/stable/clutter-Events.html#CLUTTER-PRIORITY-EVENTS:CAPS) 값을 사용하는 게 좋습니다.

5.
 [클러터 실행에 영향을 주는 환경 변수](http://docs.clutter-project.org/docs/clutter/stable/running-clutter.html)를 이용하면 성능 조율 및 디버깅에 상당한 도움을 받을 수 있습니다. 예를 들어 [`COGL_DEBUG=rectangles`](http://wiki.clutter-project.org/wiki/ClutterProfiling#Are_you_drawing_actors_unnecessarily.3F) 이나 [`CLUTTER_DEBUG="paint layout"`](http://docs.clutter-project.org/docs/clutter/stable/running-clutter.html) 등은 도움이 많이 됩니다.

6.
 [텍스트(ClutterText)](http://docs.clutter-project.org/docs/clutter/stable/ClutterText.html) 액터의 위치가 정수가 아니라면, 즉 소수점 이하 값이 존재하는 실수일 경우  글꼴 선이 흐려지거나 뭉개지는 현상이 발생합니다. 텍스트 액터 뿐 아니라 [사각형(ClutterRectangle)](http://docs.clutter-project.org/docs/clutter/stable/ClutterRectangle.html) 액터처럼 그림이 아니라 직접 그려지는 액터들도 비슷한 현상이 발생합니다. 비단 액터의 위치가 정수일 지라도 이를 포함하는 상위 컨테이너 액터의 위치가 소수점 이하를 포함하는 실수일 경우, 즉 화면(stage)상 절대 좌표가 정수가 아닐 경우 이 현상이 발생합니다. 따라서 액터의 좌표를 계산해서 지정할 경우 반드시 [`floor()`](http://www.manpagez.com/man/3/floor/) / [`ceil()`](http://www.manpagez.com/man/3/ceil/) 등의 함수를 이용해 소수점 아래 값을 없애주는 것이 좋습니다.

7.
 액터에 배경 또는 테두리를 장식하고 싶을때 보통 떠오르는 구현 방법은 두 가지가 있습니다. 첫번째는 컨테이너 액터를 이용해 사각형 액터를 맨 아래 두고 대상 액터를 위에 두는 방법을 이용한 것이고, 두 번째는 이런 작업을 하는 커스텀 액터를 직접 구현하는 것입니다. 그런데 이보다 더 좋은 방법은, [효과(ClutterEffect)](http://docs.clutter-project.org/docs/clutter/stable/ClutterEffect.html) 객체를 구현해서 사용하는 것입니다. 효과 객체가 액터 객체에 추가되면 효과의 [`pre_paint()` / `post_paint()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterEffect.html#ClutterEffectClass) 함수가 액터의 [`paint()`](http://docs.clutter-project.org/docs/clutter/stable/ClutterActor.html#ClutterActorClass) 함수 호출 전후에 자동으로 호출되므로, 동일한 디스플레이 루틴을 여러 객체에 쉽게 적용할 수 있습니다. 클러터에서 이미 기본으로 제공하는 [고급 효과](http://docs.clutter-project.org/docs/clutter/stable/ch07.html)를 사용해도 되지만, 예를 들어 [텍스트에 그림자를 넣어주는 예제](http://docs.clutter-project.org/docs/clutter/stable/ClutterEffect.html)를 그대로 이용해 [테두리 효과](http://docs.clutter-project.org/docs/clutter-cookbook/1.0/effects-basic.html)처럼 구현할 수도 있는 셈입니다.

8.
 클러터에서 직접 그리기 위해 사용하는 OpenGL 랩퍼 API [Cogl](http://docs.clutter-project.org/docs/cogl/stable/) 함수를 사용할때 [경로(Path)](http://docs.clutter-project.org/docs/cogl/stable/cogl-Path-Primitives.html) 등을 사용하지 않고 가능하다면 [기본(Primitives) 함수](http://docs.clutter-project.org/docs/cogl/stable/cogl-Primitives.html)만 사용해서 구현하는게 성능이 좋습니다.

9.
 [Cogl](http://docs.clutter-project.org/docs/cogl/stable/) 함수를 이용해 직접 그리는 방식과 모든 것을 그림 이미지로 처리하는 [텍스쳐(ClutterTexture)](http://docs.clutter-project.org/docs/clutter/stable/ClutterTexture.html)를 이용하는 방식의 장단점을 아직 잘 모르겠습니다. 다만, 텍스쳐는 내부적으로 사용하는 메모리량이 더 많은 것이 분명하고, 현재 개발 중인 시스템에서는 수많은 채널의 비디오 동시 재생을 위해 어차피 많은 텍스쳐가 사용되기 때문에 가능한 텍스쳐 사용을 자제했습니다. 하지만 영역 크기에 따라 크기가 달라지는 GUI 부분을 구현할때는 이미지 기반 텍스쳐보다 일종의 벡터 그래픽이라고 할 수 있는 Cogl 함수를 이용해 직접 그리면 훨씬 깔끔한 GUI를 얻을 수 있는 것도 사실인 것 같습니다.

10.
 비디오 재생을 위해 [비디오싱크(ClutterGstVideoSink)](http://developer.gnome.org/clutter-gst/1.3/ClutterGstVideoSink.html) 객체를 사용할 때 [갱신 우선순위 (`update-priority`)](http://developer.gnome.org/clutter-gst/1.3/ClutterGstVideoSink.html#ClutterGstVideoSink--update-priority) 속성을 [`CLUTTER_PRIORITY_REDRAW`](http://docs.clutter-project.org/docs/clutter/stable/clutter-Events.html#CLUTTER-PRIORITY-REDRAW:CAPS) 값으로 낮추면 마우스 이벤트 반응 속도를 개선할 수 있습니다.

11.
 정말로 빈번한 애니메이션을 구현할때는, 사용하기 쉽지만, 내부적으로 수많은 객체가 생성되고 소멸되는 [`clutter_actor_animate()`](http://docs.clutter-project.org/docs/clutter/stable/clutter-Implicit-Animations.html#clutter-actor-animate) 대신, 번거롭지만, [타임라인(ClutterTimeline)](http://docs.clutter-project.org/docs/clutter/stable/ClutterTimeline.html) 등을 이용해 구현하는게 CPU / 메모리 사용량을 줄이는데 도움이 됩니다.

12.
 AMD(ATI) 그래픽 카드를 장착한 리눅스 상에서 클러터를 실행할때 Catalyst 상용 X 드라이버와 최신 오픈소스 X 드라이버와 성능 차이가 거의 없어진 것 같습니다. 물론 NVidia는 상용 드라이버 성능이 월등이 더 좋고, 인텔 칩셋은 오픈 소스 드라이버만 있고 성능도 좋습니다.

이해에 도움이 될까 싶어, 아직 프로토타입이고 많은 기능이 빠져있지만, 현재 개발 중인 시스템의 동작 화면을 녹화한 영상을 보여드립니다. 녹화에 사용한 프로그램은 [gtk-recordMyDesktop](http://recordmydesktop.sourceforge.net/about.php)입니다.

<iframe width="480" height="360" src="http://www.youtube.com/embed/1w0fjm0MYo8"></iframe>

GUI 프로그래밍을 할 때마다 느끼는 점은 구현하기 위한 기술도 중요하지만, 결국 사용자를 배려하면서도 아름다움을 잃지 않는 참신한 아이디어 기반의 디자인이 더 중요하다는 점입니다. 물론 그렇기 때문에 디자이너라는 직업이 따로 있는 것이겠지만, 좋은 프로그램과 삶의 다양한 모습을 많이 보고, 많이 경험하고, 많이 참고해야 하는 지적 즐거움을 언제부터인가 프로그래머들은 남의 영역이라 멀리한 채 무미건조한 기술에만 전념하고 있는 건 아닌지 모르겠습니다.
