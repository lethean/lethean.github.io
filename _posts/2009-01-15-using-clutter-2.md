---
layout: post
title: "클러터(Clutter) 사용하기 (2)"
tags: [ Clutter ]
---

[클러터 튜토리얼](http://www.openismus.com/documents/clutter_tutorial/0.8/docs/tutorial/html/index.html) 내용을 계속 정리해 봅니다.

**액터 (Actors) - 배우**

클러터는 3차원 공간 속에서 2차원 표면(surfaces)을 다루는 캔버스 API입니다. 그래서, 기본 클러터 액터는 2차원 도형이지만 좌표는 3차원입니다. 하지만, 깊이(depth) 개념은 없습니다. 따라서 액터 테두리만 정면 화면에 걸치도록 정확하게 회전되었다면 이론적으로 안보이게 됩니다. 복잡한 3차원 객체가 필요하다면 OpenGL ES API를 이용해 새로운 액터를 구현해도 되지만, 이 방법은 나중에 살펴보기로 하고 여기서는 기본 액터 종류를 살펴보겠습니다.

-   [ClutterStage](http://clutter-project.org/docs/clutter/stable/ClutterStage.html) : 무대 자체
-   [ClutterRectangle](http://clutter-project.org/docs/clutter/stable/ClutterRectangle.html) : 사각형
-   [ClutterLabel](http://clutter-project.org/docs/clutter/stable/ClutterLabel.html) : 텍스트 표시하기
-   [ClutterEntry](http://clutter-project.org/docs/clutter/stable/ClutterEntry.html) : 사용자 편집 가능한 텍스트
-   [ClutterTexture](http://clutter-project.org/docs/clutter/stable/ClutterTexture.html) : 이미지

액터는 `clutter_container_add()`를 이용하여 스테이지에 추가할 수 있습니다. 물론 위치와 크기가 명시되어야 합니다. 모든 액터는 ClutterActor 객체에서 파생하므로 [`clutter_actor_set_position()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-position)을 이용해 x,y 좌표를 설정하고 [`clutter_actor_set_depth()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-depth)를 이용해 z 좌표를 설정합니다. z 좌표값이 클수록 화면에서 멀어집니다. [`clutter_actor_set_size()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-size)는 액터의 크기를 조절합니다. 액터의 위치는 스테이지(부모 컨테이너)의 좌상단(0,0)을 기준으로 상대적인 위치입니다. 이 기준값은 `clutter_actor_set_anchor_point()`를 이용해 변경할 수 있습니다.

GTK+ 위젯과 마찬가지로 클러터 액터도 처음 생성되었을 때는 보이는 상태가 아니기 때문에 [`clutter_actor_show()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-show)를 호출해야 비로소 보여집니다. 물론 다시 안보이게 하려면 [`clutter_actor_hide()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-hide)를 사용하면 됩니다. 또한 GTK+ 위젯 시스템과 동일한 부동 참조(floating reference) 개념을 사용하여 리소스를 관리합니다. (더 자세한 내용이 궁금하신 분은 '[GTK+ 메모리 관리](/2008/12/28/gtk-memory-management/)' 항목을 참고하세요)

모든 액터는 당연히 확대 및 축소(scaling), 회전(rotation) 시킬 수 있고, 부분적으로 투명하게 할 수도 있습니다.

액터의 크기를 확대하거나 축소하려면 [`clutter_actor_set_scale()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-scale)을 호출합니다. 유의할 점은 [`clutter_actor_set_size()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-size)로 지정한 크기는 변하지 않습니다. 그래서 [`clutter_actor_set_scale()`](clutter_actor_set_scale())을 다시 호출하면 원래 크기를 기준으로 스케일됩니다.

[`clutter_actor_set_rotation()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-rotation)을 호출하면 지정한 축(axis)을 기준으로 원하는 각도만큼 회전합니다. 예를 들어, 축으로 `CLUTTER_X_AXIS`을 지정하면 y,z 방향 인수만 지정할 수 있습니다. 이 기능 역시 액터의 위치와 크기에는 영향을 미치지 않습니다.

액터의 일부 영역만 보이도록(clipping) 하려면 [`clutter_actor_set_clip()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-clip)을 호출하면 됩니다. 예를 들어 매우 큰 컨테이너 액터를 만들고 스크롤 효과를 내면서 일부 영역만 보이도록 하는데 사용할 수 있습니다. 클립핑 영역을 해제하려면 [`clutter_actor_remove_clip()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-remove-clip)을 호출하면 됩니다. 클립핑 밖의 영역은 비디오 메모리도 사용안하고, 연산 작업에서도 제외됩니다.

액터를 현재 위치에서 상대적으로 움직이라면 [`clutter_actor_move_by()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-move-by) 또는 [`clutter_actor_set_depth()`](http://clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-set-depth)를 호출하면 됩니다.

이제 여기까지 설명한 액터의 기본 동작에 대한 예제 코드를 살펴보시기 바랍니다.

    #include <clutter/clutter.h>
    #include <stdlib.h>

    int main(int argc, char *argv[])
    {
      ClutterColor stage_color = { 0x00, 0x00, 0x00, 0xff };
      ClutterColor actor_color = { 0xff, 0xff, 0xff, 0x99 };

      clutter_init (&argc, &argv);

      /* 스테이지를 얻어 크기와 색상을 정합니다. */
      ClutterActor *stage = clutter_stage_get_default ();
      clutter_actor_set_size (stage, 200, 200);
      clutter_stage_set_color (CLUTTER_STAGE (stage), &stage_color);

      /* 스테이지에 사각형을 더합니다. */
      ClutterActor *rect = clutter_rectangle_new_with_color (&actor_color);
      clutter_actor_set_size (rect, 100, 100);
      clutter_actor_set_position (rect, 20, 20);
      clutter_container_add_actor (CLUTTER_CONTAINER (stage), rect);
      clutter_actor_show (rect);

      /* X축을 기준으로 20도 회전합니다.
       * (상단 테두리 기준으로 돌기)
       */
      clutter_actor_set_rotation (rect, CLUTTER_X_AXIS, -20, 0, 0, 0);

      /* 스테이지에 레이블을 추가합니다. */
      ClutterActor *label = clutter_label_new_full ("Sans 12", "Some Text", &actor_color);
      clutter_actor_set_size (label, 500, 500);
      clutter_actor_set_position (label, 20, 150);
      clutter_container_add_actor (CLUTTER_CONTAINER (stage), label);
      clutter_actor_show (label);

      /* X축 방향으로 300% 확대합니다. */
      clutter_actor_set_scale (label, 3.00, 1.0);

      /* 오른쪽으로 10, 위쪽으로 10 이동합니다. */
      clutter_actor_move_by (label, 10, -10);

      /* Z축으로 20 더 가깝게 이동합니다. */
      clutter_actor_set_depth (label, -20);

      /* 스테이지를 보이게 합니다. */
      clutter_actor_show (stage);

      /* 메인 이벤트 루프를 시작합니다. */
      clutter_main ();

      return EXIT_SUCCESS;
    }

이 코드를 실행하면 다음과 같은 화면을 얻을 수 있습니다.

![clutter-actor-example](/figures/clutter-actor-example.png "clutter-actor-example")
