---
layout: post
title: "클러터 1.0 애니메이션 API"
tags: [ Clutter ]
---

[클러터 1.0 버전이 릴리스](http://lists.o-hand.com/clutter/2982.html) 된지 한참 되었습니다...:) 몇몇 GLES 플랫폼에서 동작하지 않는다는 보고도 있고, clutter-gst / clutter-gtk 같은 라이브러리는 지금도 동작은 하지만 의존하는 다른 프로젝트 진행상황과 맞물린 관계로 완전히 마무리된 [1.0 버전은 조금 지연](http://lists.o-hand.com/clutter/2983.html)된다는 말도 있고 하지만, 1.0 정식 버전은 매우 많은 성능 개선과 API가 추가되었습니다.

이 글에서는 1.0 API 변경 내용 중에 우선 [애니메이션 관련 내용](http://www.clutter-project.org/docs/clutter/stable/migrating-ClutterEffect.html)을 정리해 보았습니다.

0.8 버전까지 간단한 일회성 애니메이션 효과를 구현하기 위해 사용하던 ClutterEffect API가 사라지고 `clutter_actor_animate()` 함수로 완전히 교체되었습니다. 물론 복잡한 애니메이션에는 여전히 ClutterBehaviour / ClutterTimeline / ClutterScore 등과 같은 객체를 사용해야 하지만, 단순한 효과를 위해 복잡하게 ClutterEffectTemplate 객체를 만들어 사용하던 방식이 완전히 바뀐 셈입니다.

`clutter_actor_animate()` 함수의 원형(prototype)을 보면 다음과 같습니다.

    ClutterAnimation *
    clutter_actor_animate (ClutterActor *actor,
                           gulong        mode,
                           guint         duration,
                           const gchar  *first_property_name,
                           ...);

인수를 살펴보면, 액터(actor)에 대해 [애니메이션 모드](http://clutter-project.org/docs/clutter/stable/clutter-Implicit-Animations.html#ClutterAnimationMode)(mode)와 기간(duration)을 지정한 뒤 변화시킬 속성(properties) 목록과 목표값을 원하는만큼 지정하면 됩니다. 이 함수가 돌려주는 [ClutterAnimation](http://clutter-project.org/docs/clutter/stable/clutter-Implicit-Animations.html) 객체는 애니메이션이 끝나면 "completed" 시그널을 발생하고 자동으로 소멸됩니다. 물론 단순한 애니메이션 뿐 아니라 기존 ClutterTimeline / ClutterAlpha 객체와 연동하여 더 다양하고 정교한 제어도 가능합니다.

예를 들어 다음 코드는 250 밀리초 동안 'rectangle' 액터의 크기(width\*height)를 현재 크기에서 100\*100 크기로 변경하면서 투명도를 0으로 서서히 변경합니다.

    clutter_actor_animate (rectangle, CLUTTER_LINEAR, 250,
                           "width", 100.0,
                           "height", 100.0,
                           "opacity", 0,
                           NULL);

액터의 속성(properties)을 변경하지 않고 애니메이션 동안 특정값으로 고정시킬 수도 있습니다. 다음 예제는 애니메이션 기간 동안 "rotation-angle-z" 속성을 현재 각도에서 360도로 변경하지만, "rotation-center-z" 속성은 고정된 "center" 변수값으로 고정합니다.

    clutter_actor_animate (actor, CLUTTER_EASE_IN, 100,
                           "rotation-angle-z", 360,
                           "fixed::rotation-center-z", &center,
                           NULL);

단순히 액터의 속성을 변경하는 것 뿐 아니라 ClutterAnimation 객체의 시그널 핸들러를 직접 연결할 수도 있습니다. 다음 예제는 투명도가 0이 되어 애니메이션이 완료되면 자동으로 액터를 안보이게 합니다.

    static void
    on_animation_completed (ClutterAnimation *animation,
                            ClutterActor     *actor)
    {
      clutter_actor_hide (actor);
    }

    clutter_actor_animate (actor, CLUTTER_EASE_IN, 100,
                           "opacity", 0,
                           "signal::completed",
                             on_animation_completed, actor,
                           NULL);

또는 다음과 같이 사용할 수도 있습니다.

    static void
    on_animation_completed (ClutterActor *actor,
                            gpointer data)
    {
      clutter_actor_hide (actor);
    }

    clutter_actor_animate (actor, CLUTTER_EASE_IN, 100,
                           "opacity", 0,
                           "signal-swapped::completed",
                             on_animation_completed, actor,
                           NULL);

지금까지 간단하게 설명했지만, 언제나 그렇듯이, 애니메이션 관련 API 종류와 동작 방식에 대한 더 세부적인 내용은 [튜토리얼](http://www.openismus.com/documents/clutter_tutorial/0.9/docs/tutorial/html/sec-animations.html)과 [참고 설명서](http://www.clutter-project.org/docs/clutter/stable/clutteranimation.html)를 반드시 참고하시기 바랍니다.
