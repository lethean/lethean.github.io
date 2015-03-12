---
layout: post
title: Cairo 라이브러리를 이용해 Font Awesome 아이콘 그리기
tags: [ Cairo, Clutter, FontAwesome ]
---

요즘 웹에는 글꼴 아이콘(font icon)을 많이 사용합니다. 벡터 그래픽 방식이라
확대해도 좋은 품질을 보여주고, 자주 사용되는 다양한 모양도 많이 제공됩니다.
덕분에 작은 규모의 팀에서 간단한 웹 사이트나 웹앱을 개발할 때, 디자이너의
도움(?)이 없어도 되기 때문에, 시간과 노력을 많이 절약하게 됩니다. 특히
라이센스도 자유로우면서 다양한 아이콘을 제공하는 글꼴 아이콘도 많은데,
대표적으로 [Font Awesome], [WebHostingHub Glyphs] 등이 있습니다.
물론 [구글 검색을 해보면][1] 유료제품도 있고, 직접 만들어주는 도구까지 있습니다.

그런데 글꼴 아이콘은 HTML과 CSS를 이용하는 웹 페이지에서는 쉽게 이용할 수
있지만, 데스크톱 / 임베디드 프로그램에서 사용하려면 약간 다른 방법을 사용해야
합니다. 이 글은 여러 방법 중에서 GTK+, Clutter 라이브러리에서 이용하는
[Cairo] 라이브러리를 이용해 Font Awesome 아이콘을 그리는 방법을 설명합니다.

생각해 보면 간단할 것 같은데, 막상 Cairo API를 뒤져보면 글꼴 파일을 직접
지정하는 방법이 없습니다. 막강한 기능을 제공하는 [Pango] 라이브러리 역시 글꼴
파일을 지정하는 함수는 없습니다. 모두 [Fontconfig]를 이용해 아이콘 글꼴 파일을
지정해야 합니다. 하지만 프로그램과 함께 배포해야 하는 아이콘 글꼴 파일을 시스템
글꼴 디렉터리나 사용자 글꼴 디렉터리에 복사하는 방식은 번거롭기도 하고 불가능한
경우도 있어서 가능하다면 글꼴 파일을 직접 사용하는 방법을 찾아보았습니다.

그래서 찾은 방법은 다음과 같습니다.

먼저 [FreeType] 라이브러리를 이용해 글꼴 파일을 읽은 후
[`cairo_ft_font_face_create_for_ft_face()`][cairo_ft_font_face_create_for_ft_face()]
함수를 이용해 Cairo 글꼴을 만듭니다.

```c
static cairo_font_face_t *
get_font_face (void)
{
  static cairo_font_face_t *font_face = NULL;

  if (!font_face)
    {
      FT_Library ft_library;
      FT_Face ft_face;
      FT_Error ft_error;

      ft_error = FT_Init_FreeType (&ft_library);
      if (ft_error)
        {
          g_warning ("FT_Init_FreeType() failure: %d", ft_error);
          return NULL;
        }
      ft_error = FT_New_Face (ft_library, "FontAwesome.otf", 0, &ft_face);
      if (ft_error)
        {
          g_warning ("FT_New_Face() failure: %d", ft_error);
          FT_Done_FreeType (ft_library);
          return NULL;
        }

      font_face = cairo_ft_font_face_create_for_ft_face (ft_face, 0);
    }

  return font_face;
}
```

이제 Cairo 글꼴 함수를 이용해 아이콘을 그리면 됩니다. 참고로, 아래 코드에서는
[`ClutterColor`][ClutterColor] 구조체를 이용해 아이콘 색상과 그림자 색상을
지정합니다.

