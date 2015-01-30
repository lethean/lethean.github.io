---
layout: post
title: GLib 쓰레드 프로그래밍
tags: [ Coding,  GLib,  GTK+ ]
---

소프트웨어를 개발하면서 멀티 쓰레드 방식을 사용하는 경우는 많습니다. 하지만 그만큼 복잡도가 증가해서 세심하게 고려하여 설계하지 않으면 디버깅 재앙을 얻는 경우가 많습니다. 이 글은 '[멀티쓰레드 프로그래밍 규칙](/2008/12/28/gtk-memory-management/)'에서 이어지는 내용입니다. [GTK+ 쓰레드 관련 잡설](/2006/01/20/multi-thread-gtk-programming/)은 이미 언급한 적이 있으니까, 오늘은 별도의 쓰레드로 동작하는 간단한 예제 모듈을 만들면서 몇가지 유용한 GLib 쓰레드 API를 설명하겠습니다.

### 리소스 (Resources)

한 개 이상의 쓰레드가 동작하는 방식의 소프트웨어를 설계할 경우 가장 염두에 두어야 하는 점은 **자원(resources)**입니다. 자원, 즉 리소스는 쉽게 말해 소프트웨어가 사용하는 데이터를 의미합니다. 전역 변수, 디스크 파일, 네트웍 소켓, 외부 장치 심지어 비디오 카드 같은 그래픽 장치 등이 모두 리소스입니다. 물론 넓은 의미에서 보면 리소스는 하나의 기능이나 세부 작업을 나타낼 수도 있습니다.

멀티쓰레드 프로그래밍에서 가능한 지켜야 하는 가장 중요한 원칙은 '**하나의 쓰레드만 하나의 리소스에 접근할 수 있어야 한다**'입니다. 아무 생각없이 하나의 리소스에 여러 쓰레드가 동시에 접근하도록 설계할 경우, 어쩔 수 없이 뮤텍스(mutex) 계열 API를 이용해 접근할 때마다 임계 구역을 보호해야 합니다. 그리고 이러한 기법은 소스 코드가 복잡해지고 커질수록 버그가 많아지고, 디버깅도 점점 어려워집니다. 물론, 쓰레드-풀(thread-pool) 기법처럼 성능 최적화나 확장성을 위해 멀티쓰레드를 사용하는 경우처럼 예외도 사실 많지만, 일단 이 글에서는 무시합니다.

앞서 예를 들었던 GTK+ 쓰레드 프로그래밍도 리소스 관점에서 보면, 무조건 모든 쓰레드에서 GTK+ / GDK API 호출 전후에 gdk\_threads\_\*() 계열 API를 남용해서 지독한 데드락과 이중락에 고생하던가, 아니면 GTK+ / GDK API 호출을 메인 쓰레드에서만 호출하도록 g\_idle\_add() / g\_timeout\_add() API만 이용하는 방법이 있습니다. 두번째 방법을 모델-뷰(Model-View) 개념으로 생각하면 마지막 GTK+ / GDK API 호출을 뷰(view) 갱신으로 볼 수 있고, g\_idle\_add() 계열 API는 일종의 메시지 전달로 생각할 수도 있습니다. (참고로 Sentry24DVR 2.x 버전은 첫번째 방식을, Sentry24CMS 2.x 버전은 두번째 방식을 사용합니다)

### 쓰레드 시작 / 정지 / 실행

가장 먼저 쓰레드를 만들고 종료하는 루틴을 만들어 봅시다. 편의상 모듈 이름은 'drink'라고 합니다.

    #include <glib.h>

    typedef struct _Drink Drink;
    struct _Drink
    {
      GThread *thread;
      gint running;
      GAsyncQueue *queue;
      gchar *host;
      gint port;
    };

    static gpointer
    drink_process (gpointer data)
    {
      Drink *drink = data;

      while (g_atomic_int_get (&drink->running))
        {
          // do something...
        }

      return NULL;
    }

    Drink *
    drink_new (const gchar *host, gint port)
    {
      Drink *drink;

      g_return_val_if_fail (host != NULL, NULL);
      g_return_val_if_fail (port > 0, NULL);

      drink = g_new (Drink, 1);
      drink->host = g_strdup (host);
      drink->port = port;
      drink->queue = g_async_queue_new ();

      g_atomic_int_set (&drink->running, 1);
      drink->thread = g_thread_new (drink_process, drink, TRUE, NULL);

      return drink;
    }

    void
    drink_destroy (Drink *drink)
    {
      g_return_if_fail (drink != NULL);

      g_atomic_int_set (&drink->running, 0);
      g_thread_join (drink->thread);

      g_async_queue_unref (disk->queue);
      g_free (drink->host);
      g_free (drink);
    }

