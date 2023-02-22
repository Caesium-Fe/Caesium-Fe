# Django框架学习

django的版本选取需要结合python环境版本来抉择，对应关系为：

| Django version | Python versions                                        |
| :------------- | :----------------------------------------------------- |
| 1.8            | 2.7, 3.2 (until the end of 2016), 3.3, 3.4, 3.5        |
| 1.9, 1.10      | 2.7, 3.4, 3.5                                          |
| 1.11           | 2.7, 3.4, 3.5, 3.6                                     |
| 2.0            | 3.4, 3.5, 3.6                                          |
| 2.1            | 3.5, 3.6, 3.7                                          |
| 2.2            | 3.5，3.6，3.7，3.8(在2.2.8中添加)，3.9(在2.2.17中添加) |
| 3.0            | 3.6，3.7，3.8，3.9(在3.0.11中添加)                     |
| 3.1            | 3.6，3.7，3.8，3.9(在3.1.3中添加)                      |
| 3.2            | 3.6，3.7，3.8，3.9，3.10(在3.2.9中添加)                |
| 4.0            | 3.8，3.9，3.10                                         |

```shell
# django项目创建命令：
django-admin startproject xxxxxxxx(项目名)
# django创建app
python manage.py startapp xxxxxxxx(app名)
# django服务启动
python manage.py runserver
```

项目框架的文件管理关系为：

项目名称
	默认项目文件夹[配置,路由等文件]
	功能文件夹[可为多个]
	static文件夹[存放前端js,css]
	templates[存放html]
	db文件
	manage.py管理文件
	
功能文件夹中用于填写model模型和views功能。

## Django的模板语法：

view：｛"HTML变量名" : "views变量名"｝

HTML：｛｛变量名｝｝

Django的模型：在app文件夹中创建models脚本，在脚本中引用django.db库中的models类，用ORM语法创建数据库表字段，然后用命令执行

### 命令如下：

python3 manage.py migrate   # 创建表结构

python3 manage.py makemigrations TestModel  # 让 Django 知道我们在我们的模型有一些变更

python3 manage.py migrate TestModel   # 创建表结构

会自动添加id字段作为主键

需要对应的urls文件中进行对该models脚本文件进行配置

## Django模型

### Django 模型使用自带的 ORM。

对象关系映射（Object Relational Mapping，简称 ORM ）用于实现面向对象编程语言里不同类型系统的数据之间的转换。

ORM 在业务逻辑层和数据库层之间充当了桥梁的作用。

ORM 是通过使用描述对象和数据库之间的映射的元数据，将程序中的对象自动持久化到数据库中。

### Django配置连接sql

```python
# 在项目的 settings.py 文件中找到 DATABASES 配置项，将其信息修改为：
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 数据库引擎
        'NAME': 'learn_sql',  # 数据库名称
        'USER': 'admin',  # 数据库用户名
        'PASSWORD': '123321',  # 数据库密码
        'HOST': '192.168.174.131',  # 数据库地址，本机 ip 地址 127.0.0.1 
        'PORT': '3306',  # 端口 
    }
}
```

### 接下来，告诉 Django 使用 pymysql 模块连接 mysql 数据库：

```python
# 在与 settings.py 同级目录下的 __init__.py 中引入模块和进行配置
import pymysql
pymysql.install_as_MySQLdb()
```

### 修改 TestModel/models.py 文件，代码如下：

```python
# ProjectName/AppName/models.py: 文件代码：
# models.py 
from django.db import models  
class Test(models.Model):
	name = models.CharField(max_length=20)
```

### Django模型迁移

在命令行中运行：

```shell
$ python3 manage.py migrate   # 创建表结构

$ python3 manage.py makemigrations TestModel  # 让 Django 知道我们在我们的模型有一些变更
$ python3 manage.py migrate TestModel   # 创建表结构
```

### 获取数据

Django提供了多种方式来获取数据库的内容，如下代码所示：

