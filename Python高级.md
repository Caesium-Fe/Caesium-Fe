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

# 时间日期包使用

## datetime.datetime

获取当前时间 datetime.datetime.now() 注意这个输出的是个datetime.datetime类型数据

时间数据的模板为：%Y-%m-%d %H:%M:%S 

简写：%Y-%m-%d %X

datetime.datetime类型转换为字符串类型使用：strftime，反向转换使用：strptime，注意这里的参数必须是字符串，需要年月日或者时间的话，用模板进行样式修改。

时间日期计算使用 日期+datetime.timedelta(days=时间跨度(float)，seconds=时间跨度(float)) **正负参数代表时间的前进后退**

## datetime.date

isoweekday()可以查看日期为星期几(1-7)返回值为int型

isocalendar()可以查看日期为（多少年，多少周，星期几）返回值为元组

# 高阶函数

经典用法：

```python
# 转置数据矩阵---横纵转换，但必须没行长度一致
list(map(list, zip(*data)))

#找出两个列表的交集
x = [1, 2, 3, 5, 6]
y = [2, 3, 4, 6, 7]
list(filter(lambda a: a in y, x))
---->[2, 3, 6]
```

## map 每次返回的是一个迭代器

- 把集合或者列表里的每一个元素都按照一定规则操作，生产新的列表
  或者集合

- 返回一个迭代对象（什么是可迭代？）
- map类型是一个可迭代的结构，可以使用for循环遍历
  map(func_name， n)

## reduce 每次返回的是一个结果

- reduce(function, sequence[, initial])
  function：需要传入一个有两个参数的函数，可以直接使用lambda匿名函数。
  sequence：将会从左往右遍历这个序列。
- 例子：reduce(lambda x,y: x+y, [1,2,3])，reduce(lambda x,y: x+y, [1,2,3], 100)注意多个参数时，尽量结果的类型和每个参数的类型一致。多个参数时，拼接顺序从后往前

## filter 每次返回的是一个bool结果

-  package main​import (    "fmt"    "log"    "time"​    client "github.com/influxdata/influxdb1-client/v2")​// influxdb demo​func connInflux() client.Client {    cli, err := client.NewHTTPClient(client.HTTPConfig{        Addr:     "http://127.0.0.1:8086",        Username: "admin",        Password: "",    })    if err != nil {        log.Fatal(err)    }    return cli}​// queryfunc queryDB(cli client.Client, cmd string) (res []client.Result, err error) {    q := client.Query{        Command:  cmd,        Database: "test",    }    if response, err := cli.Query(q); err == nil {        if response.Error() != nil {            return res, response.Error()        }        res = response.Results    } else {        return res, err    }    return res, nil}​// insertfunc writesPoints(cli client.Client) {    bp, err := client.NewBatchPoints(client.BatchPointsConfig{        Database:  "test",        Precision: "s", //精度，默认ns    })    if err != nil {        log.Fatal(err)    }    tags := map[string]string{"cpu": "ih-cpu"}    fields := map[string]interface{}{        "idle":   201.1,        "system": 43.3,        "user":   86.6,    }​    pt, err := client.NewPoint("cpu_usage", tags, fields, time.Now())    if err != nil {        log.Fatal(err)    }    bp.AddPoint(pt)    err = cli.Write(bp)    if err != nil {        log.Fatal(err)    }    log.Println("insert success")}​func main() {    conn := connInflux()    fmt.Println(conn)​    // insert    writesPoints(conn)​    // 获取10条数据并展示    qs := fmt.Sprintf("SELECT * FROM %s LIMIT %d", "cpu_usage", 10)    res, err := queryDB(conn, qs)    if err != nil {        log.Fatal(err)    }​    for _, row := range res[0].Series[0].Values {        for j, value := range row {            log.Printf("j:%d value:%v\n", j, value)        }    }}go
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

# 迭代器和生成器

## yield和return的区别

在一个函数里return只能执行一次，return后的语句均不会执行。

但是yield之后可以保存函数运行状态，下次继续往后执行。



## 每次生成的生成器都是不同的生成器

