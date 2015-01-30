---
layout: post
title: GTK+ 투명 배경 만들기 (OSD 효과)
tags: [ GTK+ ]
---

이번에는 컴피즈와 같은 비디오 카드 3D 기능이 필요하지 않은 기법으로 OSD 효과를 만들어 보겠습니다. 물론 부드러운 반투명 배경 등의 효과는 불가능하지만, 윈도우의 특정 영역을 아예 마스킹(masking)해서 비워버리는 방식이기 때문에 마우스 입력도 통과해 버립니다. 이번에도 역시 스크린샷 먼저!

![](/figures/osd-test.png)

여기서 사용한 기법은 [gtk\_widget\_shape\_combine\_mask()](http://library.gnome.org/devel/gtk/stable/GtkWidget.html#gtk-widget-shape-combine-mask) 함수를 이용해 위젯(윈도우)의 유효 영역을 비트맵으로 지정하는 것입니다. 여기서 비트맵이란 1비트가 하나의 픽셀을 가리키는 그래픽 형식으로, 위젯의 특정 좌표 픽셀이 유효한 영역인지는 비트맵의 해당 좌표 픽셀 비트가 1인지 여부에 따라 결정됩니다. 즉, 일반적인 경우 X 서버는 사각 형태로만 윈도우 영역을 판단하지만, 비트맵 정보가 전달되면 사각형이 아닌 복잡한 형태의 윈도우로 처리하는 것입니다. 가끔 특이한 모양의 윈도우 형태를 가진 어플리케이션이 있다면 아마도 대부분 이 기능을 이용한 것입니다. 더 정확히는  X 윈도우 Shape 확장(extension)에서 제공하는 XShapeCombineMask() API를 이용합니다. (행복하게도 이 GTK+ API는 윈도우 플랫폼에서도 정상 동작합니다)

gtk\_widget\_shape\_combine\_mask() API는 비트맵 정보로 GdkBitmap 객체를 사용하는데 이 객체를 만들기 위해서는 [gdk\_bitmap\_create\_from\_data()](http://library.gnome.org/devel/gdk/stable/gdk-Bitmaps-and-Pixmaps.html#gdk-bitmap-create-from-data) 함수와 [gdk\_pixbuf\_render\_pixmap\_and\_mask\_for\_colormap()](http://library.gnome.org/devel/gdk/stable/gdk-Pixbufs.html#gdk-pixbuf-render-pixmap-and-mask-for-colormap) 함수를 사용해야 합니다. 두번째 함수는 GdkPixbuf 객체로부터 추출하는 방식이기 때문에, 이 테스트 프로그램에서는 첫번째 함수를 이용합니다. 그런데 이 함수가 이용하는 데이터가 [XBM](http://en.wikipedia.org/wiki/XBM) 형식이라서 따로 라이브러리를 사용하지 않고 간단하게 GdkPixmap에서 추출합니다. 하지만 GdkPixmap에서 직접 픽셀 정보를 얻을 수 없으므로 GdkImage 객체로 변환한 뒤에 픽셀 정보를 얻어옵니다.

카이로(cairo) API를 사용하는 것이 대세이긴 하지만 여기서는 간단하게 [GDK Drawing API](http://library.gnome.org/devel/gdk/stable/gdk-Drawing-Primitives.html)를 이용해서 사각형과 텍스트를 표시했습니다. 텍스트는 안티 엘리어싱(anti-aliasing) 효과 때문에 테두리가 약간 지저분하지만 그럭저럭 볼만합니다. 물론, 카이로 API를 이용하면 안티 엘리어싱 효과를 끌 수 있습니다. (그런데 gnome-osd 프로그램은 어떻게 해서 깔끔하게 나오는 건지 궁금하네요)

**\*\* 업데이트 (2009.03.06) \*\***

[gdk\_draw\_layout()](http://library.gnome.org/devel/gdk/stable/gdk-Drawing-Primitives.html#gdk-draw-layout)을 이용해 텍스트를 그리는 코드와 더불어 [여기](http://live.gnome.org/OutlineLabel)를 참고해 카이로(cairo) 방식도 추가해 보았습니다.

![](/figures/osd-with-cairo.png)