```python
# ProjectName/ProjectName/testdb.py: 文件代码：

# -*- coding: utf-8 -*-
 
from django.http import HttpResponse
 
from TestModel.models import Test  # 从AppName中导入module文件
 
# 数据库操作
def testdb(request):
    # 初始化
    response = ""
    response1 = ""
    
    
    # 通过objects这个模型管理器的all()获得所有数据行，相当于SQL中的SELECT * FROM
    list = Test.objects.all()
        
    # filter相当于SQL中的WHERE，可设置条件过滤结果
    response2 = Test.objects.filter(id=1) 
    
    # 获取单个对象
    response3 = Test.objects.get(id=1) 
    
    # 限制返回的数据 相当于 SQL 中的 OFFSET 0 LIMIT 2;
    Test.objects.order_by('name')[0:2]
    
    #数据排序
    Test.objects.order_by("id")
    
    # 上面的方法可以连锁使用
    Test.objects.filter(name="runoob").order_by("id")
    
    # 输出所有数据
    for var in list:
        response1 += var.name + " "
    response = response1
    return HttpResponse("<p>" + response + "</p>")
```

### 更新数据

修改数据可以使用 save() 或 update():

```python
# -*- coding: utf-8 -*-
 
from django.http import HttpResponse
 
from TestModel.models import Test
 
# 数据库操作
def testdb(request):
    # 修改其中一个id=1的name字段，再save，相当于SQL中的UPDATE
    test1 = Test.objects.get(id=1)
    test1.name = 'Google'
    test1.save()
    
    # 另外一种方式
    #Test.objects.filter(id=1).update(name='Google')
    
    # 修改所有的列
    # Test.objects.all().update(name='Google')
    
    return HttpResponse("<p>修改成功</p>")
```

### 删除数据

删除数据库中的对象只需调用该对象的delete()方法即可：

```python
# -*- coding: utf-8 -*-
 
from django.http import HttpResponse
 
from TestModel.models import Test
 
# 数据库操作
def testdb(request):
    # 删除id=1的数据
    test1 = Test.objects.get(id=1)
    test1.delete()
    
    # 另外一种方式
    # Test.objects.filter(id=1).delete()
    
    # 删除所有数据
    # Test.objects.all().delete()
    
    return HttpResponse("<p>删除成功</p>")
```

## Django表单

### GET 方法

我们在之前的项目中创建一个 search.py 文件，用于接收用户的请求：

```python
# ProjectName/ProjectName/search.py 文件代码：
from django.http import HttpResponse
from django.shortcuts import render
# 表单
def search_form(request):
    return render(request, 'search_form.html')
 
# 接收请求数据
def search(request):  
    request.encoding='utf-8'
    if 'q' in request.GET and request.GET['q']:
        message = '你搜索的内容为: ' + request.GET['q']
    else:
        message = '你提交了空表单'
    return HttpResponse(message)
```

在模板目录 templates 中添加 search_form.html 表单：

```html
<!--ProjectName/templates/search_form.html 文件代码：-->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
    <form action="/search/" method="get">
        <input type="text" name="q">
        <input type="submit" value="搜索">
    </form>
</body>
</html>
```

urls.py 规则修改为如下形式：

```python
# ProjectName/ProjectName/urls.py 文件代码：
from django.conf.urls import url
from . import views,testdb,search
 
urlpatterns = [
    url(r'^hello/$', views.runoob),
    url(r'^testdb/$', testdb.testdb),
    url(r'^search-form/$', search.search_form),
    url(r'^search/$', search.search),
]
```

### POST 方法

上面我们使用了 GET 方法，视图显示和请求处理分成两个函数处理。

提交数据时更常用 POST 方法。我们下面使用该方法，并用一个URL和处理函数，同时显示视图和处理请求。

我们在 templates 创建 post.html：

```html
<!--ProjectName/templates/post.html 文件代码：-->
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
    <form action="/search-post/" method="post">
        {% csrf_token %}
        <input type="text" name="q">
        <input type="submit" value="搜索">
    </form>
 
    <p>{{ rlt }}</p>
</body>
</html>
```

在模板的末尾，我们增加一个 rlt 记号，为表格处理结果预留位置。

表格后面还有一个 **{% csrf_token %}** 的标签。csrf 全称是 Cross Site Request Forgery。这是 Django 提供的防止伪装提交请求的功能。POST 方法提交的表格，必须有此标签。

在HelloWorld目录下新建 search2.py 文件并使用 search_post 函数来处理 POST 请求：

```python
# ProjectName/ProjectName/search2.py 文件代码：
# -*- coding: utf-8 -*-
 
from django.shortcuts import render
from django.views.decorators import csrf
 
# 接收POST请求数据
def search_post(request):
    ctx ={}
    if request.POST:
        ctx['rlt'] = request.POST['q']
    return render(request, "post.html", ctx)
```

urls.py 规则修改为如下形式：

