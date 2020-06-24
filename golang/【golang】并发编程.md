# 【golang】并发编程

[TOC]

## 并发实现
### goroutine
golang 的 goroutine 原生支持并发，是并发式语言，而不是并行式语言

通过 `runtime.GOMAXPROCS` 设置 golang 的 `runtime` 运行时所使用的核数，相当于设置所有的 goroutine 会被调度和并行于多少个系统线程

goroutine 就是 golang 中的 coroutine（协程），即一种轻量级线程，是 go 运行时自动调度的最小执行单元，其无论是内存占用，还是创建、销毁和切换的开销都非常小

``` go
// 创建协程来执行该函数的调用
go funcName(args)

// 创建协程，执行匿名函数的调用
go func() {
    // 函数体
}(args)
```
`args` 参数的求值发生在当前协程中，而 `funcName` 函数的执行发生在新的协程中

协程执行的函数无法直接返回结果，需要使用信道来进行结果获取

主协程需要阻塞等待，以便它创建的其他协程执行完毕

### 信道
goroutine 之间通信的管道，信道需要关联一个类型，表示该类型的数据可以通过信道传输

信道是引用类型，其零值是 nil 且没有作用，因此需要用 make 函数来创建信道
``` go
// 定义关联 Type 类型的信道类型
Type TypeCh chan Type

// 创建信道
ch := make(chan Type)
```
<br>
信道默认是双向的，可以通过信道读取和发送数据：
``` go
// 读取信道的数据，并赋值到变量
varName := <- ch

// 将变量的值发送到信道
ch <- varName 
```
信道的操作默认是阻塞的，当一个协程进行信道操作发生阻塞时，go 会把控制权切换到其他协程，直到该信道操作成功后，才会解除阻塞并重新获得控制权

信道的阻塞特性能让 go 协程之间进行高效的通信，不需要用到显式锁或变量来判断通信的状态

如果一个协程操作一个信道发送或读取数据进入阻塞时，没有其它协程操作该信道读取或发送数据，则在运行时该协程会永久阻塞，形成死锁并触发恐慌

利用信道的阻塞特性等待子协程执行完毕：
``` go
func hello(done chan struct{}) {  
    fmt.Println("Hello world goroutine")
    done <- struct{}{}
}

func main() {  
    done := make(chan struct{})
    go hello(done)
    <-done
}
```
用于信号通知的信道，使用空结构体类型信道（ chan struct{} ）是对内存更友好的开发方式，因为在 go 中对于空结构体类的内存空间申请，返回的是一个固定地址，避免了可能发生的内存滥用

单向信道指只能用于发送或者接收数据的信道，即唯送信道（Send Only）和唯收信道（Receive Only）
``` go
// 定义唯送信道类型
Type TypeCh chan<- Type

// 定义唯收信道类型
Type TypeCh <-chan Type
```
在 `chan` 旁以左箭头表示发送到信道，还是从信道中接收

信道转化，只能双向信道转为单向信道：
``` go
// 转化为唯送信道
var sendCh chan<- Type = ch

// 转化为唯收信道
var receiveCh <-chan Type = ch
```

区分协程对通道的读写，使用实例：
``` go
// 用于发送数据的协程
func sendData(sendCh chan<- int) {  
    sendCh<- 10
}

// 用于接收数据的协程
func receiveData(receiveCh chan<- int) {  
    <-receiveCh
}

// 在主协程中，信道定义为双向信道
// 通过信道可以发送或接收数据
func main() {  
    ch := make(chan int)
    go sendData(ch)
    go receiveData(ch)
    // 定时执行一段时间
    <-time.After(10 * time.Second) 
}
```

关闭信道用于表示该信道不再有数据可被读取，因为向一个已关闭的信道发送数据会触发恐慌
``` go
// 关闭信道
close(ch)

// 接收数据并通过 ok 判断信道是否关闭，若信道已关闭则会读取到信道关联类型的零值
varName, ok := <-ch

// 使用 range 可以迭代读取信道，直到信道关闭
for v := range ch {
    fmt.Println("Received ",v)
}
```
信道通常情况下无需关闭，只在必须通知接收者信道不再有数据可被读取的情况才关闭

