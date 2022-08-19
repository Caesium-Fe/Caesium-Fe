### 关于循环语句，下面说法正确的有（CD）

- A. 循环语句既支持 for 关键字，也支持 while 和 do-while；
- B. 关键字 for 的基本使用方法与 C/C++ 中没有任何差异；
- C. for 循环支持 continue 和 break 来控制循环，但是它提供了一个更高级的 break，可以选择中断哪一个循环；
- D. for 循环不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量；

解析：CD。C选项中break的用法确实还有跳转到标签位置的  LOOP:  。D选项中的平行赋值: a[i], a[j] = a[j], a[i] ，逗号分隔: a[i]=a[j],a[j]=a[i]。

### 关于协程，下面说法正确是（AD）

- A. 协程和线程都可以实现程序的并发执行；
- B. 线程比协程更轻量级；
- C. 协程不存在死锁问题；
- D. 通过 channel 来进行协程间的通信；

解析：一个进程可以有多个*线程*,一个*线程*可以有多个*协程*。



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

## 重用和重声明

```go
//下面这段代码输出结果正确吗？
type Foo struct {
	bar string
}
func main() {
	s1 := []Foo{
		{"A"},
		{"B"},
		{"C"},
	}
	s2 := make([]*Foo, len(s1))
	for i, value := range s1 {
        //这里的value取的都是同一个地址
		s2[i] = &value
	}
	fmt.Println(s1[0], s1[1], s1[2])
	fmt.Println(s2[0], s2[1], s2[2])
}
/*
输出：
{A} {B} {C}
&{A} &{B} &{C}

答案解析：
参考答案及解析：s2 的输出结果错误。

s2 的输出是 &{C} &{C} &{C}，在前面题目我们提到过，for range 使用短变量声明(:=)的形式迭代变量时，变量 i、value 在每次循环体中都会被重用，而不是重新声明。所以 s2 每次填充的都是临时变量 value 的地址，而在最后一次循环中，value 被赋值为{c}。因此，s2 输出的时候显示出了三个 &{c}。
可行的解决办法如下：
for i := range s1 {
	s2[i] = &s1[i]
}
*/
```



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

不同的类型之间不能进行运算，所以运算前需要将变量的类型统一。

```go
// 关于类型转化，下面选项正确的是？
A.错
type MyInt int
var i int = 1
var j MyInt = i

B.错
type MyInt int
var i int = 1
var j MyInt = (MyInt)i

//C.对 改名变量类型 只能用于强制类型转化时使用
type MyInt int
var i int = 1
var j MyInt = MyInt(i)

D.错
type MyInt int
var i int = 1
var j MyInt = i.(MyInt)
```

## 变量声明与赋值问题