```python
g = countdown(5)
print(g.__next__()) # 5
print(g.__next__()) # 4
print("-- for ---")
for i in g:
    print(i) # 3 2 1 发射
# 调用迭代器的__next__方法，生成器顺序执行
```

```python
print(countdown(5).__next()) # 5
print(countdown(5).__next()) # 5
print(countdown(5).__next()) # 5
# 每次输出都是一样，说明每次都是不同的生成器
```

## random库

关于`random`的一个小练习，写一个生成4位随机验证码（包含数字和字母）的小程序：

```python
import random


def get_code(n):
    """
    生成指定位数的随机验证码，包含数字和字母
    :param n: 随机验证码位数
    :return:
    """
    code = ""
    for i in range(n):  # 循环n次
        number = random.randint(0, 9)  # 生成数字
        letter = chr(random.randint(65, 90))  # 利用chr将数字转成字符
        aim = random.choice([number, letter])  # 从数字和字符中选一个
        code += str(aim)  # 将每一次循环得到的结果拼接起来
    return code

if __name__ == '__main__':
    ret = get_code(4)
    print(ret)
```

## os库

```python
os.getcwd() 获取当前工作目录，即当前python脚本工作的目录路径
os.chdir("dirname")  改变当前脚本工作目录；相当于shell下cd
os.curdir  返回当前目录: ('.')
os.pardir  获取当前目录的父目录字符串名：('..')
os.makedirs('dirname1/dirname2')    可生成多层递归目录
os.removedirs('dirname1')    若目录为空，则删除，并递归到上一级目录，如若也为空，则删除，依此类推
os.mkdir('dirname')    生成单级目录；相当于shell中mkdir dirname
os.rmdir('dirname')    删除单级空目录，若目录不为空则无法删除，报错；相当于shell中rmdir dirname
os.listdir('dirname')    列出指定目录下的所有文件和子目录，包括隐藏文件，并以列表方式打印
os.remove()  删除一个文件
os.rename("oldname","newname")  重命名文件/目录
os.stat('path/filename')  获取文件/目录信息
os.sep    输出操作系统特定的路径分隔符，win下为"\\",Linux下为"/"
os.linesep    输出当前平台使用的行终止符，win下为"\t\n",Linux下为"\n"
os.pathsep    输出用于分割文件路径的字符串 win下为;,Linux下为:
os.name    输出字符串指示当前使用平台。win->'nt'; Linux->'posix'
os.system("bash command")  运行shell命令，直接显示
os.environ  获取系统环境变量
os.path.abspath(path)  返回path规范化的绝对路径
os.path.split(path)  将path分割成目录和文件名二元组返回
os.path.dirname(path)  返回path的目录。其实就是os.path.split(path)的第一个元素
os.path.basename(path)  返回path最后的文件名。如何path以／或\结尾，那么就会返回空值。即os.path.split(path)的第二个元素
os.path.exists(path)  如果path存在，返回True；如果path不存在，返回False
os.path.isabs(path)  如果path是绝对路径，返回True
os.path.isfile(path)  如果path是一个存在的文件，返回True。否则返回False
os.path.isdir(path)  如果path是一个存在的目录，则返回True。否则返回False
os.path.join(path1[, path2[, ...]])  将多个路径组合后返回，第一个绝对路径之前的参数将被忽略
os.path.getatime(path)  返回path所指向的文件或者目录的最后存取时间
os.path.getmtime(path)  返回path所指向的文件或者目录的最后修改时间
os.path.getsize(path) 返回path的大小
```

## sys库

```python
sys.argv           命令行参数List，第一个元素是程序本身路径
sys.exit(n)        退出程序，正常退出时exit(0)
sys.version        获取Python解释程序的版本信息
sys.maxint         最大的Int值
sys.path           返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
sys.platform       返回操作系统平台名称
```

## 闭包和装饰器

闭包就是在函数内引用了外部变量。

装饰器就是不改变函数的调用形式，但是对函数进行了功能上的修改。

# pandas使用

## pandas处理excel

获取excel的所有sheet名称：

