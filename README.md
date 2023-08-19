# TIL_pages

## 概要
ページ：https://izmyuta.github.io/TIL_pages/

TILをカレンダー表示するためのページ

[ここ](https://github.com/IzmYuta/TIL)で書いたmdファイルがこのリポジトリの`_posts/`にコピーされて、GitHub pagesによって表示されるという仕組み

## ワークフローの流れ
[このリポジトリ](https://github.com/IzmYuta/TIL)で今日学んだことを書く

↓

GitHub Actionsのイベントがトリガーされる

↓

トリガーを検知

↓

mdファイルをフォーマットして、コピー

↓

PRを自動作成

↓

自動でマージして反映
