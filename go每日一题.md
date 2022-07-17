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

## 切片 a、b、c 的长度和容量分别是多少？

```go
func main() {
	s := [3]int{1, 2, 3}
	a := s[:0]
	b := s[:2]
	c := s[1:2:cap(s)]
}
```

### 答案解析：

参考答案及解析：a、b、c 的长度和容量分别是 0 3、2 3、1 2。

知识点：数组或切片的截取操作。截取操作有带 2 个或者 3 个参数，形如：[i:j] 和 [i:j:k]，假设截取对象的底层数组长度为 l。在操作符 [i:j] 中，如果 i 省略，默认 0，如果 j 省略，默认底层数组的长度，截取得到的**切片长度和容量计算方法是 j-i、l-i**。操作符 [i:j:k]，k 主要是用来限制切片的容量，但是不能大于数组的长度 l，截取得到的**切片长度和容量计算方法是 j-i、k-i**。同python的切片一致，只是多了个容量的概念。

## 切片扩容-append 函数

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

