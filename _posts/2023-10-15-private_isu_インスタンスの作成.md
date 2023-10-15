---
layout: post
title: "private_isu_インスタンスの作成"
date: 2023-10-15
category: ISUCON
excerpt: ""
---
# private-isu

## 0. インスタンスの作成編
private-isuのリンク：https://github.com/catatsuy/private-isu
### マシンの起動
- リージョンが東京(ap-northeast-1)になっているか確認して、ISUCON用のAMIを検索して起動
- 無料枠(インスタンスタイプ：t2.micro)で試したいならx86_64アーキテクチャを選択すること
- 名前とタグ、キーペア、インスタンスタイプを設定したらあとはデフォルトでOK
<img width="1470" alt="5A123D85-D29B-41EF-BAAB-C4EF52FB4031" src="https://github.com/IzmYuta/TIL/assets/104307371/d71ed80f-ddb6-47c4-8795-eab3b9a347a5">

競技者用 (Ubuntu 22.04):

| 用途   |        AMI ID         |              AMI name               | 推奨インスタンスタイプ |
| ------ | :-------------------: | :---------------------------------: | ---------------------- |
| x86_64 | ami-0d92a4724cae6f07b | catatsuy_private_isu_amd64_20230917 | c6i.large              |
| arm64  | ami-0a435708a83cc3ee5 | catatsuy_private_isu_arm64_20230917 | c6g.large              |

ベンチマーカー (Ubuntu 22.04):

| 用途   |        AMI ID         |                 AMI name                  | 推奨インスタンスタイプ |
| ------ | :-------------------: | :---------------------------------------: | ---------------------- |
| x86_64 | ami-0582a2a7fbe79a30d | catatsuy_private_isu_bench_amd64_20230514 | c6i.xlarge             |
| arm64  | ami-01888a2782271061e | catatsuy_private_isu_bench_arm64_20230514 | c6g.xlarge             |



### 動作確認
- 競技者用インスタンスのセキュリティグループのインバウンドルールにHTTP(80)のポートを許可する設定を追加する
 - インスタンス -> セキュリティから選択すると楽
![2C30A96D-8811-426F-B915-379A7CD84F65_1_201_a](https://github.com/IzmYuta/TIL/assets/104307371/49fec8e8-9fd2-4933-8866-8e6e58be28c7)
<img width="1470" alt="8683C56D-65AA-4862-9FB3-6B39C37D836A" src="https://github.com/IzmYuta/TIL/assets/104307371/56b13735-b98b-4906-bd29-a97614372dce">
<img width="1470" alt="9DEC05CC-C605-4E0C-9FB7-3682DEFB852A" src="https://github.com/IzmYuta/TIL/assets/104307371/70a214c9-5a9d-4617-9bac-b3e3613c2aa7">

- グローバルIPアドレスをブラウザにコピーしてアクセス(httpで)
![ADC36BD3-6C14-4376-A753-58188BC96CE7_1_201_a](https://github.com/IzmYuta/TIL/assets/104307371/367c4a85-a1bf-4b96-9113-1ec70b21b9be)
<img width="1470" alt="C544BB56-3876-4876-B4CB-FB2EED60671B" src="https://github.com/IzmYuta/TIL/assets/104307371/8586afd1-e38b-4953-bd34-b812ff318374">
