---
layout: default
title: "投稿一覧"
---

# タブ切り替え

<div class="tab">
  <button class="tablinks" onclick="filterCategory('all')">すべて</button>
  <button class="tablinks" onclick="filterCategory('Go')">Go</button>
  <!-- 他のカテゴリのボタンも追加 -->
</div>

<div id="posts">
  {% for post in site.posts %}
    <div class="post" data-category="{{ post.category }}">
      - [{{ post.date | date : "%F" }}  {{ post.title }}]({{site.url}}{{site.baseurl}}{{post.url}}) 
    </div>
  {% endfor %}
</div>

<!-- JavaScriptでカテゴリ別にフィルタリングするコード -->
<script>
  function filterCategory(category) {
    var posts = document.querySelectorAll(".post");
    posts.forEach(function(post) {
      var postCategory = post.getAttribute("data-category");
      if (category === "all" || postCategory === category) {
        post.style.display = "block";
      } else {
        post.style.display = "none";
      }
    });
  }
</script>
