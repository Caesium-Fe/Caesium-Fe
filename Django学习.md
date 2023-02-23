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

### 请求对象: HttpRequest 对象（简称 request 对象）

以下介绍几个常用的 request 属性。

#### GET

数据类型是 QueryDict，一个类似于字典的对象，包含 HTTP GET 的所有参数。

有相同的键，就把所有的值放到对应的列表里。

取值格式：**对象.方法**。

##### get()

返回字符串，如果该键对应有多个值，取出该键的最后一个值。

#### POST

数据类型是 QueryDict，一个类似于字典的对象，包含 HTTP POST 的所有参数。

常用于 form 表单，form 表单里的标签 name 属性对应参数的键，value 属性对应参数的值。

取值格式： **对象.方法**。

##### get()

返回字符串，如果该键对应有多个值，取出该键的最后一个值。

#### body

数据类型是二进制字节流，是原生请求体里的参数内容，在 HTTP 中用于 POST，因为 GET 没有请求体。

在 HTTP 中不常用，而在处理非 HTTP 形式的报文时非常有用，例如：二进制图片、XML、Json 等。

#### path

获取 URL 中的路径部分，数据类型是字符串。

#### method

获取当前请求的方式，数据类型是字符串，且结果为大写。

### 响应对象：HttpResponse 对象

响应对象主要有三种形式：HttpResponse()、render()、redirect()。

#### HttpResponse()

返回文本，参数为字符串，字符串中写文本内容。如果参数为字符串里含有 html 标签，也可以渲染。

#### render()

返回文本，第一个参数为 request，第二个参数为字符串（页面名称），第三个参数为字典（可选参数，向页面传递的参数：键为页面参数名，值为views参数名）。

#### redirect()

重定向，跳转新页面。参数为字符串，字符串中填写页面路径。一般用于 form 表单提交后，跳转到新页面。

render 和 redirect 是在 HttpResponse 的基础上进行了封装：

- render：底层返回的也是 HttpResponse 对象
- redirect：底层继承的是 HttpResponse 对象

## Django 路由

路由简单的来说就是根据用户请求的 URL 链接来判断对应的处理程序，并返回处理结果，也就是 URL 与 Django 的视图建立映射关系。

Django 路由在 urls.py 配置，urls.py 中的每一条配置对应相应的处理方法。

**url() 方法**：普通路径和正则路径均可使用，需要自己手动添加正则首位限制符号。

**path**：用于普通路径，不需要自己手动添加正则首位限制符号，底层已经添加。

**re_path**：用于正则路径，需要自己手动添加正则首位限制符号。

### 正则路径中的分组

#### 正则路径中的无名分组

无名分组按位置传参，一一对应。

views 中除了 request，其他形参的数量要与 urls 中的分组数量一致。

#### 正则路径中的有名分组

语法： (?P<组名>正则表达式)

有名分组按关键字传参，与位置顺序无关。

views 中除了 request，其他形参的数量要与 urls 中的分组数量一致， 并且 views 中的形参名称要与 urls 中的组名对应。

##### 路由分发(include)

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

### 反向解析

随着功能的增加，路由层的 url 发生变化，就需要去更改对应的视图层和模板层的 url，非常麻烦，不便维护。

这时我们可以利用反向解析，当路由层 url 发生改变，在视图层和模板层动态反向解析出更改后的 url，免去修改的操作。

反向解析一般用在模板中的超链接及视图中的重定向。

#### 普通路径

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

#### 正则路径（无名分组）

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

#### 正则路径（有名分组）

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

### 命名空间

命名空间（英语：Namespace）是表示标识符的可见范围。

一个标识符可在多个命名空间中定义，它在不同命名空间中的含义是互不相干的。

一个新的命名空间中可定义任何标识符，它们不会与任何重复的标识符发生冲突，因为重复的定义都处于其它命名空间中。

