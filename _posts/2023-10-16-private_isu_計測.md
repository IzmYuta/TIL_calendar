---
layout: post
title: "private_isu_計測"
date: 2023-10-16
category: ISUCON
excerpt: ""
---
# private-isu

## 2.計測
### CPU・メモリ使用率
1. ベンチマーカーを実行
2. 実行中にlinuxコマンドを利用してモニタリングする
    - CPU使用率を計測：topコマンド
    - メモリ使用率：freeコマンド
3. 使用率が高いところに対してアプローチ

平常時：
<img width="1470" alt="40B82BF4-BE4A-410A-ADDB-C9E1071886C9" src="https://github.com/IzmYuta/TIL/assets/104307371/3eab0f8e-49ef-443f-b4df-e605d765ce21">
<img width="1470" alt="AF87AE1D-F0D8-4998-8871-14DA0A6E639D" src="https://github.com/IzmYuta/TIL/assets/104307371/ad721532-48eb-412a-96a4-5f1e5606b59b">

ベンチマーク実行時：
![7B8126C8-6FCD-433D-808A-FA3B2ED4A1FC_1_105_c](https://github.com/IzmYuta/TIL/assets/104307371/9e3a50d7-a94e-4952-977d-9ef609e967da)
<img width="1470" alt="7E212517-892E-4A7B-86B3-6162AA6FCD70" src="https://github.com/IzmYuta/TIL/assets/104307371/9d0b02d4-6e53-470b-b1ee-eb0712d41b45">

- ベンチマーク実行時にCPU利用率が大きく上がっていることがわかる
- 一方でメモリ使用率は平常時とほとんど変わっていなかった
- ->CPUリソースがボトルネックになっていると判断できる。
- 特にMySQLのプロセスが高い使用率になっているので、DBに対してアプローチが必要だと判断できる。 

### スロークエリログの取得
- `/etc/mysql/mysql.cnf.d/mysqld.cnf`(Dockerなら`webapp/mysql/conf.d/my.cnf`)の設定を変更することでスロークエリログの有効化ができる
- 設定を反映させるためには`systemctl restart mysql`で再起動すること

```
[mysqld]
slow_query_log      = 1
slow_query_log_file = /var/log/mysql//mysql-slow.log
long_query_time     = 0
```

| 設定 | 意味 |
|---|---|
| slow_query_log | スロークエリログを有効化する |
| slow_query_log_file | スロークエリログの出力先 |
| long_query_time | 指定秒数以上のクエリを出力 |

### スロークエリログの集計(簡易的に確認したい時)
- そのままだと膨大すぎるので、mysqldumpslowコマンドで集計する
- logファイルの閲覧ができるユーザーで実行すること(rootなど)
- ソート機能を使ってクエリ時間順に並べたりもできる

一例：
```bash
$ mysqldumpslow t /var/log/mysql/mysql-slow.log 
Count: 924  Time=0.07s (61s)  Lock=0.00s (0s)  Rows=2.8 (2586), isuconp[isuconp]@localhost
  SELECT * FROM `comments` WHERE `post_id` = N ORDER BY `created_at` DESC LIMIT N
```
- -aオプションでクエリログの詳細を見ることができる(ログが膨大なときは使わないほうがいいかも)
- -sオプションでソート方法の指定ができる。ここではt(実行時間)でソートしている。
- ` SELECT * FROM ~`が発行されたクエリ

### スロークエリログの解析(こちらがメイン)
- pt-query-digestを用いて解析する

インストール(Debian/Ubuntu)：
```bash
sudo apt update
sudo apt install percona-toolkit
```

インストール(Red Hat)：
```bash
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo yum install percona-toolkit
```

## 参考
- ISUCON本
- https://blog.bitjourney.com/entry/2017/11/09/101740