缓冲信道（Buffered Channel）是带有缓冲的信道，用于缓冲数据以实现非阻塞操作信道

缓冲信道的发送操作只有当缓冲已满时才会阻塞，而接收操作只有当缓冲为空时才会阻塞
``` go
// 创建缓冲信道，capacity 表示缓冲容量，默认为 0，即无缓冲
chanName := make(chan Type, capacity)
```
缓冲信道关闭后，对于接收操作会返回其缓冲的数据，直到缓冲为空后，才返回信道关联类型的零值和信道关闭的标志

由于缓冲信道是非阻塞的，因此在协程中可以读取到自己发送的数据，所以当两个协程之间需要通过非阻塞信道通信时，应该创建两个缓冲信道，一个用于前者写入后者接收，另一个相反

### select
select 语句用于同时监听多个信道操作的状态，完成信道操作并执行对应代码块，以实现信道的多路复用

监听过程协程会阻塞，直到存在至少一个信道操作准备就绪，如果多个信道操作同时准备就绪，则会随机选取其中之一，完成信道操作并执行对应代码块
``` go
select {
    case <-ch1:
        // 接收数据后的操作
    case ch2<- varName:
        // 发送数据后的操作
}
```
没有 case 语句的空 select 语句会一直阻塞，形成死锁并触发恐慌

### 工作池
工作池是一组等待任务分配的协程，每个协程一旦完成了所分配的任务，可继续等待任务的分配
``` go
// 任务
type Job struct {  
    id int
}

// 处理结果
type Result struct {  
    job Job
}

// 存放 Job 和 Result 类型的缓冲信道
var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

// worker 协程，处理任务
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job}
        results <- output
    }
    wg.Done()
}

// 创建 worker pool 的协程
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}

// 写入 jobs 信道，分配任务的协程
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        job := Job{i}
        jobs <- job
    }
    close(jobs)
}

// 读取 results 信道，读取处理结果的协程
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d\n", result.job.id)
    }
    done <- true
}

// main 函数
// 调用 createWorkerPool 来创建 worker
// 阻塞直到 done 信道收到 true，程序结束
func main() {
    // 先创建 done 信道
    done := make(chan bool)
    // 分配任务
    noOfJobs := 100
    go allocate(noOfJobs)
    // 读取结果
    go result(done)
    // 创建工作池
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    // 等待结果全部读取完毕
    <-done
}
```

## sync
### 等待组
本包的 `WaitGroup` 通过计数器来实现对一批 go 协程进行等待，一直阻塞到所有协程执行完毕

`WaitGroup` 的方法：

``` go
func (wg *WaitGroup) Add(delta int) 
```
为等待组的计数器增加指定值

``` go
func (wg *WaitGroup) Done()
```
为等待组的计数器减少 1

``` go
func (wg *WaitGroup) Wait()
```
阻塞直到等待组的计数器变为 0

使用实例：
``` go
// 协程需要获取 wg 的指针进行计数器操作
func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

// 为计数器增加协程数目的值，并传递 wg 的指针到协程中
func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

### 协程安全
当多个 go 协程并发运行竞争访问相同的共享资源时，如果对资源的访问顺序敏感，则这个资源在协程中使用是不安全的

也称为存在竞态条件，而导致竞态条件发生的代码区称作临界区

修复竞态条件有两种方法，一是通过缓冲信道，二是通过 sync 包提供的锁机制

利用容量为 1 的缓冲信道的阻塞特性，修复竞态条件：
``` go
// 声明公共变量
var x  = 0

// 接收 wg 指针和 ch，写入信道后来对公共变量进行修改，完成后读取信道
func increment(wg *sync.WaitGroup, ch chan struct{}) {  
    ch <- struct{}{}
    x = x + 1
    <- ch
    wg.Done()   
}

