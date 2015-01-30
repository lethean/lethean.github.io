---
layout: post
title: GObject 속성 직렬화(Serialization)하기
tags: [ Agile,  GLib ]
---

GObject 객체의 속성(properties)을 자동으로 저장하고 다시 자동으로 불러들이는 일련의 작업을 자동화할 수 있다면 편하지 않을까 생각해 본 적이 있을겁니다. 이러한 과정을 [직렬화(serialization)](http://en.wikipedia.org/wiki/Serialization)라고 부른다면, 오브젝티브-C, 자바 등과 같은 많은 언어가 이미 기본적으로 직렬화를 지원하거나 관련 라이브러리를 제공하고 있는만큼, GObject 객체 직렬화 라이브러리가 없을리가 없습니다. 예를 들어, [JSON-GLib](http://live.gnome.org/JsonGlib), [Catalina](http://git.dronelabs.com/catalina/about) 등과 같은 라이브러리를 사용하면 됩니다.

그런데, 여기서 드는 의문은, 왜 GObject 라이브러리 자체에는 정작 이 기능이 없을까입니다. 그리고, 잠깐 생각해보고 내린 결론은, 특정한 형식을 제한하기보다는, 직렬화를 위한 기본 기능만 지원하고, 어플리케이션 특성에 맞게 직렬화는 프로그래머의 자유에 맡긴게 아닌가 합니다. 위에서 언급한 라이브러리도 기능은 강력하지만, 특정 용도에서 사용하기 위해 만들어지다 보니 불필요하거나 어려운 부분이 조금 있습니다.

그래서 이 글에서는 [이전 글](/2009/08/24/oop-with-gobject-4/)에서 잠깐 언급한 객체 속성 정보 얻는 방법을 이용하여 간단한 GObject 속성 직렬화 코드를 구현하는데 필요한 몇 가지 기법을 소개하려고 합니다. 실제 직렬화 기능은 용도와 방식에 맞게 구현하면 되리라 생각합니다.

**직렬화 대상 속성 지정하기**

직렬화하려는 속성을 매번 지정하는 것보다 객체 설계시 아예 지정해버릴 수 있는 방법이 있으면 자동화에 편합니다. 이때 사용할 수 있는 기법이 `G_PARAM_USER_SHIFT` 매크로입니다. 객체 클래스 초기화 함수에서 속성을 추가(install)할때 보통 `G_PARAM_READABLE`, `G_PARAM_READWRITE` 등과 같은 미리 정의되어 있는 특성을 지정하는데, `G_PARAM_USER_SHIFT` 매크로를 이용해 사용자가 임의의 특성을 더 추가할 수 있습니다. 예를 들어 다음과 같이 정의하면 됩니다.

    #define G_PARAM_SERIALIZABLE (1 << (G_PARAM_USER_SHIFT + 1))

이제, 속성 스팩을 만들때(`g_param_spec_*()`)  `G_PARAM_SERIALIZABLE` 플래그를 함께 지정할 수 있습니다. 예를 들면 다음과 같습니다.

    pspec = g_param_spec_int ("id",
                              "ID",
                              "unique ID of the device",
                              0,
                              G_MAXINT32,
                              0,
                              G_PARAM_READWRITE | G_PARAM_SERIALIZABLE);

주의할 점은, `G_PARAM_USER_SHIFT` 매크로를 응용하는 다른 라이브러리가 있을 수 있으므로 충돌 여부를 확인해야 합니다. 예를 들어 GStreamer 라이브러리의 `GST_PARAM_USER_SHIFT` 매크로도 비슷한 역할을 합니다. 따라서 구현하려는 객체가 Gstreamer 객체를 상속받는다면 다른 값을 지정해야 합니다.

**직렬화 대상 속성 목록 얻기**

`g_object_class_list_properties()` 함수를 이용하면 객체 클래스의 모든 속성 목록을 얻을 수 있습니다. 이 함수 프로토타입을 흉내내어 직렬화 대상 속성만 추출하는 함수를 만들면 다음과 같습니다.

    GParamSpec **
    list_serializable_properties (GObject *serializable,
                                  guint   *n_properties)
    {
      GParamSpec **specs;
      GParamSpec **new_specs;
      guint        n_specs;
      guint        i;
      guint        total;

      g_return_val_if_fail (G_IS_OBJECT (serializable), NULL);

      specs = g_object_class_list_properties (G_OBJECT_GET_CLASS (serializable), &n_specs);
      new_specs = g_new0 (GParamSpec *, n_specs + 1);
      for (i = 0, total = 0; i < n_specs; i++)
        {
          GParamSpec *spec = specs[i];

          if (!(spec->flags & ECC_PARAM_SERIALIZABLE))
            continue;

          new_specs[total] = spec;
          total++;
        }
      g_free (specs);

      if (n_properties)
        *n_properties = total;

      return new_specs;
    }

이제 이 함수가 돌려주는 속성 스펙 목록을 이용해 텍스트 파일이나 데이터베이스, 또는 GConf 등을 이용하여 속성 값을 불러오거나 저장하면 됩니다.

**속성값 변환하기**

하지만 속성은 정수형, 실수형, 문자열 등 여러가지 타입인데 모든 종류의 타입을 하나씩 문자열로 변환해서 저장하는 건 비효율적 과정입니다. 이때 사용할 수 있는 함수가 `g_value_transform()`인데, 이 함수는 기본적으로 두 GValue 간 내용을 적절하게(?) 변환해 줍니다. 다음은 이 함수를 이용하여 간단하게 'key=value" 형식으로 저장한 문자열을 돌려주는 코드입니다.

    gchar *
    serialize_properties (GObject *serializable)
    {
      GParamSpec **specs;
      GString     *string;
      gchar       *str;
      guint        i;

      g_return_val_if_fail (G_IS_OBJECT (serializable), NULL);

      string = g_string_new (NULL);

      specs = list_properties (serializable, NULL);
      for (i = 0; specs[i] != NULL; i++)
        {
          GParamSpec *spec = specs[i];
          GValue      value = { 0 };
          GValue      value_str = { 0 };

          g_value_init (&value, spec->value_type);
          g_value_init (&value_str, G_TYPE_STRING);
          g_object_get_property (G_OBJECT (serializable), spec->name, &value);
          if (g_value_transform (&value, &value_str))
            g_string_append_printf (string,
                                    "%s=%sn",
                                    spec->name,
                                    g_value_get_string (&value_str));
          else
            g_warning ("failed to transform property '%s' to string", spec->name);
          g_value_unset (&value);
          g_value_unset (&value_str);
        }
      g_free (specs);

      str = string->str;
      g_string_free (string, FALSE);

      return str;
    }

그런데 한 가지 문제가 있습니다. `g_value_transform()` 함수는 기본적으로 모든 GValue 사이의 변환을 지원하지 않는다는 점입니다. 위에서 예를 든 코드는 C 언어 기본 타입을 문자열로 변환하는데, 다행히도 이 변환은 기본적으로 지원합니다. 하지만, 반대로 문자열에서 다른 타입으로 변환하는 기능은 제한적으로 지원합니다. 그래서 다음과 같이 필요한 변환 함수를 미리 등록해야 합니다.

    static void
    tranform_string_to_int (const GValue *src_value,
                            GValue       *dest_value)
    {
      gint64 value;

      value = g_ascii_strtoll (g_value_get_string (src_value), NULL, 10);
      g_value_set_int (dest_value, value);
    }

    static void
    tranform_string_to_boolean (const GValue *src_value,
                                GValue       *dest_value)
    {
      gboolean value;

      value = g_ascii_strncasecmp (g_value_get_string (src_value), "TRUE", 4) == 0 ? TRUE : FALSE;
      g_value_set_boolean (dest_value, value);
    }

    static void
    tranform_string_to_double (const GValue *src_value,
                               GValue       *dest_value)
    {
      gdouble value;

      value = g_ascii_strtod (g_value_get_string (src_value), NULL);
      g_value_set_double (dest_value, value);
    }

    static void
    register_transform_funcs (void)
    {
      struct
      {
        GType src_type;
        GType dest_type;
        GValueTransform transform_func;
      } transformers[] =
      {
        { G_TYPE_STRING, G_TYPE_INT, tranform_string_to_int },
        { G_TYPE_STRING, G_TYPE_BOOLEAN, tranform_string_to_boolean },
        { G_TYPE_STRING, G_TYPE_DOUBLE, tranform_string_to_double }
      };
      gint i;

      for (i = 0; i < G_N_ELEMENTS (transformers); i++)
        if (!g_value_type_transformable (transformers[i].src_type,
                                         transformers[i].dest_type))
          g_value_register_transform_func (transformers[i].src_type,
                                           transformers[i].dest_type,
                                           transformers[i].transform_func);
    }

당연히, 속성 타입이 C 언어 기본 타입이 아닐 경우라도 위와 같은 방식으로 변환 함수를 등록해 주면 알아서 동작합니다.

**그외...**

실제 프로젝트에서는 위와 같은 기능과 몇가지 도우미 API를 지원하는 인터페이스(GInterface) 객체를 정의해서, 각 객체가 그 인터페이스 객체를 구현(implementation)하도록 했습니다. 왜냐하면, 객체의 속성(properties)으로 나타나지 않는 내부 정보도 직렬화할 수 있기 때문입니다. 예를 들어 장치 목록 객체는 여러 장치 객체를 참조합니다. 그래서 장치 목록 객체를 직렬화하는 함수를 호출하면 실제로 장치 목록 객체는 목록에 포함된 장치 객체의 직렬화 함수를 다시 호출해 문자열을 얻어와 조합된 문자열을 돌려줍니다.

:)