**存在问题：**路由别名 name 没有作用域，Django 在反向解析 URL 时，会在项目全局顺序搜索，当查找到第一个路由别名 name 指定 URL 时，立即返回。当在不同的 app 目录下的urls 中定义相同的路由别名 name 时，可能会导致 URL 反向解析错误。

**解决：**使用命名空间。

##### 普通路径

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

## Django Admin 管理工具

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

### 激活管理工具

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

### 使用管理工具

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

### 复杂模型

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

### 自定义表单

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

### 内联(Inline)显示

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

### 列表页的显示

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

## Django ORM - 单表实例

### 数据库添加

**方式一：**模型类实例化对象

需从 app 目录引入 models.py 文件：

```
from app 目录 import models
```

并且实例化对象后要执行 **对象.save()** 才能在数据库中新增成功。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    book = models.Book(title="菜鸟教程",price=300,publish="菜鸟出版社",pub_date="2008-8-8") 
    book.save()
    return HttpResponse("<p>数据添加成功！</p>")
```

**方式二：**通过 ORM 提供的 objects 提供的方法 create 来实现（推荐）

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.create(title="如来神掌",price=200,publish="功夫出版社",pub_date="2010-10-10") 
    print(books, type(books)) # Book object (18) 
    return HttpResponse("<p>数据添加成功！</p>")
```

### 数据库查找

使用 **all()** 方法来查询所有内容。

返回的是 QuerySet 类型数据，类似于 list，里面放的是一个个模型类的对象，可用索引下标取出模型类的对象。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.all() 
    print(books,type(books)) # QuerySet类型，类似于list，访问 url 时数据显示在命令行窗口中。
    return HttpResponse("<p>查找成功！</p>")
```

**filter()** 方法用于查询符合条件的数据。

返回的是 QuerySet 类型数据，类似于 list，里面放的是满足条件的模型类的对象，可用索引下标取出模型类的对象。

pk=3 的意思是主键 primary key=3，相当于 id=3。

因为 id 在 pycharm 里有特殊含义，是看内存地址的内置函数 id()，因此用 pk。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.filter(pk=5)
    print(books)
    print("//////////////////////////////////////")
    books = models.Book.objects.filter(publish='菜鸟出版社', price=300)
    print(books, type(books))  # QuerySet类型，类似于list。
    return HttpResponse("<p>查找成功！</p>")
```

**exclude()** 方法用于查询不符合条件的数据。

返回的是 QuerySet 类型数据，类似于 list，里面放的是不满足条件的模型类的对象，可用索引下标取出模型类的对象。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.exclude(pk=5)
    print(books)
    print("//////////////////////////////////////")
    books = models.Book.objects.exclude(publish='菜鸟出版社', price=300)
    print(books, type(books))  # QuerySet类型，类似于list。
    return HttpResponse("<p>查找成功！</p>")
```

**get()** 方法用于查询符合条件的返回模型类的对象符合条件的对象只能为一个，如果符合筛选条件的对象超过了一个或者没有一个都会抛出错误。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.get(pk=5)
    books = models.Book.objects.get(pk=18)  # 报错，没有符合条件的对象
    books = models.Book.objects.get(price=200)  # 报错，符合条件的对象超过一个
    print(books, type(books))  # 模型类的对象
    return HttpResponse("<p>查找成功！</p>")
```

**order_by()** 方法用于对查询结果进行排序。

返回的是 QuerySet类型数据，类似于list，里面放的是排序后的模型类的对象，可用索引下标取出模型类的对象。

**注意：**

- a、参数的字段名要加引号。
- b、降序为在字段前面加个负号 **-**。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.order_by("price") # 查询所有，按照价格升序排列 
    books = models.Book.objects.order_by("-price") # 查询所有，按照价格降序排列
    return HttpResponse("<p>查找成功！</p>")