// 创建 wg 和 ch，并将 wg 指针和 ch 传到协程中
func main() {  
    var w sync.WaitGroup
    ch := make(chan struct{}, 1)
    for i := 0; i < 1000; i++ {
        w.Add(1)        
        go increment(&w, ch)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```

### 锁机制
通过本包中的 `Metux` 互斥锁和 `RWMetux` 读写锁这两种锁机制，可以用于修复竞态条件

两种锁都实现了以下接口：
``` go
type Locker interface {
        // 获取锁
        Lock()
        // 释放锁
        Unlock()
}
```
#### 互斥锁
互斥锁 `Metux` 用于保证任意时刻只有一个协程运行在临界区，即任意时刻只有一个协程能获取到锁（进入锁定状态），其方法：
``` go
func (m *Mutex) Lock()
```
获取互斥锁，协程进入锁定状态
``` go
func (m *Mutex) Unlock()
```
释放互斥锁，协程解除锁定状态

#### 读写锁
读写锁 `RWMetux` 保证任意时刻允许多个读操作的协程或者一个写操作的协程运行在临界区，即任意时刻有多个协程能获取到读锁（进入读锁定状态），或只有一个协程能获取到写锁（进入写锁定状态），适合协程读多写少的情况
``` go
func (rw *RWMutex) Lock()
```
获取写锁，协程进入写锁定状态
``` go
func (rw *RWMutex) Unlock()
```
释放写锁，协程解除写锁定状态
``` go
func (rw *RWMutex) RLock()
```
获取读锁，协程进入读锁定状态
``` go
func (rw *RWMutex) RUnlock()
```
释放读锁，协程解除读锁定状态

使用实例：
``` go
// 声明公共变量
var x = 0

// 接收 wg 和 m 指针，获取锁后来对公共变量进行修改，完成后释放锁
func increment(wg *sync.WaitGroup, m *sync.Mutex) {  
    m.Lock()
    x = x + 1
    m.Unlock()
    // defer m.Unlock()
    wg.Done()   
}

// 创建 wg 和 m，并将其指针传到协程中
func main() {  
    var wg sync.WaitGroup
    var m sync.Mutex
    for i := 0; i < 1000; i++ {
        wg.Add(1)        
        go increment(&wg, &m)
    }
    w.Wait()
    fmt.Println("final value of x", x)
}
```
> 锁的获取和释放必须成对出现，否则会造成死锁，使其他协程永久阻塞在锁等待状态
> 
> 可以使用 defer 语句保证协程结束后必须进行解锁

### 协程安全的映射
原生映射 map 在没有锁的情况下在协程中使用是不安全的，本包中的 `Map` 则实现了协程安全的映射

`Map` 的方法：

``` go
func (m *Map) Store(key, value interface{})
```
存储一个键值到映射

``` go
func (m *Map) Load(key interface{}) (value interface{}, ok bool)
```
从映射获取指定键的值，若存在则 `ok` 为 `true`，否则为 `false`

``` go
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
```
从映射获取指定键的值，若存在则 `loaded` 为 `true`，否则将值定的值存入并返回，此时 `loaded` 为 `false`

``` go
func (m *Map) Delete(key interface{})
```
从映射删除一个键值

``` go
func (m *Map) Range(f func(key, value interface{}) bool)
```
迭代映射，`f` 是用于接收迭代获取的键值并进行处理的函数，其返回值为 `true` 时表示继续迭代，`false` 表示停止

### 条件变量
本包中的 `Cond` 条件变量用于协调各个需要访问相同的共享资源的协程

主要的优势是在效率方面的提升，当共享资源的状态不满足条件的时候，需要操作的协程不用循环地检查其状态，等待条件变量的通知即可

`Cond` 的创建： 
``` go
func NewCond(l Locker) *Cond
```
使用指定的 `Locker`，创建一个条件变量

通过 `Cond.L` 可以获取到该 `Locker`，在发送通知以及等待通知前，必须获取到该 `Locker` 的锁

`Cond` 的方法：
``` go
func (c *Cond) Wait()
```
阻塞当前协程，直到收到该条件变量发来的通知

`Wait` 方法开始时会自动释放 `Cond.L` 的锁，结束时会自动获取 `Cond.L` 的锁

一般的使用方式：
``` go
c := sync.NewCond(&sync.Mutex{})

c.L.Lock()
// 不符合共享资源的使用条件，则等待通知
for !condition() {
   c.Wait()
}
// 使用共享资源
operation()
c.L.Unlock()
```
<br>
``` go
func (c *Cond) Signal()
```
单播通知，向一个等待该条件变量的协程发送通知

``` go
func (c *Cond) Broadcast()
```
广播通知，向所有等待该条件变量的协程发送通知

使用实例：
``` go
// 本公共变量设定为只允许同时被一个协程进行访问
var list = make([]int)

// 当列表不为空时，消费其所有元素，并发出通知继续生产 
func useList(c *sync.Cond) {
    c.L.Lock()
    if len(list) == 0 {
        c.Wait()
    }
    for _, v := range list {
        fmt.Printf("Use element %d\n", v)
    }
    c.Signal()
    c.L.Unlock()
}

// 当列表为空时，生产一批元素，并发出通知继续消费 
func pushList(c *sync.Cond) {
    c.L.Lock()
    if len(list) > 0 {
        c.Wait()
    }
    for i := 1;i<10;i++ {
        list = append(list, i)
    }
    c.Signal()
    c.L.Unlock()
}

func main {
    c := sync.NewCond(&sync.Mutex{})
    go useList(c)
    go pushList(c)
    // 定时执行一段时间
    <-time.After(10 * time.Second) 
}
```

## context
本包提供上下文，用于在函数或协程之间传递截止期限、取消信号或其他请求范围值，实现跨函数或协程的控制

``` go
type Context interface {
	// 返回截止期限，截止限期是上下文被取消的时间
	// 如果没有截止时间则 ok 为 false
	Deadline() (deadline time.Time, ok bool)

	// 返回上下文被取消时会关闭的信道
	// 如果上下文不能被取消，则返回 nil
	Done() <-chan struct{}

	// 上下文已取消后，返回非空错误以说明取消的原因
	// 上下文未被取消时，返回 nil
	Err() error

	// 返回上下文中的 key 对应的值，不存在为 nil
	Value(key interface{}) interface{}
}
```
`Context` 接口表示上下文，包含了指定的方法，这些方法均是协程安全的，且允许多次调用

### 上下文创建
``` go
// 获取一个空上下文，用于发起函数中，作为所有派生上下文的根
// 空上下文指无截止限期，不能被取消且没有值通的上下文
func Background() Context

// 同样获取一个空上下文，用于开发阶段预留的上下文
func TODO() Context

// 返回带有新 Done 信道的派生上下文（内嵌父上下文）和取消函数
// 当取消函数或者父上下文的 Done 信道被关闭时，该派生上下文被取消
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

// 返回带有截止限期和新 Done 信道的派生上下文（内嵌父上下文）和取消函数
// 当截止限期到达、取消函数或者父上下文的 Done 信道被关闭时，该派生上下文被取消
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)

// 返回带有根据超时生成的截止限期和新 Done 信道的派生上下文（内嵌父上下文）和取消函数
// 当截止限期到达、取消函数或者父上下文的 Done 信道被关闭时，该派生上下文被取消
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
}

// 返回带有键值的派生上下文（内嵌父上下文）
func WithValue(parent Context, key, val interface{}) Context
```
返回的 `Context` 上下文都是指针类型

### 上下文使用
- 上下文应该以参数方式显示传递，不要放在结构体中，并且应该作为第一个参数
- 传递上下文时不要传递 nil，未确定就传递 `context.TODO()`
- 及时取消上下文可以防止协程泄漏

使用实例：
``` go
// 生成整数发送到信道中
func gen(ctx context.Context, dst chan<- int) {
    n := 1	
    for {
        select {
        case <-ctx.Done():
            // 关闭信道并返回，避免泄漏协程
            close(dst)
            return
        case dst <- n:
            n++
        }
    }
}

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    dst := make(chan int)
    go gen(ctx, dst)
    	
    // 当循环获取信道的整数为 5 时，取消上下文
    for n := range dst {
    	if n == 5 {
    	  cancel()
    	}
    }

    // 定时运行 20 秒后返回
    <-time.After(time.Second * 20)
}
```