```go
//下面代码输出正确的是？

func main() {
	i := 1
	s := []string{"A", "B", "C"}
	i, s[i-1] = 2, "Z"
	fmt.Printf("s: %v \n", s)
}
A. s: [Z,B,C]
B. s: [A,Z,C]

/*
参考答案及解析：A。

知识点：多重赋值。

多重赋值分为两个步骤，有先后顺序：
计算等号左边的索引表达式和取址表达式，接着计算等号右边的表达式；
赋值；
所以本例，会先计算 s[i-1]，等号右边是两个表达式是常量，所以赋值运算等同于 i, s[0] = 2, "Z"。
*/

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

// 下面代码输出什么？
type Math struct {
	x, y int
}

var m = map[string]Math{
	"foo": Math{2, 3},
}

func main() {
	m["foo"].x = 4
	fmt.Println(m["foo"].x)
}
/*
答案解析：
参考答案及解析：B，编译报错 cannot assign to struct field m["foo"].x in map。错误原因：对于类似 X = Y的赋值操作，必须知道 X 的地址，才能够将 Y 的值赋给 X，但 go 中的 map 的 value 本身是不可寻址的。

有两个解决办法：

a.使用临时变量*/

type Math struct {
	x, y int
}

var m = map[string]Math{
	"foo": Math{2, 3},
}

func main() {
	tmp := m["foo"]
	tmp.x = 4
	m["foo"] = tmp
	fmt.Println(m["foo"].x)
}
// b.修改数据结构

type Math struct {
	x, y int
}

var m = map[string]*Math{
	"foo": &Math{2, 3},
}

func main() {
	m["foo"].x = 4
	fmt.Println(m["foo"].x)
	fmt.Printf("%#v", m["foo"])   // %#v 格式化输出详细信息
}
/*
references:

https://blog.csdn.net/qq_36431213/article/details/82805043
https://www.cnblogs.com/DillGao/p/7930674.html
https://haobook.readthedocs.io/zh_CN/latest/periodical/201611/zhangan.html
https://suraj.pro/post/golang_workaround/
https://blog.ijun.org/2017/07/cannot-assign-to-struct-field-in-map.html
*/

全局变量赋值的问题：
var p *int

func foo() (*int, error) {
	var i int = 5
	return &i, nil
}

func bar() {
	//use p
	fmt.Println(*p)
}

func Global_varible_assignment() {
	// 全局变量不能用 := 赋值，这种赋值属于临时变量
	// p, err := foo()
	p, err = foo()
	if err != nil {
		fmt.Println(err)
		return
	}
	bar()
	fmt.Println(*p)
}
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
//原因：在返回值处给出声明的变量那么这个变量就是被绑定的，defer中的操作也会对其造成影响。
```

## 接口问题

```go
//下面这段代码输出什么？为什么？
type People interface {
	Show()
}

type Student struct{}

func (stu *Student) Show() {

}

func main() {

	var s *Student
	if s == nil {
		fmt.Println("s is nil")
	} else {
		fmt.Println("s is not nil")
	}
	var p People = s
	if p == nil {
		fmt.Println("p is nil")
	} else {
		fmt.Println("p is not nil")
	}
}
/*参考答案及解析：s is nil 和 p is not nil。

这道题会不会有点诧异，我们分配给变量 p 的值明明是 nil，然而 p 却不是 nil。记住一点，当且仅当动态值和动态类型都为 nil 时，接口类型值才为 nil。上面的代码，给变量 p 赋值之后，p 的动态值是 nil，但是动态类型却是 *Student，是一个 nil 指针，所以相等条件不成立。*/


//下面这段代码输出什么？为什么？
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
/*
答案解析：
参考答案及解析：a、b、c 的长度和容量分别是 0 3、2 3、1 2。

知识点：数组或切片的截取操作。截取操作有带 2 个或者 3 个参数，形如：[i:j] 和 [i:j:k]，假设截取对象的底层数组长度为 l。在操作符 [i:j] 中，如果 i 省略，默认 0，如果 j 省略，默认底层数组的长度，截取得到的**切片长度和容量计算方法是 j-i、l-i**。操作符 [i:j:k]，k 主要是用来限制切片的容量，但是不能大于数组的长度 l，截取得到的**切片长度和容量计算方法是 j-i、k-i**。同python的切片一致，只是多了个容量的概念。
*/
```

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

xxxxxxxxxx pprof.Register(router)go

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

```go
//下面这段代码能否正常结束？
func main() {
	v := []int{1, 2, 3}
	for i := range v {
		v = append(v, i)
	}
}
/*
答案解析：
参考答案及解析：不会出现死循环，能正常结束。

循环次数在循环开始前就已经确定，循环内改变切片的长度，不影响循环次数。
*/

//下面这段代码输出什么？
func main() {
	var a = [5]int{1, 2, 3, 4, 5}
	var r [5]int

	for i, v := range a {
		if i == 0 {
			a[1] = 12
			a[2] = 13
		}
		r[i] = v
	}
	fmt.Println("r = ", r)
	fmt.Println("a = ", a)
}
/*
答案解析：
参考答案及解析：
    r =  [1 2 3 4 5]
    a =  [1 12 13 4 5]
range 表达式是副本参与循环，就是说例子中参与循环的是 a 的副本，而不是真正的 a。
*/


```

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
   [1 2 3 5]//这里y为什么变了，原因是y与z都是基于x的，所以在最后执行的append操作相当于修改x的第四个元素，最终导致输出的就是最后一次append操作后的数据了
   [1 2 3 5]
