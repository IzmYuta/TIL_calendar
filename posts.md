---
layout: default
title: "投稿一覧"
---

# タブ切り替え

<div class="tab">
  <button class="tablinks" onclick="filterPosts('All')">すべて</button>
  <button class="tablinks" onclick="filterPosts('Go')">Go</button>
  <!-- 他のカテゴリのボタンも追加 -->
</div>

<div id="posts">
  {% for post in site.posts %}
    <div class="post" data-category="{{ post.category }}">
      - [{{ post.date | date : "%F" }}  {{ post.title }}]({{site.url}}{{site.baseurl}}{{post.url}})
    </div>
  {% endfor %}
</div>

<script>
function filterPosts(category) {
  var posts = document.getElementsByClassName('post');
  for (var i = 0; i < posts.length; i++) {
    var post = posts[i];
    var postCategory = post.getAttribute('data-category');
    
    if (category === 'All' || postCategory === category) {
      post.style.display = 'block';
    } else {
      post.style.display = 'none';
    }
  }
}

// 初期状態ではすべてのカテゴリの記事を表示
filterPosts('All');
</script>
