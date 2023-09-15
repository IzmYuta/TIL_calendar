---
layout: default
title: "投稿一覧"
---

# 投稿一覧

<div class="tab">
  <p class="tabtext">タグ一覧</p>
  <button class="tablinks" onclick="filterCategory('all')">すべて</button>
  <button class="tablinks" onclick="filterCategory('AWS')">AWS</button>
  <button class="tablinks" onclick="filterCategory('DRF')">DRF</button>
  <button class="tablinks" onclick="filterCategory('Git')">Git</button>
  <button class="tablinks" onclick="filterCategory('Go')">Go</button>
  <button class="tablinks" onclick="filterCategory('思想')">思想</button>
  <!-- 他のカテゴリのボタンも追加 -->
</div>

<div id="posts">
  {% for post in site.posts %}
    <div class="post" data-category="{{ post.category }}">
      <a href="{{ post.url | absolute_url }}">
        <span>{{ post.date | date : "%F" }}</span>
        <span>{{ post.title }}</span>
      </a>
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