```python
# ProjectName/ProjectName/urls.py 文件代码：
from django.conf.urls import url
from . import views,testdb,search,search2
 
urlpatterns = [
    url(r'^hello/$', views.hello),
    url(r'^testdb/$', testdb.testdb),
    url(r'^search-form/$', search.search_form),
    url(r'^search/$', search.search),
    url(r'^search-post/$', search2.search_post),
]
```

## Request 对象

### path

 请求页面的全路径,不包括域名—例如, "/hello/"。

### method

请求中使用的HTTP方法的字符串表示。全大写表示。例如:   

```python
if request.method == 'GET':   
	do_something() 
elif request.method == 'POST':   
	do_something_else()
```

### GET

包含所有HTTP GET参数的类字典对象。参见QueryDict 文档。

### POST

包含所有HTTP POST参数的类字典对象。参见QueryDict 文档。

服务器收到空的POST请求的情况也是有可能发生的。也就是说，表单form通过HTTP POST方法提交请求，但是表单中可以没有数据。因此，不能使用语句if request.POST来判断是否使用HTTP POST方法；应该使用if request.method == "POST" (参见本表的method属性)。

注意: POST不包括file-upload信息。参见FILES属性。

### REQUEST

为了方便，该属性是POST和GET属性的集合体，但是有特殊性，先查找POST属性，然后再查找GET属性。借鉴PHP's $_REQUEST。

例如，如果GET = {"name": "john"} 和POST = {"age": '34'},则 REQUEST["name"] 的值是"john", REQUEST["age"]的值是"34".

强烈建议使用GET and POST,因为这两个属性更加显式化，写出的代码也更易理解。

### COOKIES

包含所有cookies的标准Python字典对象。Keys和values都是字符串。

### FILES

包含所有上传文件的类字典对象。FILES中的每个Key都是<input type="file" name="" />标签中name属性的值. FILES中的每个value 同时也是一个标准Python字典对象，包含下面三个Keys:

- filename: 上传文件名,用Python字符串表示
- content-type: 上传文件的Content type
- content: 上传文件的原始内容

注意：只有在请求方法是POST，并且请求页面中<form>有enctype="multipart/form-data"属性时FILES才拥有数据。否则，FILES 是一个空字典。

### META

包含所有可用HTTP头部信息的字典。 例如:

- CONTENT_LENGTH
- CONTENT_TYPE
- QUERY_STRING: 未解析的原始查询字符串
- REMOTE_ADDR: 客户端IP地址
- REMOTE_HOST: 客户端主机名
- SERVER_NAME: 服务器主机名
- SERVER_PORT: 服务器端口

META 中这些头加上前缀 **HTTP_** 为 Key, 冒号(:)后面的为 Value， 例如:

- HTTP_ACCEPT_ENCODING
- HTTP_ACCEPT_LANGUAGE
- HTTP_HOST: 客户发送的HTTP主机头信息
- HTTP_REFERER: referring页
- HTTP_USER_AGENT: 客户端的user-agent字符串
- HTTP_X_BENDER: X-Bender头信息

### user

是一个django.contrib.auth.models.User 对象，代表当前登录的用户。

如果访问用户当前没有登录，user将被初始化为django.contrib.auth.models.AnonymousUser的实例。

你可以通过user的is_authenticated()方法来辨别用户是否登录：

```python
if request.user.is_authenticated():
    # Do something for logged-in users.
else:
    # Do something for anonymous users.
```

只有激活Django中的AuthenticationMiddleware时该属性才可用

### session

唯一可读写的属性，代表当前会话的字典对象。只有激活Django中的session支持时该属性才可用。

### raw_post_data

原始HTTP POST数据，未解析过。 高级处理时会有用处。

-----------------------------------------------------------------------------------------------------------

Request对象也有一些有用的方法：

### _ _getitem__(key)

返回GET/POST的键值,先取POST,后取GET。如果键不存在抛出 KeyError。
这是我们可以使用字典语法访问HttpRequest对象。
例如,request["foo"]等同于先request.POST["foo"] 然后 request.GET["foo"]的操作。

### has_key()

检查request.GET or request.POST中是否包含参数指定的Key。

### get_full_path()

返回包含查询字符串的请求路径。例如， "/music/bands/the_beatles/?print=true"

### is_secure()

如果请求是安全的，返回True，就是说，发出的是HTTPS请求。

## QueryDict对象

在HttpRequest对象中, GET和POST属性是django.http.QueryDict类的实例。

