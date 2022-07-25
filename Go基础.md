# Go 并发

Go 语言支持并发，我们只需要通过 go 关键字来开启 goroutine 即可。

在并发过程中，会用到channel数据结构，它是一种环形队列结构（目前得知）

定义方法为：c chan [数据类型]

定义并分配内存方法：ch = make(chan int) -------无缓冲区通道

​									ch = make(chan int，0) -------无缓冲区通道

​									ch = make(chan int，3) -------缓冲区长度为3的通道

区别：无缓冲区的通道只能接收一个发送，当通道内有元素时，通道会阻塞，直到元素被接收。有缓冲区的通道可以接收最多缓冲区长度的元素，超出也会阻塞，直到元素被接收。



运用方法：ch <- v    // 把 v 发送到通道 ch
		  v := <-ch  // 从 ch 接收数据
					 // 并把值赋给 v

**底层结构:**

```go
type hchan struct {
    qcount   uint           // channel中的元素个数
    dataqsiz uint           // channel中循环队列的长度
    buf      unsafe.Pointer // channel缓冲区数据指针
    elemsize uint16            // buffer中每个元素的大小
    closed   uint32            // channel是否已经关闭，0未关闭
    elemtype *_type // channel中的元素的类型
    sendx    uint   // channel发送操作处理到的位置
    recvx    uint   // channel接收操作处理到的位置
    recvq    waitq  // 等待接收的sudog（sudog为封装了goroutine和数据的结构）队列由于缓冲区空间不足而阻塞的Goroutine列表
    sendq    waitq  // 等待发送的sudogo队列，由于缓冲区空间不足而阻塞的Goroutine列表

    lock mutex   // 一个轻量级锁
}
```

**channel创建:**

```go
ch := make(chan int, 3)
```

- 创建channel实际上就是在内存中实例化了一个***hchan\***结构体，并返回一个chan指针
- channle在函数间传递都是使用的这个指针，这就是为什么函数传递中无需使用channel的指针，而是直接用channel就行了，因为channel本身就是一个指针

**channel发送数据：**

```go
ch <- 1
ch <- 2
```

- 检查 recvq 是否为空，如果不为空，则从 recvq 头部取一个 goroutine，将数据发送过去，并唤醒对应的 goroutine 即可。
- 如果 recvq 为空，则将数据放入到 buffer 中。
- 如果 buffer 已满，则将要发送的数据和当前 goroutine 打包成 sudog 对象放入到 sendq中。并将当前 goroutine 置为 waiting 状态。

**channel接收数据：**

```go
<-ch
<-ch
```

- 检查sendq是否为空，如果不为空，且没有缓冲区，则从sendq头部取一个goroutine，将数据读取出来，并唤醒对应的goroutine，结束读取过程。
- 如果sendq不为空，且有缓冲区，则说明缓冲区已满，则从缓冲区中首部读出数据，把sendq头部的goroutine数据写入缓冲区尾部，并将goroutine唤醒，结束读取过程。
- 如果sendq为空，缓冲区有数据，则直接从缓冲区读取数据，结束读取过程。
- 如果sendq为空，且缓冲区没数据，则只能将当前的goroutine加入到recvq,并进入waiting状态，等待被写goroutine唤醒。

**channel规则：**

| 操作        | 空channel | 已关闭channel | 活跃中的channel |
| :---------- | :-------- | :------------ | :-------------- |
| close(ch)   | panic     | panic         | 成功关闭        |
| ch<- v      | 永远阻塞  | panic         | 成功发送或阻塞  |
| v,ok = <-ch | 永远阻塞  | 不阻塞        | 成功接收或阻塞  |

# Go语言中的指针

任何程序数据载入内存后，在内存都有他们的地址，这就是指针。而为了保存一个数据在内存中的地址，我们就需要指针变量。

比如，“永远不要高估自己”这句话是我的座右铭，我想把它写入程序中，程序一启动这句话是要加载到内存（假设内存地址0x123456），我在程序中把这段话赋值给变量`A`，把内存地址赋值给变量`B`。这时候变量`B`就是一个指针变量。通过变量`A`和变量`B`都能找到我的座右铭。

Go语言中的指针不能进行偏移和运算，因此Go语言中的指针操作非常简单，我们只需要记住两个符号：`&`（取地址）和`*`（根据地址取值）。

## 指针地址和指针类型

每个变量在运行时都拥有一个地址，这个地址代表变量在内存中的位置。Go语言中使用`&`字符放在变量前面对变量进行“取地址”操作。 Go语言中的值类型（int、float、bool、string、array、struct）都有对应的指针类型，如：`*int`、`*int64`、`*string`等。

取变量指针的语法如下：

```go
ptr := &v    // v的类型为T
```

其中：

- v:代表被取地址的变量，类型为`T`
- ptr:用于接收地址的变量，ptr的类型就为`*T`，称做T的指针类型。*代表指针。

