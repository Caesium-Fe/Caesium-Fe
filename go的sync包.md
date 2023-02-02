https://blog.csdn.net/luoye4321/article/details/82433144

一. 前言
Golang sync包提供了基础的异步操作方法，包括互斥锁Mutex，执行一次Once和并发等待组WaitGroup。
本文主要介绍sync包提供的这些功能的基本使用方法。

Mutex: 互斥锁
RWMutex：读写锁
WaitGroup：并发等待组
Once：执行一次
Cond：信号量
Pool：临时对象池
Map：自带锁的map
二. sync.Mutex
sync.Mutex称为互斥锁，常用在并发编程里面。互斥锁需要保证的是同一个时间段内不能有多个并发协程同时访问某一个资源(临界区)。
sync.Mutex有2个函数Lock和UnLock分别表示获得锁和释放锁。

```go
func (m *Mutex) Lock()
func (m *Mutex) UnLock()
```

sync.Mutex初始值为UnLock状态，并且sync.Mutex常做为其它结构体的匿名变量使用。

举个例子: 我们经常使用网上支付购物东西，就会出现同一个银行账户在某一个时间既有支出也有收入，那银行就得保证我们余额准确，保证数据无误。
我们可以简单的实现银行的支出和收入来说明Mutex的使用

```go
type Bank struct {
	sync.Mutex
	balance map[string]float64
}

// In 收入
func (b *Bank) In(account string, value float64) {
        // 加锁 保证同一时间只有一个协程能访问这段代码
	b.Lock()
	defer b.Unlock()

	v, ok := b.balance[account]
	if !ok {
		b.balance[account] = 0.0
	}
	
	b.balance[account] += v

}

// Out 支出
func (b *Bank) Out(account string, value float64) error {
        // 加锁 保证同一时间只有一个协程能访问这段代码
	b.Lock()
	defer b.Unlock()

	v, ok := b.balance[account]
	if !ok || v < value {
		return errors.New("account not enough balance")
	}
	
	b.balance[account] -= value
	return nil

}
```

三. sync.RWMutex
sync.RWMutex称为读写锁是sync.Mutex的一种变种，RWMutex来自于计算机操作系统非常有名的读者写者问题。
sync.RWMutex目的是为了能够支持多个并发协程同时读取某一个资源，但只有一个并发协程能够更新资源。也就是说读和写是互斥的，写和写也是互斥的，读和读是不互斥的。

总结起来如下

当有一个协程在读的时候，所有写的协程必须等到所有读的协程结束才可以获得锁进行写操作。
当有一个协程在读的时候，所有读的协程不受影响都可以进行读操作。
当有一个协程在写的时候，所有读、写的协程必须等到写的协程结束才可以获得锁进行读、写操作。
RWMutex有5个函数，分别为读和写提供锁操作

```go
写操作
func (rw *RWMutex) Lock()
func (rw *RWMutex) Unlock()

读操作
func (rw *RWMutex) RLock()
func (rw *RWMutex) RUnlock()

RLocker()能获取读锁，然后传递给其他协程使用。
func (rw *RWMutex) RLocker() Locker
```

举个例子，sync.Mutex一节例子里面我们没有提供查询操作，如果用Mutex互斥锁就没有办法支持多人同时查询，所以我们使用sync.RWMutex来改写这个代码

```go
type Bank struct {
	sync.RWMutex
	balance map[string]float64
}

func (b *Bank) In(account string, value float64) {
	b.Lock()
	defer b.Unlock()

	v, ok := b.balance[account]
	if !ok {
		b.balance[account] = 0.0
	}
	
	b.balance[account] += v

}

func (b *Bank) Out(account string, value float64) error {
	b.Lock()
	defer b.Unlock()

	v, ok := b.balance[account]
	if !ok || v < value {
		return errors.New("account not enough balance")
	}
	
	b.balance[account] -= value
	return nil

}

func (b *Bank) Query(account string) float64 {
	b.RLock()
	defer b.RUnlock()

	v, ok := b.balance[account]
	if !ok {
		return 0.0
	}
	
	return v

}
```