```python
df = pd.read_excel(path, sheetname=None) 
//这里的df时所有sheet名称的字典，所以需要用for循环一个一个取
df.keys()
```

去除或填充nan项：

```python
data.dropna(how = ‘all’) 
# 传入这个参数后将只丢弃全为缺失值的那些行
data.dropna(axis = 1) 
# 丢弃有缺失值的列
data.dropna(axis=1,how=“all”) 
# 丢弃全为缺失值的那些列
data.dropna(axis=0,subset = [“Age”, “Sex”]) 
# 丢弃‘Age’和‘Sex’这两列中有缺失值的行
```

```python
data.fillna(method = ‘ffill’, axis = 0) 
# 将通过前向填充 (ffill) 方法用同一列的前一个数作为填充
data.fillna(method = ‘ffill’, axis = 1)
# 将通过前向填充 (ffill) 方法用同一行的前一个数作为填充
data.fillna(method = ‘backfill’, axis = 0)
# 将通过前向填充 (ffill) 方法用同一列的后一个数作为填充
data.fillna(method = ‘backfill’, axis = 1) 
# 将通过前向填充 (ffill) 方法用同一行的后一个数作为填充
data.interpolate(method = ‘linear’, axis = 0) 
# 将通过 linear 插值使用同一列的中间值作为填充
data.interpolate(method = ‘linear’, axis = 1) 
# 将通过 linear 插值使用同一行的中间值作为填充
```

去除数据

```python
drop(labels, axis=0, level=None, inplace=False, errors='raise')
#axis为0时表示删除行，axis为1时表示删除列。
#errors='raise'会让程序在labels接收到没有的行名或者列名时抛出错误导致程序停止运行，errors='ignore'会忽略没有的行名或者列名，只对存在的行名或者列名进行操作，没有指定的话也是默认‘errors='raise'’。
#inplace默认为false，代表是否对原数据生效，boolean类型。
#level接收int或索引名，代表标签所在级别，默认为none。
#需要注意一点：在删除过后，索引也就产生了缺失，可以将索引重置-reset_index(drop=True)
```





打开需要处理的文件用read_文件类型的函数：read_csv("test.csv")  

1、**filepath_or_buffer：**数据输入的路径：可以是文件路径、可以是URL，也可以是实现read方法的任意对象(open操作)。

**2、sep：**读取csv文件时指定的分隔符，默认为逗号。注意："csv文件的分隔符" 和 "我们读取csv文件时指定的分隔符" 一定要一致。

**3、delimiter ：**分隔符的另一个名字，与 sep 功能相似。

**4、delim_whitespace ：**默认为 False，设置为 True 时，表示分割符为空白字符，可以是空格、"\t"等等。不管分隔符是什么，只要是空白字符，那么可以通过delim_whitespace=True进行读取。

**5、header：**设置导入 DataFrame 的列名称，默认为 "infer"，注意它与下面介绍的 names 参数的微妙关系。

**6、names：**当names没被赋值时，header会变成0，即选取数据文件的第一行作为列名；当 names 被赋值，header 没被赋值时，那么header会变成None。如果都赋值，这个时候，相当于先不看names，只看header，header为0代表先把第一行当做表头，下面的当成数据；然后再把表头用names给替换掉。

**7、index_col：**用于指定某列为索引，那么读取出来的数据就不会自动添加一列索引，在重新写入数据的时候就不会出现多出一列。

**8、usecols：**如果一个数据集中有很多列，但是我们在读取的时候只想要使用到的列，我们就可以使用这个参数。当多列时使用列表作为参数值。

**9、mangle_dupe_cols：**在实际工作中，我们得到的数据会很复杂，有时导入的数据会含有名字相同的列。参数 mangle_dupe_cols 会将重名的列后面多一个 .1，该参数默认为 True，如果设置为 False，会抛出不支持的异常：

**10、prefix：**当导入的数据设置header=None 时，设置此参数会自动加一行name。

通用解析参数

pandas有两个解析引擎，默认为c但是当指定的参数c无法解析时会退化成python，这个时候需要将engine指定为python，并可以加上encoding='utf-8'

