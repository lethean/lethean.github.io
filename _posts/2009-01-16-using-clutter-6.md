---
layout: post
title: "클러터(Clutter) 사용하기 (6)"
tags: [ Clutter ]
---

지금까지 예제가 단편적이라면 이번에는 조금 제대로 된 기능하는 코드입니다. 이 예제는 이미지 파일을 읽어들여 타원 주위로 회전시키며 보여줍니다. 사용자가 이미지를 클릭하면 맨 앞으로 오면서 확대되면서 파일 이름도 보여줍니다. 먼저 스크린샷부터.

![](/figures/clutter-image-viewer.png)

여러 타임라인과 움직임 객체를 이용해서 조금 복잡해 보이지만, 아마도 실제 어플리케이션은 이보다 훨씬 더 유연하고 기능적으로 동작해야겠지요. 주석을 우리말로 번역하고, 원본보다 조금 더 속도감있게 변경한 소스는 다음과 같습니다. (조금 길지요...)

    #include <clutter/clutter.h>
    #include <stdlib.h>

    static ClutterActor *stage = NULL;

    /* 파일 이름을 보여주기 위한 레이블 액터 */
    static ClutterActor *label_filename = NULL;

    /* 모든 이미지를 타원 주위로 회전화기 위한 타임라인 */
    static ClutterTimeline *timeline_rotation = NULL;

    /* 이미지 하나를 위로 올리고 확대하기 위한 타임라인과 움직임 객체 */
    static ClutterTimeline *timeline_moveup = NULL;
    static ClutterBehaviour *behaviour_scale = NULL;
    static ClutterBehaviour *behaviour_path = NULL;
    static ClutterBehaviour *behaviour_opacity = NULL;

    /* 이미지 목록을 보여줄 타원의 좌표와 크기 */
    static const gint ELLIPSE_Y = 390;
    static const gint ELLIPSE_HEIGHT = 450; /* 90도 회전된 상태에서 앞뒤 거리 */
    static const gint IMAGE_HEIGHT = 100;

    static double angle_step = 30;

    typedef struct _Item
    {
      ClutterActor *actor;
      ClutterBehaviour *ellipse_behaviour;
      gchar* filepath;
    } Item;

    /* 맨 앞으로 오게 할 이미지 항목 */
    static Item *item_at_front = NULL;

    static GSList *list_items = NULL;

    static void rotate_all_until_item_is_at_front (Item *item);

    static gdouble
    angle_in_360 (gdouble angle)
    {
      gdouble result = angle;

      while (result >= 360)
        result -= 360;

      return result;
    }

    static void
    on_foreach_clear_list_items (gpointer data, gpointer user_data)
    {
      Item* item = (Item*)data;

      /* 액터는 스테이지가 없어질때 자동으로 정리되므로 해제할 필요가 없습니다. */
      g_object_unref (item->ellipse_behaviour);
      g_free (item->filepath);
      g_free (item);
    }

    static void
    scale_texture_default (ClutterActor *texture)
    {
      int pixbuf_height = 0;

      /* 이미지의 세로 크기를 얻습니다. */
      clutter_texture_get_base_size (CLUTTER_TEXTURE (texture),
                     NULL, &pixbuf_height);

      const gdouble scale = pixbuf_height ?
        IMAGE_HEIGHT /  (gdouble)pixbuf_height : 0;

      /* 기준 높이에 맞게 스케일링합니다. */
      clutter_actor_set_scale (texture, scale, scale);
    }

    static void
    load_images (const gchar* directory_path)
    {
      g_return_if_fail (directory_path);

      /* 현재 이미지 목록을 비웁니다. */
      g_slist_foreach (list_items, on_foreach_clear_list_items, NULL);
      g_slist_free (list_items);

      /* 새로운 목록을 초기화합니다. */
      list_items = NULL;

      /* 디렉토리에 있는 이미지 목록을 얻습니다. */
      GError *error = NULL;
      GDir* dir = g_dir_open (directory_path, 0, &error);
      if (error)
        {
          g_warning ("g_dir_open() failed: %sn", error->message);
          g_clear_error (&error);
          return;
        }

      const gchar* filename = NULL;
      while ((filename = g_dir_read_name(dir)))
        {
          gchar* path = g_build_filename (directory_path, filename, NULL);

          /* 이미지 파일로부터 텍스쳐 액터를 만듭니다. */
          ClutterActor *actor = clutter_texture_new_from_file (path, NULL);
          if (actor)
        {
          Item* item = g_new0 (Item, 1);

          item->actor = actor;
          item->filepath = g_strdup (path);

          /* 모든 이미지가 같은 높이가 되도록 스케일링합니다. */
          scale_texture_default (item->actor);

          list_items = g_slist_append (list_items, item);
        }

          g_free (path);
        }

      g_dir_close (dir);
    }

    static gboolean
    on_texture_button_press (ClutterActor *actor,
                 ClutterEvent *event,
                 gpointer      user_data)
    {
      /* 이미지 회전 타임라인이 실행중이면 이벤트를 무시합니다.
       * 즉, 이미지가 움직이고 있는 도중에 발생하는 마우스 버튼 클릭을 무시합니다.
       */
      if (timeline_rotation && clutter_timeline_is_playing (timeline_rotation))
        {
          printf ("on_texture_button_press(): ignoringn");
          return FALSE;
        }

      Item *item = (Item *) user_data;

      /* 선택한 아이템이 맨 앞에 올때까지 이미지 목록을 회전시킵니다. */
      rotate_all_until_item_is_at_front (item);

      return TRUE;
    }

    static void
    add_to_ellipse_behaviour (ClutterTimeline *timeline_rotation,
                  gdouble          start_angle,
                  Item            *item)
    {
      g_return_if_fail (timeline_rotation);

      ClutterAlpha *alpha =
        clutter_alpha_new_full (timeline_rotation,
                    CLUTTER_ALPHA_SINE_INC,
                    NULL,
                    NULL);

      /* 타원을 따라 동작할 움직임 객체를 만듭니다. */
      item->ellipse_behaviour =
        clutter_behaviour_ellipse_new (alpha,
                       320, ELLIPSE_Y,
                       ELLIPSE_HEIGHT, ELLIPSE_HEIGHT,
                       CLUTTER_ROTATE_CW,
                       angle_in_360 (start_angle),
                       angle_in_360 (start_angle + 360));

      /* X축 기준으로 타원의 축 기울입니다. */
      clutter_behaviour_ellipse_set_angle_tilt (
        CLUTTER_BEHAVIOUR_ELLIPSE (item->ellipse_behaviour),
        CLUTTER_X_AXIS,
        -90);

      /* ClutterAlpha 객체는 따로 해제할 필요가 없습니다. */

      /* 액터에 움직임을 적용합니다. */
      clutter_behaviour_apply (item->ellipse_behaviour, item->actor);
    }

    static void
    add_image_actors (void)
    {
      int x = 20;
      int y = 0;
      gdouble angle = 0;
      GSList *list = list_items;

      /* 이미지 갯수로 회전시 이미지 간격을 계산합니다. */
      if (list)
        angle_step = 360 / g_slist_length (list);

      while (list)
        {
          /* 이미지 액터를 스테이지에 넣습니다. */
          Item *item = (Item *) list->data;
          ClutterActor *actor = item->actor;
          clutter_container_add_actor (CLUTTER_CONTAINER (stage), actor);

          /* 초기 좌표를 지정합니다. */
          clutter_actor_set_position (actor, x, y);
          y += 100;

          /* 기본적으로 스테이지만 이벤트를 발생할 수 있으므로,
           * 액터도 이벤트를 발생할 수 있게 합니다.
           */
          clutter_actor_set_reactive (actor, TRUE);

          /* 버튼 클릭 시그널에 핸들러 함수를 연결합니다. */
          g_signal_connect (actor, "button-press-event",
                G_CALLBACK (on_texture_button_press), item);

          /* 타원 액터에 움직임을 추가합니다. */
          add_to_ellipse_behaviour (timeline_rotation, angle, item);
          angle += angle_step;

          clutter_actor_show (actor);

          list = g_slist_next (list);
        }
    }

    /* 이 기그널 핸들러는 선택한 이미지가 확대되어 위로 이동하는 타임라인이
     * 완료되었을때 호출됩니다.
     */
    static void
    on_timeline_moveup_completed (ClutterTimeline* timeline,
                                  gpointer         user_data)
    {
      /* 타임라인 객체를 해제합니다. */
      g_object_unref (timeline_moveup);
      timeline_moveup = NULL;

      g_object_unref (behaviour_scale);
      behaviour_scale = NULL;

      g_object_unref (behaviour_path);
      behaviour_path = NULL;

      g_object_unref (behaviour_opacity);
      behaviour_opacity = NULL;
    }

    /* 이 시그널 핸들러는 이미지가 타원을 따라 회전하는 타임라인이 완료되었을때
     * 호출됩니다.
     */
    static void
    on_timeline_rotation_completed (ClutterTimeline* timeline,
                                    gpointer         user_data)
    {
      /* 모든 이미지가 회전하다가 클릭한 이미지가 맨 앞에 온 상태입니다.
       * 이제 맨 앞의 이미지를 크게 보여주고 파일 이름도 표시합니다.
       */

      /* 이미지를 변형합니다. */
      ClutterActor *actor = item_at_front->actor;
      timeline_moveup = clutter_timeline_new(15 /* frames */,
                         30 /* frames per second */);
      ClutterAlpha *alpha = clutter_alpha_new_full (timeline_moveup,
                            CLUTTER_ALPHA_SINE_INC,
                            NULL, NULL);

      /* 현재 크기에서 약 2배 크기로 확대합니다. */
      gdouble scale_start = 0;
      clutter_actor_get_scale (actor, &scale_start, NULL);
      const gdouble scale_end = scale_start * 1.8;

      behaviour_scale =
        clutter_behaviour_scale_new (alpha,
                     scale_start, scale_start,
                     scale_end, scale_end);
      clutter_behaviour_apply (behaviour_scale, actor);

      /* 그림을 위 방향을 이동합니다. */
      ClutterKnot knots[2];
      knots[0].x = clutter_actor_get_x (actor);
      knots[0].y = clutter_actor_get_y (actor);
      knots[1].x = knots[0].x;
      knots[1].y = knots[0].y - 250;
      behaviour_path =
        clutter_behaviour_path_new (alpha, knots, G_N_ELEMENTS(knots));
      clutter_behaviour_apply (behaviour_path, actor);

      /* 파일 이름을 조금씩 보여줍니다. */
      clutter_label_set_text (CLUTTER_LABEL (label_filename),
                  item_at_front->filepath);
      behaviour_opacity = clutter_behaviour_opacity_new (alpha, 0, 255);
      clutter_behaviour_apply (behaviour_opacity, label_filename);

      /* 모든 움직임(behaviours)을 시작합니다.
       * 또한 완료되었을때 핸들러를 연결합니다.
       */
      g_signal_connect (timeline_moveup, "completed",
                        G_CALLBACK (on_timeline_moveup_completed), NULL);
      clutter_timeline_start (timeline_moveup);
    }

    static void
    rotate_all_until_item_is_at_front (Item *item)
    {
      g_return_if_fail (item);

      clutter_timeline_stop(timeline_rotation);

      /* 선택한 이미지를 위로 올려 보여주는 타임라인이 동작중이라면
       * 당장 멈추게 합니다.
       */
      if (timeline_moveup)
        clutter_timeline_stop (timeline_moveup);

      clutter_actor_set_opacity (label_filename, 0);

      /* 선택한 이미지 항목의 번호를 얻습니다. */
      const gint pos = g_slist_index (list_items, item);

      g_assert (pos != -1);

      if (!item_at_front && list_items)
        item_at_front = (Item *) list_items->data;

      /* 현재 맨 앞에 있는 항목의 번호를 얻습니다. */
      gint pos_front = 0;
      if (item_at_front)
        pos_front = g_slist_index (list_items, item_at_front);

      g_assert (pos_front != -1);

      /* 첫번째 항목의 시작 / 끝 각도를 계산합니다. */
      const gdouble angle_front = 180;
      gdouble angle_start = angle_front - (angle_step * pos_front);
      angle_start = angle_in_360 (angle_start);
      gdouble angle_end = angle_front - (angle_step * pos);

      gdouble angle_diff = 0;

      /* 모든 이미지 항목의 끝 각도를 설정합니다. */
      GSList *list = list_items;
      while (list)
        {
          Item *this_item = (Item*)list->data;

          /* 이미지 크기를 원래대로 되돌립니다. */
          scale_texture_default (this_item->actor);

          angle_start = angle_in_360 (angle_start);
          angle_end = angle_in_360 (angle_end);

          /* 현재 맨 앞 있는 항목일 경우 360도 더 회전하게 함니다. */
          if (item_at_front == item)
        angle_end += 360;
          angle_end = angle_in_360 (angle_end);

          clutter_behaviour_ellipse_set_angle_start (
            CLUTTER_BEHAVIOUR_ELLIPSE (this_item->ellipse_behaviour),
        angle_start);

          clutter_behaviour_ellipse_set_angle_end (
            CLUTTER_BEHAVIOUR_ELLIPSE (this_item->ellipse_behaviour), angle_end);

          /* 선택한 항목일 경우 */
          if (this_item == item)
        {
          if (angle_start < angle_end)
            angle_diff =  angle_end - angle_start;
          else
            angle_diff = 360 - (angle_start - angle_end);
        }

          /* TODO: Set the number of frames, depending on the angle.
           * otherwise the actor will take the same amount of time to reach
           * the end angle regardless of how far it must move, causing it to
           * move very slowly if it does not have far to move.
           */
          angle_end += angle_step;
          angle_start += angle_step;
          list = g_slist_next (list);
        }

      /* 속도는 같지만 이동해야 할 거리만큼 프레임 수를 조절합니다. */
      gint pos_to_move = 0;
      if (pos_front < pos)
        {
          const gint count = g_slist_length (list_items);
          pos_to_move = count + (pos - pos_front);
        }
      else
        {
          pos_to_move = pos_front - pos;
        }

      clutter_timeline_set_n_frames (timeline_rotation, angle_diff / 5);

      /* 타임라인이 끝난뒤 맨 앞에 위치해야할 항목을 기억합니다. */
      item_at_front = item;

      clutter_timeline_start (timeline_rotation);
    }

    int
    main (int argc, char *argv[])
    {
      ClutterColor stage_color = { 0xB0, 0xB0, 0xB0, 0xff }; /* light gray */

      clutter_init (&argc, &argv);

      /* 스테이지를 얻어 크기와 색상을 정합니다. */
      stage = clutter_stage_get_default ();
      clutter_actor_set_size (stage, 800, 600);
      clutter_stage_set_color (CLUTTER_STAGE (stage), &stage_color);

      /* 파일 이름을 보여주기 위한 레이블 액터를 만들고, 일단 안보이게 합니다. */
      label_filename = clutter_label_new ();
      ClutterColor label_color = { 0x60, 0x60, 0x90, 0xff }; /* blueish */
      clutter_label_set_color (CLUTTER_LABEL (label_filename), &label_color);
      clutter_label_set_font_name (CLUTTER_LABEL (label_filename), "Sans 24");
      clutter_actor_set_position (label_filename, 10, 10);
      clutter_actor_set_opacity (label_filename, 0); /* hidden */
      clutter_container_add_actor (CLUTTER_CONTAINER (stage), label_filename);
      clutter_actor_show (label_filename);

      /* 이미지 목록 밑에 보여줄 사각형을 만듭니다. */
      ClutterColor rect_color = { 0xff, 0xff, 0xff, 0xff }; /* white */
      ClutterActor *rect = clutter_rectangle_new_with_color (&rect_color);
      clutter_actor_set_height (rect, ELLIPSE_HEIGHT + 20);
      clutter_actor_set_width (rect, clutter_actor_get_width (stage) + 100);

      /* 사각형을 이미지 목록 밑에 위치하도록 합니다. */
      clutter_actor_set_position (rect,
        -(clutter_actor_get_width (rect) - clutter_actor_get_width (stage)) / 2,
        ELLIPSE_Y + IMAGE_HEIGHT - (clutter_actor_get_height (rect) / 2));

      /* X축을 기준으로 사각형을 눕힙니다. */
      clutter_actor_set_rotation (rect, CLUTTER_X_AXIS, -90,
                      0, (clutter_actor_get_height (rect) / 2), 0);
      clutter_container_add_actor (CLUTTER_CONTAINER (stage), rect);
      clutter_actor_show (rect);

      /* 스테이지를 보이게 합니다. */
      clutter_actor_show (stage);

      /** 이미지를 회전시킬 타임라인을 만들고,
       * 회전이 끝났을때 실행할 핸들러를 연결합니다.
       */
      timeline_rotation = clutter_timeline_new(60 /* frames */,
                           30 /* frames per second */);
      g_signal_connect (timeline_rotation, "completed",
                G_CALLBACK (on_timeline_rotation_completed), NULL);

      /* 이미지를 로드하고 각각에 대한 액터를 만듭니다. */
      load_images ("./images/");
      add_image_actors ();

      /* clutter_timeline_set_loop(timeline_rotation, TRUE); */

      /* 첫번째 이미지가 선택된 것처럼 회전을 시작합니다. */
      if (list_items)
        rotate_all_until_item_is_at_front ((Item*)list_items->data);

      /* 메인 이벤트 루프를 시작합니다. */
      clutter_main ();

      /* 모든 이미지 목록을 해제합니다. */
      g_slist_foreach(list_items, on_foreach_clear_list_items, NULL);
      g_slist_free (list_items);

      g_object_unref (timeline_rotation);

      return EXIT_SUCCESS;
    }