```c
static void
draw_font_icon (cairo_t            *cr,
                double              x,
                double              y,
                int                 centering,
                const char         *code,
                double              size,
                const ClutterColor *color,
                const ClutterColor *shadow_color,
                double              shadow_offset)
{
  cairo_font_options_t *options;

  cairo_save (cr);

  options = cairo_font_options_create ();
  cairo_font_options_set_antialias (options, CAIRO_ANTIALIAS_GRAY);
  cairo_font_options_set_hint_style (options, CAIRO_HINT_STYLE_FULL);
  cairo_set_font_options (cr, options);
  cairo_font_options_destroy (options);

  cairo_set_font_face (cr, get_font_face ());
  cairo_set_font_size (cr, size);

  {
    cairo_text_extents_t extents;

    cairo_text_extents (cr, code, &extents);
    if (!centering)
      {
        x -= extents.x_bearing;
        y -= extents.y_bearing;
      }
    else
      {
        x += (extents.x_bearing -extents.width * 0.5);
        y += (-extents.y_bearing -extents.height * 0.5);
      }
  }

  if (shadow_color && shadow_color->alpha > 0 && shadow_offset != 0.0)
    {
      cairo_save (cr);
      cairo_move_to (cr, x + shadow_offset, y + shadow_offset);
      clutter_cairo_set_source_color (cr, shadow_color);
      cairo_show_text (cr, code);
      cairo_restore (cr);
    }

  clutter_cairo_set_source_color (cr, color);
  cairo_move_to (cr, x, y);
  cairo_show_text (cr, code);

  cairo_restore (cr);
}
```

아이콘을 회전하려면 다음과 같이 미리 캔버스를 돌린 후에 아이콘을 그리면 됩니다.
예를 들어, [`ClutterCanvas::draw`][ClutterCanvas::draw] 시그널 핸들러는 다음과
같이 작성합니다.

```c
static gboolean
draw_icon_content (ClutterCanvas *canvas,
                   cairo_t       *cr,
                   gint           width,
                   gint           height,
                   gpointer       user_data)
{
  const double rotate_degree = 180.0;

  /* Clear the contents of the canvas. */
  cairo_save (cr);
  cairo_set_operator (cr, CAIRO_OPERATOR_CLEAR);
  cairo_paint (cr);
  cairo_restore (cr);

  if (rotate_degree > 0.0)
    {
      cairo_translate (cr, width / 2, height / 2);
      cairo_rotate (cr, rotate_degree * (G_PI / 180.0));
      cairo_translate (cr, - width / 2, - height / 2);
    }
  draw_font_icon (cr, ...);

  return TRUE;
}
```

그런데
[Font Awesome 아이콘 목록]에서 원하는 아이콘은 찾았는데, 아이콘 이름에 해당하는
실제 글꼴의 문자 코드(code)는 어떻게 알 수 있을까요? 웹 페이지를 무식하게
파싱해서 C 언어 헤더 파일을 자동으로 생성하는 [Node.js 스크립트][2]를 이용하면
다음과 같은 결과를 얻을 수 있습니다.

```c
#define FA_ICON_ADJUST "\xef\x81\x82" /* &#xf042; */
#define FA_ICON_ADN "\xef\x85\xb0" /* &#xf170; */
#define FA_ICON_ALIGN_CENTER "\xef\x80\xb7" /* &#xf037; */
#define FA_ICON_ALIGN_JUSTIFY "\xef\x80\xb9" /* &#xf039; */
#define FA_ICON_ALIGN_LEFT "\xef\x80\xb6" /* &#xf036; */
#define FA_ICON_ALIGN_RIGHT "\xef\x80\xb8" /* &#xf038; */
#define FA_ICON_AMBULANCE "\xef\x83\xb9" /* &#xf0f9; */
#define FA_ICON_ANCHOR "\xef\x84\xbd" /* &#xf13d; */
#define FA_ICON_ANDROID "\xef\x85\xbb" /* &#xf17b; */
#define FA_ICON_ANGELLIST "\xef\x88\x89" /* &#xf209; */
```

:)

[1]: https://www.google.co.kr/search?q=font+icon
[2]: https://gist.github.com/lethean/bdd5a657f103b6cb0c23
[cairo_ft_font_face_create_for_ft_face()]: http://cairographics.org/manual/cairo-FreeType-Fonts.html#cairo-ft-font-face-create-for-ft-face
[Cairo]: http://cairographics.org/
[ClutterCanvas::draw]: https://developer.gnome.org/clutter/stable/ClutterCanvas.html#ClutterCanvas-draw
[ClutterColor]: https://developer.gnome.org/clutter/stable/clutter-Colors.html#ClutterColor
[Font Awesome 아이콘 목록]: http://fortawesome.github.io/Font-Awesome/icons/
[Font Awesome]: http://fortawesome.github.io/Font-Awesome/
[Fontconfig]: http://www.freedesktop.org/wiki/Software/fontconfig/
[FreeType]: http://freetype.org/
[Pango]: http://www.pango.org/
[WebHostingHub Glyphs]: http://www.webhostinghub.com/glyphs/
