---
layout: post
title: "last_login"
date: 2023-09-26
category: DRF
excerpt: ""
---
# Djangoでのlast_loginの更新方法

Djangoのlast_loginフィールドは、ユーザーモデルがDjangoのデフォルトのAbstractBaseUserまたはAbstractUserを拡張している場合に、ユーザーがログインするたびに自動的に更新されます。これはDjangoのセッション管理と認証フレームワークに組み込まれています。

last_loginフィールドの更新は、以下の手順で行われます：

1. ユーザーがログインし、認証が成功すると、Djangoは`login`ビューまたは`auth.login`メソッドを使用して`last_login`フィールドを更新します。
2. `login`ビューまたは`auth.login`メソッドは、`update_last_login`関数を呼び出します。
3. `update_last_login`関数は、Userモデルの`last_login`フィールドを現在の日時に設定します。具体的には、`timezone.now()`を使用して現在の日時を取得し、`last_login`フィールドにセットします。
