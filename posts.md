---
layout: default
title: "投稿一覧"
---

# 投稿一覧

<style>
/* ボタンの基本スタイル */
.tablinks {
  background-color: #f2f2f2;
  border: none;
  color: #333;
  padding: 10px 20px;
  cursor: pointer;
  transition: background-color 0.3s;
}

/* 選択中のボタンのスタイル */
.tablinks.active {
  background-color: #007bff;
  color: #fff;
}
</style>

<div class="tab">
  <p class="tabtext">タグ</p>
  <button class="tablinks" onclick="filterCategory('all')">すべて</button>
  <button class="tablinks" onclick="filterCategory('AWS')">AWS</button>
  <button class="tablinks" onclick="filterCategory('DRF')">DRF</button>
  <button class="tablinks" onclick="filterCategory('git')">git</button>
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