xxxxxxxxxx package main​import (    "fmt"​    "github.com/Shopify/sarama")​// kafka consumer​func main() {    consumer, err := sarama.NewConsumer([]string{"127.0.0.1:9092"}, nil)    if err != nil {        fmt.Printf("fail to start consumer, err:%v\n", err)        return    }    partitionList, err := consumer.Partitions("web_log") // 根据topic取到所有的分区    if err != nil {        fmt.Printf("fail to get list of partition:err%v\n", err)        return    }    fmt.Println(partitionList)    for partition := range partitionList { // 遍历所有的分区        // 针对每个分区创建一个对应的分区消费者        pc, err := consumer.ConsumePartition("web_log", int32(partition), sarama.OffsetNewest)        if err != nil {            fmt.Printf("failed to start consumer for partition %d,err:%v\n", partition, err)            return        }        defer pc.AsyncClose()        // 异步从每个分区消费信息        go func(sarama.PartitionConsumer) {            for msg := range pc.Messages() {                fmt.Printf("Partition:%d Offset:%d Key:%v Value:%v", msg.Partition, msg.Offset, msg.Key, msg.Value)            }        }(pc)    }}go

sync.WaitGroup有3个函数

```go
func (wg *WaitGroup) Add(delta int)  Add添加n个并发协程
func (wg *WaitGroup) Done()  Done完成一个并发协程
func (wg *WaitGroup) Wait()  Wait等待其它并发协程结束
```

sync.WaitGroup在Golang编程里面最常用于协程池，下面这个例子会同时启动1000个并发协程。

```go
func main() {
     wg := &sync.WaitGroup{}
     for i := 0; i < 1000; i++ {
         wg.Add(1)
         go func() {
	     defer func() {
		wg.Done()
	     }()
	     time.Sleep(1 * time.Second)
	     fmt.Println("hello world ~")
	 }()
     }
     // 等待所有协程结束
     wg.Wait()
     fmt.Println("WaitGroup all process done ~")
}
```

sync.WaitGroup没有办法指定最大并发协程数，在一些场景下会有问题。例如操作数据库场景下，我们不希望某一些时刻出现大量连接数据库导致数据库不可访问。所以，为了能够控制最大的并发数，推荐使用github.com/remeh/sizedwaitgroup，用法和sync.WaitGroup非常类似。

下面这个例子最多只有10个并发协程，如果已经达到10个并发协程，只有某一个协程执行了Done才能启动一个新的协程。

```go
import  "github.com/remeh/sizedwaitgroup"

func main() {
     //最大10个并发
     wg := sizedwaitgroup.New(10)
     for i = 0; i < 1000; i++ {
         wg.Add()
         go func() {
	     defer func() {
		wg.Done()
	     }()
	     time.Sleep(1 * time.Second)
	     fmt.Println("hello world ~")
	 }()
     }
     // 等待所有协程结束
     wg.Wait()
     fmt.Println("WaitGroup all process done ~")
}
```

五. sync.Once
sync.Once指的是只执行一次的对象实现，常用来控制某些函数只能被调用一次。sync.Once的使用场景例如单例模式、系统初始化。
例如并发情况下多次调用channel的close会导致panic，解决这个问题我们可以使用sync.Once来保证close只会被执行一次。

sync.Once的结构如下所示，只有一个函数。使用变量done来记录函数的执行状态，使用sync.Mutex和sync.atomic来保证线程安全的读取done。

```go
type Once struct {
	m    Mutex     #互斥锁
	done uint32    #执行状态
}

func (o *Once) Do(f func())
```

举个例子，1000个并发协程情况下只有一个协程会执行到fmt.Printf，多次执行的情况下输出的内容还不一样，因为这取决于哪个协程先调用到该匿名函数。

```go
func main() {
     once := &sync.Once{}

     for i := 0; i < 1000; i++ {
     go func(idx int) {
        once.Do(func() {
    	time.Sleep(1 * time.Second)
    	fmt.Printf("hello world index: %d", idx)
        })
     }(i)
     }
    
     time.Sleep(5 * time.Second)

}
```

六. sync.Cond
sync.Cond指的是同步条件变量，一般需要与互斥锁组合使用，本质上是一些正在等待某个条件的协程的同步机制。

```go
// NewCond returns a new Cond with Locker l.
func NewCond(l Locker) *Cond {
    return &Cond{L: l}
}

// A Locker represents an object that can be locked and unlocked.
type Locker interface {
    Lock()
    Unlock()
}
```

sync.Cond有3个函数Wait、Signal、Broadcast

