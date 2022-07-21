---
typora-root-url: typora-user-images
---

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

## 变量声明与赋值问题

```go
func Dif_var_make() {
	// 以下只是声明m变量，但并没有分配内存
	//顾不可以直接进行赋值操作
	// var m map[string]int
	// m["a"] = 1
	//由于没有 "b" 元素，v会返回值类型对应的零值
	// if v := m["b"]; v != nil {
	// 	fmt.Println(v)
	// }

	m := make(map[string]int)
	m["a"] = 1
	//由于没有 "b" 元素，ok就为false
	if v, ok := m["b"]; ok {
		fmt.Println(v)
	}
}

//下面代码中，x 已声明，y 没有声明，判断每条语句的对错。
1.x, _ := f()
2.x, _ = f()
3.x, y := f()
4.x, y = f()
//知识点：变量的声明。
//1.错，x 已经声明，不能使用 :=；
//2.对；
//3.对，当多值赋值时，:= 左边的变量无论声明与否都可以；
//4.错，y 没有声明。
```

问题在以上注释中给与解释。

## 变量和返回值之间的关系

```go
func increaseA() int {
	var i int
	defer func() {
		i++
	}()
	return i
}

func increaseB() (r int) {
	defer func() {
		r++
	}()
	return r
}

func main() {
	fmt.Println(increaseA())
	fmt.Println(increaseB())
}//0 1
//原因：
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

**永远不要使用一个指针指向一个接口类型，因为它已经是一个指针。**

```go
package day8

type S struct {
}

func f(x interface{}) {
}

func g(x *interface{}) {
}

func main() {
	s := S{}
	p := &s
	f(s) //A
	g(s) //B
	f(p) //C
	g(p) //D
}


// A、B、C、D 哪些选项有语法错误？
// 参考答案及解析：BD。
// 函数参数为 interface{} 时可以接收任何类型的参数，包括用户自定义类型等，即使是接收指针类型也用 interface{}，而不是使用 *interface{}。

```

## 实际使用

```go
// A B处补充完整，让程序正常输出
type S struct {
	m string
}

func f() *S {
    return __  //A    &S{"foo"}
}

func main() {
    p := __    //B    f()或者*f()
	fmt.Println(p.m) //print "foo"
}
```



## 切片 a、b、c 的长度和容量分别是多少？

```go
func main() {
	s := [3]int{1, 2, 3}
	a := s[:0] //[]
	b := s[:2] //[1,2]
	c := s[1:2:cap(s)] //[2]
}
```

### 答案解析：

参考答案及解析：a、b、c 的长度和容量分别是 0 3、2 3、1 2。

知识点：数组或切片的截取操作。截取操作有带 2 个或者 3 个参数，形如：[i:j] 和 [i:j:k]，假设截取对象的底层数组长度为 l。在操作符 [i:j] 中，如果 i 省略，默认 0，如果 j 省略，默认底层数组的长度，截取得到的**切片长度和容量计算方法是 j-i、l-i**。操作符 [i:j:k]，k 主要是用来限制切片的容量，但是不能大于数组的长度 l，截取得到的**切片长度和容量计算方法是 j-i、k-i**。同python的切片一致，只是多了个容量的概念。

### 切片的本质

切片的本质就是对底层数组的封装，它包含了三个信息：底层数组的指针、切片的长度（len）和切片的容量（cap）。

举个例子，现在有一个数组`a := [8]int{0, 1, 2, 3, 4, 5, 6, 7}`，切片`s1 := a[:5]`，相应示意图如下。

![slice_01](E:\markdownInJob\typora-user-images\slice_01.png)

切片`s2 := a[3:6]`，相应示意图如下：

![slice_02](E:\markdownInJob\typora-user-images\slice_02.png)

## 切片扩容-append 函数

Go语言的内建函数`append()`可以为切片动态添加元素。 可以一次添加一个元素，可以添加多个元素，也可以添加另一个切片中的元素（后面加…）。

```go
func main(){
	var s []int
	s = append(s, 1)        // [1]
	s = append(s, 2, 3, 4)  // [1 2 3 4]
	s2 := []int{5, 6, 7}  
	s = append(s, s2...)    // [1 2 3 4 5 6 7]
}
```

**注意：**通过var声明的零值切片可以在`append()`函数直接使用，无需初始化。

```go
var s []int
s = append(s, 1, 2, 3)
```

没有必要像下面的代码一样初始化一个切片再传入`append()`函数使用，

```go
s := []int{}  // 没有必要初始化
s = append(s, 1, 2, 3)

