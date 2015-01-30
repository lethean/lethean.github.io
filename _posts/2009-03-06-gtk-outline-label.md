---
layout: post
title: GTK+ 글자 외곽선 효과 (GtkOutlineLabel)
tags: [ GTK+ ]
---

[GTK+ 메일링 리스트](http://mail.gnome.org/archives/gtk-list/2009-March/msg00031.html)에서 카이로(cairo) API를 이용해 깔끔하게 [외곽선 효과](http://live.gnome.org/OutlineLabel)를 구현하는 방법의 글을 보고 테스트 삼아 위젯으로 만들어 보았습니다. 이름하여 'GtkOutlineLabel' 위젯, 실행 화면은 다음과 같습니다.

![](/figures/gtk-outline-label.png)

API는 간단하게 외곽선 색상과 굵기를 지정할 수 있는 기능만 있습니다. 다음은 테스트 프로그램의 일부입니다.

    int
    main (int argc, char **argv)
    {
      GtkWidget *window;
      GtkWidget *label;
      GtkWidget *vbox;

      gtk_init (&argc, &argv);

      window = gtk_window_new (GTK_WINDOW_TOPLEVEL);
      make_transparent_window (window);
      gtk_widget_show (window);

      vbox = gtk_vbox_new (FALSE, 0);
      gtk_widget_show (vbox);
      gtk_container_add (GTK_CONTAINER (window), vbox);

      label = gtk_outline_label_new ("<span font="Bold 50">Hello, 안녕?</span>");
      gtk_widget_show (label);
      gtk_box_pack_start (GTK_BOX (vbox), label, TRUE, TRUE, 0);

      label = gtk_outline_label_new ("<span font="Bold 30" color="red">Hello, 안녕?</span>");
      gtk_widget_show (label);
      gtk_box_pack_start (GTK_BOX (vbox), label, TRUE, TRUE, 0);

      gtk_outline_label_set_line_color (GTK_OUTLINE_LABEL (label), "#000000");
      gtk_outline_label_set_line_width (GTK_OUTLINE_LABEL (label), 1.0);

      gtk_main ();

      return 0;
    }
