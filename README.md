### DRF 源码解析

#### 1.routers
```
## drf 路由的写法
from rest_framework.routers import DefaultRouter
from django.urls import include, path

from test_drf.views import AnimalViewSet

router = DefaultRouter(trailing_slash=False)
router.register('animal', AnimalViewSet)

urlpatterns = [
    path('', include(router.urls))
]

```
我们知道  urlpatterns 是django自带固定的变量，在上层 url incldue 的时候会用到这个变量
所以关键点就在于 router.urls ，我们可以打印一下这个值，

```
[
<URLPattern '^animal$' [name='animal-list']>,
<URLPattern '^animal\.(?P<format>[a-z0-9]+)/?$' [name='animal-list']>,
<URLPattern '^animal/(?P<pk>[^/.]+)$' [name='animal-detail']>,
<URLPattern '^animal/(?P<pk>[^/.]+)\.(?P<format>[a-z0-9]+)/?$' [name='animal-detail']>,
<URLPattern '^$' [name='api-root']>, <URLPattern '^\.(?P<format>[a-z0-9]+)/?$' [name='api-root']>
]

```