true_values和false_value可以指定什么数据为true或false true_values=["数据值"]

skipfooter：从文件末尾过滤行，解析引擎退化为 Python。这是因为 C 解析引擎没有这个特性。



返回的是读取了的数据，按照第一行的标题来命名每一列，所以可以直接按照列的名称来取出整列的数据。

只会按照列来循环 for col in test: print(test[col])

loc属性可以返回 指定索引 对应到某一行的数据。

```python
data = {
 "calories": [420, 380, 390],
 "duration": [50, 40, 45]
}

df = pd.DataFrame(data, index = ["day1", "day2", "day3"])
# 指定索引
print(df.loc["day2"])
```

## Pandas JSON

常规json可以这样处理：

```python
# 这里的url 可以是文件的网址，也可以是文件的路径
df = pd.read_json(url)
print(df)
```

内嵌的json：

```json
{
  "school_name": "ABC primary school",
  "class": "Year 1",
  "students": [
  {
    "id": "A001",
    "name": "Tom",
    "math": 60,
    "physics": 66,
    "chemistry": 61
  },
  {
    "id": "A002",
    "name": "James",
    "math": 89,
    "physics": 76,
    "chemistry": 51
  },
  {
    "id": "A003",
    "name": "Jenny",
    "math": 79,
    "physics": 90,
    "chemistry": 78
  }]
}
```

可以这样处理，需要使用到 **json_normalize()** 方法将内嵌的数据完整的解析出来：

```python
import pandas as pd
import json
# 使用 Python JSON 模块载入数据
with open('nested_list.json','r') as f:
  data = json.loads(f.read())
# 展平数据
df_nested_list = pd.json_normalize(data, record_path =['students'])
print(df_nested_list)
```

# pandas的120道小题

1.将字典创建为DataFrame

```python
data = {
    "grammar": ["Python","c","Java","Go",np.nan,"SQL","PHP","Python"],
    "score": [1,2,np.nan,4,5,6,7,10]
}
df = pd.DataFrame(data)
```

2.提取含有字符串"Python"

2.提取含有"Python"的行

```python
# 方法1
df[df["grammar"]=="Python"]
# 方法2
result = df["grammar"].str.contains("Python")
result.fillna(value=False, inplace=True)
df[result]
```

3.输出df的所有列名

```python
df.columns
```

4.修改第二列列名为"popularity"

```python
df.rename(columns={"score":"popularity"},inplace=True)
```

5.统计grammar列中每种编程语言出现的次数

```python
df["grammar"].value_counts()
```

6.将空值用上下值的平均值填充

```python
df["popularity"]=df["popularity"].fillna(df["popularity"].interpolate())
```

7.提取popularity列中值大于3的行

```python
df[df["popularity"]>3]
```

8.按照grammar列进行去除重复值

```python
df.drop_duplicates(["grammar"])
```

9.计算popularity列平均值

```python
df["popularity"].mean()
```

10.将grammar列转换为list

```python
df["grammar"].to_list()
```

11.将DataFrame保存为Excel

```python
df.to_excel("test.xlsx")
```

12.查看数据行列数

```python
# shape是属于df的属性
df.shape
```

13.提取popularity列值大于3小于7的行

```python
df[(df["popularity"]>3)&(df["popularity"]<7)]
```

14.交换两列位置

```python
# 方法1
cols = df.colums([1,0])
df = df[cols]
# 方法2
temp = df["popularity"]
df.drop(labels=["popularity"],axis=1,inplace=True)
df.insert(0,"popularity",temp)
```

15.提取popularity列最大值所在行

```python
df[df["popularity"]==df["popularity"].max()]
```

16.查看最后5行数据

```python
df.tail()
```

17.删除最后一行数据

```python
df.drop([len(df)-1],inplace=True)
```

18.添加一行数据["Perl",6.6]

```python
row = {"grammar":"Perl","popularity":"6.6"}
df = df.append(row,ignore_index=True)
```

19.对数据按照"popularity"列值的大小排序

```python
df.sort_values("popularity",inplace=True)
```

20.统计grammar列每个字符串的长度