```

**reverse()** 方法用于对查询结果进行反转。

返回的是 QuerySe t类型数据，类似于 list，里面放的是反转后的模型类的对象，可用索引下标取出模型类的对象。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    # 按照价格升序排列：降序再反转
    books = models.Book.objects.order_by("-price").reverse()
    return HttpResponse("<p>查找成功！</p>")
```

**count()** 方法用于查询数据的数量返回的数据是整数。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.count() # 查询所有数据的数量 
    books = models.Book.objects.filter(price=200).count() # 查询符合条件数据的数量
    return HttpResponse("<p>查找成功！</p>")
```

**first()** 方法返回第一条数据返回的数据是模型类的对象也可以用索引下标 **[0]**。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.first() # 返回所有数据的第一条数据
    return HttpResponse("<p>查找成功！</p>")
```

**last()** 方法返回最后一条数据返回的数据是模型类的对象不能用索引下标 **[-1]**，ORM 没有逆序索引。

```python
# app01/views.py: 文件代码：
from django.shortcuts import render,HttpResponse
from app01 import models 
def add_book(request):
    books = models.Book.objects.last() # 返回所有数据的最后一条数据
    return HttpResponse("<p>查找成功！</p>")
```

**exists()** 方法用于判断查询的结果 QuerySet 列表里是否有数据。

返回的数据类型是布尔，有为 true，没有为 false。

**注意：**判断的数据类型只能为 QuerySet 类型数据，不能为整型和模型类的对象。

```python
# 实例
from django.shortcuts import render,HttpResponse
from app01 import models
def add_book(request):
    books = models.Book.objects.exists()
    # 报错，判断的数据类型只能为QuerySet类型数据，不能为整型
    books = models.Book.objects.count().exists()
    # 报错，判断的数据类型只能为QuerySet类型数据，不能为模型类对象
    books = models.Book.objects.first().exists()  
    return HttpResponse("<p>查找成功！</p>")
```

**values()** 方法用于查询部分字段的数据。

返回的是 QuerySet 类型数据，类似于 list，里面不是模型类的对象，而是一个可迭代的字典序列，字典里的键是字段，值是数据。

**注意：**

- 参数的字段名要加引号

- 想要字段名和数据用 **values**

- ```python
  # 实例
  from django.shortcuts import render,HttpResponse
  from app01 import models
  def add_book(request):
      # 查询所有的id字段和price字段的数据
      books = models.Book.objects.values("pk","price")
      print(books[0]["price"],type(books)) # 得到的是第一条记录的price字段的数据
      return HttpResponse("<p>查找成功！</p>")
  ```

**values_list()** 方法用于查询部分字段的数据。

返回的是 QuerySet 类型数据，类似于 list，里面不是模型类的对象，而是一个个元组，元组里放的是查询字段对应的数据。

**注意：**

- 参数的字段名要加引号
- 只想要数据用 values_list

```python
# 实例

from django.shortcuts import render,HttpResponse
from app01 import models
def add_book(request):
    # 查询所有的price字段和publish字段的数据
    books = models.Book.objects.values_list("price","publish")
    print(books)
    print(books[0][0],type(books)) # 得到的是第一条记录的price字段的数据
    return HttpResponse("<p>查找成功！</p>")
```

**distinct()** 方法用于对数据进行去重。

返回的是 QuerySet 类型数据。

**注意：**

- 对模型类的对象去重没有意义，因为每个对象都是一个不一样的存在。
- distinct() 一般是联合 values 或者 values_list 使用。

```python
#实例
from django.shortcuts import render,HttpResponse
from app01 import models
def add_book(request):
    # 查询一共有多少个出版社
    books = models.Book.objects.values_list("publish").distinct() # 对模型类的对象去重没有意义，因为每个对象都是一个不一样的存在。
    books = models.Book.objects.distinct()
    return HttpResponse("<p>查找成功！</p>")
```

**filter()** 方法基于双下划线的模糊查询（exclude 同理）。

**注意：**filter 中运算符号只能使用等于号 = ，不能使用大于号 > ，小于号 < ，等等其他符号。