QueryDict类似字典的自定义类，用来处理单键对应多值的情况。

QueryDict实现所有标准的词典方法。还包括一些特有的方法：

### _ _getitem__

和标准字典的处理有一点不同，就是，如果Key对应多个Value，_ _getitem__()返回最后一个value。

### _ _setitem__

设置参数指定key的value列表(一个Python list)。注意：它只能在一个mutable QueryDict 对象上被调用(就是通过copy()产生的一个QueryDict对象的拷贝).

### get()

如果key对应多个value，get()返回最后一个value。

### update()

参数可以是QueryDict，也可以是标准字典。和标准字典的update方法不同，该方法添加字典 items，而不是替换它们:

```python
>>> q = QueryDict('a=1')
>>> q = q.copy() # to make it mutable
>>> q.update({'a': '2'})
>>> q.getlist('a')
 ['1', '2']
>>> q['a'] # returns the last
['2']
```

### items()

和标准字典的items()方法有一点不同,该方法使用单值逻辑的_ _getitem__():

```python
>>> q = QueryDict('a=1&a=2&a=3')
>>> q.items()
[('a', '3')]
```

### values()

和标准字典的values()方法有一点不同,该方法使用单值逻辑的_ _getitem__():



此外, **QueryDict**也有一些方法，如下：

### copy()

返回对象的拷贝，内部实现是用Python标准库的copy.deepcopy()。该拷贝是mutable(可更改的) — 就是说，可以更改该拷贝的值。

### getlist(key)

返回和参数key对应的所有值，作为一个Python list返回。如果key不存在，则返回空list。 It's guaranteed to return a list of some sort..

### setlist(key,list_)

设置key的值为list_ (unlike _ _setitem__()).

### appendlist(key,item)

添加item到和key关联的内部list.

### setlistdefault(key,list)

和setdefault有一点不同，它接受list而不是单个value作为参数。

### lists()

和items()有一点不同, 它会返回key的所有值，作为一个list, 例如:

```python
>>> q = QueryDict('a=1&a=2&a=3')
>>> q.lists()
[('a', ['1', '2', '3'])]
```

### urlencode()

返回一个以查询字符串格式进行格式化后的字符串(例如："a=2&b=3&b=5")。



## Django视图

一个视图函数，简称视图，是一个简单的 Python 函数，它接受 Web 请求并且返回 Web 响应。

响应可以是一个 HTML 页面、一个 404 错误页面、重定向页面、XML 文档、或者一张图片...

无论视图本身包含什么逻辑，都要返回响应。代码写在哪里都可以，只要在 Python 目录下面，一般放在项目的 views.py 文件中。

每个视图函数都负责返回一个 HttpResponse 对象，对象中包含生成的响应。

视图层中有两个重要的对象：请求对象(request)与响应对象(HttpResponse)。

## 请求对象: HttpRequest 对象（简称 request 对象）

以下介绍几个常用的 request 属性。

### GET

数据类型是 QueryDict，一个类似于字典的对象，包含 HTTP GET 的所有参数。

有相同的键，就把所有的值放到对应的列表里。

取值格式：**对象.方法**。

#### get()

返回字符串，如果该键对应有多个值，取出该键的最后一个值。

### POST

数据类型是 QueryDict，一个类似于字典的对象，包含 HTTP POST 的所有参数。

常用于 form 表单，form 表单里的标签 name 属性对应参数的键，value 属性对应参数的值。

取值格式： **对象.方法**。

#### get()

返回字符串，如果该键对应有多个值，取出该键的最后一个值。

### body

数据类型是二进制字节流，是原生请求体里的参数内容，在 HTTP 中用于 POST，因为 GET 没有请求体。

在 HTTP 中不常用，而在处理非 HTTP 形式的报文时非常有用，例如：二进制图片、XML、Json 等。

### path

获取 URL 中的路径部分，数据类型是字符串。

### method

获取当前请求的方式，数据类型是字符串，且结果为大写。

## 响应对象：HttpResponse 对象

响应对象主要有三种形式：HttpResponse()、render()、redirect()。

### HttpResponse()

返回文本，参数为字符串，字符串中写文本内容。如果参数为字符串里含有 html 标签，也可以渲染。

### render()

返回文本，第一个参数为 request，第二个参数为字符串（页面名称），第三个参数为字典（可选参数，向页面传递的参数：键为页面参数名，值为views参数名）。

### redirect()

重定向，跳转新页面。参数为字符串，字符串中填写页面路径。一般用于 form 表单提交后，跳转到新页面。

