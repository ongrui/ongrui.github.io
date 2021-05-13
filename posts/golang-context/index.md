# Golang之Context


对context的源码理解还没有很深，当前只处于学习阶段

## Context是啥

context是go中跨越API边界以及进程之间的取消信号和其他请求范围的值 --摘自contex.context.go

在Go1.7版本中引入了context，主要用于在goroutine之间传递上下文消息，包括取消信号，超时时间，截止时间，键值对

context是并发安全的

## 为什么需要Context

在go中，每个请求需要开启一个goroutine去处理，但是在大部分业务流程中，一个goroutine并不能很好的满足需求，通常一个请求需要开启多个goroutine来处理该请求，比如一些预处理的工作

但是在业务中由于某些原因（比如操作被取消，请求超时等），业务流程可能会被中断，那么由该请求开启的一些goroutine就会变成孤儿协程，造成资源浪费

那么此时就需要来通知这些goroutine关闭

在go中channel+select可以用来给子goroutine发送信号，通知它们关闭，但是在某些场景下用该方式就会比较复杂，搞不好容易写出问题

于是context应运而生

## Context包

### context接口：

- `Deadline()`：返回context的截止时间，通过判断该时间，函数就能决定是否进行下面的操作

- `Done()`：返回一个只读的channel，表示context被取消的信号，当这个channel被关闭时，说明context被取消了。
  - channel为空且不关闭时，读不出任何东西，当关闭时，则会读出存储类型的零值
- `Err()`：返回channel被关闭的原因
- `Value()`：获取设置的key所对应的val

### Canceler接口：

- 该接口定义了cancel方法和Done方法，用来取消context
- 实现了这两个方法的struct是：`*cancelCtx`和`*timeCtx`

### emptyCtx

`type emptyCtx int`

- emptyCtx永远不会取消，没有值，也没有截止日期

- 它不是struct {}，因为此类型的var必须具有不同的地址

该类型体实现了context接口中定义的方法

下面定义了两个变量

```GO
var (
	background = new(emptyCtx)
	todo       = new(emptyCtx)
)
```

然后通过下面两个方法对外开放

```go
// Background returns a non-nil, empty Context. It is never canceled, has no
// values, and has no deadline. It is typically used by the main function,
// initialization, and tests, and as the top-level Context for incoming
// requests.
func Background() Context {
   return background
}

// TODO returns a non-nil, empty Context. Code should use context.TODO when
// it's unclear which Context to use or it is not yet available (because the
// surrounding function has not yet been extended to accept a Context
// parameter).
func TODO() Context {
   return todo
}
```

- Background主要用来初始化根节点，是context的默认值，并且还作为上下文的根节点存在

- TODO方法返回一个非空的context，当不知道要传入那个context时，可以传入context.TODO

### cancelCtx结构体

 cancelCtx可以被取消，该结构体实现了canceler接口，它将Context作为它的一个匿名字段，这样该结构体就可以被看作成一个context

`Done()`方法：该方法被调用时才会make一个存放struct的channel，然后把该channel返回

`Err()`方法：该方法返回一个context取消的原因

`cancel()`方法：通过关闭c.done，递归取消传递下去的所有子节点，从父节点删除自己，子goroutine接收关闭信号的方式就是select只读的c.done

### WithCancel函数

该函数需要传入一个根节点，上面说的Background，然后会返回一个子context，和一个用于取消子context的cancel函数，一旦调用cancel函数，当前context的子节点就会select到关闭的c.done，从而关闭goroutine

```GO
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}
```

- `newCancelCtx`将传入的上下文包装成私有结构体context.cancelCtx
- `propagateCancel`函数会构建上下文之间的关联，当父context被取消时，取消子context

```GO
func propagateCancel(parent Context, child canceler) {
	done := parent.Done()
	if done == nil {
		return // 父上下文不会触发取消信号
	}
	select {
	case <-done:
		child.cancel(false, parent.Err()) // 父上下文已经被取消
		return
	default:
	}

	if p, ok := parentCancelCtx(parent); ok {
		p.mu.Lock()
		if p.err != nil {
			child.cancel(false, p.err)
		} else {
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
	} else {
		go func() {
			select {
			case <-parent.Done():
				child.cancel(false, parent.Err())
			case <-child.Done():
			}
		}()
	}
}
```

对上面代码的解读

- 如果context.Done为空时，则不会出发取消事件，函数直接返回

- 当传入的根节点的context对象的Done被关闭时，child也就是子context会被直接取消
- `parentCancelCtx`函数返回父级的基础`*cancelCtx`，如果是父级也是`*context`时，