__in 用于读取区间，= 号后面为列表 。

```python
# 实例
from django.shortcuts import render,HttpResponse
from app01 import models
def add_book(request):
    # 查询价格为200或者300的数据
    books = models.Book.objects.filter(price__in=[200,300])
    return HttpResponse("<p>查找成功！</p>")
```

**__gt** 大于号 ，= 号后面为数字。

```python
# 查询价格大于200的数据 
books = models.Book.objects.filter(price__gt=200)
```

**__gte** 大于等于，= 号后面为数字。

```
# 查询价格大于等于200的数据 
books = models.Book.objects.filter(price__gte=200)
```

**__lt** 小于，=号后面为数字。

```
# 查询价格小于300的数据 
books=models.Book.objects.filter(price__lt=300)
```

**__lte** 小于等于，= 号后面为数字。

```
# 查询价格小于等于300的数据 
books=models.Book.objects.filter(price__lte=300)
```

**__range** 在 ... 之间，左闭右闭区间，= 号后面为两个元素的列表。

```
books=models.Book.objects.filter(price__range=[200,300])
```

**__contains** 包含，= 号后面为字符串。

```
books=models.Book.objects.filter(title__contains="菜")
```

**__icontains** 不区分大小写的包含，= 号后面为字符串。

```
books=models.Book.objects.filter(title__icontains="python") # 不区分大小写
```

**__startswith** 以指定字符开头，= 号后面为字符串。

```
books=models.Book.objects.filter(title__startswith="菜")
```

**__endswith** 以指定字符结尾，= 号后面为字符串。

```python
books=models.Book.objects.filter(title__endswith="教程")
```

**__year** 是 DateField 数据类型的年份，= 号后面为数字。

```python
books=models.Book.objects.filter(pub_date__year=2008) 
```

**__month** 是DateField 数据类型的月份，= 号后面为数字。

```python
books=models.Book.objects.filter(pub_date__month=10) 
```

**__day** 是DateField 数据类型的天数，= 号后面为数字。

```python
books=models.Book.objects.filter(pub_date__day=01)
```

### 数据库删除

**方式一：**使用模型类的 **对象.delete()**。

**返回值：**元组，第一个元素为受影响的行数。

```python
books=models.Book.objects.filter(pk=8).first().delete()
```

**方式二**：使用 QuerySet **类型数据.delete()**(推荐)

**返回值：**元组，第一个元素为受影响的行数。

```python
books=models.Book.objects.filter(pk__in=[1,2]).delete()
```

**注意：**

- a. Django 删除数据时，会模仿 SQL约束 ON DELETE CASCADE 的行为，也就是删除一个对象时也会删除与它相关联的外键对象。
- b. delete() 方法是 QuerySet 数据类型的方法，但并不适用于 Manager 本身。也就是想要删除所有数据，不能不写 all。

```python
books=models.Book.objects.delete()　 # 报错
books=models.Book.objects.all().delete()　　 # 删除成功
```

### 数据库修改

**方式一：**

```shell
模型类的对象.属性 = 更改的属性值
模型类的对象.save()
```

**返回值：**编辑的模型类的对象。

```python
books = models.Book.objects.filter(pk=7).first() 
books.price = 400 
books.save()
```

**方式二：**QuerySet 类型数据.update(字段名=更改的数据)（推荐）

**返回值：**整数，受影响的行数

```python
# 实例
from django.shortcuts import render,HttpResponse
from app01 import models
def add_book(request):
    books = models.Book.objects.filter(pk__in=[7,8]).update(price=888)
    return HttpResponse(books)
```

## Django ORM – 多表实例

表与表之间的关系可分为以下三种：

- **一对一**: 一个人对应一个身份证号码，数据字段设置 unique。
- **一对多**: 一个家庭有多个人，一般通过外键来实现。
- **多对多**: 一个学生有多门课程，一个课程有很多学生，一般通过第三个表来实现关联。