render 和 redirect 是在 HttpResponse 的基础上进行了封装：

- render：底层返回的也是 HttpResponse 对象
- redirect：底层继承的是 HttpResponse 对象

# Django 路由

路由简单的来说就是根据用户请求的 URL 链接来判断对应的处理程序，并返回处理结果，也就是 URL 与 Django 的视图建立映射关系。

Django 路由在 urls.py 配置，urls.py 中的每一条配置对应相应的处理方法。

**url() 方法**：普通路径和正则路径均可使用，需要自己手动添加正则首位限制符号。

**path**：用于普通路径，不需要自己手动添加正则首位限制符号，底层已经添加。

**re_path**：用于正则路径，需要自己手动添加正则首位限制符号。

## 正则路径中的分组

### 正则路径中的无名分组

无名分组按位置传参，一一对应。

views 中除了 request，其他形参的数量要与 urls 中的分组数量一致。

### 正则路径中的有名分组

语法： (?P<组名>正则表达式)

有名分组按关键字传参，与位置顺序无关。

views 中除了 request，其他形参的数量要与 urls 中的分组数量一致， 并且 views 中的形参名称要与 urls 中的组名对应。

#### 路由分发(include)

**存在问题**：Django 项目里多个app目录共用一个 urls 容易造成混淆，后期维护也不方便。

**解决**：使用路由分发（include），让每个app目录都单独拥有自己的 urls。

**步骤：**

- 1、在每个 app 目录里都创建一个 urls.py 文件。
- 2、在项目名称目录下的 urls 文件里，统一将路径分发给各个 app 目录。

```python
# 实例
from django.contrib import admin
from django.urls import path,include # 从 django.urls 引入 include
urlpatterns = [
    path('admin/', admin.site.urls),
    path("app01/", include("app01.urls")),
    path("app02/", include("app02.urls")),
]
```

## 反向解析

随着功能的增加，路由层的 url 发生变化，就需要去更改对应的视图层和模板层的 url，非常麻烦，不便维护。

这时我们可以利用反向解析，当路由层 url 发生改变，在视图层和模板层动态反向解析出更改后的 url，免去修改的操作。

反向解析一般用在模板中的超链接及视图中的重定向。

### 普通路径

在 urls.py 中给路由起别名，**name="路由别名"**。

```
path("login1/", views.login, name="login")
```

在 views.py 中，从 django.urls 中引入 reverse，利用 **reverse("路由别名")** 反向解析:

```
return redirect(reverse("login"))
```

在模板 templates 中的 HTML 文件中，利用 **{% url "路由别名" %}** 反向解析。

```
<form action="{% url 'login' %}" method="post"> 
```

### 正则路径（无名分组）

在 urls.py 中给路由起别名，**name="路由别名"**。

```
re_path(r"^login/([0-9]{2})/$", views.login, name="login")
```

在 views.py 中，从 django.urls 中引入 reverse，利用 **reverse("路由别名"，args=(符合正则匹配的参数,))** 反向解析。

```
return redirect(reverse("login",args=(10,)))
```

在模板 templates 中的 HTML 文件中利用 **{% url "路由别名" 符合正则匹配的参数 %}** 反向解析。

```
<form action="{% url 'login' 10 %}" method="post"> 
```

### 正则路径（有名分组）

在 urls.py 中给路由起别名，**name="路由别名"**。

```
re_path(r"^login/(?P<year>[0-9]{4})/$", views.login, name="login")
```

在 views.py 中，从 django.urls 中引入 reverse，利用 **reverse("路由别名"，kwargs={"分组名":符合正则匹配的参数})** 反向解析。

```
return redirect(reverse("login",kwargs={"year":3333}))
```

在模板 templates 中的 HTML 文件中，利用 **{% url "路由别名" 分组名=符合正则匹配的参数 %}** 反向解析。

```
<form action="{% url 'login' year=3333 %}" method="post">
```

## 命名空间

命名空间（英语：Namespace）是表示标识符的可见范围。

一个标识符可在多个命名空间中定义，它在不同命名空间中的含义是互不相干的。

一个新的命名空间中可定义任何标识符，它们不会与任何重复的标识符发生冲突，因为重复的定义都处于其它命名空间中。

**存在问题：**路由别名 name 没有作用域，Django 在反向解析 URL 时，会在项目全局顺序搜索，当查找到第一个路由别名 name 指定 URL 时，立即返回。当在不同的 app 目录下的urls 中定义相同的路由别名 name 时，可能会导致 URL 反向解析错误。

