# Python高级

python 可以在运行（不是程序动起来，而是可以在后面的代码中进行修改或追加或删除）时，对类添加属性或方法。

在没有将pip配置为环境变量时使用方法为：python -m pip install [packagename]

input函数的格式为统一字符串。

zfill函数可以在输出字符前添加0，达到格式标准。

添加属性直接类名.属性初始化值，这种是永久生成这个属性；

​			对象.属性初始化值，这种是只属于这个对象的属性。

添加方法，需要用到import types， types.MethodType(方法名，对象名/类名)

​					这里的对象名或类名的应用范围同上。

在调用实例方法时，默认会传输self对象参数，而调用类方法时（@classmethod装饰器），传输的时cls参数。

## 时间日期包使用

获取当前时间 datetime.datetime.now() 注意这个输出的是个datetime.datetime类型数据

时间数据的模板为：%Y-%m-%d %H:%M:%S

datetime类型转换为字符串类型使用：strftime，反向转换使用：strptime



## 高阶函数

map

- 把集合或者列表里的每一个元素都按照一定规则操作，生产新的列表
  或者集合

- 返回一个迭代对象（什么是可迭代？）
- map类型是一个可迭代的结构，可以使用for循环遍历
  map(func_name， n)

reduce

- reduce(function, sequence[, initial])
  function：需要传入一个有两个参数的函数，可以直接使用lambda匿名函数。
  sequence：将会从左往右遍历这个序列。
- 例子：reduce(lambda x,y: x+y, [1,2,3])，reduce(lambda x,y: x+y, [1,2,3], 100)注意多个参数时，尽量结果的类型和每个参数的类型一致。多个参数时，拼接顺序从后往前

filter

- 过滤函数：filter()函数用于过滤序列，过滤掉不符合条件的元素，返回符合条件的元素组成新列表。
- filter(function, iterable) function为函数，iterable为序列
- 例子: filter(lambda x: x%2==1,[1,2,3,4,5,6,7,8,9,10]) 结果为[1,3,5,7,9]

isinstance

- isinstance(iterable, (type1,type2,···))
- iterable: 需要判断的变量
- (type···): 类型名称，当判断多个类型，用元组的形式写，而不是列表

sort

- key/cmp=: 可自行定义排序规则，cmp只在python2版本有
- sort: 只能用于list类型，不会产生新的list，在原有的list对象进行排序


sorted

- key/cmp=: 同sort
- 会产生新的list类型对象

**for i in range(len())与len()后在丢入循环range中的效率都是一致的。**

逻辑判断的顺序为NOT-->AND-->OR

## Numpy库使用

规定小数的位数：round函数只能规定一个数，numpy.round函数可以规定一个list的所有元素。

Numpy.char.add([],[])将两个存字符串元素的list进行对应位置拼接，['a','c'] ['b','d'] -->['a','b'] ['c','d']

Numpy.char.multiply('Runoob ',3)将字符串进行多次重复拼接。

Numpy.char.center('str', 长度,fillchar = '*') 函数用于将字符串居中，并使用指定字符在左侧和右侧进行填充。

Numpy.char.capitalize('str') 函数将字符串的第一个字母转换为大写。

Numpy.char.title('str') 函数将字符串的每个单词的第一个字母转换为大写

Numpy.char.lower('str'or'list') 函数对数组的每个元素转换为小写。它对每个元素调用 str.lower。

Numpy.char.upper('str'or'list') 函数对数组的每个元素转换为大写。它对每个元素调用 str.upper

Numpy.char.split('str',sep='.') 通过指定分隔符对字符串进行分割，并返回数组。默认情况下，分隔符为空格。

Numpy.char.splitlines() 函数以换行符作为分隔符来分割字符串，并返回数组。**\n**，**\r**，**\r\n** 都可用作换行符。

Numpy.char.strip('str'or'list','char') 函数用于移除开头或结尾处的特定字符。list的每个元素都会进行一遍删除。

Numpy.char.join('char'or'char list','str'or'list') 函数通过指定分隔符来连接数组中的元素或字符串。当为list时，如果需要分别对每个元素进行分隔，则需要数量一致。