### 创建模型

接下来我们来看下多表多实例。

```python
# 实例
class Book(models.Model):
    title = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    pub_date = models.DateField()
    publish = models.ForeignKey("Publish", on_delete=models.CASCADE)
    authors = models.ManyToManyField("Author")


class Publish(models.Model):
    name = models.CharField(max_length=32)
    city = models.CharField(max_length=64)
    email = models.EmailField()


class Author(models.Model):
    name = models.CharField(max_length=32)
    age = models.SmallIntegerField()
    au_detail = models.OneToOneField("AuthorDetail", on_delete=models.CASCADE)


class AuthorDetail(models.Model):
    gender_choices = (
        (0, "女"),
        (1, "男"),
        (2, "保密"),
    )
    gender = models.SmallIntegerField(choices=gender_choices)
    tel = models.CharField(max_length=32)
    addr = models.CharField(max_length=64)
    birthday = models.DateField()
```

**说明：**

- 1、EmailField 数据类型是邮箱格式，底层继承 CharField，进行了封装，相当于 MySQL 中的 varchar。
- 2、Django1.1 版本不需要联级删除：on_delete=models.CASCADE，Django2.2 需要。
- 3、一般不需要设置联级更新.
- 4、外键在一对多的多中设置：**models.ForeignKey("关联类名", on_delete=models.CASCADE)**。
- 5、OneToOneField = ForeignKey(...，unique=True)设置一对一。
- 6、若有模型类存在外键，创建数据时，要先创建外键关联的模型类的数据，不然创建包含外键的模型类的数据时，外键的关联模型类的数据会找不到。

### 表结构

**书籍表 Book**：title 、 price 、 pub_date 、 publish（外键，多对一） 、 authors（多对多）

**出版社表 Publish**：name 、 city 、 email

**作者表 Author**：name 、 age 、 au_detail（一对一）

**作者详情表 AuthorDetail**：gender 、 tel 、 addr 、 birthday

以下是表格关联说明：

![Django-orm2_1](E:\markdownInJob\typora-user-images\Django-orm2_1.png)

### 插入数据

我们在 MySQL 中执行以下 SQL 插入操作：

```python
insert into app01_publish(name,city,email) values ("华山出版社", "华山", "hs@163.com"), ("明教出版社", "黑木崖", "mj@163.com")
 
# 先插入 authordetail 表中多数据
insert into app01_authordetail(gender,tel,addr,birthday) values (1,13432335433,"华山","1994-5-23"), (1,13943454554,"黑木崖","1961-8-13"), (0,13878934322,"黑木崖","1996-5-20") 

# 再将数据插入 author，这样 author 才能找到 authordetail 
insert into app01_author(name,age,au_detail_id) values ("令狐冲",25,1), ("任我行",58,2), ("任盈盈",23,3)
```

### ORM - 添加数据

#### 一对多(外键 ForeignKey)

**方式一:** 传对象的形式，返回值的数据类型是对象，书籍对象。

**步骤：**

- a. 获取出版社对象
- b. 给书籍的出版社属性 publish 传出版社对象

```python
# app01/views.py 文件代码：
def add_book(request):
    #  获取出版社对象
    pub_obj = models.Publish.objects.filter(pk=1).first()
    #  给书籍的出版社属性publish传出版社对象
    book = models.Book.objects.create(title="菜鸟教程", price=200, pub_date="2010-10-10", publish=pub_obj)
    print(book, type(book))
    return HttpResponse(book)
```

方式二: 传对象 id 的形式(由于传过来的数据一般是 id,所以传对象 id 是常用的)。

一对多中，设置外键属性的类(多的表)中，MySQL 中显示的字段名是:**外键属性名_id**。

返回值的数据类型是对象，书籍对象。

**步骤：**

- a. 获取出版社对象的 id
- b. 给书籍的关联出版社字段 publish_id 传出版社对象的 id