**解决：**使用命名空间。

### 普通路径

定义命名空间（include 里面是一个元组）格式如下：

```
include(("app名称：urls"，"app名称"))
```

实例：

```
path("app01/", include(("app01.urls","app01"))) 
path("app02/", include(("app02.urls","app02")))
```

在 app01/urls.py 中起相同的路由别名。

```
path("login/", views.login, name="login")
```

在 views.py 中使用名称空间，语法格式如下：

```
reverse("app名称：路由别名")
```

实例：

```
return redirect(reverse("app01:login")
```

在 templates 模板的 HTML 文件中使用名称空间，语法格式如下：

```
{% url "app名称：路由别名" %}
```

实例：

```
<form action="{% url 'app01:login' %}" method="post">
```

# Django Admin 管理工具

Django 提供了基于 web 的管理工具。

Django 自动管理工具是 django.contrib 的一部分。你可以在项目的 settings.py 中的 INSTALLED_APPS 看到它：

```python
# ProjectName/ProjectName/settings.py 文件代码：
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
)
```

django.contrib是一套庞大的功能集，它是Django基本代码的组成部分。

------

## 激活管理工具

通常我们在生成项目时会在 urls.py 中自动设置好，我们只需去掉注释即可。

配置项如下所示：

```python
# ProjectName/ProjectName/urls.py 文件代码：
# urls.py
from django.conf.urls import url
from django.contrib import admin
 
urlpatterns = [
    url(r'^admin/', admin.site.urls),
]
```

当这一切都配置好后，Django 管理工具就可以运行了。

------

## 使用管理工具

启动开发服务器，然后在浏览器中访问 http://127.0.0.1:8000/admin/，得到默认的管理界面。

你可以通过命令 **python manage.py createsuperuser** 来创建超级用户，如下所示：

```shell
# python manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: admin@runoob.com
Password:
Password (again):
Superuser created successfully.
[root@solar HelloWorld]#
```

为了让 admin 界面管理某个数据模型，我们需要先注册该数据模型到 admin。比如，我们之前在 TestModel 中已经创建了模型 Test 。修改 TestModel/admin.py:

```python
# ProjectName/TestModel/admin.py: 文件代码：

from django.contrib import admin
from TestModel.models import Test
 
# Register your models here.
admin.site.register(Test)
```

刷新后即可看到 Testmodel 数据表在界面上显示。

## 复杂模型

管理页面的功能强大，完全有能力处理更加复杂的数据模型。

先在 TestModel/models.py 中增加一个更复杂的数据模型：

```python
# ProjectName/TestModel/models.py: 文件代码：

from django.db import models
 
# Create your models here.
class Test(models.Model):
    name = models.CharField(max_length=20)
 
class Contact(models.Model):
    name   = models.CharField(max_length=200)
    age    = models.IntegerField(default=0)
    email  = models.EmailField()
    def __unicode__(self):
        return self.name
 
class Tag(models.Model):
    contact = models.ForeignKey(Contact, on_delete=models.CASCADE,)
    name    = models.CharField(max_length=50)
    def __unicode__(self):
        return self.name
```

这里有两个表。Tag 以 Contact 为外部键。一个 Contact 可以对应多个 Tag。

我们还可以看到许多在之前没有见过的属性类型，比如 IntegerField 用于存储整数。

在 TestModel/admin.py 注册多个模型并显示：

```python
# ProjectName/TestModel/admin.py: 文件代码：
from django.contrib import admin
from TestModel.models import Test,Contact,Tag
 
# Register your models here.
admin.site.register([Test, Contact, Tag])
```

在以上管理工具我们就能进行复杂模型操作。

如果你之前还未创建表结构，可使用以下命令创建：

```shell
$ python manage.py makemigrations TestModel  # 让 Django 知道我们在我们的模型有一些变更
$ python manage.py migrate TestModel   # 创建表结构
```

## 自定义表单

我们可以自定义管理页面，来取代默认的页面。比如上面的 "add" 页面。我们想只显示 name 和 email 部分。修改 TestModel/admin.py:

```python
# ProjectName/TestModel/admin.py: 文件代码：
from django.contrib import admin
from TestModel.models import Test,Contact,Tag
 
# Register your models here.
class ContactAdmin(admin.ModelAdmin):
    fields = ('name', 'email')
 
admin.site.register(Contact, ContactAdmin)
admin.site.register([Test, Tag])
```

