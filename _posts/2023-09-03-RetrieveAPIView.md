---
layout: post
title: "RetrieveAPIView"
date: 2023-09-03
category: DRF
excerpt: ""
---
# RetrieveAPIViewの動作

まず、django.rest_framework.genericsのRetrieveAPIViewの実装を見る
```python
class RetrieveAPIView(mixins.RetrieveModelMixin, GenericAPIView):
    """
    Concrete view for retrieving a model instance.
    """
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
```
self.retrieveメソッドで何をしているのか、RetrieveModelMixinを見てみる。
```python
class RetrieveModelMixin:
    """
    Retrieve a model instance.
    """
    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        return Response(serializer.data)
```
self.get_objectでは何をしているのかを、GenericAPIViewを見る。
```python
    def get_object(self):
        """
        Returns the object the view is displaying.

        You may want to override this if you need to provide non-standard
        queryset lookups.  Eg if objects are referenced using multiple
        keyword arguments in the url conf.
        """
        queryset = self.filter_queryset(self.get_queryset())

        # Perform the lookup filtering.
        lookup_url_kwarg = self.lookup_url_kwarg or self.lookup_field

        assert lookup_url_kwarg in self.kwargs, (
            'Expected view %s to be called with a URL keyword argument '
            'named "%s". Fix your URL conf, or set the `.lookup_field` '
            'attribute on the view correctly.' %
            (self.__class__.__name__, lookup_url_kwarg)
        )

        filter_kwargs = {self.lookup_field: self.kwargs[lookup_url_kwarg]}
        obj = get_object_or_404(queryset, **filter_kwargs)

        # May raise a permission denied
        self.check_object_permissions(self.request, obj)

        return obj
```
以上のことから、RetribeAPIViewは
- まずget_object()を実行
  - get_queryset()で取得したクエリセットに対して以下の処理を実行
  - URLのパスパラメータ名と一致するクラスのフィールドを検索
  - 一致したクラスフィールドの中から、URLのパスパラメータの値でフィルタリングして、get_object_or_404()する
  - モデルオブジェクトが返却される
- 取得したモデルオブジェクトインスタンスをJSONオブジェクトに(デ)シリアライズする
- レスポンスとしてJSONオブジェクトを返す