## new和make

指针类型变量在使用赋值前需要先分配内存，如果只是声明而没有进行分配内存，会引起程序panic，如下例子：

```go
func main() {
	var a *int
	*a = 100  // 这句会造成程序panic
	fmt.Println(*a)

	var b map[string]int
	b["沙河娜扎"] = 100
	fmt.Println(b)
}
```

所以这时需要使用new或make来声明了。

## NEW

new函数不太常用，使用new函数得到的是一个类型的指针，并且该指针对应的值为该类型的零值。举个例子：

```go
func main() {
	a := new(int)
	b := new(bool)
	fmt.Printf("%T\n", a) // *int
	fmt.Printf("%T\n", b) // *bool
	fmt.Println(*a)       // 0
	fmt.Println(*b)       // false
}	
```

上述案例中的改写方法为：

```go
func main() {
	var a *int
	a = new(int)
	*a = 10
	fmt.Println(*a)
}
```

## make

make也是用于内存分配的，区别于new，它只用于slice、map以及chan的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了。

make函数是无可替代的，我们在使用slice、map以及channel的时候，都需要使用make进行初始化，然后才可以对它们进行操作。

上述案例中的改写方法为：

```go
func main() {
	var b map[string]int
	b = make(map[string]int, 10)
	b["沙河娜扎"] = 100
	fmt.Println(b)
}
```

### new与make的区别

1. 二者都是用来做内存分配的。
2. make只用于slice、map以及channel的初始化，返回的还是这三个引用类型本身；
3. 而new用于类型的内存分配，并且内存对应的值为类型零值，返回的是指向类型的指针。

```go
//评论区的小插曲
func main(){	
	e := 10
	b := &e
	fmt.Printf("e:%d prt:%d \n", e, &e)
	fmt.Printf("b:%p type:%T \n", b, b)
	fmt.Println(&b)//这里看到的是变量b的内存地址，而不是b里存的地址
	fmt.Println(&e)
}
```

# map

map是一种无序的基于`key-value`的数据结构，Go语言中的map是引用类型，必须初始化才能使用。

## map定义

Go语言中 `map`的定义语法如下：

```go
map[KeyType]ValueType
```

其中，

- KeyType:表示键的类型。
- ValueType:表示键对应的值的类型。

map类型的变量默认初始值为nil，需要使用make()函数来分配内存。语法为：

```go
make(map[KeyType]ValueType, [cap])
```

其中cap表示map的容量，该参数虽然不是必须的，但是我们应该在初始化map的时候就为其指定一个合适的容量。

## map基本使用

map中的数据都是成对出现的，map的基本使用示例代码如下：

```go
func main() {
	scoreMap := make(map[string]int, 8)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	fmt.Println(scoreMap)
	fmt.Println(scoreMap["小明"])
	fmt.Printf("type of a:%T\n", scoreMap)
}
```

输出：

```bash
map[小明:100 张三:90]
100
type of a:map[string]int
```

map也支持在声明的时候填充元素，例如：

```go
func main() {
	userInfo := map[string]string{
		"username": "武汉小王子",
		"password": "123456",
	}
	fmt.Println(userInfo) //
}
```

## 判断某个键是否存在

Go语言中有个判断map中键是否存在的特殊写法，格式如下:

```go
value, ok := map[key]
```

## map的遍历

Go语言中使用`for range`遍历map。

```go
func main() {
	scoreMap := make(map[string]int)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	scoreMap["娜扎"] = 60
	for k, v := range scoreMap {
		fmt.Println(k, v)
	}
}
```

但我们只想遍历key的时候，可以按下面的写法：

```go
func main() {
	scoreMap := make(map[string]int)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	scoreMap["娜扎"] = 60
	for k := range scoreMap {
		fmt.Println(k)
	}
}
```

**注意：** 遍历map时的元素顺序与添加键值对的顺序无关。

## 使用delete()函数删除键值对

使用`delete()`内建函数从map中删除一组键值对，`delete()`函数的格式如下：

```go
delete(map, key)
```

其中，

- map:表示要删除键值对的map
- key:表示要删除的键值对的键

## 按照指定顺序遍历map

```go
func main() {
	rand.Seed(time.Now().UnixNano()) //初始化随机数种子

	var scoreMap = make(map[string]int, 200)

	for i := 0; i < 100; i++ {
		key := fmt.Sprintf("stu%02d", i) //生成stu开头的字符串
		value := rand.Intn(100)          //生成0~99的随机整数
		scoreMap[key] = value
	}
	//取出map中的所有key存入切片keys
	var keys = make([]string, 0, 200)
	for key := range scoreMap {
		keys = append(keys, key)
	}
	//对切片进行排序
	sort.Strings(keys)
	//按照排序后的key遍历map
	for _, key := range keys {
		fmt.Println(key, scoreMap[key])
	}
}
```

