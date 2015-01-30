---
layout: post
title: "클러터(Clutter) 사용하기 (4)"
tags: [ Clutter ]
---

[클러터 튜토리얼](http://www.openismus.com/documents/clutter_tutorial/0.8/docs/tutorial/html/index.html) 계속 이어집니다.

**타임라인 (Timelines)**

[ClutterTimeline](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html)은 시간이 흐르는 동안 액터의 모양이나 위치를 자동으로 변경하는데 사용합니다. 다음에 설명할 효과(effect)나 움직임(behaviour)와 함께 사용할 수도 있지만 직접 사용할 수도 있습니다.

타임라인 객체는 지정한 초당 프레임 수를 기준으로 시간이 되면 각 프레임이 그려질 수 있도록 [`new-frame`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#ClutterTimeline-new-frame) 시그널을 발생합니다. 그러면 시그널 핸들러에서 액터의 속성을 변경하면 됩니다. 물론 스테이지에 있는 여러 액터의 속성을 동시에 변경할 수도 있습니다. (물론 이외에도 몇 가지 시그널이 더 있습니다)

타임라인 객체를 만드는 [`clutter_timeline_new()`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#clutter-timeline-new) 함수는 총 프레임 수(n\_frames)와 초당 프레임 수(fps)를 인수로 받습니다. 따라서 이 값을 기준으로 전체 지속 시간을 예측할 수 있습니다. 타임라인은 [`clutter_timeline_start()`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#clutter-timeline-start)를 호출해야 비로소 시작합니다. [`clutter_timeline_stop()`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#clutter-timeline-stop)을 호출하기 전까지 [`clutter_timeline_set_loop()`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#clutter-timeline-set-loop)를 이용하여 무한히 반복하게 할 수도 있습니다.

여러 타임라인을 동시에 시작하거나 정지, 또는 순서대로 실행하려면 [ClutterScore](http://clutter-project.org/docs/clutter/stable/ClutterScore.html) 객체를 사용합니다. 일종의 타임라인 제어기 역할을 하는 셈입니다. [`clutter_score_append()`](http://clutter-project.org/docs/clutter/stable/ClutterScore.html#clutter-score-append)는 두 개의 타임라인을 인수로 받는데, 첫번째는 부모(parent) 타임라인이고 두번째가 추가할 타임라인입니다. 추가되는 타임라인은 부모가 끝난 다음에 실행됩니다. 부모가 NULL인 타임라인은 [`clutter_timeline_start()`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#clutter-timeline-start)을 호출하면 바로 시작합니다.

예를 들어 다음 예제는 timeline1이 끝난 다음에 timeline2, timeline3가 동시에 시작합니다.

      ClutterTimeline *timeline1, *timeline2, *timeline3;
      ClutterScore *score;

      timeline1 = clutter_timeline_new_for_duration (1000);
      timeline2 = clutter_timeline_new_for_duration (500);
      timeline3 = clutter_timeline_new_for_duration (500);

      score = clutter_score_new ();

      clutter_score_append (score, NULL,       timeline1);
      clutter_score_append (score, timeline1, timeline2);
      clutter_score_append (score, timeline1, timeline3);

      clutter_score_start (score);

클러터 액터와 달리 타임라인 객체는 부동 참조(floating reference)가 없기 때문에 사용이 끝난 다음에는 `g_object_unref()`로 리소스를 해제해야 합니다. 사용이 끝나는 시점이 모호할 경우 [`completed`](http://clutter-project.org/docs/clutter/stable/ClutterTimeline.html#ClutterTimeline-completed) 시그널에 핸들러를 등록해서 처리해도 됩니다.

다음 예제는 하나가 끝난 뒤 다른 하나가 시작하는 두 개의 타임라인을 포함하는 하나의 스코어 객체를 보여줍니다. 이 스코어 객체는 무한히 반복하고, 첫번째 타임라인은 사각형을 회전시키며 두번째 타임라인은 사각형을 수평을 이동시킵니다.

    #include <clutter/clutter.h>
    #include <stdlib.h>

    ClutterActor *rect = NULL;

    gint rotation_angle = 0;
    gint color_change_count = 0;

    /* 사각형을 회전시키고 색상을 변경 */
    void
    on_timeline_rotation_new_frame (ClutterTimeline *timeline,
                                    gint             frame_num,
                                    gpointer         data)
    {
      rotation_angle += 1;
      if(rotation_angle >= 360)
        rotation_angle = 0;

      /* X축을 기준으로 시계방향으로 회전시킵니다. */
      clutter_actor_set_rotation (rect, CLUTTER_X_AXIS,
        rotation_angle, 0, 0, 0);

      /* 색삭을 변경합니다.
       * (This is a silly example, making the rectangle flash):
       */
      ++color_change_count;
      if(color_change_count > 100)
        color_change_count = 0;

      if(color_change_count == 0)
      {
        ClutterColor rect_color = { 0xff, 0xff, 0xff, 0x99 };
        clutter_rectangle_set_color (CLUTTER_RECTANGLE (rect), &rect_color);
      }
      else if (color_change_count == 50)
      {
        ClutterColor rect_color = { 0x10, 0x40, 0x90, 0xff };
        clutter_rectangle_set_color (CLUTTER_RECTANGLE (rect), &rect_color);
      }
    }

    /* 사각형 움직이기 */
    void
    on_timeline_move_new_frame (ClutterTimeline *timeline,
                                gint             frame_num,
                                gpointer         data)
    {
      gint x_position = clutter_actor_get_x (rect);

      x_position += 1;
      if (x_position >= 150)
        x_position = 0;

      clutter_actor_set_x (rect, x_position);
    }

    int main(int argc, char *argv[])
    {
      ClutterColor stage_color = { 0x00, 0x00, 0x00, 0xff };
      ClutterColor rect_color = { 0xff, 0xff, 0xff, 0x99 };

      clutter_init (&argc, &argv);

      /* 스테이지를 얻어 크기와 색상을 정합니다. */
      ClutterActor *stage = clutter_stage_get_default ();
      clutter_actor_set_size (stage, 200, 200);
      clutter_stage_set_color (CLUTTER_STAGE (stage), &stage_color);

      /* 스테이지에 사각형을 추가합니다. */
      rect = clutter_rectangle_new_with_color (&rect_color);
      clutter_actor_set_size (rect, 70, 70);
      clutter_actor_set_position (rect, 50, 100);
      clutter_container_add_actor (CLUTTER_CONTAINER (stage), rect);
      clutter_actor_show (rect);

      /* 스테이지를 보이게 합니다. */
      clutter_actor_show (stage);

      /* 스코어 객체를 하나 만들어 타임라인 객체 두개를 추가합니다.
       * 두번째 타임라인은 첫번째가 멈춘 뒤에 시작합니다. */
      ClutterScore *score = clutter_score_new ();
      clutter_score_set_loop (score, TRUE);

      ClutterTimeline *timeline_rotation =
        clutter_timeline_new (200 /* 총프레임수 */, 120 /* 초당프레임수(fps) */);
      g_signal_connect (timeline_rotation, "new-frame",
                        G_CALLBACK (on_timeline_rotation_new_frame), NULL);
      clutter_score_append (score, NULL, timeline_rotation);

      ClutterTimeline *timeline_move =
        clutter_timeline_new (200 /* 총프레임수 */, 120 /* 초당프레임수(fps) */);
      g_signal_connect (timeline_move, "new-frame",
                        G_CALLBACK (on_timeline_move_new_frame), NULL);
      clutter_score_append (score,  timeline_rotation, timeline_move);

      clutter_score_start (score);

      /* 메인 이벤트 루프를 시작합니다. */
      clutter_main ();

      g_object_unref (timeline_rotation);
      g_object_unref (timeline_move);
      g_object_unref (score);

      return EXIT_SUCCESS;
    }

이제 클러터를 이용한 기본적인 애니메이션 원리를 알게 되었군요.
