---
layout: post
title: "클러터(Clutter) 사용하기 (5)"
tags: [ Clutter ]
---

[클러터 튜토리얼](http://www.openismus.com/documents/clutter_tutorial/0.8/docs/tutorial/html/index.html) 계속 이어집니다.

**효과(Effects) - 자연스러운 애니메이션
**

클러터는 애니메이션을 쉽게 구현하기 위해 효과(effect) 기능을 제공합니다. 시간에 따라 하나의 액터 속성을 변경하기 위해 타임라인과 간단한 숫자 계산을 이용한 [여러가지 효과(effect) 함수](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html)를 이용할 수 있습니다. 예를 들어 [`clutter_effect_fade()`](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html#clutter-effect-fade)는 액터의 불투명도(opacity)를 단계적으로 변화시키고, [`clutter_effect_rotate()`](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html#clutter-effect-rotate)는 단계적으로 액터를 회전시키는데(rotate), 이때 불투명도와 회전도는 프로그래머가 등록하는 `alpha_func` 콜백을 호출해서 계산합니다.

효과 기능을 사용하려면 제일 먼저 [`clutter_effect_template_new()`](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html#clutter-effect-template-new)를 이용하여 ClutterEffectTemplate 객체를 하나 만들어야 합니다. 이때 타임라인 객체와 ClutterAlphaFunc 형태의 콜백을 함께 지정해야 합니다. 이 콜백은 [`clutter_alpha_get_timeline()`](http://clutter-project.org/docs/clutter/0.8/ClutterAlpha.html#clutter-alpha-get-timeline)을 호출해서 타임라인 객체를 얻은 뒤, 현재 프레임 번호([`clutter_timeline_get_current_frame()`](http://clutter-project.org/docs/clutter/0.8/ClutterTimeline.html#clutter-timeline-get-current-frame))와 총 프레임 수([`clutter_timeline_get_n_frames()`](http://clutter-project.org/docs/clutter/0.8/ClutterTimeline.html#clutter-timeline-get-n-frames))를 기반으로 알파값을 계산하여 반환하면 됩니다. 반환하는 값은 0과 [`CLUTTER_ALPHA_MAX`\_ALPHA](CLUTTER_ALPHA_MAX_ALPHA) 사이의 값이어야 하며, 이 값의 의미는 사용하는 효과에 따라 다릅니다. 예를 들어 [`clutter_effect_fade()`](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html#clutter-effect-fade)를 사용할때 [`CLUTTER_ALPHA_MAX_ALPHA`](CLUTTER_ALPHA_MAX_ALPHA) 값은 불투명도 100%를 의미합니다. 물론 모든 계산을 직접 할 수도 있지만 [`CLUTTER_ALPHA_SINE`](http://clutter-project.org/docs/clutter/0.8/ClutterAlpha.html#CLUTTER-ALPHA-SINE:CAPS)처럼 미리 정의된 콜백 함수를 이용하면 쉽게 자연스러운 움직임을 얻을 수 있습니다. 다음 그림은 미리 정의된 몇가지 알파 콜백 함수가 반환하는 값을 그래프로 표현한 것입니다.

![](/figures/clutter-alpha-func.png)

이렇게 만들어진 ClutterEffectTemplate 객체를 사용하려는 [`clutter_effect_*()`](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html) 함수를 호출하면서 전달하면 됩니다.

여러가지 타임라인을 여러가지 효과와 사용할때는 [`clutter_effect_template_set_timeline_clone()`](http://clutter-project.org/docs/clutter/0.8/clutter-Clutter-Effects.html#clutter-effect-template-set-timeline-clone)을 이용하여 타임라인을 복제하도록 하면, 원본 타임라인을 변경해서 다른 효과에 사용할 수 있으므로, 다른 효과에 영향을 주지 않고 쉽게 재활용이 가능합니다. ClutterTimeline 객체와 마찬가지로 ClutterEffectTemplate 객체도 사용이 끝난후 `g_object_unref()`를 이용하여 리소스를 해제해야 합니다. 참고로, 효과 함수를 호출한 이후 템플릿을 더 이상 사용하지 않는다면 바로 해제해도 됩니다.

효과 함수는 실제로 [ClutterBehaviour](http://clutter-project.org/docs/clutter/stable/ClutterBehaviour.html) 객체를 감싼 단순화된 API라고 말할 수 있습니다. 하지만 효과 함수는 한 번에 하나의 액터만 제어할 수 있고 타임라인이 실행중인 동안에는 효과를 변경할 수 없습니다. 이러한 단점을 피하고 더 복잡한 효과를 원한다면 움직임(Behaviours) 객체를 직접 사용해야 합니다.

**움직임(Behaviours) - 더 자연스럽고 강력한 애니메이션**

효과 기능은 간단하지만, 더 복잡하고 다양한 애니메이션을 제어하기에는 부족하기 때문에 가끔은 움직임(Behaviours) 객체를 직접 사용해야 하는 경우가 있습니다. 효과 함수와 달리 움직임 객체를 사용하면 여러 액터를 동시에 제어할 수 있고 타임라인이 실행중이라도 움직임 파라메터를 변경할 수 있습니다. 예를 들어 [ClutterBehaviourPath](http://clutter-project.org/docs/clutter/stable/ClutterBehaviourPath.html)는 액터를 지정한 경로(path)를 따라 움직이는데, 매 프레임마다 `alpha_func` 콜백을 호출해서 경로상의 위치를 계산합니다. 다음 그림은 경로 상에서 알파 함수의 효과를 그래프로 표현한 것입니다.

![](/figures/clutter-path-alpha-func.png)

콜백 함수 동작 방식은 효과(effect) 함수에서 사용하는 것과 동일합니다.

타임라인이 무한루프 방식으로 동작하지 않을 경우, 움직임(behaviour)의 타임라인이 시작되면 움직임은 항상 마지막 점에 도달한 뒤 거기서 끝납니다. 예를 들자면, 액터는 타임라인에서 지정한 총 프레임 수와 초당 프레임 수에서 지정한 만큼 변경되면서 마지막 지점에 도달할때까지 경로를 따라 움직입니다.

액터와 마찬가지로 [ClutterAlpha](http://clutter-project.org/docs/clutter/stable/ClutterAlpha.html) 객체도 부동 참조(floating reference)를 가지고 있으므로 움직임 객체에 더한 다음에 따로 리소스를 해제할 필요가 없습니다. 하지만 움직임 객체는 아니므로 사용이 끝난 뒤에 `g_object_unref()`를 이용하여 리소스를 해제해야 합니다.

클러터에서 제공하는 기본 움직임은 다음과 같습니다.

-   [ClutterBehaviourBspline](http://clutter-project.org/docs/clutter/stable/ClutterBehaviourBspline.html): 스플라인을 따라 액터가 이동
-   [ClutterBehaviourDepth](http://clutter-project.org/docs/clutter/stable/ClutterBehaviourDepth.html): Z축을 따라 액터가 이동
-   [ClutterBehaviourEllipse](http://clutter-project.org/docs/clutter/stable/ClutterBehaviourEllipse.html): 타원을 따라 액터가 이동
-   [ClutterBehaviourOpacity](http://clutter-project.org/docs/clutter/0.8/ClutterBehaviourOpacity.html): 액터의 불투명도 변화
-   [ClutterBehaviourPath](http://clutter-project.org/docs/clutter/0.8/ClutterBehaviourPath.html): 일련의 점으로 정의된 경로를 따라 액터가 이동
-   [ClutterBehaviourRotate](http://clutter-project.org/docs/clutter/0.8/ClutterBehaviourRotate.html): 액터가 회전
-   [ClutterBehaviourScale](http://clutter-project.org/docs/clutter/0.8/ClutterBehaviourScale.html): 액터가 확대 또는 축소

