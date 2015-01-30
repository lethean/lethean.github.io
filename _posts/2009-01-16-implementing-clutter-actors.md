---
layout: post
title: Clutter 액터 구현하기 (Implementing Actors)
tags: [ Clutter ]
---

[(클러터 튜토리얼](http://www.openismus.com/documents/clutter_tutorial/0.8/docs/tutorial/html/index.html) 내용이 조금 더 남아 있지만 여기까지만 정리할 생각입니다. 나머지는 아직까지 관심 밖이라서... 아무튼 [클러터 API 문서](http://www.clutter-project.org/docs/clutter/stable/)는 대강이라도 한 번 훑어봐야 더 정확하게 사용할 수 있을 것 같습니다)

클러터가 제공하는 액터만으로 뭔가 부족함을 느낀다면 이제 직접 새로운 액터를 구현할 때입니다. 새로운 액터를 구현하는 작업은 GTK+ 위젯처럼 GObject 기반 객체를 만드는 과정과 거의 비슷합니다. 제일 먼저 `G_DEFINE_TYPE()` 매크로를 이용해 ClutterActor 파생 객체를 정의합니다. 예를 들어 새로운 객체 이름이 ClutterTriangle이라면 다음과 같습니다.

    G_DEFINE_TYPE (ClutterTriangle, clutter_triangle, CLUTTER_TYPE_ACTOR);

그리고 `ClutterActor::paint()` 가상 함수를 구현합니다.

    static void
    clutter_triangle_class_init (ClutterTriangleClass *klass)
    {
      ClutterActorClass *actor_class = CLUTTER_ACTOR_CLASS (klass);

      actor_class->paint = clutter_triangle_paint;
      ...
    }

기본적인 액터 정보는 ClutterActor 기본 클래스에서 얻을 수 있습니다. 가령 [`clutter_actor_get_geometry()`](http://www.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-get-geometry) 함수로 좌표와 크기를 얻을 수 있고, [`clutter_actor_get_opacity()`](http://www.clutter-project.org/docs/clutter/stable/ClutterActor.html#clutter-actor-get-opacity) 함수로 불투명도를 얻을 수 있습니다.

ClutterActor::paint() 함수에서 실제 그리기 작업을 구현할때는 OpenGL API를 사용해야 합니다. OpenGL ES나 일반적인 OpenGL 환경에서 문제없이 동작하게 하려면 클러터에서 제공하는 COGL API를 사용합니다. 예를 들면 [`cogl_rectangle()`](http://www.clutter-project.org/docs/cogl/stable/cogl-Primitives.html#cogl-rectangle)이나 [`cogl_push_matrix()`](http://www.clutter-project.org/docs/cogl/stable/cogl-General-API.html#cogl-push-matrix) 등이 그것입니다.

그리기 함수와 더불어 지정한 색상으로 실루엣(silhouette)을 그려주는 `ClutterActor::pick()` 가상 함수도 구현해야 합니다. 클러터는 이 함수를 이용해 스크린외영역(offscreen)에 단일한 색상으로 모든 액터의 실루엣을 그려 놓은 뒤 커서가 위치한 좌표의 액터가 무엇인지 확인하는데 사용합니다. 따라서 새로운 액터가 단순하다면 그리기 함수에서 사용한 코드를 그대로 사용해도 됩니다.

나머지 대부분의 가상 함수는 필요한 경우에만 다시 구현하면 됩니다.

다음 예제는 삼각형을 그려주는 액터를 구현한 코드입니다.

파일 : triangle-actor.h

    #ifndef _CLUTTER_TUTORIAL_TRIANGLE_ACTOR_H
    #define _CLUTTER_TUTORIAL_TRIANGLE_ACTOR_H

    #include <glib-object.h>
    #include <clutter/clutter-actor.h>
    #include <clutter/clutter-color.h>

    G_BEGIN_DECLS

    #define CLUTTER_TYPE_TRIANGLE clutter_triangle_get_type()

    #define CLUTTER_TRIANGLE(obj) 
      (G_TYPE_CHECK_INSTANCE_CAST ((obj), 
      CLUTTER_TYPE_TRIANGLE, ClutterTriangle))

    #define CLUTTER_TRIANGLE_CLASS(klass) 
      (G_TYPE_CHECK_CLASS_CAST ((klass), 
      CLUTTER_TYPE_TRIANGLE, ClutterTriangleClass))

    #define CLUTTER_IS_TRIANGLE(obj) 
      (G_TYPE_CHECK_INSTANCE_TYPE ((obj), 
      CLUTTER_TYPE_TRIANGLE))

    #define CLUTTER_IS_TRIANGLE_CLASS(klass) 
      (G_TYPE_CHECK_CLASS_TYPE ((klass), 
      CLUTTER_TYPE_TRIANGLE))

    #define CLUTTER_TRIANGLE_GET_CLASS(obj) 
      (G_TYPE_INSTANCE_GET_CLASS ((obj), 
      CLUTTER_TYPE_TRIANGLE, ClutterTriangleClass))

    typedef struct _ClutterTriangle        ClutterTriangle;
    typedef struct _ClutterTriangleClass   ClutterTriangleClass;
    typedef struct _ClutterTrianglePrivate ClutterTrianglePrivate;

    struct _ClutterTriangle
    {
      ClutterActor           parent;

      /*< private >*/
      ClutterTrianglePrivate *priv;
    }; 

    struct _ClutterTriangleClass
    {
      ClutterActorClass parent_class;
    };

    GType clutter_triangle_get_type (void) G_GNUC_CONST;

    ClutterActor *clutter_triangle_new              (void);
    ClutterActor *clutter_triangle_new_with_color   (const ClutterColor *color);

    void          clutter_triangle_get_color        (ClutterTriangle   *triangle,
                                                     ClutterColor       *color);
    void          clutter_triangle_set_color        (ClutterTriangle   *triangle,
                                                     const ClutterColor *color);

    G_END_DECLS

    #endif

파일 : triangle-actor.c

    #include "triangle_actor.h"
    #include <cogl/cogl.h>

    G_DEFINE_TYPE (ClutterTriangle, clutter_triangle, CLUTTER_TYPE_ACTOR);

    enum
    {
      PROP_0,
      PROP_COLOR
    };

    #define CLUTTER_TRIANGLE_GET_PRIVATE(obj) 
      (G_TYPE_INSTANCE_GET_PRIVATE ((obj), 
       CLUTTER_TYPE_TRIANGLE, ClutterTrianglePrivate))

    struct _ClutterTrianglePrivate
    {
      ClutterColor color;
    };

    static void
    do_triangle_paint (ClutterActor *self, const ClutterColor *color)
    {
      ClutterTriangle        *triangle = CLUTTER_TRIANGLE(self);
      ClutterTrianglePrivate *priv;
      ClutterGeometry         geom;
      ClutterFixed            coords[6];

      triangle = CLUTTER_TRIANGLE(self);
      priv = triangle->priv;

      cogl_push_matrix();

      clutter_actor_get_geometry (self, &geom);

      cogl_color (color);

      /* Paint a triangle:
       *
       * The parent paint call will have translated us into position so
       * paint from 0, 0 */
      coords[0] = CLUTTER_INT_TO_FIXED (0);
      coords[1] = CLUTTER_INT_TO_FIXED (0);

      coords[2] = CLUTTER_INT_TO_FIXED (0);
      coords[3] = CLUTTER_INT_TO_FIXED (geom.height);

      coords[4] = CLUTTER_INT_TO_FIXED (geom.width);
      coords[5] = CLUTTER_INT_TO_FIXED (geom.height);

      cogl_path_polygon (coords, 3);
      cogl_path_fill ();

      cogl_pop_matrix();
    }

    static void
    clutter_triangle_paint (ClutterActor *self)
    {
      ClutterTriangle *triangle = CLUTTER_TRIANGLE(self);
      ClutterTrianglePrivate *priv = triangle->priv;

      /* Paint the triangle with the actor's color: */
      ClutterColor color;
      color.red   = priv->color.red;
      color.green = priv->color.green;
      color.blue  = priv->color.blue;
      color.alpha = clutter_actor_get_opacity (self);

      do_triangle_paint (self, &color);
    }

    static void
    clutter_triangle_pick (ClutterActor *self, const ClutterColor *color)
    {
      /* Paint the triangle with the pick color, offscreen.
         This is used by Clutter to detect the actor under the cursor
         by identifying the unique color under the cursor. */
      do_triangle_paint (self, color);
    }

    static void
    clutter_triangle_set_property (GObject      *object,
                                   guint         prop_id,
                                   const GValue *value,
                                   GParamSpec   *pspec)
    {
      ClutterTriangle *triangle = CLUTTER_TRIANGLE(object);

      switch (prop_id)
        {
        case PROP_COLOR:
          clutter_triangle_set_color (triangle, g_value_get_boxed (value));
          break;
        default:
          G_OBJECT_WARN_INVALID_PROPERTY_ID (object, prop_id, pspec);
          break;
      }
    }

    static void
    clutter_triangle_get_property (GObject    *object,
                                   guint       prop_id,
                                   GValue     *value,
                                   GParamSpec *pspec)
    {
      ClutterTriangle *triangle = CLUTTER_TRIANGLE(object);
      ClutterColor     color;

      switch (prop_id)
        {
        case PROP_COLOR:
          clutter_triangle_get_color (triangle, &color);
          g_value_set_boxed (value, &color);
          break;
        default:
          G_OBJECT_WARN_INVALID_PROPERTY_ID (object, prop_id, pspec);
          break;
        }
    }

    static void
    clutter_triangle_finalize (GObject *object)
    {
      G_OBJECT_CLASS (clutter_triangle_parent_class)->finalize (object);
    }

    static void
    clutter_triangle_dispose (GObject *object)
    {
      G_OBJECT_CLASS (clutter_triangle_parent_class)->dispose (object);
    }

    static void
    clutter_triangle_class_init (ClutterTriangleClass *klass)
    {
      GObjectClass        *gobject_class = G_OBJECT_CLASS (klass);
      ClutterActorClass *actor_class = CLUTTER_ACTOR_CLASS (klass);

      /* Provide implementations for ClutterActor vfuncs: */
      actor_class->paint = clutter_triangle_paint;
      actor_class->pick = clutter_triangle_pick;

      gobject_class->finalize     = clutter_triangle_finalize;
      gobject_class->dispose      = clutter_triangle_dispose;
      gobject_class->set_property = clutter_triangle_set_property;
      gobject_class->get_property = clutter_triangle_get_property;

      /**
       * ClutterTriangle:color:
       *
       * The color of the triangle.
       */
      g_object_class_install_property (gobject_class,
                                       PROP_COLOR,
                   g_param_spec_boxed ("color",
                                       "Color",
                                       "The color of the triangle",
                                       CLUTTER_TYPE_COLOR,
                                       G_PARAM_READABLE | G_PARAM_WRITABLE));

      g_type_class_add_private (gobject_class, sizeof (ClutterTrianglePrivate));
    }

    static void
    clutter_triangle_init (ClutterTriangle *self)
    {
      ClutterTrianglePrivate *priv;

      self->priv = priv = CLUTTER_TRIANGLE_GET_PRIVATE (self);

      priv->color.red = 0xff;
      priv->color.green = 0xff;
      priv->color.blue = 0xff;
      priv->color.alpha = 0xff;
    }

    /**
     * clutter_triangle_new:
     *
     * Creates a new #ClutterActor with a rectangular shape.
     *
     * Return value: a new #ClutterActor
     */
    ClutterActor*
    clutter_triangle_new (void)
    {
      return g_object_new (CLUTTER_TYPE_TRIANGLE, NULL);
    }

    /**
     * clutter_triangle_new_with_color:
     * @color: a #ClutterColor
     *
     * Creates a new #ClutterActor with a rectangular shape
     * and with @color.
     *
     * Return value: a new #ClutterActor
     */
    ClutterActor *
    clutter_triangle_new_with_color (const ClutterColor *color)
    {
      return g_object_new (CLUTTER_TYPE_TRIANGLE,
                   "color", color,
                   NULL);
    }

    /**
     * clutter_triangle_get_color:
     * @triangle: a #ClutterTriangle
     * @color: return location for a #ClutterColor
     *
     * Retrieves the color of @triangle.
     */
    void
    clutter_triangle_get_color (ClutterTriangle *triangle,
                                ClutterColor     *color)
    {
      ClutterTrianglePrivate *priv;

      g_return_if_fail (CLUTTER_IS_TRIANGLE (triangle));
      g_return_if_fail (color != NULL);

      priv = triangle->priv;

      color->red = priv->color.red;
      color->green = priv->color.green;
      color->blue = priv->color.blue;
      color->alpha = priv->color.alpha;
    }

    /**
     * clutter_triangle_set_color:
     * @triangle: a #ClutterTriangle
     * @color: a #ClutterColor
     *
     * Sets the color of @triangle.
     */
    void
    clutter_triangle_set_color (ClutterTriangle   *triangle,
                     const ClutterColor *color)
    {
      ClutterTrianglePrivate *priv;

      g_return_if_fail (CLUTTER_IS_TRIANGLE (triangle));
      g_return_if_fail (color != NULL);

      g_object_ref (triangle);

      priv = triangle->priv;

      priv->color.red = color->red;
      priv->color.green = color->green;
      priv->color.blue = color->blue;
      priv->color.alpha = color->alpha;

      clutter_actor_set_opacity (CLUTTER_ACTOR (triangle),
                                 priv->color.alpha);

      if (CLUTTER_ACTOR_IS_VISIBLE (CLUTTER_ACTOR (triangle)))
        clutter_actor_queue_redraw (CLUTTER_ACTOR (triangle));

      g_object_notify (G_OBJECT (triangle), "color");
      g_object_unref (triangle);
    }

테스트 프로그램은 다음과 같습니다.

    #include <clutter/clutter.h>
    #include "triangle_actor.h"
    #include <stdlib.h>

    int main(int argc, char *argv[])
    {
      ClutterColor stage_color = { 0x00, 0x00, 0x00, 0xff };
      ClutterColor actor_color = { 0xff, 0xff, 0xff, 0x99 };

      clutter_init (&argc, &argv);

      /* Get the stage and set its size and color: */
      ClutterActor *stage = clutter_stage_get_default ();
      clutter_actor_set_size (stage, 200, 200);
      clutter_stage_set_color (CLUTTER_STAGE (stage), &stage_color);

      /* Add our custom actor to the stage: */
      ClutterActor *actor = clutter_triangle_new_with_color (&actor_color);
      clutter_actor_set_size (actor, 100, 100);
      clutter_actor_set_position (actor, 20, 20);
      clutter_container_add_actor (CLUTTER_CONTAINER (stage), actor);
      clutter_actor_show (actor);

      /* Show the stage: */
      clutter_actor_show (stage);

      /* Start the main loop, so we can respond to events: */
      clutter_main ();

      return EXIT_SUCCESS;
    }
