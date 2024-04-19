---
layout: post
title: "minecraft_server"
date: 2024-03-30
category: minecraft
excerpt: ""
---
# Minecraftサーバーについて

参考：<https://minecraftjapan.miraheze.org/wiki/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC>

## 1. サーバーの種類について

大別すると以下の3つ

### バニラサーバー

- 公式サイトからサーバー用の.jarをダウンロードして作成したサーバー。

### MODサーバー

- バニラサーバーに対して、ForgeやFabricなどのAPIを搭載したもの。

### プラグインサーバー

- PaperMC,SpigotMC,PurPurMCなどが該当する
- 独自APIを実装した.jarファイルを利用して作成する
- バニラサーバーとは異なる挙動をすることがある

## 2. サーバーの必要スペック

- CPU: 4~6core
- Mem: 最低1GB, 4GB以上推奨、8GB以上で安心
- ストレージ：チャンクロードが頻繁に発生するため、ランダムアクセス能力の高いものを推奨
- OS: 64bit Linux推奨

###

## Velocityでの鯖立て

- メイン鯖でのonline_modeはfalseにすること
  - online_modeがtrueだとユーザーの検証が行われる
  - velocity側でも検証は行えるが、2重に検証することはできない
  - そのためvelocityのみtrueにし、そのほかのサーバーではfalseにする
- メイン鯖にはFabricを導入すること
  - バニラのサーバーにはユーザー情報の転送機能がなく、プロキシとの通信が行えない
  - Fabric MODを導入し、ユーザー情報を転送できるようにするplugin "FabricProxy-Lite"を追加することで行える
  - バニラサーバーであっても、Vanilla Cordを使えば転送できなくはないが、多分Fabricにした方が楽だと思う



