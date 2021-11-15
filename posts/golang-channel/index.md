# Golang之channel


##  什么是Channel

- channel是golang中的一种数据类型，它像一个管道或者是队列，规则是先入先出，可以保证数据的顺序。

- 声明channel时需要指定channel中存放数据的类型
- channel是并发安全的，它可以被用来做程间通信

- channel是引用类型

## channel的结构

runtime.hchan

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

- `qcount` channel中的元素个数
- `dataqsiz` channel中循环队列的长度
- `buf` channel的缓冲区指针
- `sendx` channel的发送操作处理到的位置
- `recvx` channel的接收操作处理到的位置
- `recvq` channel等待读取的goroutine队列
- `sendq` channel等待发送的goroutine队列



## 创建channel

创建channel时需要使用make来创建

channel分为带缓冲区的channel和不带缓冲区的channel

```go
//带缓冲区的
ch := make(chan string, 3)
//不带缓冲区的
ch := make(chan int)
```

当make的时候会对传入的缓冲区大小进行判断，如果make的时候缓冲区大小没有传值，那么当前缓冲区大小就会被设置为默认值0，也就是没有缓冲区

## 发送数据

```go
ch <- i
```

### 发送数据需要注意的地方

- 1.如目标channel创建时定义存放的数据类型是int类型时，那么发送数据时只能发送int类型的数据

```go
func main() {
	ch := make(chan int, 3)
	ch <- "test" //cannot use "test" (type untyped string) as type int in send
}
```

- 2.当目标channel没有缓冲区，且等待接收队列中没有goroutine，会阻塞发送

```go
func main() {
	ch := make(chan int)
	ch <- 1 //fatal error: all goroutines are asleep - deadlock!
}
```

- 3.当目标channel处于关闭状态时，会报panic，提示通道已关闭

```go
func main() {
	ch := make(chan int, 3)
	close(ch)
	ch <- 1 //panic: send on closed channel
}
```

- 4.当目标channel缓冲区已满时，且等待队列中没有goroutine，会阻塞发送

```go
func main() {
	ch := make(chan int, 1)
	ch <- 1
	ch <- 2 //fatal error: all goroutines are asleep - deadlock!
}
```

- 5.当声明的目标channel为只读的channel时，无法写入数据

```go
func main() {
	var ch <- chan int
	ch <- 2 //invalid operation: ch <- 2 (send to receive-only type <-chan int)
}
```

## 读取数据

```go
a := <- ch
```

### 读取数据需要注意的地方

- 1.目标channel有缓冲区，但缓冲区中没有数据时，读取会阻塞

```go
func main() {
	ch := make(chan int, 2)
	a := <-ch //fatal error: all goroutines are asleep - deadlock!
	fmt.Println(a)
}
```

- 2.目标channel没有缓冲区，且发送队列没有数据时，读取会阻塞

```go
func main() {
	ch := make(chan int)
	a := <-ch //fatal error: all goroutines are asleep - deadlock!
	fmt.Println(a)
}
```

- 3.目标channel为关闭状态时，会返回channel中数据类型的零值

```go
func main() {
	ch := make(chan int, 3)
	a := <-ch
	fmt.Println(a) //0
}
```

- 4.目标channel为只写channel时，读取数据会失败

```go
func main() {
	var ch chan<- int
	ch = make(chan int, 3)
	a := <-ch //invalid operation: <-ch (receive from send-only type chan<- int)
	fmt.Println(a)
}
```

## 关于读取阻塞

上面学习到如果channel没有内容或者发送队列没有内容时，则会读取阻塞

如果我们有需要等待channel中有数据然后读取的情况下，可以用select来解决读取阻塞的问题

```go
func main() {
	var wg sync.WaitGroup
	ch := make(chan int, 1)

	wg.Add(1)
	defer wg.Wait()

	go func() {
		time.Sleep(5 * time.Second)
		time.Sleep(1000)
		ch <- 1
		wg.Done()
	}()

	for {
		select {
		case a := <-ch:
			fmt.Printf("读取到了数据: %v\n", a)
			return
		default:
			fmt.Println("没有读取到数据..")
			time.Sleep(1 * time.Second)
		}
	}
}
```

## 关于panic

上面学习发送数据的时候学到了如果向已经关闭的channel中发送数据的话会报panic

在Golang中，panic会导致程序退出

所以为了防止出现这种问题，可以使用recover来捕获panic，不让程序异常退出

```go
func main() {
	ch := make(chan int, 1)
	close(ch)
	res := readCh(ch)
	if res{
		fmt.Println("读取成功...")
	}else {
		fmt.Println("读取失败...")
	}
	fmt.Println("继续后续工作......")
}

func readCh(ch chan <- int) bool {
    // 添加了recover来捕获panic时，主进程不会退出,这里处理失败之后主进程还能进行后续工作
    // 如果这里没有捕获panic，则会导致整个程序的退出
	defer func() {
		if err := recover(); err != nil{
			fmt.Println(err) //send on closed channel
		}
	}()
	ch <- 1
	return true
}
```

- 这里的recover也可以运用到其它容易发生panic的地方

## 学习资料

https://www.bilibili.com/video/BV1ME411Y71o?spm_id_from=333.999.0.0

https://halfrost.com/go_channel/#toc-16

https://note.youdao.com/ynoteshare/index.html?id=65b5a66c02398d0317f90882e20c16c0&type=note&_time=1636960393412

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#641-%E8%AE%BE%E8%AE%A1%E5%8E%9F%E7%90%86