```go
// Wait 等待通知
func (c *Cond) Wait()
// Signal 单播通知
func (c *Cond) Signal()
// Broadcast 广播通知
func (c *Cond) Broadcast()
```

举个例子，sync.Cond用于并发协程条件变量。

```go
var sharedRsc = make(map[string]interface{})
func main() {
    var wg sync.WaitGroup
    wg.Add(2)
    m := sync.Mutex{}
    c := sync.NewCond(&m)
    

    go func() {
        // this go routine wait for changes to the sharedRsc
        c.L.Lock()
        for len(sharedRsc) == 0 {
            c.Wait()
        }
        fmt.Println(sharedRsc["rsc1"])
        c.L.Unlock()
        wg.Done()
    }()
    
    go func() {
        // this go routine wait for changes to the sharedRsc
        c.L.Lock()
        for len(sharedRsc) == 0 {
            c.Wait()
        }
        fmt.Println(sharedRsc["rsc2"])
        c.L.Unlock()
        wg.Done()
    }()
    
    // this one writes changes to sharedRsc
    c.L.Lock()
    sharedRsc["rsc1"] = "foo"
    sharedRsc["rsc2"] = "bar"
    c.Broadcast()
    c.L.Unlock()
    wg.Wait()

}
```

七. sync.Pool
sync.Pool指的是临时对象池，Golang和Java具有GC机制，因此很多开发者基本上都不会考虑内存回收问题，不像C++很多时候开发需要自己回收对象。
Gc是一把双刃剑，带来了编程的方便但同时也增加了运行时开销，使用不当可能会严重影响程序的性能，因此性能要求高的场景不能任意产生太多的垃圾。
sync.Pool正是用来解决这类问题的，Pool可以作为临时对象池来使用，不再自己单独创建对象，而是从临时对象池中获取出一个对象。

sync.Pool有2个函数Get和Put，Get负责从临时对象池中取出一个对象，Put用于结束的时候把对象放回临时对象池中。

```go
func (p *Pool) Get() interface{}
func (p *Pool) Put(x interface{})
```

看一个官方例子

```go
var bufPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func timeNow() time.Time {
    return time.Unix(1136214245, 0)
}

func Log(w io.Writer, key, val string) {
    // 获取临时对象，没有的话会自动创建
    b := bufPool.Get().(*bytes.Buffer)
    b.Reset()
    b.WriteString(timeNow().UTC().Format(time.RFC3339))
    b.WriteByte(' ')
    b.WriteString(key)
    b.WriteByte('=')
    b.WriteString(val)
    w.Write(b.Bytes())
    // 将临时对象放回到 Pool 中
    bufPool.Put(b)
}

func main() {
    Log(os.Stdout, "path", "/search?q=flowers")
}
```

从上面的例子我们可以看到创建一个Pool对象并不能指定大小，所以sync.Pool的缓存对象数量是没有限制的(只受限于内存)，那sync.Pool是如何控制缓存临时对象数的呢？
sync.Pool在init的时候注册了一个poolCleanup函数，它会清除所有的pool里面的所有缓存的对象，该函数注册进去之后会在每次Gc之前都会调用，因此sync.Pool缓存的期限只是两次Gc之间这段时间。正因Gc的时候会清掉缓存对象，所以不用担心pool会无限增大的问题。

正因为如此sync.Pool适合用于缓存临时对象，而不适合用来做持久保存的对象池(连接池等)。

八. sync.Map
Go在1.9版本之前自带的map对象是不具有并发安全的，很多时候我们都得自己封装支持并发安全的Map结构，如下所示给map加个读写锁sync.RWMutex。

```go
type MapWithLock struct {
    sync.RWMutex
    M map[string]Kline
}
```

Go1.9版本新增了sync.Map它是原生支持并发安全的map，sync.Map封装了更为复杂的数据结构实现了比之前加读写锁锁map更优秀的性能。

sync.Map总共5个函数，用法和原生的map差点很多

```go
// 查询一个key
func (m *Map) Load(key interface{}) (value interface{}, ok bool)
// 设置key value
func (m *Map) Store(key, value interface{})
// 如果key存在则返回key对应的value，否则设置key value
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
// 删除一个key
func (m *Map) Delete(key interface{})
// 遍历map，仍然是无序的
func (m *Map) Range(f func(key, value interface{}) bool)
```