Numpy.array(需要指定参数)函数来生成多维矩阵数组。Numpy.ndim属性查看数组的维数。Numpy.array(数组，ndmin=数)，ndmin参数用来定义数组的维数。多维数组在取索引时，下标索引参数从左往右 ==》多维数组从外往里。

Numpy.array.dtype属性可以查看数组元素的数据类型，也可以通过dtype=‘数据类型’来更改元素的数据类型。当遇到无法强制转换的数据类型，会直接报错ValueError。

Numpy.array().astype()函数创建数组的副本，并允许您将数据类型指定为参数。

副本与视图的区别：副本用.copy()函数创建，视图用.view()函数创建。
副本和数组视图之间的主要区别在于副本是一个新数组，而这个视图只是原始数组的视图。
可以用base属性来进行区分副本和视图。

Numpy.array.shape属性可以获取数组的组成形状（各维度里拥有的元素个数）

Numpy.array.reshape(参数可以为多个)函数可以重塑数组的形状，参数的个数对应新数组的维度，参数从左往右 ==》对应新数组从外往里元素的个数。
注意：新数组的元素个数应与原数组个数一致，否则报错。但是可以使用 -1 来做参数，这样就会自动计算元素的摆放。



## 面向对象

类在初始化时，若有构造方法则会构造方法，并且这个方法是可以传参数的，若没有，则会直接生成实例。

创建对象属性时，可以在构造方法里用，self.属性字段 格式来初始化， 也可以用在构造方法同级，属性字段 格式来初始化。

## pyautogui库

可以实现控制电脑的鼠标和键盘

## pyWin32库

实现windows下软件的自动脚本

# python操作数据库

下面以sqlite3为例：

sqlite3.connect()打开一个到SQLite数据库文件database的连接。

connection.cursor()创建一个cursor对象，将在Python数据库编程中用到。

cursor.execute(sql)执行一个sql语句，注意sql 语句可以被参数化使用。即sql="""insert into *** values(%s,%s,%s)""" cursor.execute(sql, ('3','ajshf','男'))

cursor.executescript(sql)该例程一旦接收到脚本，会执行多个sql语句。sql语句应该用分号分隔。connection.commit()提交当前的事务。

connection.rollback()回滚至上一次调用commit()对数据库所做的更改。

connection.close()关闭数据库连接。

cursor.fetchone()获取查询结果集中的下一行，返回一个单一的序列，当没有更多可用的数据时，则返回 None。

cursor.fetchmany()获取查询结果集中的下一行组数据，返回一个列表。

cursor.fetchall()获取查询结果集中所有的数据行，返回一个列表。

# Django框架学习

项目框架的文件管理关系为：

项目名称
	默认项目文件夹[配置,路由等文件]
	功能文件夹[可为多个]
	static文件夹[存放前端js,css]
	templates[存放html]
	db文件
	manage.py管理文件
	
功能文件夹中用于填写model模型和views功能。

## url配置问题根据django的版本来：

Django1.1.x 版本：使用url()：普通路径和正则路径均可使用，需要自己手动添加正则首位限制符号。

2.2.x之后的版本：path：用于普通路径，不需要自己手动添加正则首位限制符号，底层已经添加。re_path：用于正则路径，需要自己手动添加正则首位限制符号。

总结：Django1.1.x 版本中的 url 和 Django 2.2.x 版本中的 re_path 用法相同。

## Django的模板语法：

view：｛"HTML变量名" : "views变量名"｝

HTML：｛｛变量名｝｝

Django的模型：在app文件夹中创建models脚本，在脚本中引用django.db库中的models类，用ORM语法创建数据库表字段，然后用命令执行

## 命令如下：

python3 manage.py migrate   # 创建表结构

python3 manage.py makemigrations TestModel  # 让 Django 知道我们在我们的模型有一些变更

python3 manage.py migrate TestModel   # 创建表结构

会自动添加id字段作为主键

需要对应的urls文件中进行对该models脚本文件进行配置

### Django 中 Cookie 的语法

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
rep =HttpResponse || render || redirect 
rep.delete_cookie(key)
```

## Django中的Middleware

https://m.runoob.com/django/django-middleware.html

