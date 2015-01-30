---
layout: post
title: "싱글턴(Singleton) GObject 객체 만들기"
tags: [ Agile,  GLib ]
---

좋은 [블로그 포스트](http://blogs.gnome.org/xclaesse/2010/02/11/how-to-make-a-gobject-singleton/)가 올라왔길래, 우리말로 정리해 보았습니다.

GObject 기반 객체 지향 프로그래밍에서 싱글턴 패턴을 사용하려면 대개 다음과 같은 함수를 추가합니다.

    FooBar*
    foo_bar_get_default (void)
    {
      static FooBar *self = NULL;

      if (self == NULL)
        self = foo_bar_new ();

      return self;
    }

하지만 이렇게 구현할 경우 몇가지 단점이 있는데, 돌려받은 객체를 실수로 해제할 경우 문제를 일으킬 수 있고, 프로그램이 종료할때까지 객체가 소멸되지 않아 메모리 누수가 발생할 수 있습니다. 또한 사용자가 `g_object_new(FOO_TYPE_BAR, NULL)` 방식으로 객체를 생성하면 결국 새 객체가 만들어지기 때문에 싱글턴 객체로 동작하지 않습니다.

그래서, [Empathy](http://live.gnome.org/Empathy) 프로젝트에서는 다음과 같이 싱글턴 객체를 구현하고 있습니다.

    static GObject*
    constructor (GType type,
                 guint n_construct_params,
                 GObjectConstructParam *construct_params)
    {
      static GObject *self = NULL;

      if (self == NULL)
        {
          self = G_OBJECT_CLASS (foo_bar_parent_class)->constructor (
            type, n_construct_params, construct_params);
          g_object_add_weak_pointer (self, (gpointer) &self);
          return self;
        }

      return g_object_ref (self);
    }

    static void
    foo_bar_class_init (FooBarClass *klass)
    {
      GObjectClass *object_class = G_OBJECT_CLASS (klass);

      object_class->constructor = constructor;
    }

이 방식을 이용하면 `g_object_new()`를 이용해 객체를 만들어도 항상 동일한 객체를 돌려줍니다. 더불어 객체의 참조카운터를 증가해서 돌려주기 때문에, 일반 객체처럼, 사용이 끝나면 `g_object_unref()`를 호출해 객체를 해제하면 됩니다. 물론 마지막 사용이 끝나는 시점에서는 자동으로 객체가 소멸되고 객체 포인터도 NULL값으로 초기화됩니다.([g\_object\_add\_weak\_pointer()](http://library.gnome.org/devel/gobject/stable/gobject-The-Base-Object-Type.html#g-object-add-weak-pointer) 함수가 이 역할을 합니다)

물론 빈번한 객체 생성 / 소멸 호출을 막기 위해 프로그램 전반적으로 객체를 유지하든, 필요한 때만 생성해서 사용하도록 할 지 여부는 이 객체를 사용하는 프로그램이 선택할 수 있습니다. 따라서 라이브러리 코드를 작성할 경우 반드시 이 방식으로 싱글턴 객체를 제공하는게 좋습니다.

참고로, 위 구현은 멀티쓰레드가 동시에 접근하는 경우 안전하지 않습니다. 그러므로, 필요하다면, 뮤텍스나 [g\_once()](http://library.gnome.org/devel/glib/stable/glib-Threads.html#g-once) 등을 이용해 객체 포인터를 보호해야 합니다.
