---
layout: post
title: GObject Private 데이터 접근 오버헤드 줄이기
tags: [ GLib,  GTK+ ]
---

GTK+ 개발자 메일링 리스트에서 [GTK+ 속도 관련 질의 응답](http://mail.gnome.org/archives/gtk-devel-list/2008-December/thread.html#00061)이 오가는 걸 지켜보는 도중 g\_type\_class\_add\_private() + g\_type\_instance\_get\_private() 조합을 사용하면 편하지만, 오버헤드가 크고 느리기 때문에 이를 [줄일 수 있는 다른 방법을 소개한 내용](http://mail.gnome.org/archives/gtk-devel-list/2008-December/msg00072.html)이 있어 결론만 정리해 보았습니다.

그 방법은, [그놈 프로그래밍 가이드라인](http://developer.gnome.org/doc/guides/programming-guidelines/binary.html#PRIVATE)에서도 명시했듯이,  객체 데이터 선언시 'priv' 등과 같은 필드를 선언하고, g\_type\_instance\_get\_private() 함수로 내부 데이터 주소를 얻어 'priv' 필드에 저장해 둡니다. 그리고 다음부터는 그 필드를  이용하여 내부 데이터(Private Data)에 직접 접근하는 방식입니다.

예를 들어 기존의 코드가 다음과 같다면,

    /* foo-object.h */
    typedef struct _FooObject FooObject;
    struct _FooObject
    {
      GObject parent;
    };

    gint foo_object_do_something (FooObject *foo);

    /* foo-object.c */
    typedef struct _FooObjectPrivate FooObjectPrivate;
    struct _FooObjectPrivate
    {
      gboolean eating;
      gint size;
    };

    #define FOO_OBJECT_GET_PRIVATE(obj) 
      (G_TYPE_INSTANCE_GET_PRIVATE ((obj), 
       FOO_TYPE_OBJECT, 
       FooObjectPrivate))

    static void
    foo_object_class_init (FooObjectClass *klass)
    {
      /* ... */
      g_type_class_add_private (G_OBJECT_CLASS (klass), 
                                sizeof (FooBarPrivate));
    } 

    gint
    foo_object_do_something (FooObject *foo)
    {
      FooObjectPrivate *priv;

      priv = FOO_OBJECT_GET_PRIVATE (foo);
      priv->eating = TRUE;
      /* ... */
    }

다음과 같이 변경하면, 매번 g\_type\_instance\_get\_private() 함수를 호출하는 오버헤드를 줄일 수 있습니다.

    /* foo-object.h */
    typedef struct _FooObjectPrivate FooObjectPrivate;
    typedef struct _FooObject FooObject;
    struct _FooObject
    {
      GObject parent;
      FooObjectPrivate *priv;
    };

    gint foo_object_do_something (FooObject *foo);

    /* foo-object.c */
    struct _FooObjectPrivate
    {
      gboolean eating;
      gint size;
    };

    #define FOO_OBJECT_GET_PRIVATE(obj) 
      (((FooObject *) (obj))->priv)

    static void
    foo_object_class_init (FooObjectClass *klass)
    {
      /* ... */
      g_type_class_add_private (G_OBJECT_CLASS (klass), 
                                sizeof (FooBarPrivate));
    } 

    static void
    foo_object_init (FooObject *obj)
    {
      /* ... */
      obj->priv = 
        G_TYPE_INSTANCE_GET_PRIVATE (obj, 
                                     FOO_TYPE_OBJECT, 
                                     FooObjectPrivate);
    }

    gint
    foo_object_do_something (FooObject *foo)
    {
      FooObjectPrivate *priv;

      priv = FOO_OBJECT_GET_PRIVATE (foo);
      priv->eating = TRUE;
      /* ... */
    }

물론 위에서 설명한 방식 대신 객체 초기화시 내부 데이터(private)를 아예 따로 할당해서 관리하는  방식도 비슷하지만, g\_type\_class\_add\_private() 함수를 통해 추가한 메모리는 GLib 라이브러리가 알아서 관리해주기 때문에 더 편합니다.
