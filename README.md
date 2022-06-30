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
 
 可以看到,最终的返回值为内嵌view，并且内嵌view的返回值为 self.dispatch, 但是 ViewSetMixin 没有 dispatch 方法
 
 这个就是Mixin的思想，我提供一些依赖于别人的方法
 
 这个dispatch 就是真正视图函数，每一个请求进来都会进入这个dispatch方法，非常的关键这个方法
 
 那我们就来看看这个关键的 dispatch 方法
 
 ```python3
     def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?
        try:
            self.initial(request, *args, **kwargs)
            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed
            response = handler(request, *args, **kwargs)
        except Exception as exc:
            response = self.handle_exception(exc)
        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response
 ```
 
 首先会做一个 initialize_request，看名字就知道是对request初始化，实际是把request重新包装了一下,并且传了parsers，authenticators，negotiator，parser_context进去
 ```python3
     def initialize_request(self, request, *args, **kwargs):
        """
        Returns the initial request object.
        """
        parser_context = self.get_parser_context(request)
        return Request(
            request,
            parsers=self.get_parsers(),  # 解析器
            authenticators=self.get_authenticators(),  # 认证器
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )
 ```
 
 接下来接续看 dispatch ,关键点：
 ```python3
 self.initial(request, *args, **kwargs) # 中最后三行，分别执行了 认证，权限，频率控制
 ```
 
 来看认证, 只执行了一个request.user，我们要知道这个request已经drf包装后的request的，所以我们继续看request.user
 
 ```python3
     def perform_authentication(self, request):
        """
        Perform authentication on the incoming request.

        Note that if you override this and simply 'pass', then authentication
        will instead be performed lazily, the first time either
        `request.user` or `request.auth` is accessed.
        """
        request.user
 ```
 
 可以看到 request.user 中 调用了 _authenticate,而 _authenticate 使用了之前包装的时候传进来的authenticators，
 
 然后在调用authenticate，并且返回值应该是两个，第一个给user, 第二个给auth
 
 这就是为什么drf定义的 BASE authenticator 必须写 authenticate方法，
 
 ```python3
     def _authenticate(self):
        """
        Attempt to authenticate the request using each authentication instance
        in turn.
        """
        for authenticator in self.authenticators:
            try:
                user_auth_tuple = authenticator.authenticate(self)
            except exceptions.APIException:
                self._not_authenticated()
                raise

            if user_auth_tuple is not None:
                self._authenticator = authenticator
                self.user, self.auth = user_auth_tuple
                return

        self._not_authenticated()
 ```
 
 
