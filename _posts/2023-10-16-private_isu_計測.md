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
- `pt-query-digest /var/log/mysql/mysql-slow.log`でログを解析できる
- teeコマンドを使ってファイルに書き込むこともできる

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

使用例：
```bash
# 標準出力させる場合
pt-query-digest /var/log/mysql/mysql-slow.log
# ファイル出力させるとき
pt-query-digest /var/log/mysql/mysql-slow.log | tee /home/isucon/private_isu/digest_$(date +%Y%m%d%H%M).txt
```

結果の見方：
- 全体として3部構成になっている
    - 全体的な統計
    - ランキング
    - 各クエリの詳細

まずはランキングから見ていく
- 主なチューニング対象は次の2つ：
    - 実行回数が少ないのに、1回あたりのクエリの負荷が高く、上位に食い込むもの
    - 1回あたりのクエリの負荷は小さいが、実行回数が多く、上位に食い込むもの
```txt
# Profile
# Rank Query ID                      Response time Calls R/Call V/M   Item
# ==== ============================= ============= ===== ====== ===== ====
#    1 0x624863D30DAC59FA16849282... 61.8901 71.2%   924 0.0670  0.00 SELECT comments
#    2 0x422390B42D4DD86C7539A5F4... 20.8125 23.9%   935 0.0223  0.00 SELECT comments
# MISC 0xMISC                         4.2685  4.9% 15408 0.0003   0.0 <23 ITEMS>
```

| 要素 | 意味 |
|---|---|
| Query ID | クエリのハッシュ値 |
| Response Time | 実行時間の合計と全体に占める割合 |
| Calls | 実行された回数 |
| R/Call | 1回あたりの時間 |

次に、ランキング上位のクエリの詳細を見ていく
- クエリの実行回数は924回、実行時間は62秒かかっている
    - Lock timeが小さいため、他のスレッドの影響は受けていなさそう
    - Rows sentとRows examineに着目すると、2000行返すのに、8800万行も読み込んでいることがわかる
    - クエリにはLIMIT 3がついており、最大でも3行しか返さないのに、毎回10万行近く読み込んでいる
    - -> インデックスを付与して、読み込む行数を減らすように改善する
```txt
# Query 1: 10.38 QPS, 0.70x concurrency, ID 0x624863D30DAC59FA16849282195BE09F at byte 2929867
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: 2023-10-16T01:51:19 to 2023-10-16T01:52:48
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          5     924
# Exec time     71     62s    65ms   138ms    67ms    68ms     3ms    65ms
# Lock time     18     3ms     2us    35us     3us     3us     1us     2us
# Rows sent      0   2.53k       0       3    2.80    2.90    0.73    2.90
# Rows examine  48  88.12M  97.66k  97.66k  97.66k  97.04k       0  97.04k
# Query size     3  74.08k      80      83   82.09   80.10    0.18   80.10
# String:
# Databases    isuconp
# Hosts        localhost
# Users        isuconp
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms  ################################################################
# 100ms  #
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT * FROM `comments` WHERE `post_id` = 9981 ORDER BY `created_at` DESC LIMIT 3\G
```

| 要素 | 意味 |
|---|---|
| Count | 実行されたクエリ数 |
| Exec time | クエリの実行にかかった時間 |
| Lock time | クエリの実行までにかかった時間。他のスレッドによるロックの待ち時間 |
| Rows sent | クエリを実行し、クライアントに返した行数 |
| Rows examine | クエリを実行時に検索した行数 |
| Query size | 実行したクエリの長さ(文字数) |

## 参考
- ISUCON本
- https://blog.bitjourney.com/entry/2017/11/09/101740