```python
# app01/views.py 文件代码：
def add_book(request):
    #  获取出版社对象
    pub_obj = models.Publish.objects.filter(pk=1).first()
    #  获取出版社对象的id
    pk = pub_obj.pk
    #  给书籍的关联出版社字段 publish_id 传出版社对象的id
    book = models.Book.objects.create(title="冲灵剑法", price=100, pub_date="2004-04-04", publish_id=pk)
    print(book, type(book))
    return HttpResponse(book)
```

### 多对多(ManyToManyField)：在第三张关系表中新增数据

**方式一:** 传对象形式，无返回值。

**步骤：**

- a. 获取作者对象
- b. 获取书籍对象
- c. 给书籍对象的 authors 属性用 add 方法传作者对象

```python
# app01/views.py 文件代码：
def add_book(request):
    #  获取作者对象
    chong = models.Author.objects.filter(name="令狐冲").first()
    ying = models.Author.objects.filter(name="任盈盈").first()
    #  获取书籍对象
    book = models.Book.objects.filter(title="菜鸟教程").first()
    #  给书籍对象的 authors 属性用 add 方法传作者对象
    book.authors.add(chong, ying)
    return HttpResponse(book)
```

**方式二:** 传对象id形式，无返回值。

**步骤：**

- a. 获取作者对象的 id
- b. 获取书籍对象
- c. 给书籍对象的 authors 属性用 add 方法传作者对象的 id

```python
# app01/views.py 文件代码：
def add_book(request):
    #  获取作者对象
    chong = models.Author.objects.filter(name="令狐冲").first()
    #  获取作者对象的id
    pk = chong.pk
    #  获取书籍对象
    book = models.Book.objects.filter(title="冲灵剑法").first()
    #  给书籍对象的 authors 属性用 add 方法传作者对象的id
    book.authors.add(pk)
```

### 关联管理器(对象调用)

**前提：**

- 多对多（双向均有关联管理器）
- 一对多（只有多的那个类的对象有关联管理器，即反向才有）

**语法格式：**

```
正向：属性名
反向：小写类名加 _set
```

**注意：**一对多只能反向

**常用方法：**

**add()**：用于多对多，把指定的模型对象添加到关联对象集（关系表）中。

**注意：**add() 在一对多(即外键)中，只能传对象（ *QuerySet数据类型），不能传 id（*[id表]）。

***[ ]** 的使用:

\# 方式一：传对象

```python
book_obj = models.Book.objects.get(id=10)
author_list = models.Author.objects.filter(id__gt=2)
book_obj.authors.add(*author_list)  # 将 id 大于2的作者对象添加到这本书的作者集合中
# 方式二：传对象 id
book_obj.authors.add(*[1,3]) # 将 id=1 和 id=3 的作者对象添加到这本书的作者集合中
return HttpResponse("ok")
```

反向：**小写表名_set**

```python
ying = models.Author.objects.filter(name="任盈盈").first()
book = models.Book.objects.filter(title="冲灵剑法").first()
ying.book_set.add(book)
return HttpResponse("ok")
```

**create()**：创建一个新的对象，并同时将它添加到关联对象集之中。

返回新创建的对象。

```python
pub = models.Publish.objects.filter(name="明教出版社").first()
wo = models.Author.objects.filter(name="任我行").first()
book = wo.book_set.create(title="吸星大法", price=300, pub_date="1999-9-19", publish=pub)
**print**(book, type(book))
**return** HttpResponse("ok")
```

**remove()**：从关联对象集中移除执行的模型对象。

对于 ForeignKey 对象，这个方法仅在 null=True（可以为空）时存在，无返回值。

```python
# 实例
author_obj =models.Author.objects.get(id=1)
book_obj = models.Book.objects.get(id=11)
author_obj.book_set.remove(book_obj)
return HttpResponse("ok")
```

**clear()**：从关联对象集中移除一切对象，删除关联，不会删除对象。