var s = make([]int)  // 没有必要初始化
s = append(s, 1, 2, 3)
```

每个切片会指向一个底层数组，这个数组的容量够用就添加新增元素。当底层数组不能容纳新增的元素时，切片就会自动按照一定的策略进行“扩容”，此时该切片指向的底层数组就会更换。“扩容”操作往往发生在`append()`函数调用时，所以我们通常都需要用原变量接收append函数的返回值。

举个例子：

```go
func main() {
	//append()添加元素和切片扩容
	var numSlice []int
	for i := 0; i < 10; i++ {
		numSlice = append(numSlice, i)
		fmt.Printf("%v  len:%d  cap:%d  ptr:%p\n", numSlice, len(numSlice), cap(numSlice), numSlice)
	}
}
//输出：
[0]  len:1  cap:1  ptr:0xc0000a8000
[0 1]  len:2  cap:2  ptr:0xc0000a8040
[0 1 2]  len:3  cap:4  ptr:0xc0000b2020
[0 1 2 3]  len:4  cap:4  ptr:0xc0000b2020
[0 1 2 3 4]  len:5  cap:8  ptr:0xc0000b6000
[0 1 2 3 4 5]  len:6  cap:8  ptr:0xc0000b6000
[0 1 2 3 4 5 6]  len:7  cap:8  ptr:0xc0000b6000
[0 1 2 3 4 5 6 7]  len:8  cap:8  ptr:0xc0000b6000
[0 1 2 3 4 5 6 7 8]  len:9  cap:16  ptr:0xc0000b8000
[0 1 2 3 4 5 6 7 8 9]  len:10  cap:16  ptr:0xc0000b8000
```

从上面的结果可以看出：

1. `append()`函数将元素追加到切片的最后并返回该切片。
2. 切片numSlice的容量按照1，2，4，8，16这样的规则自动进行扩容，每次扩容后都是扩容前的2倍。

append()函数还支持一次性追加多个元素。 例如：

```go
var citySlice []string
// 追加一个元素
citySlice = append(citySlice, "北京")
// 追加多个元素
citySlice = append(citySlice, "上海", "广州", "深圳")
// 追加切片
a := []string{"成都", "重庆"}
citySlice = append(citySlice, a...)// 注意追加切片时后面要加...
fmt.Println(citySlice) //[北京 上海 广州 深圳 成都 重庆]
```

## 切片的扩容策略

- 首先判断，如果新申请容量（cap）大于2倍的旧容量（old.cap），最终容量（newcap）就是新申请的容量（cap）。
- 否则判断，如果旧切片的长度小于1024，则最终容量(newcap)就是旧容量(old.cap)的两倍，即（newcap=doublecap），
- 否则判断，如果旧切片长度大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始循环增加原来的1/4，即（newcap=old.cap,for {newcap += newcap/4}）直到最终容量（newcap）大于等于新申请的容量(cap)，即（newcap >= cap）
- 如果最终容量（cap）计算值溢出，则最终容量（cap）就是新申请容量（cap）。

## 追加元素

- 通过`append()`函数可以在切片的尾部追加 N 个元素

```go
  var a []int
  a = append(a, 1)                    // 追加一个元素
  a = append(a, 1, 2, 3)                // 追加多个元素
  a = append(a, []int{1, 2, 3}...)    // 追加一个切片，注意追加切片时后面要加...
```

- 使用 append()函数也可以在切片头部添加元素

```go
  a = append([]int{0}, a...)            // 在开头添加一个元素
  a = append([]int{1, 2, 3}, a...)    // 在开头添加一个切片
