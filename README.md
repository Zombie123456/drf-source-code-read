### DRF 源码解析 - viewset 篇

#### 前言
DRF 框架基于Django的用于构建 Web API 的强大而灵活的工具包。该文章旨在解析drf的源码的关键部分

官方 [source code](https://github.com/encode/django-rest-framework)

导入drf的GenericViewSet，这就是我们的入口
```python3
from rest_framework.viewsets import GenericViewSet
```

可以看到该class什么都没有写，只是继承了两个类，我们先看ViewSetMixin
 ```python3
 class GenericViewSet(ViewSetMixin, generics.GenericAPIView):
    """
    The GenericViewSet class does not provide any actions by default,
    but does include the base set of generic view behavior, such as
    the `get_object` and `get_queryset` methods.
    """
    pass
 ```
 
 ViewSetMixin  主要重写了as_view方法，该方法是用于Django CBV 的方式，在DRF中routers也会调用viewset的as_view方法，
 ```python3

 ```
 
 可以看到 最终的返回值为内嵌view，并且内嵌view的返回值为 self.dispatch, 但是 ViewSetMixin 没有 dispatch方法