- child会被加入根context的children列表中，等待根context发送取消信号
- 如果父级不是`*context`的话，则开启一个新的goroutine监听根节点的Done()，和child.Done()两个channel，如果根节点的Done()被关闭时，则调用child的cancel取消子上下文

所以该函数的作用是在根context和子context之间同步取消信号，保证根context被取消后，子context也能收到取消信号

#### context.concelCtx.cancel()

该方法会关闭context中的channel并同步取消信号

### timerCtx

该结构体基于`cancelCtx`，添加了time.Timer和deadline

```go
// A timerCtx carries a timer and a deadline. It embeds a cancelCtx to
// implement Done and Err. It implements cancel by stopping its timer then
// delegating to cancelCtx.cancel.
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}
```

通过cancelCtx来实现Done和Err

计时器到期时则通知cancelCtx.cancel来关闭context

`*timerCtx.cancel`

```go
func (c *timerCtx) cancel(removeFromParent bool, err error) {
	c.cancelCtx.cancel(false, err)
	if removeFromParent {
		// Remove this timerCtx from its parent cancelCtx's children.
		removeChild(c.cancelCtx.Context, c)
	}
	c.mu.Lock()
	if c.timer != nil {
		c.timer.Stop()
		c.timer = nil
	}
	c.mu.Unlock()
}
```

第二行调用了contextCtx的cancel方法

第三行，如果removeFromParent为true，则调用removeChild方法从父节点中删除子节点

第八行，如果定时器不为空，则停止计时，防止时间到了之后再次发起取消信号

### WithTimeout函数

```GO
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}
```

该函数返回的是WithDeadline函数，参数是一个根context对象和当前时间+超时时间

### WithDeadline函数

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
   if cur, ok := parent.Deadline(); ok && cur.Before(d) {
      // The current deadline is already sooner than the new one.
      return WithCancel(parent)
   }
   c := &timerCtx{
      cancelCtx: newCancelCtx(parent),
      deadline:  d,
   }
   propagateCancel(parent, c)
   dur := time.Until(d)
   if dur <= 0 {
      c.cancel(true, DeadlineExceeded) // deadline has already passed
      return c, func() { c.cancel(false, Canceled) }
   }
   c.mu.Lock()
   defer c.mu.Unlock()
   if c.err == nil {
      c.timer = time.AfterFunc(dur, func() {
         c.cancel(true, DeadlineExceeded)
      })
   }
   return c, func() { c.cancel(true, Canceled) }
}
```

官方注释写的是该函数返回父context的children，并将截止日期调整为不晚于传入的时间

第二行，如果父级的截止时间更早，则返回

`c := &timerCtx`：新建timerCtx对象，deadline的值为传入的截止时间，context使用传入的跟节点

第十行，调用propagateCancel函数，当父节点被取消时取消子节点

` dur := time.Until(d)`：得到一个当前时间到d的时间差

如果截止时间已过，则调用cancel方法，并传入超过截止时间的错误

如果新建的context的err为空，则返回一个截止时间的计时器

### valueCtx

```go
type valueCtx struct {
   Context
   key, val interface{}
}
```

valueCtx继承了Context

```go
func (c *valueCtx) String() string {
   return contextName(c.Context) + ".WithValue(type " +
      reflectlite.TypeOf(c.key).String() +
      ", val " + stringify(c.val) + ")"
}