drink\_new() 함수는 지정한 호스트 / 포트 번호를 이용하여 새로운 Drink 객체를 만듭니다. 그리고 앞으로 나올 모든 데이터는 각각 자신이 속한 Drink 객체만 접근합니다. 즉, Drink 객체를 하나의 리소스로 여기면 됩니다. drink\_destroy() 함수는 쓰레드가 종료할때까지 기다렸다가 Drink 객체를 해제하고 마무리합니다.

쓰레드 함수 무한 루프는 간단하게 정수형 변수를 플래그처럼 사용합니다. 제대로 하려면 플래그 변수 역시 뮤텍스 API로 보호해주어야 하지만 대부분의 경우 간단한 원자연산자(atomic operator)로 처리가 가능합니다. 일단 이렇게 만들어 둡시다.

마지막으로 설명할 API가 [GAsyncQueue](http://library.gnome.org/devel/glib/stable/glib-Asynchronous-Queues.html) 객체인데, 가장 중요한 역할을 담당하는 물건입니다. 설명 그대로 이 API는 쓰레드간 비동기 통신(asynchronous communication between threads)을 하는데 사용합니다. 이 객체를 생성하는데는 g\_async\_queue\_new(), 없애기 위해서는 g\_async\_queue\_unref() 함수를 이용하는데, 일단 지금은 만들어만 놓습니다.

### API 추가 + 메시지 전달

제일 먼저 하고 싶은 일은 미리 지정한 서버에 TCP 연결을 하거나, 끊고 싶습니다. 이를 비동기큐를 이용해서 간단하게 구현해 봅시다.

    enum
    {
      DRINK_MSG_CONNECT = 1,
      DRINK_MSG_SHUTDOWN = 2,
    };

    void
    drink_connect (Drink *drink)
    {
      g_return_if_fail (drink != NULL);

      g_async_queue_push (drink->queue,
                          GINT_TO_POINTER (DRINK_MSG_CONNECT));
    }

    void
    drink_shutdown (Drink *drink)
    {
      g_return_if_fail (drink != NULL);

      g_async_queue_push (drink->queue,
                          GINT_TO_POINTER (DRINK_MSG_SHUTDOWN));
    }

이렇게 하면 drink\_connect() / drink\_shutdown() 함수를 호출하면 g\_async\_queue\_push() 함수를 이용해 메시지를 큐에 넣기만 하고 아무 일도 안합니다. (참고 : 모듈 외부에서 볼때는 내부 구현에 쓰레드를 사용하는지, 메시지 큐를 이용하는지 등은 공개되지도 않고, 공개할 필요도 없습니다) 이제 drink\_process() 함수를 다음과 같이 수정합니다.

    static gpointer
    drink_process (gpointer data)
    {
      Drink *drink = data;

      while (g_atomic_int_get (&drink->running))
        {
          do {
            GTimeVal tval;
            gpointer msg;

            /* wait for messages */
            g_get_current_time (&tval);
            g_timeval_add (&tval, 10000); /* 10msec */
            msg = g_async_queue_timed_pop (drink->queue, &tval);
            if (!msg)
              break;

            switch (GPOINTER_TO_INT (msg))
              {
              case DRINK_MSG_CONNECT:
                // do connect work...
                break;
              case DRINK_MSG_SHUTDOWN:
                // do shutdown work...
                break;
              default:
                g_warning ("unknown drink msg");
                break;
              }
          } while (1);

          // do something else ...
        }

      return NULL;
    }

보는 바와 같이 메시지 큐에서 메시지를 꺼내어 메시지에 해당하는 작업을 처리합니다. 만일 메시지 큐를 사용하지 않고 drink\_connect() 함수에서 직접 연결 작업을 수행하면 쓰레드 부분과 공유하는 부분을 모두 뮤텍스로 보호해야 하지만 이처럼 모든 작업을 담당 쓰레드가 처리하도록 메시지만 전송하면 실행 순서도 맞고 쓰레드가 자료 공유를 걱정할 필요도 없게 됩니다.

여기서 사용한 g\_async\_queue\_timed\_pop() 함수는 지정한 시간 동안 아무 메시지도 없으면 NULL을 돌려줍니다. 비슷한 함수로 g\_async\_queue\_pop() 함수는 메시지가 올때까지 무한정 기다랍니다. g\_async\_queue\_try\_pop() 함수는 메시지가 없을 경우 바로 NULL을 돌려줍니다. 만일 쓰레드 함수 자체적인 작업은 없고 100% 외부에서 메시지가 올때만 작업이 수행된다면 g\_async\_queue\_pop() 함수를 사용하는 것이 더 좋습니다. 프로세스 동기화나 수면 상태(sleep) 등을 다른 작업을 하면서 자체적으로 하는 경우라면 g\_async\_queue\_try\_pop() 함수가 유용합니다.

이 예제에서는 단순하게 10 밀리초 여유를 두고 메시지를 확인하고, 그외 다른 작업을 처리하도록 했습니다.

### 쓰레드 종료 다듬기

예제 처음에 있던 쓰레드 종료 코드가 너무 단순해서 조금 불안할 지도 모르겠네요. 메시지 큐에 데이터가 있을때 종료되면 메모리 누수도 있을 것 같고... 그래서 쓰레드 종료도 하나의 메시지로 처리하도록 하려고 합니다. 수정하는 부분은 다음과 같습니다.

    enum
    {
      DRINK_MSG_STOP_THREAD = -1,
      DRINK_MSG_CONNECT = 1,
      DRINK_MSG_SHUTDOWN = 2,
    };

    static gpointer
    drink_process (gpointer data)
    {
      Drink *drink = data;

      while (TRUE)
        {
          GTimeVal tval;
          gpointer msg;

          /* wait for messages */
          g_get_current_time (&tval);
          g_timeval_add (&tval, 10000); /* 10msec */
          msg = g_async_queue_timed_pop (drink->queue, &tval);
          if (msg)
            {
              if (msg == GINT_TO_POINTER (DRINK_MSG_STOP_THREAD)) break;

              switch (GPOINTER_TO_INT (msg))
                {
                case DRINK_MSG_CONNECT:
                  // do connect work...
                  break;
                case DRINK_MSG_SHUTDOWN:
                  // do shutdown work...
                  break;
                default:
                  g_warning ("unknown drink msg");
                  break;
                }
            }

          // do something else ...
        }

    void
    drink_destroy (Drink *drink)
    {
      g_return_if_fail (drink != NULL);

      g_async_queue_push (drink->queue, GINT_TO_POINTER (DRINK_MSG_STOP_THREAD));
      g_thread_join (drink->thread);

      g_async_queue_unref (disk->queue);
      g_free (drink->host);
      g_free (drink);
    }

쓰레드 함수 무한루프 조건문이 조금 변경되었을 뿐 기본적인 원리는 동일합니다.

### 마지막, 조금 더 개선...

이놈의 메시지 방식을 사용하면 대부분 프로그래머는 쉽게 switch() 문의 유혹을 떨쳐버리지 못합니다. 근데, 만일 당신이 매우 성능 좋은 메시징 서비스를 만들고 있다면 이런 방식의 코드는 유지보수도 힘들고 성능도 나쁠 수 있습니다. 메시지-함수 테이블을 유지해도 되고, 여러가지 방법이 있겠지만 여기서는 약간 가독성(readability)과 유지보수에 중점을 둔 방식을 설명하려 합니다.

    typedef struct _DrinkMsg DrinkMsg;
    struct _DrinkMsg
    {
      void (*func) (Drink *drink, gpointer data1, gpointer data2);
      gpointer data1;
      gpointer data2;
    };

    static gpointer
    drink_process (gpointer data)
    {
      ...
      DrinkMsg *msg;

      msg = g_async_queue_try_pop (drink->queue);
      if (msg)
        {
          if (msg == GINT_TO_POINTER (-1))
            break;
          msg->func (drink, msg->data1, msg->data2);
          g_slice_free1 (msg);
        }
      ...
    }

    void
    drink_destroy (Drink *drink)
    {
      ...
      g_async_queue_push (drink->queue, GINT_TO_POINTER (-1));
      g_thread_join (drink->thread);
      ...
    }

    static void
    drink_connect_real (Drink *drink, gpointer data1, gpointer data2)
    {
      gchar *host = data1;
      gint port = GPOINTER_TO_INT (data2);

      // do connect work...

      g_free (host);
    }

    void
    drink_connect (Drink *drink, const gchar *host, gint port)
    {
      DrinkMsg *msg;

      msg = g_slice_new (DrinkMsg);
      msg->func = drink_connect_real;
      msg->data1 = g_strdup (host);
      msg->data2 = GINT_TO_POINRTER (port);
      g_async_queue_push (drink->queue, msg);
    }

뭐... 더 이상의 설명은 피곤해서...

궁금한 API는 직접 매뉴얼을 한 번 뒤져보시길... :)
