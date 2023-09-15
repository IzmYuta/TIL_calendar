---
layout: default
title: "投稿一覧"
---

#　投稿一覧


<div class="tab">
  <a href="{{ page.url }}#all" class="tablinks">すべて</a>
  <a href="{{ page.url }}#go" class="tablinks">Go</a>
  <!-- 他のカテゴリのリンクも追加 -->
</div>

{% for post in site.posts %}
  <div id="{{ post.category }}" class="post">
    - [{{ post.date | date : "%F" }}  {{ post.title }}]({{site.url}}{{site.baseurl}}{{post.url}})
  </div>
{% endfor %}
