---
layout: default
title: "投稿一覧"
---



<div class="tab">
  <button class="tablinks" onclick="filterCategory('all')">すべて</button>
  <button class="tablinks" onclick="filterCategory('Go')">Go</button>
  <!-- 他のカテゴリのボタンも追加 -->
</div>

<div id="posts">
  {% for post in site.posts %}
    <div class="post" data-category="{{ post.category }}" data-content="{{ post.content | markdownify }}">
    </div>
  {% endfor %}
</div>

<!-- JavaScriptでカテゴリ別にフィルタリングするコード -->
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
<script>
  function filterCategory(category) {
    var posts = document.querySelectorAll(".post");
    posts.forEach(function(post) {
      var postCategory = post.getAttribute("data-category");
      var content = post.getAttribute("data-content");
      var renderedContent = marked(content);

      if (category === "all" || postCategory === category) {
        post.innerHTML = renderedContent;
      } else {
        post.innerHTML = ""; // 非表示
      }
    });
  }
</script>