```python
df["grammar"] = df["grammar"].fillna("R")
df["len_str"] = df["grammar"].map(lambda x: len(x))
```

21.读取本地excel数据

```python
df.read_excel(file_path)
```

22.查看df数据前5行

```python
df.head()
```

23.将salary列数据转换为最大值

```python
# 方法1 salary列的类型为字符串，所以需要先整理成整型
import re
def func(df):
	list = df["salary"].split('-')
	smin = int(list[0].strip('k'))
	smax = int(list[1].strip('k'))
	df["salary"] = int((smin+smax) / 2 * 1000)
	
df = df.apply(func, axis=1)

# 方法2 直接使用.ix方法或 .iloc方法
```

24.将数据根据学历进行分组并计算平均薪资

```
df.groupby("education").mean()
```

25.将createTime列时间转换为月-日

```python
# 弹幕里说pandas不能使用for，使用apply
for i in range(len(df)):
	df,ix[i,0] = df.ix[i,0].to_pydatetime().strftime("%m-%d")
```

26.查看索引、数据类型和内存信息

```python
df.info()
```

27.查看数值型列的汇总统计

```python
df.describe()
```

28.新增一列根据salary将数据分为三组

```python
bins = [0,5000,20000,50000]
group_names = ["","",""]
df['categories'] = pd.cut(df['salary'], bins, labels=group_names)
```

29.按照salary列对数据降序排列

```python
df.sort_values('salary', ascending=True/False)
```

30.取出第33行数据

```python
df.loc[32]
```

31.计算salary列的中位数

```python
np.median(df['salary'])
```

32.绘制薪资水平频率分布直方图

```python
import matplotlib.pylot as plt
df.salary.plot(kind='hist')
```

33.绘制薪资水平密度曲线

```python
df.salary.plot(kind='kde',xlim(0,8000))
```

34.删除最后一列categories

```python
df.drop(columns=['categories'],inplace=True)
```

35.将df的第一列与第二列合并为新的一列

```python
df['test'] = df['education'] + df['creatTime']
```

36.将education列与salary列合并微信的一列

```python
df['test1'] = df['salary'].map(str)+ df['education']
```

37.计算salary最大值与最小值之差

```python
df[['salary']].apply(lambda x:x.max()-x.min())
```

38.将第一行与最后一行拼接

```python
pd.concat([df[:1],df[-2:-1]])
```

39.将第8行数据添加至末尾

```python
df.append(df.iloc[7])
```

40.查看每列的数据类型

```python
df.dtypes
```

41.将creatTime列设置为索引

```python
df.set_index("creatTime")
```

42.生成一个和df长度相同的随机数dataframe

```python
df1 = pd.DataFrame(pd.Series(np.random.randint(1,10,135)))
```

43.将上一题生成的dataframe与df合并

```python
df = pd.concat([df,df1],axis=1)
```

44.生成新的一列new为salary列减去之前生成随机数列

```python
df["new"] = int(df['salary']) - int(df[0])
```

45.检查数据中是否含有任何缺失值

```python
df.isnull().value.any()
```

46.将salary列类型转换为浮点数

```python
df['salary'].astype(np.float64)
```

47.计算salary大于10000的次数

```python
len(df[df['salary']>10000])
```

48.查看每种学历出现的次数

```python
# 方法1
len(df[df['education']])
# 方法2
df.education.value_counts()
```

49.查看education列共有几种学历

```python
df['education'].nunique()
```

50.提取salary与new列的和大于60000的最后三行

```python
df1 = df[['salary','new']]
rowsums = df1.apply(np.sum,axis=1)
res = df.iloc[np.where(rowsums>60000)[0][-3:],:]
```

## 金融数据处理

51.使用绝对路径读取本地Excel数据



52.查看数据前三行





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

# Flask框架

Flask基于Werkzeug WSGI工具包和Jinja2模板引擎。

在调试过程中开发使用app.run(debug=True)启动flask项目。

## 路由：

1.使用@app.route('/madasd')将路由与函数绑定。