```

```go
//下面这段代码输出什么？为什么？
func main() {
	s1 := []int{1, 2, 3}
	s2 := s1[1:]
	s2[1] = 4
	fmt.Println(s1)
	s2 = append(s2, 5, 6, 7)
	fmt.Println(s1)
}
/*
[1 2 4]
[1 2 4]
我们已经知道，golang 中切片底层的数据结构是数组。当使用 s1[1:] 获得切片 s2，和 s1 共享同一个底层数组，这会导致 s2[1] = 4 语句影响 s1。

而 append 操作会导致底层数组扩容，生成新的数组，因此追加数据后的 s2 不会影响 s1。

但是为什么对 s2 赋值后影响的却是 s1 的第三个元素呢？这是因为切片 s2 是从数组的第二个元素开始，s2 索引为 1 的元素对应的是 s1 索引为 2 的元素。
*/


// 下面这段代码输出什么？

func change(s ...int) {
	s = append(s,3)
}

func main() {
	slice := make([]int,5,5)
	slice[0] = 1
	slice[1] = 2
	change(slice...)
	fmt.Println(slice)
	change(slice[0:2]...)
	fmt.Println(slice)
}

/*
答案解析：
参考答案及解析：

[1 2 0 0 0]
[1 2 3 0 0]
知识点：可变函数、append()操作。

Go 提供的语法糖...，可以将 slice 传进可变函数，不会创建新的切片。第一次调用 change() 时，append() 操作使切片底层数组发生了扩容，原 slice 的底层数组不会改变； 第二次调用change() 函数时，使用了操作符[i,j]获得一个新的切片，假定为 slice1，
它的底层数组和原切片底层数组是重合的，不过 slice1 的长度、容量分别是 2、5，所以在 change() 函数中对 slice1 底层数组的修改会影响到原切片。
*/
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
// defer面试题
func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	x := 1
	y := 2
    // 先执行calc("A", 1, 2)得到结果，再将calc("AA", 1, 3)入栈
	defer calc("AA", x, calc("A", x, y))
	x = 10
    // 同上calc("B", 10, 2), calc("BB", 10, 12)
	defer calc("BB", x, calc("B", x, y))
	y = 20
}
//问，上面代码的输出结果是？
// [A,1,2,3] [B,10,2,12] [BB,10,12,22] [AA,1,3,4]
"提示：defer注册要延迟执行的函数时该函数所有的参数都需要确定其值"

//经典案例
func f1() int {
	x := 5
	defer func() {
		x++
	}()
	return x
}

func f2() (x int) {
	defer func() {
		x++
	}()
	return 5
}

func f3() (y int) {
	x := 5
	defer func() {
		x++
	}()
	return x
}
func f4() (x int) {
	defer func(x int) {
		x++
	}(x)
	return 5
}
func main() {
	fmt.Println(f1()) // 5
	fmt.Println(f2()) // 6
	fmt.Println(f3()) // 5
	fmt.Println(f4()) // 5
}