以上代码定义了一个 ContactAdmin 类，用以说明管理页面的显示格式。

里面的 fields 属性定义了要显示的字段。

由于该类对应的是 Contact 数据模型，我们在注册的时候，需要将它们一起注册。

我们还可以将输入栏分块，每个栏也可以定义自己的格式。修改 TestModel/admin.py为：

```python
# ProjectName/TestModel/admin.py: 文件代码：
from django.contrib import admin
from TestModel.models import Test,Contact,Tag
 
# Register your models here.
class ContactAdmin(admin.ModelAdmin):
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',), # CSS
            'fields': ('age',),
        }]
    )
 
admin.site.register(Contact, ContactAdmin)
admin.site.register([Test, Tag])
```

上面的栏目分为了 Main 和 Advance 两部分。classes 说明它所在的部分的 CSS 格式。

## 内联(Inline)显示

上面的 Contact 是 Tag 的外部键，所以有外部参考的关系。

而在默认的页面显示中，将两者分离开来，无法体现出两者的从属关系。我们可以使用内联显示，让 Tag 附加在 Contact 的编辑页面上显示。

修改TestModel/admin.py：

```python
# HelloWorld/TestModel/admin.py: 文件代码：
from django.contrib import admin
from TestModel.models import Test,Contact,Tag
 
# Register your models here.
class TagInline(admin.TabularInline):
    model = Tag
 
class ContactAdmin(admin.ModelAdmin):
    inlines = [TagInline]  # Inline
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]
 
    )
 
admin.site.register(Contact, ContactAdmin)
admin.site.register([Test])
```

## 列表页的显示

在 Contact 输入数条记录后

我们也可以自定义该页面的显示，比如在列表中显示更多的栏目，只需要在 ContactAdmin 中增加 list_display 属性:

```python
# HelloWorld/TestModel/admin.py: 文件代码：
from django.contrib import admin
from TestModel.models import Test,Contact,Tag
 
# Register your models here.
class TagInline(admin.TabularInline):
    model = Tag
 
class ContactAdmin(admin.ModelAdmin):
    list_display = ('name','age', 'email') # list
    inlines = [TagInline]  # Inline
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]
 
    )
 
admin.site.register(Contact, ContactAdmin)
admin.site.register([Test])
```

搜索功能在管理大量记录时非常有，我们可以使用 search_fields 为该列表页增加搜索栏：

```python
# HelloWorld/TestModel/admin.py: 文件代码：
from django.contrib import admin
from TestModel.models import Test,Contact,Tag
 
# Register your models here.
class TagInline(admin.TabularInline):
    model = Tag
 
class ContactAdmin(admin.ModelAdmin):
    list_display = ('name','age', 'email') # list
    search_fields = ('name',)
    inlines = [TagInline]  # Inline
    fieldsets = (
        ['Main',{
            'fields':('name','email'),
        }],
        ['Advance',{
            'classes': ('collapse',),
            'fields': ('age',),
        }]
 
    )
 
admin.site.register(Contact, ContactAdmin)
admin.site.register([Test])
```

#### 问题解决

```python
Django 在根据 models 生成数据库表时报 __init__() missing 1 required positional argument: 'on_delete' 错误

原因：

在 django2.0 后，定义外键和一对一关系的时候需要加 on_delete 选项，此参数为了避免两个表里的数据不一致问题，不然会报错：TypeError: __init__() missing 1 required positional argument: 'on_delete'。

举例说明：

user=models.OneToOneField(User)owner=models.ForeignKey(UserProfile)
需要改成：

user=models.OneToOneField(User,on_delete=models.CASCADE) --在老版本这个参数（models.CASCADE）是默认值
owner=models.ForeignKey(UserProfile,on_delete=models.CASCADE) --在老版本这个参数（models.CASCADE）是默认值参数
说明：on_delete 有 CASCADE、PROTECT、SET_NULL、SET_DEFAULT、SET() 五个可选择的值。

 CASCADE：此值设置，是级联删除。
 PROTECT：此值设置，是会报完整性错误。
 SET_NULL：此值设置，会把外键设置为 null，前提是允许为 null。
 SET_DEFAULT：此值设置，会把设置为外键的默认值。
 SET()：此值设置，会调用外面的值，可以是一个函数。一般情况下使用 CASCADE 就可以了。
```



## Django 中 Cookie 的语法

设置 cookie:

```django
rep.set_cookie(key,value,...) 
rep.set_signed_cookie(key,value,salt='加密盐',...)
```

获取 cookie:

```
request.COOKIES.get(key)
```

删除 cookie:

```
rep = HttpResponse || render || redirect 
rep.delete_cookie(key)
```

## Django 中使用Redis

### 1.安装django-redis库

### 2.配置

打开Django的配置文件，比如说setting.py，里面设置CACHES项

```django
CACHES = {
	"default" : {
		"BACKEND" : "django_redis.cache.RedisCache",
		"LOCATION" : "redis://127.0.0.1:6379/1",
		"OPTIONS" : {
			"CLIENT_CLASS" : "django_redis.client.Defaultclient",
		}
	}
}
```

一个CACHES里可以配置多个redis连接信息，每一个都有自己的别名（alias），上面的“default”就是别名，到时候可以通过不同别名连接不同redis数据库

LOCATION是连接的信息，包括ip端口用户密码等，如果不需要用户密码则可以省略不写，django-redis支持三种连接协议，如下

| 协议      | 说明                   | 举例                                                     |
| --------- | ---------------------- | -------------------------------------------------------- |
| redis://  | 普通的TCP套接字连接    | redis://[[username]:[password]]@localhost:6379/0         |
| rediss    | SSL方式的TCP套接字连接 | rediss://[[username]:[password]]@localhost:6379/0        |
| rediss:// | Unix域套接字连接       | unix://[[username]:[password]]@/path/to/socket.sock?db=0 |

### 3.使用redis存储session

Django默认的Session是存储在sql数据库中，但我们都知道普通的数据会被数据存储在硬盘上，速度没有那么快，如果想改成存储在redis中，只需要在配置文件中修改

```django
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
```

### 4.redis连接超时时间设置

连接超时的秒数可以在配置项里设置，SOCKET_CONNECT_TIMEOUT表示连接redis的超时时间，SOCKET_TIMEOUT表示使用redis进行读写操作超时时间

```django
CACHES = {
	"default" ： {
		"OPTIONS" : {
			"SOCKET_CONNECT_TIMEOUT" : 5,  # 连接redis超时时间
			"SOCKET_TIMEOUT" : 5,  # redis读写操作超时时间
		}
	}
}
```

4.使用redis
4.1 使用默认redis
如果你想使用默认的redis，也就是在配置文件里设置的别名为“default”的redis，可以引用django.core.cache里的cache

```python
from django.core.cache import cache

cache.set("name", "冰冷的希望", timeout=None)
print(cache.get("name"))
```

4.2 使用指定redis（原生redis）
当你在配置文件里写了多个redis连接，可以通过别名指定要使用哪个redis

```python
from django_redis import get_redis_connection

redis_conn = get_redis_connection("chain_info")
redis_conn.set("name", "icy_hope")
print(redis_conn.get("name"))
```

要注意，通过get_redis_connection()获取得到的客户端是原生Redis客户端，虽然基本上支持所有的原生redis命令，但它返回的数据是byte类型，你需要自己decode

5.连接池
使用连接池的好处是不用管理连接对象，它会自动创建一些连接对象并且尽可能重复使用，所以相当来说性能会好一点

5.1 配置连接池
要使用连接池，首先要在Django的配置文件里写上连接池的最大连接数

```django
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        ...
        "OPTIONS": {
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}
        }
    }
}
```

5.2 使用连接池
我们可以通过连接别名确定要使用哪个redis，然后正常执行命令就行，我们不用在乎它创建了哪些连接实例，但你可以通过connection_pool的_created_connections属性查看当前创建了多少个连接实例

```python
from django_redis import get_redis_connection

redis_conn = get_redis_connection("default")
redis_conn.set("name", "冰冷的希望")
print(redis_conn.get("name"))

# 查看目前已创建的连接数量
connection_pool = redis_conn.connection_pool
print(connection_pool._created_connections)
```

5.3 自定义连接池
Django-redis默认的连接的类是DefaultClient，如果你有更高的定制需求，可以新建一个自己的类，继承ConnectionPool

```python
from redis.connection import ConnectionPool

class MyPool(ConnectionPool):
    pass
```

有了这个类之后还需要在Django的配置文件里指定它

```django
"OPTIONS": {
    "CONNECTION_POOL_CLASS": "XXX.XXX.MyPool",
}
```





## Django中的Middleware

https://m.runoob.com/django/django-middleware.html