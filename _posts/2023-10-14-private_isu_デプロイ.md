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
1. github上でISUCON用のプライベートリポジトリを作る
2. マシンにssh接続(コンソール上でEC2 Instance Connectからsshするのが一番楽？)
   ![18DE798E-61D4-4EBF-ABEA-7A199C6E512C_1_201_a](https://github.com/IzmYuta/TIL/assets/104307371/01dc18b0-d611-4a03-84c2-d80b6c22b50c)
4. sshキーを作成(参考：https://qiita.com/tamorieeeen/items/c24f8285448b607b12dd)
5. アプリケーションコードのリポジトリに移動(cd private_isu)
6. git init
7. git add と git commit
8. デプロイキーを登録してsshでpushする(参考：https://qiita.com/tamorieeeen/items/c24f8285448b607b12dd)

### その他
- マシンの起動
  - ISUCON用のAMIを選択して起動
  - リージョンが東京(ap-northeast-1)になっているか確認
    <img width="1470" alt="5A123D85-D29B-41EF-BAAB-C4EF52FB4031" src="https://github.com/IzmYuta/TIL/assets/104307371/d71ed80f-ddb6-47c4-8795-eab3b9a347a5">
- 動作確認
  - セキュリティグループを編集してHTTP(80)のポートを許可するように編集
  - グローバルIPアドレスをブラウザに入力してアクセス
