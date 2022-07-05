```go
func main() {
	str := "hello"
    // str1 := []byte(str) 将字符串变为数组存储
    str[0] = 'x'  // 这里不能直接修改
    // str = string(str1) 将数组变回字符串
	fmt.Println(str)
}
```

以上输出结果为Compilation Error(编译错误)，因为Go中的字符串是只读的。

## 变量声明

变量声明方式有多种：

```go
//var声明（需要制定类型） 
var var1 int32   var1 = 32 ; 
//":="进行赋值（不需要指定类型）
var2 := 32 ;
//var声明（不指定类型） 
var var3 = 32
//多变量声明
name, age = 'avc', 18
//多变量声明
var(name1 = "cav" age = 11)
```

#### 字符型变量：

golang没有专门的字符类型，一般使用byte保存字符。

go的字符串由字节组成，没有字符。所以需要对其修改时，使用对应字节长度的数组：byte 等同于int8，常用来处理ASCII字符，rune 等同于int32，常用来处理Unicode或utf-8字符。

当使用格式化输出时需要用Printf("%f ", str1)。

#### 字符串变量：

在字符串赋值时可以使用 " " 也可以使用 ``(反引号，键盘左上角)，他们的区别在于前者可以识别转义字符，后者不会。

```go
varstr := "hello" + " world" //字符串可通过 + 连接
varstr += "！" // 可以通过 += 来对自身进行拓展
```

#### 类型转换：

go中不支持类型自动转换，需要显式转换。

```go
varsa := 12.9
varsa2 := int(varsa) // 这样会损失精度， 0.9直接被丢弃
aa := fmt.Sprintf("%f",varsa) // 转换为字符串
var3, error3 := strconv.ParseFloat(aa, 32) // 字符串转值类型
```

## 接口问题

```go
type A interface {
	ShowA() int
}

type B interface {
	ShowB() int
}

type Work struct {
	i int
}

func (w Work) ShowA() int {
	return w.i + 10
}

func (w Work) ShowB() int {
	return w.i + 20
}

func Test2() {
	c := Work{3}
	var a A = c
	var b B = c
	fmt.Println(a.ShowA())
	fmt.Println(b.ShowB())
}//输出结果为：13 23
```

这个问题反映的是接口，一种类型实现多个接口，结构体work分别实现了接口A、B,所以接口变量a,b分别实现各自的方法，输出不同结果。