//案例2
type Person struct {
	age int
}
func Test1() {
	person := &Person{28}
	// 1. 28
	defer fmt.Println(person.age)
	// 2. 29
	defer func(p *Person) {
		fmt.Println(p.age)
	}(person)
	// 3. 29
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

//return 之后的 defer 语句会执行吗，下面这段代码输出什么？

var a bool = true
func main() {
	defer func(){
		fmt.Println("1")
	}()
	if a == true {
		fmt.Println("2")
		return
	}
	defer func(){
		fmt.Println("3")
	}()
}
/*
参考答案及解析：
2
1
defer 关键字后面的函数或者方法想要执行必须先注册，return 之后的 defer 是不能注册的， 也就不能执行后面的函数或方法。
*/
```

## go的if else

```go
func main() {
	if a := 1; false {
	} else if b := 2; false {
	} else {
		println(a, b)
	}
}
// 正常运行 输出 1 2
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

# iota 的用法

```go
// 下面这段代码输出什么？
const (
	a = iota
	b
)
const (
	// iota在遇到const关键字会被重置为0
	// const里的技巧：当第一个常量被赋值后
	// 后续常量如果没有赋值，那么值同上一个一致
	name = "name"
	c    = iota
	d
)

func Iota_learn() {
	fmt.Println(a)
	fmt.Println(b)
	fmt.Println(c)
	fmt.Println(d)
} // 0 1 1 2
// iota 是 golang 语言的常量计数器，只能在常量的表达式中使用。
//iota 在 const 关键字出现时将被重置为0，const中每新增一行常量声明将使 iota 计数一次。

// 下面这段代码输出什么？
//当类型定义了String方法
// 那么在使用fmt输出时就会自动调用String
type Direction int

const (
	//  iota只需要给一个const赋值
	// 那么剩下的就会依次赋值
	North Direction = iota
	East
	South
	West
)

func (d Direction) String() string {
	return [...]string{"North", "East", "South", "West"}[d]
}

// 有个例外 就是当类型为指针类型时
// 并不会自动调用，得手动
// func (d *Direction) String() string {
// 	return [...]string{"North", "East", "South", "West"}[*d]
// }

func Iota_test1() {
	fmt.Println(South)
}
/*
参考答案及解析：South。知识点：iota 的用法、类型的 String() 方法。

根据 iota 的用法推断出 South 的值是 2；另外，如果类型定义了 String() 方法，当使用 fmt.Printf()、fmt.Print() 和 fmt.Println() 会自动使用 String() 方法，实现字符串的打印。

Reference:

https://wiki.jikexueyuan.com/project/the-way-to-go/10.7.html
https://www.sunansheng.com/archives/24.html
https://yourbasic.org/golang/iota/
*/
```

# select中管道抢占死锁问题

```go
// 以下代码的输出结果：
package main

import "sync"

func main() {
	var wg sync.WaitGroup
	foo := make(chan int)
	bar := make(chan int)
	wg.Add(1)
	go func() {
		defer wg.Done()
		select {
		case foo <- <-bar:
		default:
			println("default")
		}
	}()
	wg.Wait()
}
/*
答案解析：
这是我根据火丁笔记发的一篇文章：《一个 select 死锁问题》 进行的修改，以便更好理解。

按常规理解，go func 中的 select 应该执行 default 分支，程序正常运行。但结果却不是，而是死锁。可以通过该链接测试：https://play.studygolang.com/p/kF4pOjYXbXf。

原因文章也解释了，Go 语言规范中有这么一句：

For all the cases in the statement, the channel operands of receive operations and the channel and right-hand-side expressions of send statements are evaluated exactly once, in source order, upon entering the “select” statement. The result is a set of channels to receive from or send to, and the corresponding values to send. Any side effects in that evaluation will occur irrespective of which (if any) communication operation is selected to proceed. Expressions on the left-hand side of a RecvStmt with a short variable declaration or assignment are not yet evaluated.

不知道大家看懂没有？于是，最后来了一个例子验证你是否理解了：为什么每次都是输出一半数据，然后死锁？（同样，这里可以运行查看结果：https://play.studygolang.com/p/zoJtTzI7K5T）
*/
package main

import (
	"fmt"
	"time"
)

func talk(msg string, sleep int) <-chan string {
	ch := make(chan string)
	go func() {
		for i := 0; i < 5; i++ {
			ch <- fmt.Sprintf("%s %d", msg, i)
			time.Sleep(time.Duration(sleep) * time.Millisecond)
		}
	}()
	return ch
}

func fanIn(input1, input2 <-chan string) <-chan string {
	ch := make(chan string)
	go func() {
		for {
			select {
			case ch <- <-input1:
			case ch <- <-input2:
			}
		}
	}()
	return ch
}

func main() {
	ch := fanIn(talk("A", 10), talk("B", 1000))
	for i := 0; i < 10; i++ {
		fmt.Printf("%q\n", <-ch)
	}
}
/*
这是 StackOverflow 上的一个问题：https://stackoverflow.com/questions/51167940/chained-channel-operations-in-a-single-select-case。

关键点和文章开头例子一样，在于 select case 中两个 channel 串起来，即 fanIn 函数中：
*/
select {
case ch <- <-input1:
case ch <- <-input2:
}

// 如果改为这样就一切正常：
select {
case t := <-input1:
  ch <- t
case t := <-input2:
  ch <- t
}
结合这个更复杂的例子分析 Go 语言规范中的那句话。

对于 select 语句，在进入该语句时，会按源码的顺序对每一个 case 子句进行求值：这个求值只针对发送或接收操作的额外表达式。

比如：

// ch 是一个 chan int；
// getVal() 返回 int
// input 是 chan int
// getch() 返回 chan int
select {
  case ch <- getVal():
  case ch <- <-input:
  case getch() <- 1:
  case <- getch():
}
//在没有选择某个具体 case 执行前，例子中的 getVal()、<-input 和 getch() 会执行。这里有一个验证的例子：https://play.studygolang.com/p/DkpCq3aQ1TE。

package main

import (
	"fmt"
)

func main() {
	ch := make(chan int)
	go func() {
		select {
		case ch <- getVal(1):
			fmt.Println("in first case")
		case ch <- getVal(2):
			fmt.Println("in second case")
		default:
			fmt.Println("default")
		}
	}()

	fmt.Println("The val:", <-ch)
}

func getVal(i int) int {
	fmt.Println("getVal, i=", i)
	return i
}
无论 select 最终选择了哪个 case，getVal() 都会按照源码顺序执行：getVal(1) 和 getVal(2)，也就是它们必然先输出：

getVal, i= 1
getVal, i= 2
你可以仔细琢磨一下。

现在回到 StackOverflow 上的那个问题。

每次进入以下 select 语句时：

select {
case ch <- <-input1:
case ch <- <-input2:
}
<-input1 和 <-input2 都会执行，相应的值是：A x 和 B x（其中 x 是 0-5）。但每次 select 只会选择其中一个 case 执行，所以 <-input1 和 <-input2 的结果，必然有一个被丢弃了，也就是不会被写入 ch 中。因此，一共只会输出 5 次，另外 5 次结果丢掉了。（你会发现，输出的 5 次结果中，x 比如是 0 1 2 3 4）

而 main 中循环 10 次，只获得 5 次结果，所以输出 5 次后，报死锁。

虽然这是一个小细节，但实际开发中还是有可能出现的。比如文章提到的例子写法：

// ch 是一个 chan int；
// getVal() 返回 int
// input 是 chan int
// getch() 返回 chan int
select {
  case ch <- getVal():
  case ch <- <-input:
  case getch() <- 1:
  case <- getch():
}
因此在使用 select 时，一定要注意这种可能的问题。

不要以为这个问题不会遇到，其实很常见。最多的就是 time.After 导致内存泄露问题，网上有很多文章解释原因，如何避免，其实最根本原因就是因为 select 这个机制导致的。

比如如下代码，有内存泄露（传递给 time.After 的时间参数越大，泄露会越厉害），你能解释原因吗？

package main

import (
    "time"
)

func main()  {
    ch := make(chan int, 10)

    go func() {
        var i = 1
        for {
            i++
            ch <- i
        }
    }()

    for {
        select {
        case x := <- ch:
            println(x)
        case <- time.After(30 * time.Second):
            println(time.Now().Unix())
        }
    }
}
/*
答案解析:
原因是在for+select+time处理的情境下,如果 case <- time.After(30 * time.Second) 这样创建定时任务，会因为for循环每次调用select都会重新初始化一个全新的计时器（Timer）,从而导致一直在不停创建任务，而任务也不能被释放，最终造成内存溢出。
解决办法：
在for循环前可以提前声明定时任务，并保存在变量中，case语句直接调用就好。
*/
```
