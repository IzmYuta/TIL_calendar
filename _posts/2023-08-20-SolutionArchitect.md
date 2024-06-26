---
layout: post
title: "SolutionArchitect"
date: 2023-08-20
category: AWS
excerpt: ""
---
# VPCについて
## IPアドレスの基礎
### 概要

- ネットワーク機器やWebサイトの場所を表している、住所のようなもの
- ICANNという非営利団体が管理
- 重複が許されない32ビットの数値データ
- 0.0.0.0 ~ 255.255.255.255の範囲をとる
- IPアドレスはネットワークインターフェースカード(NIC)に割り当てられる
  - NICがホストマシンにアタッチされることでIPアドレスが割り振られる
- IPv4は枯渇しつつある
  - IPv6への移行が進められている

### グローバルIPアドレスとプライベートIPアドレス

- グローバル：世界で一意の値
- プライベート：利用範囲を制限することで自由に割り振ることができる

## サブネットマスクとサブネット
### プライベートネットワークの作り方

- ネットワークの範囲を決める
  - =IPアドレスの利用可能な範囲
  - (例)10.0.1.0/16
  - IPアドレス(10.0.1.0)+サブネットマスク(/16)によって構成

### サブネットマスクとIPアドレス

- IPアドレスは00000000.00000000.00000000.00000000の8×4=32ビットで構成される
- サブネットの数値の意味：左からNビットまでが固定される。/NのことをCIDRという。
  -(例)192.0.1.0/16 -> 192.0.1.0 ~ 192.0.255.255 までがサブネットの範囲になる
- サブネットマスクによって固定される数値の部分をネットワーク部、残りの部分をホスト部という
- 単なるネットワークには/16、その中でサブネットを分ける時は/24を使うのが一般的

### CIDRとIPアドレス数

| サブネットマスク | IPアドレス数 |
| --- | --- |
| /16 | 65536 |
| /18 | 16384 |
| /20 | 4096 |
| /22 | 1024 |
| /24 | 256 |
| /26 | 64 |
| /28 | 16 |

### サブネットの役割

- サブネットによってマシンをグループ化することで、該当のIPアドレスを見つけやすくする
- また、グループごとに役割を変える、ということもできる

## VPCの概要

VPCとは、AWSのネットワークからユーザー専用の領域を切り出すことができる、仮想ネットワークのサービス

### 特徴

- リージョン内に5つまでVPC設定可能(上限緩和申請をすれば増やせる)
- IPアドレス範囲(CIDR)を洗濯して仮想ネットワークを構築
  - 最小： /28
  - 最大： /16
- VPCはプライベートIPアドレスによってネットワークレンジを設定(10.0.2.0/24などがよく使われる)
- サブネットの作成、ルートテーブルやネットワークゲートウェイの設定などをして、仮想ネットワーク環境を完全に制御できる
- 必要に応じてクラウド内外のネットワーク同士を接続したり、外部のネットワークと接続することが可能
  - VPC同士、オンプレミスのプライベートネットワークとの接続が可能

### 利用できるIPアドレスの数

上の表から、AWS管理用のIPアドレス5つを引いたものが実際に使えるIPアドレスの数

### AWS管理IPアドレス(/24の場合)

| ホストアドレス | 用途 |
| --- | --- |
| .0 | ネットワークアドレス |
| .1 | VPCルータ |
| .2 | Amazonが提供するDNSサービス |
| .3 | AWSで予約されているIPアドレス |
| .255 | ブロードキャストアドレス |

### VPCの構成

- 1つのVPC、1つのサブネットが最小構成
- 単一のサブネットがAZの範囲になる
- サブネットを追加することで複数AZにVPCを広げることができる
- **ただし、VPCはリージョンを超えることはできない**

### デフォルトVPCについて

- AWSアカウントを作成すると、自動的に各リージョンに1つずつデフォルトVPCをデフォルトサブネットが作成される
- サイズ/16のIPv4 CIDRブロック(172.31.0.0/16)のVPCを作成する
- 各AZにサイズ/20のデフォルトサブネットを作成する
- インターネットゲートウェイ、セキュリティグループ、ネットワークアクセスコントロールリスト(ACL)が関連付けられている
- AWSアカウントにはデフォルトDHCPオプションが関連付けられている
- パブリックとプライベートのDNSホスト名が付与される

### カスタムVPCの作成方法

**コンソール画面から作成**
1. VPCを作成
2. サブネットを作成
3. インターネット経路(Gateway)を設定
4. VPCへのトラフィック許可の設定(ネットワークACL)

**VPCウィザードから作成**
- 視覚的にネットワークを構成することができる
- 上の操作を一括で行うことができる

### サブネットについて

- サブネットはCIDRで分割されたネットワーク
- 1サブネット1AZ。1つのサブネット内で複数のAZにまたがることはできない。
- 1つのVPC内に200個のサブネットを作成することができる
- サブネットはVPCのCIDR範囲内で定義する必要がある
- サブネットのタイプはインターネットゲートウェイへの経路がルートテーブルで設定されるか否かで決まる
  - 設定されるとパブリックサブネット
  - 設定されないとプライベートサブネット
  - サブネット作成時はルートテーブルに設定されないので、プライベートサブネットとして作成される
-  サブネットの使い分け
  - 公開したいサービス：パブリックサブネット
  - セキュリティを高めたいサービス：プライベートサブネット
- プライベートサブネットへのアクセス方法
  - パブリックサブネットに接続されたサーバーを経由してアクセス(踏み台サーバー)
  - レスポンスを返したい時はNATゲートウェイを使う

### ゲートウェイの種類

| 種類 | 用途 |
| --- | --- |
| インターネットゲートウェイ| ・インターネットの出入り口になるゲートウェイ<br>・デフォルトゲートウェイとして使われる<br>・インターネットゲートウェイをVPCに1つ配置する |
| NATゲートウェイ | ・プライベートサブネットのリソースから、インターネットへのトラフィックを可能にするためのゲートウェイ<br>・プライベートアドレスをパブリックアドレスに変換してインターネットゲートウェイに連携させる |
| Egress-Only Internet Gateway | ・IPv6向けのインターネットゲートウェイ<br>・IPv6経由でのVPCからインターネットへの送信を可能にする<br>・インターネットからのインスタンスへの接続(Ingress)を防ぐ |
| カスタマーゲートウェイ | ・オンプレミス環境と接続する際に使うゲートウェイ<br>・カスタマーゲートウェイデバイス、またはソフトウェアアプリケーションに関する情報をAWSに提供する |
| 仮想プライベートゲートウェイ | ・仮想プライベートゲートウェイは、VPNトンネルのAmazon側にあるルーター<br>・VPN接続時に利用 |

### VPCにおけるDNS

| 種類 | 用途 |
| --- | --- |
| enableDnsHostname | ・パブリックIPアドレスを持つインスタンスが、対応するパブリックDNSホスト名を取得するかどうかの設定<br>・この属性がtrueで、enableDnsSupportもtrueの場合、VPC内のインスタンスはDNSホスト名を取得する |
| enableDnsSupport | ・DNS解決がサポートされているかをどうかの設定<br>・この属性がfalseの場合、パブリックDNSホスト名をIPアドレスに変換するAmazon Route53 Resolverサーバーが機能しなくなる<br>・trueの場合、Amazonが提供するDNSサーバー(169.254.169.253)へのクエリ、またはリザーブドIPアドレス(VPC IPv4のネットワーク範囲に2をプラスしたアドレス)へのクエリが成功する |

- 何に使うのか？
  - Route53でプライベートホストゾーンにカスタムDNSドメイン名を利用したい場合に必要 
