---
layout: post
title: "Singleton"
date: 2023-08-22
category: 思想
excerpt: ""
---
# Singleton
## 概要
- シングルトン(クラス)とは、システム中にそのクラスのインスタンスが1つしか存在しないという、デザインパターンの1つ
- 最初の1つ目のインスタンスをシステム全体で使い回す
- シングルトンはステートレスなオブジェクトであるべき
  - ステートレス：状態を持たない。すなわち過去のデータが未来に影響を与えない。ユーザーによって挙動を変えない。
- 利用例：設定ファイル、ログファイル
- 類：シングルインスタンス

## 利点
- 指定したクラスのインスタンスが1つしか存在しないことを保証することができる

## 欠点
- インスタンスが1つしかないので、マルチスレッド下では同時編集など注意しなければならない
- リレーションがあると、それも一緒に消えてしまう可能性がある(on_delete=CASCADEの場合)
  - = 変更履歴が残せない

## 実装方法(Django)
厳密にはシングルトンではないが...
```python
class SingletonModel(models.Model):
    # Singleton Django Model

    class Meta:
        abstract = True

    def save(self, *args, **kwargs):
        # Save object to the database. Removes all other entries if there are any.

        self.__class__.objects.exclude(id=self.id).delete()
        super().save(*args, **kwargs)
```
参考：https://stackoverflow.com/questions/49735906/how-to-implement-singleton-in-django

