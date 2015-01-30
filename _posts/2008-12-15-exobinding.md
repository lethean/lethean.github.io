---
layout: post
title: ExoBinding 소개
tags: [ GTK+ ]
---

[Xfce](http://www.xfce.org/) 프로젝트에서 사용하는 라이브러리에 포함되어 있는 [ExoBinding](http://www.xfce.org/documentation/api/exo/exo-Binding-Properties-Functions.html) 이라는 (객체)함수가 있는데, 매우 흥미로워서 소개합니다. (참고로, '[ExoBinding and Settings Management](http://mbarnes.livejournal.com/1899.html)' 블로그를 통해 알게된 내용입니다)

ExoBinding 함수는 기본적으로 GObject 기반 두 객체의 속성(properties) 값을 동기화합니다. 즉, 속성을 묶는(binding) 역할을 합니다. 속성 묶음(binding)의 한쪽 속성이 바뀌면 다른쪽 속성을 자동적으로 새로운 값으로 변경합니다. 단방향(uni-directional) 뿐 아니라 상호(mutual) 바인딩도 지원하는데, 필요할 경우 변환(transform) 함수를 지정할 수도 있습니다. 또한 복수 객체가 복잡하게 묶여 있어도 무한 루프를 자동으로 제거합니다.

예를 들어 (기혼자일 경우에만 배우자 이름을 입력받도록 하는 경우처럼) 사용자에게 문자열을 입력받는 GtkEntry 위젯이 있고, 이 위젯의 활성화(sensitive) 여부를 결정하는 GtkCheckButton 위젯이 있을 경우 다음과 같은 코드 한 줄이면 모든게 해결됩니다.

    {
      GtkWidget *button;
      GtkWidget *entry;

      button = gtk_check_button_new_with_label ("Are you married?");
      entry = gtk_entry_new ();

      exo_binding_new (G_OBJECT (button), "active",
                       G_OBJECT (entry), "sensitive");
    }

위 코드처럼, 엔트리 위젯의 'sensitive'를 체크버튼 위젯의 'active'에 연결만 하면 됩니다. 시그널 핸들러를 작성해서 복잡하게 처리할 필요가 전혀 없어져 버립니다. 내부적으로 묶음(binding)에 사용한 정보는 둘 중 하나의 객체가 사라지면 자동적으로 사라지므로, 메모리 누수를 걱정할 필요도 없습니다.

이 객체에 관심을 가지는 이유는 활용도가 무궁무진하기 때문입니다. 특히 복잡한 설정 화면을 만들때 여러가지 조건으로 활성화 / 비활성화, 보여주기 / 안보여주기, 같은 값으로 위젯 간 동기화 등의 기능을 시그널 핸들러와 API를 이용해 직접 코딩하면 실수도 많고 코드 분량도 많아지는데, 위 방식을 이용하면 매우 깔끔하게 처리가 가능합니다. 더 나아가 [libgconf-bridge](http://wiki.openmoko.org/wiki/Libgconf-bridge)처럼 GConf 설정 내용을 그대로 GObject 속성에 반영하는 라이브러리와 결합할 경우, 설정 화면 만드는 일은 더욱 더 간편해질 수 있습니다.

만일 이 기능이 Glade에 통합될 수 있다면, 정말로 UI 부분만 따로 프로그래밍할 수 있는 여건이 마련될 수도 있을 것 같은데... 조만간에 소스 코드를 분석해서 프로젝트에 활용할 방법을 찾아볼 예정입니다.
