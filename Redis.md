# Redis简介：

## windows版本

打开cmd，进入到解压后的路径，输入redis-server redis.windows.conf，服务启动

但关闭cmd后，服务就会停止

打开cmd，进入redis文件夹下，输入redis-server.exe --service-install redis.windows.conf --loglevel verbose回车，设置服务命令（需要管理员权限）

服务启动后可以运行redis客户端，redis-cli.exe

客户端内的命令有：set, get......

事务标记开始：MULTI 

命令入队：INCR

执行：EXEC  注意当EXEC前出现命令(语法？)错误，redis会终止事务进行报错，

   而出现命令(语法没错，数据key，value？)错误，redis会忽略这条指令继续执行事务。

监视key是否被改动过：watch 在发现key被更改后，在执行EXEC是就会返回nil，表示事务无法被触发。

## Linux版本

```
 查看redis是否正常打开指令
 ps aux | grep redis-server
 
 连接redis指令
 redis-cli
 
 如果出现NOAUTH Authentication required
 使用： auth 密码 ，就可以解决
 
 
 
```



### string类型

```redis
设置键值
set key value
设置过期时间
setex key second value
设置已有数据过期时间
expire key time
设置多个键值
mset key1 value1 key2 value2 ...
追加值
append key value
获取值，不存在则为nil
get key
获取多个键的值
mget key1 key2 ...
```

### 键操作

```redis
查找键，参数支持正则表达式
keys pattern
查看所有键
keys *
判断键是否存在，如果存在返回1，不存在返回0
exists key1
查看键对应的value的类型
type key
删除键及对应的值
del key1 key2 ...
查看有效时间，以秒为单位
ttl key -1位永久有效
```

### hash

```redis
结构：
键key:{
	域field ： 值value
}
设置单个属性
hset key field value
设置多个属性
hmset key field1 value1 field2 value2 ...
获取指定键所有的属性
hkeys key
获取一个属性的值
hget key field
获取多个属性的值
hmget key field1 field2 ...
获取所有属性的值
hvals key
删除属性，属性对应的值会一起被删除
hdel key field1 field2 ...
```

### list列表的元素类型为string  按照插入顺序排序

```redis
在左侧插入数据
lpush key value1 value2 …
在右侧插⼊数据
rpush key value1 value2 …
在指定元素的前或后插⼊新元素
linsert key before或after 现有元素 新元素
例3：在键为a1的列表中元素b前加⼊3
linsert a1 before b 3

设置指定索引位置的元素值
-索引从左侧开始，第⼀个元素为0
-索引可以是负数，表示尾部开始计数，如-1表示最后⼀个元素
lset key index value

删除指定元素
-将列表中前count次出现的值为value的元素移除
-count > 0: 从头往尾移除
-count < 0: 从尾往头移除
-count = 0: 移除所有
lrem key count value
```

### set

```redis
添加元素
sadd key member1 member2 …
返回所有的元素
smembers key
删除指定元素
srem key value
```



## 针对redis中的内容扩展

### flushall 清空数据库中的所有数据

针对各种数据类型它们的特性，使用场景如下:

| 类型         | 特性                                                         |
| ------------ | ------------------------------------------------------------ |
| 字符串string | 用于保存一些项目中的普通数据，只要键值对的都可以保存，例如，保存 session,定时记录状态 |
| 哈希hash     | 用于保存项目中的一些字典数据，但是不能保存多维的字典，例如，商城的购物车 |
| 列表list     | 用于保存项目中的列表数据，但是也不能保存多维的列表，例如，队列，秒杀，医院的挂号 |
| 无序集合set  | 用于保存项目中的一些不能重复的数据，可以用于过滤，例如，投票海选的时候，过滤候选人 |
| 无序集合set  | 用于保存项目中一些不能重复，但是需要进行排序的数据，分数排行榜. |

# django 中配置使用 redis

```python
# 安装
pip install redis
# 设置redis缓存
CACHES = {
    # 默认缓存
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # 项目上线时,需要调整这里的路径
        "LOCATION": "redis://127.0.0.1:6379/0",
    	"OPTIONS": {
        	"CLIENT_CLASS": "django_redis.client.DefaultClient",
        	# "CONNECTION_POOL_KWARGS": {
        	#    "max_connections": 1000,
        	#    "encoding": 'utf-8'
        	# },
        	# "PASSWORD": "foobared" # 如果设置了登录密码，那么这里写密码
    	}
	},

	# 提供存储短信验证码
	"sms_code": {
    	"BACKEND": "django_redis.cache.RedisCache",
    	"LOCATION": "redis://127.0.0.1:6379/1",
    	"OPTIONS": {
        	"CLIENT_CLASS": "django_redis.client.DefaultClient",
    	}
	},
}
```

```python
# 使用：
import redis

conn = redis.Redis(
    host='127.0.0.1',
    port=6379,
    encoding='utf-8',
    db=0,
    decode_responses=True  # 自动了解码返回数据
)

conn.set('name2', 'chao')
name = conn.get('name2')
print(str(name, 'utf-8'))
```

上述官方不推荐，因为每次操作redis会建立一次连接，效率会比较低，可以使用redis连接池来替换。

## 使用 redis 链接池：

```python
pool = redis.ConnectionPool(
    host='127.0.0.1',
    port=6379,
    encoding='utf-8',
    db=0,
    max_connections=100,

)

conn = redis.Redis(connection_pool=pool)
conn.set('name3','yu')
value = conn.get('name3')
print(value)

# conn.set('name', "吴超", ex=10)
# # 根据键获取值：如果存在获取值（获取到的是字节类型）；不存在则返回None
# value = conn.get('name')
# print(value)
# bytes 转 字符串：str(object,'utf-8')
```

## django-redis

```python
安装: pip install django-redis

在views里使用：

from django_redis import get_redis_connection
def login(request):
    conn = get_redis_connection('sms_code')  # 默认key是:default(settings中配置的)
    conn.set('xx3', 'oo3')
    
目前没有使用这个方式。
from django.core.cache import cache
这个方式似乎不需要再次进行连接，而是在django在加载配置文件时，就预先连接好了。
cache.get(key)； cache.set(key, value)
```

## 使用redis时出现过的问题：

```
1.在代码中设置了当天日期（字符串类型）作为key，可是在redis库中查看key为前缀添加了':1:'字符，导致直接查看库内容，查找不到，但是不影响代码执行。
	使用的指令：keys *用于查看redis中的键值。
可是这个 ':1:'字符 不知道从哪来的？？？
这个是django自带的缓存键版本，所以这个在redis-cli中查看这个键值时，查找不到，但是这个并不会影响到cache.get，因为这个会自动添加上版本号，所以影响不大。
2.


```