对于 ForeignKey 对象，这个方法仅在 null=True（可以为空）时存在。

无返回值。

```python
#  清空独孤九剑关联的所有作者
book = models.Book.objects.filter(title="菜鸟教程").first()
book.authors.clear()
```

### ORM 查询

基于对象的跨表查询。

```
正向：属性名称
反向：小写类名_set
```

#### 一对多

查询主键为 1 的书籍的出版社所在的城市（正向）。

```python
#  实例
book = models.Book.objects.filter(pk=10).first()
res = book.publish.city
print(res, type(res))
return HttpResponse("ok")
```

查询明教出版社出版的书籍名（反向）。

反向：**对象.小写类名_set(pub.book_set)** 可以跳转到关联的表(书籍表)。

**pub.book_set.all()**：取出书籍表的所有书籍对象，在一个 QuerySet 里，遍历取出一个个书籍对象。

```python
# 实例
pub = models.Publish.objects.filter(name="明教出版社").first()
res = pub.book_set.all()
for i in res:
    print(i.title)
return HttpResponse("ok")
```

### 一对一

查询令狐冲的电话（正向）

正向：对象.属性 (author.au_detail) 可以跳转到关联的表(作者详情表)

```python
# 实例
author = models.Author.objects.filter(name="令狐冲").first()
res = author.au_detail.tel
print(res, type(res))
return HttpResponse("ok")
```

查询所有住址在黑木崖的作者的姓名（反向）。

一对一的反向，用 **对象.小写类名** 即可，不用加 _set。

反向：对象.小写类名(addr.author)可以跳转到关联的表(作者表)。

```python
# 实例
addr = models.AuthorDetail.objects.filter(addr="黑木崖").first()
res = addr.author.name
print(res, type(res))
return HttpResponse("ok")
```

### 多对多

菜鸟教程所有作者的名字以及手机号（正向）。

正向：**对象.属性(book.authors)**可以跳转到关联的表(作者表)。

作者表里没有作者电话，因此再次通过**对象.属性(i.au_detail)**跳转到关联的表（作者详情表）。

```python
# 实例
book = models.Book.objects.filter(title="菜鸟教程").first()
res = book.authors.all()
for i in res:
    print(i.name, i.au_detail.tel)
return HttpResponse("ok")
```

查询任我行出过的所有书籍的名字（反向）。

```python
# 实例
author = models.Author.objects.filter(name="任我行").first()
res = author.book_set.all()
for i in res:
    print(i.title)
return HttpResponse("ok")
```

### 基于双下划线的跨表查询

#### 正向：属性名称__跨表的属性名称 反向：小写类名__跨表的属性名称

#### 一对多

查询菜鸟出版社出版过的所有书籍的名字与价格。

```python
# 实例
res = models.Book.objects.filter(publish__name="菜鸟出版社").values_list("title", "price")
```

反向：通过 小**写类名__跨表的属性名称（book__title，book__price）** 跨表获取数据。

```python
# 实例
res = models.Publish.objects.filter(name="菜鸟出版社").values_list("book__title","book__price")
return HttpResponse("ok")
```

#### 多对多

查询任我行出过的所有书籍的名字。

正向：通过 属性名称__跨表的属性名称(authors__name) 跨表获取数据：

```python
res = models.Book.objects.filter(authors__name="任我行").values_list("title")
```

反向：通过 小写类名__跨表的属性名称（book__title） 跨表获取数据：

```python
res = models.Author.objects.filter(name="任我行").values_list("book__title")
```

#### 一对一

查询任我行的手机号。

正向：通过 **属性名称__跨表的属性名称(au_detail__tel)** 跨表获取数据。

```python
res = models.Author.objects.filter(name="任我行").values_list("au_detail__tel")
```

反向：通过 **小写类名__跨表的属性名称（author__name）** 跨表获取数据。

```python
res = models.AuthorDetail.objects.filter(author__name="任我行").values_list("tel")
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