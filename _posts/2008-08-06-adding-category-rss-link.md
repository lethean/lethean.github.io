---
layout: post
title: "카테고리 RSS 링크 추가"
tags: [ WordPress ]
---

워드프레스로 블로그를 옮기면서 다짐했던 것 중 하나가 가능한 소스를 건드리지 않고 일반 사용자처럼 사용하자였는데... 카테고리별 RSS 피드를 왜 설정에서 쉽게 변경하지 못하고, 테마나 위젯 소스를 건드려야만 하도록 했을까, 결국 위젯 코드를 수정해서 원하는 기능을 얻었습니다.

    public_html/wp-includes/widgets.php 2008-07-22 12:09:23
    @@ -745,6 +745,8 @@
      <ul>
      <?php
        $cat_args['title_li'] = '';
    +   $cat_args['feed'] = 'rss';
    +   $cat_args['feed_image'] = 'rssfeed.jpg';
        wp_list_categories($cat_args);
      ?>
      </ul>

더이상 코드를 수정할 일 없기만을 바랄 뿐...