```

## 注:从头部添加元素会引起内存的重分配，导致已有元素全部复制一次。因此从头部添加元素的开销要比从尾部添加元素大很多

## 注意：一般情况下，使用 append 函数追加新的元素时，都会用原切片变量接收返回值来获得更新。不然就会出现以下状况，每次的结果都会被下次覆盖掉。

```go
func AppendDemo() {
       x := make([]int, 0, 10)
       x = append(x, 1, 2, 3)
       y := append(x, 4)
       z := append(x, 5)
       fmt.Println(x)
       fmt.Println(y)
       fmt.Println(z)
   }//output
   [1 2 3]
   [1 2 3 5]
   [1 2 3 5]
```

## 使用copy()函数复制切片

由于切片是引用类型，所以a和b其实都指向了同一块内存地址。修改b的同时a的值也会发生变化。

Go语言内建的`copy()`函数可以迅速地将一个切片的数据复制到另外一个切片空间中，`copy()`函数的使用格式如下：

```bash
copy(destSlice, srcSlice []T)
```

其中：

- srcSlice: 数据来源切片
- destSlice: 目标切片

举个例子：

```go
func main() {
	// copy()复制切片
	a := []int{1, 2, 3, 4, 5}
	c := make([]int, 5, 5)
	copy(c, a)     //使用copy()函数将切片a中的元素复制到切片c
	fmt.Println(a) //[1 2 3 4 5]
	fmt.Println(c) //[1 2 3 4 5]
	c[0] = 1000
	fmt.Println(a) //[1 2 3 4 5]
	fmt.Println(c) //[1000 2 3 4 5]
}
```

## 从切片中删除元素

Go语言中并没有删除切片元素的专用方法，我们可以使用切片本身的特性来删除元素。 代码如下：

```go
func main() {
	// 从切片中删除元素
	a := []int{30, 31, 32, 33, 34, 35, 36, 37}
	// 要删除索引为2的元素
	a = append(a[:2], a[3:]...)
	fmt.Println(a) //[30 31 33 34 35 36 37]
}
```

总结一下就是：要从切片a中删除索引为`index`的元素，操作方法是

```go
a = append(a[:index], a[index+1:]...)
```



## defer输出顺序

```go
type Person struct {
	age int
}
func Test1() {
	person := &Person{28}
	// 1.28
	defer fmt.Println(person.age)
	// 2.29
	defer func(p *Person) {
		fmt.Println(p.age)
	}(person)
	// 3.29
	defer func() {
		fmt.Println(person.age)
	}()
	person.age = 29
}
/*
参考答案及解析：29 29 28。变量 person 是一个指针变量 。

1.person.age 此时是将 28 当做 defer 函数的参数，会把 28 缓存在栈中，等到最后执行该 defer 语句的时候取出，即输出 28；

2.defer 缓存的是结构体 Person{28} 的地址，最终 Person{28} 的 age 被重新赋值为 29，所以 defer 语句最后执行的时候，依靠缓存的地址取出的 age 便是 29，即输出 29；

3.很简单，闭包引用，输出 29；

又由于 defer 的执行顺序为先进后出，即 3 2 1，所以输出 29 29 28。
*/
```

## go的switch case

fallthrough 语法可以执行满足条件的case的下一个case，是为了兼容C语言中的case设计的。

## goto(跳转到指定标签)

`goto`语句通过标签进行代码间的无条件跳转。`goto`语句可以在快速跳出循环、避免重复退出上有一定的帮助。Go语言中使用`goto`语句能简化一些代码的实现过程。 例如双层嵌套的for循环要退出时：

```go
func gotoDemo1() {
	var breakFlag bool
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				// 设置退出标签
				breakFlag = true
				break
			}
			fmt.Printf("%v-%v\n", i, j)
		}
		// 外层for循环判断
		if breakFlag {
			break
		}
	}
}
```

使用`goto`语句能简化代码：

```go
func gotoDemo2() {
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				// 设置退出标签
				goto breakTag
			}
			fmt.Printf("%v-%v\n", i, j)
		}
	}
	return
	// 标签
breakTag:
	fmt.Println("结束for循环")
}
```

## 9*9乘法表

```go
func jiujiu(){
    for i:=1;i<10;i++{
        for j:=1;j<=i;j++{
            fmt.Printf("%v * %v = %2v ",i,j,i*j)
        }
    	fmt.Println()
    }
}
```

