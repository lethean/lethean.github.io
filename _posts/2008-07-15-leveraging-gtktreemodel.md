---
layout: post
title: GtkTreeModel 확장하기
tags: [ GTK+ ]
---

GTK+ 프로그래밍에서 많이 사용하는 GtkTreeModel(GtkListStore / GtkTreeStore)에는 일반적으로 데이터(객체)에서 표시할 내용만 추가해서 사용합니다. 따라서 실제로 데이터가 변경되면 그때마다 GtkTreeModel 내용을 변경해주어야 합니다.(일종의 동기화) 하지만 이러한 프로그래밍 방식은 매우 귀찮고 개발 시간도 오래 걸리는 것은 물론 런타임 오버헤드도 발생할 수 밖에 없습니다. 아예 GtkTreeModel에서 하나의 컬럼에 데이터(객체)를 넣고 관리하는 방법도 있지만, 이 역시 이러한 오버헤드와 비효율은 피할 수 없습니다.

[leveraging GtkTreeModel](http://davyd.livejournal.com/252351.html)

위 글에서 저자는 기존 GtkListStore / GtkTreeStore 객체를 상속받은 새로운 GtkTreeModel 인터페이스를 구현하는 방법을 이용해 이러한 오버헤드를 줄이는 방법을 설명하고 있습니다.

방법은 다음과 같습니다.

먼저 GtkTreeModel 객체에는 실제 데이터(객체) 하나만 넣습니다. 그리고 외부에 공개하는 컬럼 갯수와 각 컬럼의 종류(type), 값(value)을 에뮬레이션하기 위해 다음 세가지 API를 오버라이딩(overriding)합니다.

1.  get\_n\_columns ()
2.  get\_column\_type ()
3.  get\_value ()

위 세 API는 GtkTreeModel 외부에서 원하는 행(row)의 컬럼(column) 값을 가져올때만 호출됩니다. 특히 GtkTreeView 등과 같은 위젯에 연결되어 있을 경우 행이 열 개이거나 백만 개이거나 상관없이 현재 화면에 보이는 행의 값만 가져오기 위해 호출됩니다.

물론 문자열 조합이 복잡하거나 연산이 복잡한 경우 약간의 오버헤드가 발생할 수 있지만, 메모리 사용량은 확 줄어들고 동기화 걱정 자체가 없다는 장점이 더 큽니다.
