---
layout: post
title: "Singleton"
date: 2023-08-22
category: 思想
excerpt: ""
---
# Singleton
## 概要
- シングルトン(クラス)とは、プログラム中にそのクラスのインスタンスが1つしか存在しないという、デザインパターンの1つ
- シングルトンはステートレスなオブジェクトであるべき
  - ステートレス：状態を持たない。すなわち過去のデータが未来に影響を与えない。ユーザーによって挙動を変えない。


## 実装方法
```python
class SingletonModel(models.Model):
    """Singleton Django Model"""

    class Meta:
        abstract = True

    def save(self, *args, **kwargs):
        """
        Save object to the database. Removes all other entries if there
        are any.
        """
        self.__class__.objects.exclude(id=self.id).delete()
        super(SingletonModel, self).save(*args, **kwargs)
```