2.使用app.add_url_rule()函数来将路由与函数绑定。参数为（'/','hello',函数名）

路由还可以是动态变换的：

在route()注解中加入路由和变量。例如:

```python
@app.route('/hello/<name>')

def hello_name(name):

	return name
```

**注意一点的是路由在写成斜杠结尾时，可以不用在路由中明确写出也可访问，但是末尾斜杠省掉，则路由一定不能出现斜杠，否则返回404**

url_for()函数用于动态构建特定函数的URL。都是来自flask库。

```python
from flask import Flask, redirect, url_for, request

@app.route('/admin') 
def hello_admin():   
	return 'Hello Admin'

@app.route('/guest/<guest>')
def hello_guest(guest):   
	return 'Hello %s as Guest' % guest

@app.route('/user/<name>') 
def hello_user(name):   
	if name =='admin':      
		return redirect(url_for('hello_admin'))  
	else:      
		return redirect(url_for('hello_guest', guest = name))
```

Flask默认的路由响应为GET请求，但也可以通过route装饰器提供方法参数来进行配置。将以下脚本另存为login.html

```html
<html>
   <body>
      <form action = "http://localhost:5000/login" method = "post">
         <p>Enter Name:</p>
         <p><input type = "text" name = "nm" /></p>
         <p><input type = "submit" value = "submit" /></p>
      </form>
   </body>
</html>
```

现在在Python shell中输入以下脚本：

```python
from flask import Flask, redirect, url_for, request
app = Flask(__name__)

@app.route('/success/<name>')
def success(name):
   return 'welcome %s' % name

@app.route('/login',methods = ['POST', 'GET'])
def login():
   if request.method == 'POST':
      # 请求的表单信息通过name属性里的值来获取
      user = request.form['nm']
      return redirect(url_for('success',name = user))
   else:
      user = request.args.get('nm')
      return redirect(url_for('success',name = user))

if __name__ == '__main__':
   app.run(debug = True)
```

flask的模板语言为  {{ 变量名 }}    用render_template( 'html文件' ，html中的变量名=视图函数中的变量名)

## Flask Request对象

Request对象的重要属性如下所列：

- **Form** - 它是一个字典对象，包含表单参数及其值的键和值对。
- **args** - 解析查询字符串的内容，它是问号（？）之后的URL的一部分。
- **Cookies** - 保存Cookie名称和值的字典对象。
- **files** - 与上传文件有关的数据。
- **method** - 当前请求方法。

## Flask Cookies

make_response属于Flask文件，所以直接导入。

```python
from flask import Flask, make_response, request # 注意需导入 make_response

app = Flask(__name__)

@app.route("/set_cookies")
def set_cookie():
    resp = make_response("success")
    resp.set_cookie("w3cshool", "w3cshool",max_age=3600)    
    return resp

@app.route("/get_cookies")
def get_cookie():
    # 获取名字为Itcast_1对应cookie的值
    cookie_1 = request.cookies.get("w3cshool")  
    return cookie_1

@app.route("/delete_cookies")
def delete_cookie():
    # 注意删除，只是让cookie过期
    resp = make_response("del success")
    resp.delete_cookie("w3cshool")
    return resp
```

## Flask 重定向和错误

redirect()函数用来返回一个响应对象，并将用户重定向到指定状态码的另一个目标位置。错误状态码可以使用abort()函数来进行更改。这两个函数都在flask文件中导入。

## Flask 消息闪现



## Flask蓝图(BluePrint)

Flask项目的文件目录为：功能模块(蓝图)文件夹放功能模块代码，static文件夹放js脚本，templates文件夹放html静态页面，app.py启动文件。

注意蓝图模块，需要导入模块    from flask import Blueprint

蓝图模块对象的生成：

```python
upload = Blueprint("upload", __name__, template_folder="../templates", static_folder='../static')
```

## Flask扩展包

Flask-email邮件扩展包：使用邮件服务器来进行邮件的发送。

Flask-WTF表单验证扩展包：用来验证提交的表单的内容是否符合要求。

Flask-sijax简单异步请求包：使用jQuery.ajax来发出AJAX请求。
