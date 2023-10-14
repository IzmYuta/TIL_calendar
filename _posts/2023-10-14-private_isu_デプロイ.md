---
layout: post
title: "private_isu_デプロイ"
date: 2023-10-14
category: ISUCON
excerpt: ""
---
# private-isu攻略

## デプロイ編
- ISUCONではまずマシンが提供される
  - githubリポジトリは提供されないので自分で作る必要がある
  - マシン上でコマンドを実行する必要がある
    1. github上でISUCON用のプライベートリポジトリを作る
    2. アプリケーションコードのリポジトリに移動
    3. git init
    4. git add と git commit
    5. デプロイキーでssh pushする

### 具体的にやったこと
- マシンの起動
  - ISUCON用のAMIを選択して起動
- 動作確認
  - セキュリティグループを編集してHTTP(80)のポートを許可するように編集
  - グローバルIPアドレスをブラウザに入力してアクセス
