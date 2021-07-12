# `Route`类介绍 
#### 注意：`Route`类仅适用于 `ViewSetView`类视图
已实现的基础类有
* BaseRoute [抽象]
* DefaultRoute [常用]

上述类最终都会生成可适用于 `spot.add_route()` 使用的数据，格式如下
> 由于Route类是为 urls.py 文件服务的所以先看一下urls.py -> urls 需要的数据格式
```python 
# urls.py -> urls 如下格式的数据 List[Dict]
[{
    'handler': TestView,
    'uri': /test,
    'name': 'test',
    'is_base':False, # 缺省为 False
    'methods':['GET','POST'] # 缺省为 ALL_METHOD
}]

# DefaultRoute.urls 生成的数据格式
[{
    'handler': TestView,
    'uri': /test,
    'name': 'test',
    'is_base':False,
}]

# 所以在具体使用时只需要如下即可
route = DefaultRoute()
route.register_route(TestView, '/test', test)
'....'
urls = route.urls

```
> 源代码

```python

class DefaultRoute(BaseRoute):
    def register_route(self, viewset: object, prefix: str, name: str = None, is_base: bool = False):
        """
        注册路由到路由管理类
        :param viewset: 视图，无需
        :param prefix:
        :param name:
        :param is_base:
        :return:
        """

    def get_viewset_method_list(self, viewset):
        """
        得到viewSet所有请求方法
        :param viewset: 类视图
        :return:
        """

    @property
    def urls(self):
        """得到 urls.py 需要的数据"""

    def initialize(self, destination: Union[Sanic, Blueprint]):
        """注册路由到destination中"""
```