## 元素为map类型的切片

下面的代码演示了切片中的元素为map类型时的操作：

```go
func main() {
	var mapSlice = make([]map[string]string, 3)
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v\n", index, value)
	}
	fmt.Println("after init")
	// 对切片中的map元素进行初始化
	mapSlice[0] = make(map[string]string, 10)
	mapSlice[0]["name"] = "小王子"
	mapSlice[0]["password"] = "123456"
	mapSlice[0]["address"] = "武汉"
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v\n", index, value)
	}
}
```

## 值为切片类型的map

下面的代码演示了map中值为切片类型的操作：

```go
func main() {
	var sliceMap = make(map[string][]string, 3)
	fmt.Println(sliceMap)
	fmt.Println("after init")
	key := "中国"
	value, ok := sliceMap[key]
	if !ok {
		value = make([]string, 0, 2)
	}
	value = append(value, "北京", "上海")
	sliceMap[key] = value
	fmt.Println(sliceMap)
}
```

# 函数

Go语言中支持函数、匿名函数和闭包，并且函数在Go语言中属于“一等公民”。

## 函数定义

Go语言中定义函数使用`func`关键字，具体格式如下：

```go
func 函数名(参数)(返回值){
    函数体
}
```

其中：

- 函数名：由字母、数字、下划线组成。但函数名的第一个字母不能是数字。在同一个包内，函数名也称不能重名（包的概念详见后文）。
- 参数：参数由参数变量和参数变量的类型组成，多个参数之间使用`,`分隔。
- 返回值：返回值由返回值变量和其变量类型组成，也可以只写返回值的类型，多个返回值必须用`()`包裹，并用`,`分隔。
- 函数体：实现指定功能的代码块。

我们先来定义一个求两个数之和的函数：

```go
func intSum(x int, y int) int {
	return x + y
}
```

函数的参数和返回值都是可选的，例如我们可以实现一个既不需要参数也没有返回值的函数：

```go
func sayHello() {
	fmt.Println("Hello 沙河")
}
```

## 函数的调用

定义了函数之后，我们可以通过`函数名()`的方式调用函数。 例如我们调用上面定义的两个函数，代码如下：

```go
func main() {
	sayHello()
	ret := intSum(10, 20)
	fmt.Println(ret)
}
```

注意，调用有返回值的函数时，可以不接收其返回值。

## 参数

### 类型简写

函数的参数中如果相邻变量的类型相同，则可以省略类型，例如：

```go
func intSum(x, y int) int {
	return x + y
}
```

上面的代码中，`intSum`函数有两个参数，这两个参数的类型均为`int`，因此可以省略`x`的类型，因为`y`后面有类型说明，`x`参数也是该类型。

### 可变参数

可变参数是指函数的参数数量不固定。Go语言中的可变参数通过在参数名后加`...`来标识。

注意：可变参数通常要作为函数的最后一个参数。

举个例子：

```go
func intSum2(x ...int) int {
	fmt.Println(x) //x是一个切片
	sum := 0
	for _, v := range x {
		sum = sum + v
	}
	return sum
}
```

调用上面的函数：

```go
ret1 := intSum2()
ret2 := intSum2(10)
ret3 := intSum2(10, 20)
ret4 := intSum2(10, 20, 30)
fmt.Println(ret1, ret2, ret3, ret4) //0 10 30 60
```

固定参数搭配可变参数使用时，可变参数要放在固定参数的后面，示例代码如下：

```go
func intSum3(x int, y ...int) int {
	fmt.Println(x, y)
	sum := x
	for _, v := range y {
		sum = sum + v
	}
	return sum
}
```

调用上述函数：

```go
ret5 := intSum3(100)
ret6 := intSum3(100, 10)
ret7 := intSum3(100, 10, 20)
ret8 := intSum3(100, 10, 20, 30)
fmt.Println(ret5, ret6, ret7, ret8) //100 110 130 160
```

本质上，函数的可变参数是通过切片来实现的。

## 返回值

Go语言中通过`return`关键字向外输出返回值。

### 多返回值

Go语言中函数支持多返回值，函数如果有多个返回值时必须用`()`将所有返回值包裹起来。

举个例子：

```go
func calc(x, y int) (int, int) {
	sum := x + y
	sub := x - y
	return sum, sub
}
```

### 返回值命名

函数定义时可以给返回值命名，并在函数体中直接使用这些变量，最后通过`return`关键字返回。

例如：

```go
func calc(x, y int) (sum, sub int) {
	sum = x + y
	sub = x - y
	return
}
```

### 返回值补充

当我们的一个函数返回值类型为slice时，nil可以看做是一个有效的slice，没必要显示返回一个长度为0的切片。

```go
func someFunc(x string) []int {
	if x == "" {
		return nil // 没必要返回[]int{}
	}
	...
}
```