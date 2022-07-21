# go fmt库

### 通用占位符

| 占位符 |                说明                |
| :----: | :--------------------------------: |
|   %v   |          值的默认格式表示          |
|  %+v   | 类似%v，但输出结构体时会添加字段名 |
|  %#v   |           值的Go语法表示           |
|   %T   |            打印值的类型            |
|   %%   |               百分号               |

### 布尔型

| 占位符 |    说明     |
| :----: | :---------: |
|   %t   | true或false |

### 整型

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|   %b   |                         表示为二进制                         |
|   %c   |                    该值对应的unicode码值                     |
|   %d   |                         表示为十进制                         |
|   %o   |                         表示为八进制                         |
|   %x   |                   表示为十六进制，使用a-f                    |
|   %X   |                   表示为十六进制，使用A-F                    |
|   %U   |          表示为Unicode格式：U+1234，等价于”U+%04X”           |
|   %q   | 该值对应的单引号括起来的go语法字符字面值，必要时会采用安全的转义表示 |

### 浮点数与复数

| 占位符 |                          说明                          |
| :----: | :----------------------------------------------------: |
|   %b   |   无小数部分、二进制指数的科学计数法，如-123456p-78    |
|   %e   |              科学计数法，如-1234.456e+78               |
|   %E   |              科学计数法，如-1234.456E+78               |
|   %f   |           有小数部分但无指数部分，如123.456            |
|   %F   |                        等价于%f                        |
|   %g   | 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出） |
|   %G   | 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出） |

### 字符串和[]byte

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|   %s   |                   直接输出字符串或者[]byte                   |
|   %q   | 该值对应的双引号括起来的go语法字符串字面值，必要时会采用安全的转义表示 |
|   %x   |           每个字节用两字符十六进制数表示（使用a-f            |
|   %X   |          每个字节用两字符十六进制数表示（使用A-F）           |

### 指针

| 占位符 |              说明              |
| :----: | :----------------------------: |
|   %p   | 表示为十六进制，并加上前导的0x |

### 宽度标识符

宽度通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。精度通过（可选的）宽度后跟点号后跟的十进制数指定。如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为0。举例如下：

| 占位符 |        说明        |
| :----: | :----------------: |
|   %f   | 默认宽度，默认精度 |
|  %9f   |  宽度9，默认精度   |
|  %.2f  |  默认宽度，精度2   |
| %9.2f  |    宽度9，精度2    |
|  %9.f  |    宽度9，精度0    |

### 其他flag

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|  ’+’   | 总是输出数值的正负号；对%q（%+q）会生成全部是ASCII字符的输出（通过转义）； |
|  ’ ‘   | 对数值，正数前加空格而负数前加负号；对字符串采用%x或%X时（% x或% X）会给各打印的字节之间加空格 |
|  ’-’   | 在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）； |
|  ’#’   | 八进制数前加0（%#o），十六进制数前加0x（%#x）或0X（%#X），指针去掉前面的0x（%#p）对%q（%#q），对%U（%#U）会输出空格和单引号括起来的go字面值； |
|  ‘0’   | 使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面； |



# go defer语句

defer一般用于资源的释放和异常的捕捉，defer后的语句会在程序进行最后的return后再执行。

在 defer 归属的函数即将返回时，将延迟处理的语句按 defer 的逆序进行执行，也就是说，先被 defer 的语句最后被执行，最后被 defer 的语句，最先被执行。

```go
func CopyFile(dstName, srcName string) (written int64, err error){
	src, err := os.Open(srcName)
	if err != nil{
		return
	}
    dst, err := os.Creat(dstName)
    if err != nil{
        return
    }
    dst.Close()
    src.Close()
    return
}
```

以上代码中，如果后一个资源在打开后出现异常，那么前面所有打开的资源都没有办法关闭，所以最好在后面的异常处理中，对前面的资源进行释放。

但是如果所有的异常处理中都需要加上这个重复的释放资源代码，会很繁琐。

这个时候就用到了defer关键字。可以在每个占用的资源后面接上释放资源的代码并使用defer，这样最终都会将这些资源释放掉。

```go
src, err := os.Open(srcName)
defer src.Close()
if err != nil{
    return
}
```

# go的flag库



Go语言内置的`flag`包实现了命令行参数的解析，`flag`包使得开发命令行工具更为简单。

## os.Args

如果你只是简单的想要获取命令行参数，可以像下面的代码示例一样使用`os.Args`来获取命令行参数。

```go
//os.Args demo
func main() {
	//os.Args是一个[]string
	if len(os.Args) > 0 {
		for index, arg := range os.Args {
			fmt.Printf("args[%d]=%v\n", index, arg)
		}
	}
}
```

将上面的代码执行`go build -o "args_demo"`编译之后，执行：

```bash
$ ./args_demo a b c d
args[0]=./args_demo
args[1]=a
args[2]=b
args[3]=c
args[4]=d
```

`os.Args`是一个存储命令行参数的字符串切片，它的第一个元素是执行文件的名称。

# 异常的捕捉

go 并没有 try...catch模块，那是怎么进行异常捕捉和处理的呢？

go会出现panic异常，当出现这个异常后程序就会暂停了。

```go
func executePanic(){
	defer fmt.Println("defer func")
	panic("This is Panic Situation")
	fmt.Println("The function executes Completely")
}
func main(){
	executePanic()
	fmt.Println("Main block is executed completely...")
}
```

上述代码中，使用了手动执行panic函数来触发异常。当异常出现后，函数还是会调用defer中的函数，然后异常退出。上述代码输出如下：

```go
defer func
panic: This is Panic Situation 
程序的错误信息
```

但是如果defer后加上recover函数，程序就能正常运行，main函数能正常运行后退出。

```go
func executePanic(){
	defer func(){
		if errMsg := recover();errMsg != nil{
			fmt.Println(errMsg)
		}
	}()
	panic("This is Panic Situation")
	fmt.Println("The function executes Completely")
}
func main(){
	executePanic()
	fmt.Println("Main block is executed completely...")
}
```

此时我们在executePanic中用recover，那么挂掉的只是executePanic，他不会抛到main中，所以main能继续运行，继而main开辟的其他函数也能继续运行。