func (c *valueCtx) Value(key interface{}) interface{} {
   if c.key == key {
      return c.val
   }
   return c.Context.Value(key)
}
```

valueCtx实现了String和Value方法

value方法返回key的val，如果当前节点不存在，则查找上一个节点，这里是一个递归操作

使用WithValue函数来创建valueCtx

```go
func WithValue(parent Context, key, val interface{}) Context {
   if key == nil {
      panic("nil key")
   }
   if !reflectlite.TypeOf(key).Comparable() {
      panic("key is not comparable")
   }
   return &valueCtx{parent, key, val}
}
```

官方的注释写的是返回父项的副本，key必须可以比较，不能为string类型(string可以比较吧，而且我试了string也可以啊:disappointed_relieved:)或者其它内置类型，避免使用上下文在程序包之间发生冲突

## 使用Context

context的根节点使用context.Background()函数来创建，上面也有写到，Background函数会返回一个emptyCtx，一个空的context，不能被取消，没有值也没有超时时间

创建子节点的方法

```GO
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
func WithValue(parent Context, key, val interface{}) Context {
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
```

一个一个来写一下

### WithCancel

WithCancel 传入一个context的根节点，返回一个context的副本，和一个cancel方法

context的副本用于传入到子goroutine中

cancel用于通知子goroutine关闭

```go
package main

import (
   "context"
   "fmt"
   "sync"
   "time"
)

var wg sync.WaitGroup

func main() {
   Work()
}

func Work() {
   ctx, cancel := context.WithCancel(context.Background())
   wg.Add(1)
   go Preloading(ctx) //开启一个子goroutine，并把返回的context的副本传入
   time.Sleep(time.Second * 3)
   cancel() //调用cancel关闭context.done，子goroutine通过select ctx.Done来关闭进程
   wg.Wait()
}

func Preloading(ctx context.Context) {

    fmt.Println("loading...")
	// 等待发出关闭信号
    for {
		fmt.Println("wait...")
        time.Sleep(time.Second)
        select {
            case <-ctx.Done(): //当cancel被调用时，context.done会被关闭，这里会select到ctx.Done的零值
            fmt.Println("ctx.Done close")
            wg.Done()
            return
        default:
            fmt.Println("ctx.Done not close")
        }
   }
}
```

### WithValue

该函数将传入的键值对传入到根节点

传入一个根节点，和kv对，k必须是可以比较的，返回一个context的副本，用于传入子goroutine

一个简单的例子

```go
package main

import (
   "context"
   "fmt"
)


func main() {
   Work()
}

func Work() {
   ctx := context.WithValue(context.Background(), 1, "a")
   Preloading(ctx)
}

func Preloading(ctx context.Context) {
   id, ok := ctx.Value(1).(string) //如果找到了为1的key，则返回1的val和ok
   if ok {
      fmt.Printf("work id 1 is val: %s", id)
   }else {
      fmt.Printf("not work id 1\n")
   }
}
```

### WithTimeout

传入一个context根节点和一个时段

返回一个context根节点的副本和一个cancel

context根节点的副本用于传入子goroutine，cancel用于通知子goroutine关闭

一个简单的例子

```go
package main

import (
   "context"
   "fmt"
   "sync"
   "time"
)

var wg sync.WaitGroup

func main() {
   Work()
}

func Work() {
   ctx, cancel := context.WithTimeout(context.Background(), time.Second*5)

   wg.Add(1)
   go Preloading(ctx)
   time.Sleep(time.Second * 6)
   cancel() //如果工作已经完毕了，但是时间还没到，可以调用cancel来关闭
   wg.Wait()
}

func Preloading(ctx context.Context) {
   for {
      fmt.Println("wait...")
      time.Sleep(time.Second)
      select {
      case <-ctx.Done():
         fmt.Println("ctx.Done close")
         wg.Done()
         return
      default:
         fmt.Println("ctx.Done not close")
      }
   }
}
```

### WithDeadline

跟WithTimeout很相似

传入一个context根节点和一个未来时间点，返回一个context根节点的副本和cancel

context的副本用于传入子goroutine，cancel用于通知子goroutine关闭

一个简单的例子

```go
package main

import (
   "context"
   "fmt"
   "sync"
   "time"
)

var wg sync.WaitGroup

func main() {
   Work()
}

func Work() {
   d := time.Now().Add(time.Second * 5) //定义一个未来时间点
   ctx, cancel := context.WithDeadline(context.Background(), d)

   wg.Add(1)
   go Preloading(ctx)
   time.Sleep(time.Second * 6)
   defer cancel() //如果工作已经完毕了，但是时间还没到，可以调用cancel来关闭
   wg.Wait()
}

func Preloading(ctx context.Context) {
   for {
      fmt.Println("wait...")
      time.Sleep(time.Second)
      select {
      case <-ctx.Done():
         fmt.Println("ctx.Done close")
         wg.Done()
         return
      default:
         fmt.Println("ctx.Done not close")
      }
   }
}
```



## 总结

像 `WithCancel`、`WithDeadline`、`WithTimeout`、`WithValue` 这些创建函数

实际上是创建了一个个的链表结点而已。我们知道，对链表的操作，通常都是 `O(n)` 复杂度的，效率不高。

那么，context 包到底解决了什么问题呢？

答案是：`cancelation`。仅管它并不完美，但它确实很简洁地解决了问题。

--摘自：`https://qcrao.com/2019/06/12/dive-into-go-context/`



- 在使用中对容易造成内存泄漏的地方，要特别注意cancel的调用

## 学习资料

Go SDK 1.14.4 context

https://qcrao.com/2019/06/12/dive-into-go-context/

https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/

https://www.liwenzhou.com/posts/Go/go_context/#autoid-0-1-4




















