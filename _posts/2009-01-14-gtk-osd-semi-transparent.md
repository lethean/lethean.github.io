---
layout: post
title: GTK+ 반투명 배경 만들기 (OSD 효과)
tags: [ GTK+ ]
---

[PyGTK를 이용해 데스크탑 위젯을 만드는 블로그](http://bloc.eurion.net/archives/2009/standalone-pygtk-desktop-widgets/)를 보고 이를 C 언어로 바꾸어 보았습니다. Compiz나 Metacity의 컴포지팅(compositing) 기능이 활성화되어 있을 경우에만 제대로 동작합니다. 먼저 스크린샷부터.

![](/figures/trans-test.png)

여기서 사용한 기법을 요약하면, 윈도우 바탕에 RGBA 컬러맵을 할당하고 배경화면을 직접 그리도록 설정한뒤 "expose-event" 발생시 Cairo API를 이용하여 원하는 투명 배경을 그리는 겁니다. 소스 코드는 다음과 같습니다.

    #include <gtk/gtk.h>

    static void
    make_desktop_window (GtkWidget *window)
    {
      gtk_window_set_type_hint (GTK_WINDOW (window), GDK_WINDOW_TYPE_HINT_DOCK);
      gtk_window_set_keep_below (GTK_WINDOW (window), TRUE);
      gtk_window_set_decorated (GTK_WINDOW (window), FALSE);
      gtk_window_stick (GTK_WINDOW (window));
    }

    static gboolean
    window_exposed (GtkWidget      *widget,
                    GdkEventExpose *event,
                    gpointer        user_data)
    {
      cairo_t   *cr;

      cr = gdk_cairo_create (widget->window);

      /* Make it transparent */
      cairo_set_operator (cr, CAIRO_OPERATOR_CLEAR);
      gdk_cairo_region (cr, event->region);
      cairo_fill_preserve (cr);

      /* Make it semi-transparent */
      cairo_set_source_rgba (cr, 0.0, 0.0, 0.0, 0.5);
      cairo_set_operator (cr, CAIRO_OPERATOR_OVER);
      cairo_fill (cr);

      cairo_destroy (cr);

      return FALSE;
    }

    static void
    make_transparent_window (GtkWidget *window)
    {
      GdkScreen *screen;
      GdkColormap *colormap;

      screen = gtk_widget_get_screen (window);
      if (!screen)
        {
          g_warning ("failed to get window's screen");
          return;
        }

      colormap = gdk_screen_get_rgba_colormap (screen);
      if (!colormap)
        {
          g_warning ("failed to get RGBA colormap");
          return;
        }

      gtk_widget_set_colormap (window, colormap);
      gtk_widget_set_app_paintable (window, TRUE);
      g_signal_connect (window, "expose-event", G_CALLBACK (window_exposed), NULL);
    }

    static void
    button_clicked (GtkButton *button,
    gpointer   user_data)
    {
      static gint counter = 0;
      GtkWidget  *label = user_data;
      gchar      *str;

      str = g_strdup_printf ("<span size='larger' " 
                             "weight='bold' " 
                             "color='white'>안녕하신가? %d</span>",
                             ++counter);
      gtk_label_set_markup (GTK_LABEL (label), str);
      g_free (str);
    }

    int
    main (int argc, char **argv)
    {
      GtkWidget *window;
      GtkWidget *vbox;
      GtkWidget *label;
      GtkWidget *button;

      gtk_init (&argc, &argv);

      window = gtk_window_new (GTK_WINDOW_TOPLEVEL);
      make_transparent_window (window);
      if (0) make_desktop_window (window);
      gtk_widget_show (window);

      vbox = gtk_vbox_new (TRUE, 10);
      gtk_widget_show (vbox);
      gtk_container_add (GTK_CONTAINER (window), vbox);

      label = gtk_label_new ("안녕하신가?");
      gtk_widget_show (label);
      gtk_box_pack_start (GTK_BOX (vbox), label, TRUE, TRUE, 10);

      button = gtk_button_new_with_label ("Click Here!");
      gtk_widget_show (button);
      gtk_box_pack_start (GTK_BOX (vbox), button, FALSE, FALSE, 10);

      g_signal_connect (button, "clicked", G_CALLBACK (button_clicked), label);

      gtk_main ();

      return 0;
    }

참고로, 이를 더 응용하면 필요한 위젯은 모두  반투명하게 만들 수도 있겠지만, 제대로 하려면 Murrine 등과 같은 테마 엔진처럼 GTK+ 테마에서 처리하는게 더 바른 선택일 것 같습니다.

**[업데이트 - 2009.01.16]
**

GTK+ 테마 엔진 "pixmap"을 이용하고 투명도를 지원하는 PNG / SVG 형식의 이미지를 사용하면 버튼 등과 같은 위젯도 투명도가 적용됩니다. 다음은 위 테스트 프로그램을 이러한 테마를 이용했을때 스크린샷입니다.

![](/figures/gtk-pixmap-engine-transpareny.png)

GTK+ 테마 엔진을 새로 구현할 필요없이 pixmap 엔진만 사용해도 어플리케이션 전체적으로 반투명 효과를 얻을 수 있습니다. 그런데 pixmap 엔진 관련된 문서가 별로 없어서... 그나마 찾아낸 건 이 정도 뿐입니다.

-   [http://live.gnome.org/GnomeArt/Tutorials/GtkThemes](//live.gnome.org/GnomeArt/Tutorials/GtkThemes)
-   [https://stage.maemo.org/svn/maemo/projects/haf/trunk/sapwood/README](//stage.maemo.org/svn/maemo/projects/haf/trunk/sapwood/README)
-   [http://www.nabble.com/Pixmap-Theme-Engine-options-td18393063.html](//www.nabble.com/Pixmap-Theme-Engine-options-td18393063.html)

