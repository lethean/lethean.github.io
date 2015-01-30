---
layout: post
title: GtkCellRenderer 이해하기
tags: [ GTK+ ]
---

GTK+ 프로그램 개발시 [GtkTreeView](http://library.gnome.org/devel/gtk/stable/GtkTreeView.html) / [GtkComboBox](http://library.gnome.org/devel/gtk/stable/GtkComboBox.html) 위젯은 상당히 많이 사용함에도 불구하고, 주위를 둘러보면 그저 이미 만들어져 있는 코드를 복사 붙여넣기 식으로 개발하는 사람이 많습니다. 그래서 조금만 고급 기능(?)이 필요한 경우 어디부터 시작해야할 지 몰라 당황하는 사람이 대부분인 것 같습니다. 그래서, 이 글에서는 [GtkTreeView](http://library.gnome.org/devel/gtk/stable/GtkTreeView.html) / [GtkComboBox](http://library.gnome.org/devel/gtk/stable/GtkComboBox.html) 위젯 등에서 데이터를 표시하는데 사용하는 [GtkCellLayout](http://library.gnome.org/devel/gtk/stable/GtkCellLayout.html) / [GtkCellRenderer](http://library.gnome.org/devel/gtk/stable/GtkCellRenderer.html) 객체가 어떤 방식으로 동작하는지 간략하게 설명하려 합니다.

### GtkCellRenderers in GtkCellLayout in GtkTreeView

먼저 GtkTreeView 위젯이 표시될때 객체가 어떻게 구성되는지 설명합니다.

GtkCellRenderer 객체는 데이터 하나를 표시하는데 사용합니다. 그림 하나, 텍스트 하나, 컴보박스 하나 등등... 하지만 GtkCellRenderer 객체를 GtkTreeView 같은 위젯에 바로 넣을 수는 없고, 여러 GtkCellRenderer 객체를 하나의 GtkCellLayout 객체에 넣을 수 있습니다. 그리고 다시 GtkCellLayout 객체를 GtkTreeView 객체에 넣을 수 있습니다. 이 관계를 일종의 수식으로 표현하면 아래와 같습니다.

    GtkCellLayout1 <- GtkCellRenderer1 + GtkCellRenderer2 + GtkCellRenderer3
    GtkCellLayout2 <- GtkCellRednerer4 + GtkCellRenderer5
    GtkTreeView <- GtkCellLayout1 + GtkCellLayout2

GtkCellLayout 객체는 GtkTreeView 위젯에 표시될때 하나의 컬럼(column)으로 표시됩니다. 사용하는 GTK+ 테마에 따라 컬럼 단위로 구분선이 그어질 수도 있고, 제목을 클릭하면 정렬(sort)하는 기능을 구현하면 컬럼 단위로 정렬됩니다. 사실 GtkCellLayout은 직접 사용할 수 있는 객체가 아니고 인터페이스(interface) 역할만 하며, 이 인터페이스를 구현한 객체를 실제로 사용합니다. 예를 들면 [GtkTreeViewColumn](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html "GtkTreeViewColumn") / [GtkIconView](http://library.gnome.org/devel/gtk/stable/GtkIconView.html "GtkIconView") / [GtkCellView](http://library.gnome.org/devel/gtk/stable/GtkCellView.html "GtkCellView") / [GtkEntryCompletion](http://library.gnome.org/devel/gtk/stable/GtkEntryCompletion.html "GtkEntryCompletion") / [GtkComboBox](http://library.gnome.org/devel/gtk/stable/GtkComboBox.html "GtkComboBox") / [GtkComboBoxEntry](http://library.gnome.org/devel/gtk/stable/GtkComboBoxEntry.html "GtkComboBoxEntry") 등입니다. 따라서 위 수식은 GtkComboBox 위젯 등에도 동일하게 적용됩니다.

중요한 점은 GtkCellLayout 객체는 실제로 데이터를 표시하는 객체가 아니라 일종의 컨테이너 역할만 하고, 실제로 데이터를 표시하는 작업은 GtkCellRenderer 객체가 담당한다는 점입니다. 그리고 더 중요한 사실은, 이렇게 구성할때 행(row)에 대한 레이아웃과 렌더러만 구성할 수 있다는 점입니다. 즉 GtkTreeModel 객체에 들어있는 모든 데이터를 행(row) 단위 레이아웃을 거쳐 표시합니다.

### GtkCellRenderer 렌더링

프로그래머가 GtkCellRenderer 객체를 직접 사용하는 경우는 거의 없습니다. 대부분 이를 상속한 [GtkCellRendererText](http://library.gnome.org/devel/gtk/stable/GtkCellRendererText.html) / [GtkCellRendererPixbuf](http://library.gnome.org/devel/gtk/stable/GtkCellRendererPixbuf.html) / [GtkCellRendererProgresss](http://library.gnome.org/devel/gtk/stable/GtkCellRendererProgress.html) / [GtkCellRendererToggle](http://library.gnome.org/devel/gtk/stable/GtkCellRendererToggle.html) 등의 객체를 사용합니다.

GtkCellRenderer 객체는 기본적으로 상태 정보가 없습니다. 대신 객체 속성(properties)을 기준으로 표시하고 프로그래머 역시 속성을 변경해서 원하는 내용을 표시합니다. GTK+ 위젯 시스템은 당연히 데이터 표시가 필요할때만  GtkCellRenderer 객체 속성을 변경해서 원하는 내용을 원하는 위치에 표시합니다.

예를 들어 GtkTreeModel 객체에 100개의 행(row) 데이터가 있고, 현재 화면에는 1번째부터 10번째까지 행의 데이터만 표시된다고 가정하면 GTK+는 먼저 첫번째 행의 데이터에 대하여 모든 레이아웃, 다시 레이아웃의 모든 렌더러에게 데이터를 표시할 좌표를 알려주고, 렌더러의 속성을 트리모델의 데이터 값으로 변경합니다. 그러면 렌더러는 속성이 변경되었므로 그 속성에 따라 내용을 표시합니다. 그리고 2번째 행으로 이동해 동일한 작업을 수행합니다. 이 작업을 화면에 보이는 10번째 행까지만 반복합니다. 참고로, 이 작업은 EXPOSE 이벤트나 스크롤 이벤트가 발생했을 경우에도 필요한 영역의 데이터만 표시하기 위해 동일하게 동작합니다.

이제 무심코 사용하는 [gtk\_tree\_view\_column\_new\_with\_attributes()](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html#gtk-tree-view-column-new-with-attributes) API를 한번 분석해 봅시다. 문서에서는 다음과 같이 설명하고 있습니다.

> Creates a new [GtkTreeViewColumn](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html "GtkTreeViewColumn") with a number of default values. This is equivalent to calling [`gtk_tree_view_column_set_title()`](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html#gtk-tree-view-column-set-title "gtk_tree_view_column_set_title ()"), [`gtk_tree_view_column_pack_start()`](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html#gtk-tree-view-column-pack-start "gtk_tree_view_column_pack_start ()"), and [`gtk_tree_view_column_set_attributes()`](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html#gtk-tree-view-column-set-attributes "gtk_tree_view_column_set_attributes ()") on the newly created [GtkTreeViewColumn](http://library.gnome.org/devel/gtk/stable/GtkTreeViewColumn.html "GtkTreeViewColumn").
>
> Here's a simple example:
>
>      enum { TEXT_COLUMN, COLOR_COLUMN, N_COLUMNS };
>      ...
>      {
>        GtkTreeViewColumn *column;
>        GtkCellRenderer   *renderer = gtk_cell_renderer_text_new ();
>
>        column = gtk_tree_view_column_new_with_attributes ("Title",
>                                                           renderer,
>                                                           "text", TEXT_COLUMN,
>                                                           "foreground", COLOR_COLUMN,
>                                                           NULL);
>      }
>
> |-------------|
> | *`title` :* |
> | *`cell`* :  |
> | *`...`* :   |
> | *Returns* : |
>
기본값으로 GtkTreeViewColumn 객체를 만드는데, 이는 gtk\_tree\_view\_column\_new()로 새로운 객체를 만들고gtk\_tree\_view\_column\_set\_title()로 제목을 정한 다음, gtk\_tree\_view\_colunn\_pack\_start()로 렌더러를 추가한 뒤, gtk\_tree\_view\_column\_set\_attributes()로 어트리뷰트를 설정한다고 되어있습니다.

여기서 말하는 어트리뷰트(attribute)는, 위 예제에서 보는 것처럼, 렌더러의 속성(property)과("text", "foreground"), 트리모델에서 해당 데이터가 위치한 컬럼 번호(TEXT\_COLUMN, COLOR\_COLUMN)로 구성됩니다. 즉, 앞서 설명한 루프 작업시 어트리뷰트 정보를 이용해 렌더러의 속성을 트리모델의 해당 컬럼 데이터를 가져와서 매번 g\_object\_set() API를 이용해 설정하면 렌더러가 해당 좌표에 표시하게 됩니다.

### GtkCellRenderer Advanced?

모든 GtkCellRenderer 객체가 반드시 어트리뷰트 방식으로 동작할 필요는 없습니다. 예를 들어 컴보박스에 표시되는 텍스트 앞에 동일한 아이콘을 표시하기 위해 굳이 트리모델에 새로운 컬럼을 추가할 필요는 없습니다. 다만 다음과 같은 방식으로 텍스트 렌더러를 추가하기 전에 처리해주면 됩니다.

    c = gtk_cell_renderer_pixbuf_new ();
    g_object_set (G_OBJECT (c), "stock-id", GTK_STOCK_HARDDISK, NULL);
    gtk_tree_view_column_set_pack_start (treeview_column, c, FALSE);

참고로 위 예제에서는 "stock-id" 속성을 사용했지만, 당연히 다른 속성을 사용해도 됩니다.

또한 [gtk\_cell\_layout\_set\_cell\_data\_func()](http://library.gnome.org/devel/gtk/stable/GtkCellLayout.html#gtk-cell-layout-set-cell-data-func) 함수를 이용하면, 렌더러 실행시 호출할 함수를 등록할 수 있습니다. 따라서 반드시 트리모델 데이터 뿐 아니라 외부 데이터나 조건을 기준으로 렌더러의 속성을 가공할 수 있습니다.

예를 들어 GtkComboBox를 이용해 트리구조 데이터를 서브메뉴 방식으로 표시하다보면, 기본적으로 서브메뉴 위에 상위 항목이 제목처럼 선택할 수 있게 표시됩니다. 이 항목이 불필요한 경우가 있는데, 다음과 같은 코드를 사용하면 됩니다.

    static void
    combo_box_cell_data_func(GtkCellLayout   *cell_layout,
                             GtkCellRenderer *cell,
                             GtkTreeModel    *tree_model,
                             GtkTreeIter     *iter,
                             gpointer        data)
    {
      gboolean sensitive;
      sensitive = !gtk_tree_model_iter_has_child (tree_model, iter);
      g_object_set (cell, "sensitive", sensitive, NULL);
    }

    ...
    gtk_cell_layout_set_cell_data_func (GTK_CELL_LAYOUT (combobox),
                                        renderer,
                                        combo_box_cell_data_func,
                                        NULL, NULL);

이제, 다른 어플리케이션 소스나 GTK+ 문서를 한번 정독하면 GtkTreeView / GtkComboBox 위젯을 다루는 작업은 더이상 어렵지 않을까요?
