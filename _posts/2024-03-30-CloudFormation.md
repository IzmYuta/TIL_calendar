---
layout: post
title: "CloudFormation"
date: 2024-03-30
category: AWS
excerpt: ""
---
# CloudFormationの構文

参考：<https://dev.classmethod.jp/articles/cloudformation-beginner01/>

## 基本

CloudFormationはいくつかのセクションで区切られる。
実は、Resources以外はなくても良い。

- Format Version
- Description
- Metadata
- Parameters
- Mappings
- **Resources**
- Outputs

## Resources

```yaml
Resources:
  <Logical ID>:
    Type: <Resource type>
    Properties:
      <Set of properties...>
```

### Logical ID

- テンプレート内で一意なID。
- テンプレートの中で他のリソースを参照する場合などは、このIDを利用
- スタックのリソース一覧にも、このIDでリソースが表示

### Resource type

- 実際に作成するリソースのタイプ。
- 対応しているリソースタイプはAWS リソースプロパティタイプのリファレンスに一覧がある。
-　<https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html>

### Resource properties

- 各リソースの作成時に指定するプロパティ
- リソースタイプによって利用できるプロパティは異なるので、公式ドキュメントとにらめっこしながら、指定

## 応用

### クロススタック参照

他のスタックからリソースのIDなど持ってくる必要があるとき、次のようにして参照することができる
<https://aws.amazon.com/jp/blogs/news/aws-cloudformation-%E3%81%AE%E6%9B%B4%E6%96%B0-yaml%E3%80%81%E3%82%AF%E3%83%AD%E3%82%B9%E3%82%B9%E3%82%BF%E3%83%83%E3%82%AF%E5%8F%82%E7%85%A7%E3%80%81%E7%B0%A1%E7%95%A5%E5%8C%96%E3%81%95%E3%82%8C/>
