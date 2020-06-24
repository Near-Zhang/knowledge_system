# 【python】高性能编程

[TOC]

## 概念
### 内存管理
**虚拟储存器机制**

操作系统通过虚拟储存器机制来进行内存管理，为每个进程提供了一个独立的、连续的、一致的虚拟地址空间，当进程访问虚拟地址空间时，操作系统和处理器会合作完成地址翻译，将虚拟地址转化为合适的物理地址，最终访问到达物理内存或磁盘
![](media/15910682512490.jpg)
> MMU：CPU 中负责地址翻译的内存管理单元，通过访问操作系统维护的映射表完成翻译
> Main Memory：主存，即物理内存

实现的关键能力：
- 为每个进程提供独立的、连续的、一致的地址空间，解决了进程并发的内存管理问题

- 为每个进程提供大于物理内存的地址空间，解决了物理内存不足的问题

- 允许不同的虚拟地址映射为相同的物理地址，在程序之间实现有效而安全的通信

- 将物理内存看作磁盘的高速缓存，在物理内存中只保存活动区域，并在需要时和磁盘进行数据交换，提高了物理内存的使用效率

**内核空间和用户空间**

操作系统将分配到进程的虚拟地址空间划分为两部分，高位部分为内核空间，低位部分为用户空间

比如在 32 位的 linux 操作系统中，将高位的 1GB 地址空间（从虚拟地址 0xC0000000 到 0xFFFFFFFF ）划分为内核空间，而低位的 3GB 地址空间（从虚拟地址0x00000000到0xBFFFFFFF）划分为用户空间，而对于 64 位的操作系统，这两个空间则非常大了

操作系统的核心是内核，可以使用特权指令以及访问底层硬件设备，独立于用户进程，内核空间是只允许内核访问的地址空间，保证了内核的安全，不受用户进程的影响

不同进程的内核空间所映射的物理地址空间是相同的，而用户空间所映射的物理地址空间是不同的，也就是说不同进程的 **内核空间是共享的，用户空间是独立的**
![](media/15910733351940.jpg)

### 并发和并行
并行（Parallellism）指在一个时间点上同时处理多个任务的能力，并行只存在于多处理器系统中

![](media/15559215505627.jpg)

![](media/15559223175584.jpg)

并发（Concurrency）指在一段时间间隔内交替处理多个任务的能力，并发既可存在于单处理器中，也可存在于多处理器系统中，但每个处理器在一个时间点上只有一个任务在处理

也就是说，并发和并行并非是互斥的，并发可以存在于并行系统中，但是每条并行的一个时间点上只有一个任务在处理，比如一个开启了 2 进程，每个进程 4 个协程的系统，就是个并行度为 2，并行度为 8 的系统

![](media/15559215240519.jpg)

![](media/15559222952970.jpg)

在运行速度上，并行系统不一定比并发系统更快，由于运行中的多个任务之间通常是需要通信的，然而这些任务在并行系统上由不同的处理器处理，它们之间的通信开销比较大，但在并发系统上，这种通信开销就很小

### 进程、线程、协程
#### 进程（Process）
进程是程序的基本运行实体，是操作系统进行资源分配和调度的基本单位，每个进程都占有一个独立的、连续的、一致的地址空间，其中包括共享的内核空间和独立的用户空间

进程是线程的容器，并且至少包含一个线程，这个线程被称为其主线程，进程内的线程可以创建新线程，除了进程 0 以外，每个进程都由其父所进程创建

**进程的执行状态**
- 就绪状态（Ready）

  进程已获得除处理器外的所需资源，只是在等待分配处理器资源，只要分配了处理器进程就可执行
  就绪状态的进程可以按多个优先级来划分队列。如一个进程由于时间片用完而进入就绪状态时，排入低优先级队列，而当进程由 I/O 操作完成而进入就绪状态时，排入高优先级队列
  
- 运行状态（Running）

  进程占用处理器资源，处于此状态的进程的数目小于等于处理器的数目
  在没有其他进程可以执行时，如所有进程都在阻塞状态，系统通常会自动执行系统的空闲进程

- 阻塞状态（Blocked）

  系统由于进程等待某种条件，如 I/O 操作或进程同步，在条件满足之前无法继续执行，该事件发生前即使把处理器资源分配给该进程，该进程也无法执行

<br>
![](media/15910941122563.jpg)

<br>
**进程的创建和组成**

进程的创建过程：
- 内核为进程分配地址空间，读取程序代码到用户空间
- 内核为进程分配 PCB 和其他资源
- 内核为进程保存状态信息到 PCB，并将进程放到运行队列中等待处理器资源进行执行

进程的实体组成被称为 **进程上下文**，分为 **进程控制块（PCB）、用户空间** 两个部分

当内核需要切换到另一个进程，会保存当前进程上下文再进行切换，而下次再切换回来时，根据之前保存的进程上下文，就能还原到原来的状态继续执行下去

PCB 是记录进程状态及其他相关信息的 `task_struct` 数据结构，存在于内核空间，是进程存在的唯一标识，在进程被创建时由内核为其分配，其中的主要结构如下：
- 描述信息：PID、进程名、用户 ID、父进程 ...
- 控制信息：当前执行状态、优先级、计时信息、通信信息 ...
- 资源信息：占用内存信息、I/O设备信息、打开的文件信息 ...
- CPU 现场保护结构：进程挂起时保存的当前状态，后续用于恢复执行，寄存器值、堆栈数据 ...

PCB 的组织结构：
- 链表方式：根据进程状态将所有 PCB 分为多个链表
- 索引方式：根据进程状态建立多个索引表，索引值指向各个 PCB

进程的用户空间结构：

![-w165](media/15910878041053.jpg)
                     
- 堆栈段：存储进程运行需要的动态数据
    - 栈段（stack）：可动态向下扩张地址空间，存储局部变量，函数调用时的参数和返回值，作用域执行结束后会被立即回收，操作方式类似数据结构中的栈，先进后出
    - 空白段：用于扩张堆段或栈段，以及 mmap 内存映射
    - 堆段（heap）：可动态向上扩张地址空间，存储通过系统调用如 alloc、new，动态分配的内存，通过系统调用如 free、delete 进行回收，操作方式类似数据结构中的链表
    
- 数据段：存储进程运行需要的静态数据，如环境变量，常量、全局变量
    - data 段：已初始化的数据
    - bss 段 ：未初始化的数据
    
- 文本段：
    - text：存储程序代码转化得到的 CPU 二进制执行指令

**进程间通信（IPC）的方式**

- 管道（Pipe）：半双工的通信方式，同一时间只允许单向通信，而且只能在有亲缘关系（父子进程）的进程间使用

- 命名管道（FIFO）：半双工的通信方式，但允许在无亲缘关系的进程间使用

- 消息队列（Message Queue）：是由消息组成的链表，存放在内核中并由消息队列标识符标识

- 信号量（Semaphore）：是一个计数器，用来控制多个进程对共享资源的访问，常作为一种锁机制

- 共享内存（Shared Memory）：是通过映射一块多个进程可以共享的地址空间，一个进程创建后多个进程都可以访问，是最快的通信方式

- 套接字（Socket）：是一种客户端和服务端形式的通信方式

- 信号（Signal）：是一种比较复杂的通信方式，用于通知和接收进程某个事件的发生

#### 线程（Thread）
线程是进程中一个单一顺序的执行流，是进程中的实际执行单位，是处理器进行调度和执行的基本单位，被采用了基于时间片轮转的抢占式调度方式

在单处理器系统中的调度和执行：

![-w363](media/15911681473220.jpg)

线程必须存在于进程中，一个进程中可以存在一个或多个线程，其中的所有线程共享该进程的地址空间，但拥有独立的 **线程控制块（TCB）和线程栈**，这两者就是 **线程上下文**，即线程的实体组成

![-w364](media/15911669524036.jpg)

TCB 是记录线程状态及其他相关信息的 `task_struct` 数据结构，存在于内核空间，是线程存在的唯一标识，在进程被创建时由内核为其分配，由于其数据结构、组织结构都和 PCB 一致，因此线程也被称是操作系统实现的轻量级进程

TCB 存储着一部分和 PCB 共有的信息，也有一部分特别的信息，如线程 ID 、所属进程或线程的指针、线程状态、...

线程栈是从所属进程的空白区中映射出来的，和进程的栈段不同的是，线程栈不能动态增长，一旦用完则会内存溢出，线程栈之间原则上是独立的，但由于线程共享所属进程的地址空间，因此通过地址还是可以进行互相访问

**进程和线程的区别**
- 多个进程的地址空间是独立的，同一进程的线程共享进程的内存空间，但拥有独立的线程栈

- 线程可以直接访问所属进程的数据段，子进程则拥有一份从父进程复制的独立数据段

- 多个进程之间必须通过 IPC 方式进行通信，同一进程的线程之间可以直接通信

- 线程的创建开销很少，而进程的创建需要大量复制其父进程的内容，开销较大，并且线程
  上下文切换要比进程上下文切换的效率高得多

- 线程可以控制同一进程下的其他线程，进程只能控制其子进程

- 对主线程进行操作可能会影响同一进程下其他的线程，对父进程的操作不会影响其子进程

#### 协程（Coroutine）
协程是一种用户态的轻量级线程，和线程或者进程的最大区别，就是线程或者进程的切换受操作系统控制，而协程的切换由自身控制，即由获得当前执行权的协程来控制

协程必须存在于线程中，一个线程中可以存在一个或多个协程，其中的所有协程共享该线程的地址空间，但它也拥有独立的协程上下文

协程的优势：
- 没有线程上下文切换的开销
- 既没有线程间为各原子操作加锁的开销，也没有进程间进行通信和同步的开销
- 原子操作是指不会被线程调度机制打断的操作，如果一个操作包含多个原子操作，可以通过锁机制来保证该操作的完整性

协程的缺陷：
- 多协程本质也是单线程，无法利用多核处理器，除非配合多进程或者多线程来利用多核处理器，但 CPython 解释器由于 GIL 的限制，多线程也无法利用多核处理器
- 在获得当前执行权的协程发生阻塞时，会阻塞掉整个线程

### 全局解释锁（GIL）
在 Cpython 解释器中引入了一个全局解释锁（GLI），来保护对 python object 的访问，防止多个线程同时执行不同的 python bytecode，从而导致不可预期的结果

Cpython 解释器虽然是通过操作系统的原生线程接口（linux 上为 pthread）来创建线程，由操作系统来调度线程的执行，但是其中的每个线程被调度唤醒前，都需要先获得 GIL，才能通过解释器运行对应的 python bytecods，从而利用到处理器的资源

GIL 这个限制如果在单处理器的环境下不会有任何问题，因为任何一个被唤醒的线程都能成功获得 GIL（只存在一个核心，新线程被唤醒时老线程就会挂起同时释放 GIL），但在多处理器的环境下，被其中一个核心唤醒的线程由于获得 GIL 的线程还在其他核心上运行，未释放 GIL，这段等待获得 GIL 的时间该线程也无法运行

也就是说，同一个时刻只有一个线程可以执行 python bytecods，因此在 Cpython 解释器上实现的多线程编程，即使运行在多处理器的环境下，实际上也只能利用到相当于当个处理器的资源

避免 GIL 限制的方式：
- 选用不同的解释器：GIL 是解释器的特性，并不是 python 语言的特性，比如 JPython PyPy 就没有 GIL

- 选用不同的编程方式：只有多线程编程会受到限制，比如多进程、多协程就不会受到影响

**GIL 和 线程锁**

为什么 Cpython 已经存在 GIL，在多线程编程中还需要使用线程锁？

因为 GIL 只能保证 python 对象的访问安全，无法保证线程中操作的完整性，从而产生的竞态条件，即结果与线程执行顺序有关，如图：

![-w693](media/15913335173899.jpg)

那么为什么有了用户态的线程锁，Cpython 还需要 GIL？

一个重要原因是为了降低 Cpython 程序的开发的复杂度，比如 CPython 解释器的内存回收机制，可以理解为有一个独立的线程每过一段时间会做一次全局轮询，对无标识符指向的内存进行回收，假设一个线程 del 了一个变量，然后在内存回收线程在清空这个变量的过程中，另一个线程正好又重新给这个还没来及得清空的内存空间赋值了，结果就有可能新赋值的数据被删除了，为了解决类似的问题，Cpython 解释器就简单粗暴地加了 GIL，后续许多新特性也依赖了 GIL

### IO 模型
#### 缓存 IO
IO 模型是关于 **缓存 IO** 的概念，缓存 IO 又称为 **标准 IO**，是绝大多数文件系统的默认 IO 机制，linux 内核在缓存 IO 机制下，要完成一次 IO 读或写操作的步骤如下：

- 读操作：内核先从 IO 设备中把数据读取到内核空间的缓冲区，再从这个内核的空间缓冲区把数据拷贝到用户空间

- 写操作：内核先把数据从用户空间拷贝到内核空间的缓冲区，再从这个内核空间的缓冲区把数据写入到 IO 设备

> 内核会为进程所打开的 IO 设备在内核空间分配一块缓冲区，而每个打开的 IO 设备都被视为文件，由一个文件描述符（fd）所指向
>
> 在 linux 中一切皆文件，内核为每个进程维护了一张用于记录已打开文件的文件描述表，文件描述符则是该表的索引，是一个非负整数，每当进程调用内核打开一个现有文件或者新建一个文件时，内核都会分配一个文件描述符并返回给进程，而进程也通过该文件描述符在调用内核时标识所要操作的文件

这种 IO 机制的优点是减少了进程发起系统调用的次数，缺点是增加了处理器和内存的开销，因为在过程中数据需要在用户空间和内核空间进行拷贝

#### 区分和比较
最对于一个进程来说，IO 操作实际上是通过系统调用内核，来对 IO 设备在内核空间的缓冲区进行读写，而并非直接对 IO 设备进行读写，这个是内核负责完成的操作

以最常用的缓存 IO，即 网络 IO 为例，对于进程来说，进行一次 IO 读或写操作的耗时，分为以下两个阶段：
- 等待数据准备
    - 读操作：等待 socket 在内核空间的缓冲区可读，即内核已经把数据从 socket 读取到缓冲区
    - 写操作：等待 socket 在内核空间的缓冲区可写，即内核可以把数据从缓冲区写入到 socket

- 等待数据拷贝
    - 读操作：等待内核把数据从 socket 在内核空间的缓冲区拷贝到用户空间
    - 写操作：等待内核把数据从用户空间拷贝到 socket 在内核空间的缓冲区

使用不同 IO 模型的进程，区别就在于对上述两个阶段的处理以及所处状态，在 linux 中存在以下 5 种 IO 模型：
- 阻塞 IO（Blocking IO）
- 非阻塞 IO（Nonblocking IO）
- IO 多路复用（IO Multiplexing）
- 信号驱动 IO（Signal Driven IO）
- 异步 IO（Asynchronous IO）

> 下面关于各个 IO 模型的描述，是基于进程的读操作来说明的，写操作虽然在过程上有些不同，但在原理和特点是上是一致的

<br>
**阻塞 IO**

![-w716](media/15919542751927.jpg)

特点：进程在等待数据准备阶段和等待数据拷贝阶段都是阻塞的

详细过程：
- 进程通过调用函数发起系统调用后进入阻塞状态
- 等待内核完成进行数据准备和数据拷贝，返回成功指示
- 进程解除阻塞状态并得到数据

<br>
**非阻塞 IO**

![-w720](media/15919544082707.jpg)

特点：进程在等待数据准备阶段不是阻塞的，但需要不断的发起系统调用来询问内核是否完成数据准备

详细过程：
- 进程不断通过函数发起系统调用，若内核未完成数据准备，则立即返回异常，进程不会进入阻塞状态
- 一旦内核完成了数据准备，进程再次发起系统调用，就会进入阻塞状态
- 等待内核完成进行数据拷贝，返回成功指示
- 进程解除阻塞状态并得到数据

<br>
**IO 多路复用**

![-w751](media/15919877192306.jpg)

特点：和阻塞 IO 类似，进程在等待数据准备阶段和等待数据拷贝阶段都是阻塞的，但进程是一次性对多个 socket 进行了等待数据准备

详细过程：
- 进程通过调用多路复用函数发起系统调用后进入阻塞状态
- 内核同时监测多个 socket，一旦发现完成数据准备的 socket，返回对应的可读条件
- 进程解除阻塞状态，并根据可读条件对特定 socket 发起系统调用后，再次进入阻塞状态，由于没有等待数据准备的阶段，可以调用非阻塞 I/O 函数
- 等待内核完成进行数据拷贝，返回成功指示
- 进程解除阻塞状态并得到数据

注意：对于 sockct 服务端，IO 多路复用的优势就是可以同时处理多个连接，但如果连接数很低的话，使用 IO 多路复用就不一定比使用多线程 + 阻塞 IO 的性能要高，因为通过多路复用函数进行系统调用时，内核需要有很多额外的开销

**信号驱动 IO**

![-w745](media/15919877727907.jpg)

特点：进程在等待数据准备阶段不阻塞，并且只需要发起一次系统调用

详细过程：
- 进程通过 SIGIO 信号处理程序进行系统调用，内核立即返回，进程继续执行
- 直到 SIGIO 信号处理程序接收到内核完成数据准备后递交的 SIGIO 信号，进程通过调用函数进行系统调用，进行阻塞状态
- 等待内核完成进行数据拷贝，返回成功指示
- 进程解除阻塞状态并得到数据

注意：信号驱动 IO 不适合处理 TCP socket，因为 TCP socket 的信号产生的过于频繁，连接请求完成、断开连接发起、断开连接完成、数据可读、数据可写等都会产生SIGIO，而这里只需要 SIGIO 信号在数据可读或数据可写时产生。但是适合处理 UDP socket，因为 SIGIO 信号只在数据报到达以及发生异步错误才会产生

<br>
**异步 IO**

![-w718](media/15919881794407.jpg)

特点：进程在数据准备阶段和数据拷贝阶段进程都不需要阻塞，调用过程完全异步

详细过程：
- 进程通过调用异步操作函数进行系统调用后，内核立即返回，进程继续执行
- 内核完成数据准备和数据拷贝之后，会给进程递交一个信号，通知异步操作已经完成
- 进程得到数据

<br>
**各个模型比较**

![-w529](media/15919885243484.jpg)

阻塞指当调用过程不能立即得到调用结果时，当前线程会被挂起直到函数返回调用结果，阻塞是一种主动行为，并且不占用处理器资源

异步指发起 IO 操作的系统调用后，完全不需要阻塞等待内核处理，当前线程可以继续运行，当内核完成操作后会再进行信号回调通知

阻塞 IO、非阻塞 IO、IO 多路复用、信号驱动 IO 这四种模式都有进程等待内核处理的阶段，因此都属于同步 IO

但对发起进程来说，等待数据拷贝阶段的耗时是远低于等待数据准备阶段的，因此对于仅没有等待数据准备阶段的 IO 模型，如IO 多路复用、信号驱动 IO，也可以算实现了 IO 的异步化

#### IO 多路复用
##### select
工作原理：
- 进程通过 select 函数发起系统调用后，进入阻塞状态，传入三个文件描述符列表，分别表示所要监测的 IO 设备状态是可读、可写和触发异常

- 内核需要先把所有文件描述符列表从用户空间拷贝到内核空间，并进行轮询操作

- 轮询操作指对文件描述符列表分别轮询找到其中进入对应状态的文件描述符，且组成对应状态的结果列表，若存在不为空的结果列表，则返回这些结果列表，即结果列表从内核空间拷贝到用户空间，否则进入睡眠，直到超时或接收到驱动设备的中断通知，唤醒并进行轮询操作

- 进程解除阻塞，并得到对应状态的结果列表

其所剩不多的优点是良好跨平台支持，目前几乎在所有的操作系统上支持

其缺点主要包括以下三点：
- 最大支持同时监测的文件描述符数量为 1024，由于存储 文件描述符列表的数据结构 FD_SETSIZE 宏定义的限制，可以通过修改该宏定义并重新编译内核来去除该限制

- 每次系统调用都需要把所有文件描述符列表从用户空间拷贝到内核空间，这个开销随着所要监测的文件描述符数变大时会变得很大

- 需要遍历所有需要监测的文件描述符来找到具体已经进入状态的文件描述符，时间复杂度为 $O(n)$，这个时间所要监测的文件描述符数量变大时会变得很大

使用方式：
``` py
import select

# 阻塞直到返回进入可读、可写和触发异常状态的三个文件描述符列表
# rfdlist、wfdlist、xfdlist 分别是需要监控可读、可写和触发异常状态的文件描述符列
# timeout 是阻塞超时，默认永不超时
select.select(rfdlist, wfdlist, xfdlist, timeout=None)


# 用例：实现并发的 socket 服务端
import select, socket

ss = socket.socket()
ss.setblocking(0)
ss.bind(('', 9998))
ss.listen()

rfdlist = [ss]
wfdlist = []

while True:
    rable, wable, exceptional = select.select(rfdlist, wfdlist, rfdlist)
    for s in rable:
        if s == ss:
            conn, cli = s.accept()
            rfdlist.append(conn)
        else:
            date = s.recv(1024)
            if date:
                try:
                    s.send(date)
                except ConnectionResetError:
                    rfdlist.remove(s)
            else:
                s.close()
                rfdlist.remove(s)
``` 

##### poll
工作原理本质上和 select 没有太大区别，但是 poll 只能被 Unix 平台支持，并且是通过传入包含所需监控的文件描述符和对应状态的的结构进行调用的，内核通过一个链表来存储这些信息，而调用返回的是以文件描述符和对应状态作为元素的列表

由于使用链表作为存储需要监测文件描述符的数据结构，没有了最大数量的限制，但和 select 同样的，开销会随着监测文件描述符数量变大时变得很大，效率降低

使用方式：
``` py
import select

# 对应状态的事件表示，底层是二进制数字，可用按位或来合并事件，用按位与来判断事件
select.POLLIN   # 可读数据（包括 socket 关闭）
select.POLLPRI  # 可读带外（紧迫）数据
select.POLLOUT  # 可写数据
select.POLLERR  # 发生异常
select.POLLHUP  # 发生挂起
select.POLLNVAL # 不是文件描述符

# 初始化 poll 对象
poll = select.poll()

# poll 对象的方法：

# 注册需要监测的文件描述符和状态
# fd 是文件描述符编号
poll.register(fd, eventmask)

# 解除已注册的文件描述符
poll.unregister(fd)

# 阻塞直到返回包含进入状态事件的 (fd, eventmask) 列表
# fd 是文件描述符编号，eventmask 是事件标记
poll.poll(timeout=None)


# 用例：实现并发的 socket 服务端
import select, socket

ss = socket.socket()
ss.setblocking(0)
ss.bind(('', 9998))
ss.listen(1024)

eventmask = select.POLLIN | select.POLLPRI | select.POLLHUP | select.POLLERR
fd_sock_dict = {ss.fileno(): ss}
poller = select.poll()
poller.register(ss,eventmask)

while True:
    for fd, event in poller.poll():
        s = fd_sock_dict[fd]
        if event & ( select.POLLIN | select.POLLPRI ):
            if s == ss:
                conn, cli = s.accept()
                poller.register(conn, eventmask)
                fd_sock_dict[conn.fileno()] = conn
            else:
                date = s.recv(1024)
                if date:
                    try:
                        s.send(date)
                    except ConnectionResetError:
                        poller.unregister(fd)
                else:
                    s.close()
                    poller.unregister(fd)
```

##### epoll
工作原理：
- 当一个 epoll 对象被创建时，内核使用 mmap 机制将一块用户空间地址和一块内核空间地址映射到相同的物理地址，并在该地址空间建立了一个 epoll 文件系统，且在该文件系统上创建一个 file 结点，并在 file 结点中创建了一个红黑树和就绪链表，一个 file 结点代表并服务于 一个 epoll 对象，用于维护其相关数据

- file 结点中的红黑树用来存储需要监测的文件描述符和对应状态，以支持快速的查找、插入、删除，时间复杂度$O(lg_{2}N)$，并且在插入新的需要监测的文件描述符和对应状态时，会在内核的中断处理程序中注册一个回调，将发生中断事件的文件描述符和对应状态存入到就绪链表中，内核返回成功指示

- 进程通过 epoll 对象发起系统调用，进行阻塞状态，直到超时或内核接收到驱动设备的中断通知，把就绪链表中的内容拷贝到指定的用户空间，然后清空就绪链表

- 最后内核检查这些文件描述符，如果是设为 LT 模式的文件描述符，并且有未处理的事件时，又把该文件描述符拷贝回到刚清空的就绪链表

- 进程解除阻塞，得到以文件描述符和对应状态作为元素的列表

> epoll 的两种工作模式：
> 
> LT（水平触发）是根据当前状态而触发，ET（边缘触发）模式是根据状态改变而触发
> 
> 对于 LT（水平触发）模式，通过调用 epoll 对象后得到监测到事件发生的文件描述符，若该文件描述符的对应事件没有被处理，则下次调用时，结果仍会再次包含该文件描述符，而对于 ET（边缘触发）模式，在这种情况下则不会包含该文件描述符
> 
> 边缘触发的性能要更高一些，但是代码实现更复杂一些

epoll 的缺点是只能被 2.6 版本以上的 Linux 内核所支持，而优点有以下：
- 除了支持 select、poll 使用的 LT 模式，还支持 ET 模式

- 没有同时监测文件描述符最大数量的限制，这个最大值由使用的内存空间决定，1G 的内存空间大概可以支持检测 10 万个文件描述符

- 由于 nmap 机制，没有多次把文件描述符从用户空间拷贝到内核空间的开销

- 内核无需历遍所有文件描述符，通过为每个文件描述符注册中断回调函数的方式，进而根据就绪链表获得具体进入监测状态的文件描述符

epoll 对比 select、poll 主要是解决了随着需要监测文件描述符的数量提升时性能下降的问题，但在需要监测的文件描述符较少的情况下，三者性能上并没有很大差别，因此使用epoll 模型的服务端能支持超大并发的访问

使用方式：
``` py
import select

# 对应状态的事件表示，底层是二进制数字，可用按位或来合并事件，用按位与来判断事件
select.EPOLLIN      # 可读数据（包括 socket 关闭）
select.EPOLLPRI     # 可读带外（紧迫）数据
select.EPOLLOUT     # 可写数据
select.EPOLLERR     # 发生异常
select.EPOLLHUP     # 发生挂起
select.EPOLLET      # 设为边缘触发模式

# 初始化 epoll 对象
epoll = select.epoll()

# epoll 对象的方法：

# 注册需要监测的文件描述符和状态
# fd 是文件描述符编号
epoll.register(fd, eventmask)

# 解除已注册的文件描述符
epoll.unregister(fd)

# 阻塞直到返回包含进入状态事件的 (fd, eventmask) 列表
# fd 是文件描述符编号，eventmask 是事件标记
epoll.poll(timeout=None) 

# 返回 epoll 对象自身的文件描述符编号
epoll.fileno()                      

# 关闭 epoll 对象，使用完毕后需要关闭，否则会占有资源，包括文件描述符
epoll.close()               
                            

# 用例：实现并发的 socket 服务端
import select, socket

ss = socket.socket()
ss.setblocking(0)
ss.bind(('', 9998))
ss.listen(1024)

eventmask = select.EPOLLIN | select.EPOLLPRI | select.EPOLLHUP | select.EPOLLERR
fd_sock_dict = {ss.fileno(): ss}
epoll = select.epoll()
epoll.register(ss, eventmask)

while True:
    for fd,event in epoll.poll():
        s = fd_sock_dict[fd]
        if event & ( select.EPOLLIN | select.EPOLLPRI ):
            if s == ss:
                conn, cli = s.accept()
                epoll.register(conn, eventmask)
                fd_sock_dict[conn.fileno()] = conn
            else:
                date = s.recv(1024)
                if date:
                    try:
                        s.send(date)
                    except ConnectionResetError:
                        epoll.unregister(fd)
                else:
                    s.close()
                    epoll.unregister(fd)
        elif event & select.EPOLLHUP:
            print('OUT')
            s.close()
            epoll.unregister(fd)
```

##### 最优实现
selectors 模块提供根据系统来自动选择最优的 IO 多路复用模型，简化开发过程并且更好地兼容各个系统：
- linux：epoll	
- windows：select	
- macOS：kqueue（原理上等同于 epoll）

使用方式：
``` py
import selectors

# 对应状态的事件表示，底层是二进制数字，可用按位或来合并事件，用按位与来判断事件
selectors.EVENT_READ      # 数据可读
selectors.EVENT_WRIT      # 可写数据

# 返回一个自动选择的 IO 多路复用模型对象 selector
selector = selectors.DefaultSelector()                  

# selector 对象的方法：

# 注册需要监测的文件描述符和事件标识
# data 是可附带的额外数据，对于 select() 返回结果中的 key，可通过 key.data 取出
selector.register(fd, eventmask, data=None)        

# 解除已注册的文件描述符
selector.unregister(fd)

# 阻塞直到超时或返回包含所有进入事件状态的 (key, events) 的列表
# key 是一个 {'fileobj': 文件对象, 'fd': 文件描述符, 'events': 事件标记, 'data': 附带数据} 的字典
# events 是事件标记
selector.select(timeout=None)               
            
# 关闭 selector 对象，使用完毕后需要关闭，否则会占有资源，包括文件描述符
selector.close()                                    


# 用例：实现并发的 socket 服务端
import selectors, socket

def accept(sock):
    conn, addr = sock.accept()
    print('accepted', conn, 'from', addr)
    conn.setblocking(0)
    mode.register(conn, selectors.EVENT_READ,handle)

def handle(conn):
    data = conn.recv(1000)
    if data:
        conn.send(data)
    else:
        mode.unregister(conn)
        conn.close()

mode = selectors.DefaultSelector()
sock = socket.socket()
sock.bind(('localhost', 9998))
sock.listen(1024)
sock.setblocking(0)
mode.register(sock, selectors.EVENT_READ, accept)

# 事件循环，循环监测 socket 的 io 事件，执行回调函数来处理 socket
while True:
    events = mode.select()
    for key, mask in events:
        callback = key.data
        callback(key.fileobj)
```

### 事件驱动
事件驱动一种编程模式，即程序的执行流由外部事件来所决定。其特点是包含一个处理线程，通常是一个事件获取循环，对已注册的事件进行监听和获取，一旦这些事件被外部触发，就会通过回调机制对其进行相应的处理

![](media/15922198014290.jpg)

结合多协程 + IO 多路复用所实现的协程事件驱动模型：
- 存在一个进行事件循环的处理协程，它会不断从事件列表迭代提取事件，将事件的回调作为任务来创建任务协程，再切换到任务协程中对事件进行处理

- 事件列表中的事件，不仅由外部事件源添加，也包括主协程通过调用 IO 多路复用获取的 IO 事件

- 任务协程执行中若遇到 IO 操作，会向 IO 多路复用对象注册监听相关文件描述符的事件，并创建一个切换回当前协程的回调，然后切换回主协程

- 当 IO 事件被主协程从事件列表中提取出来时，通过 IO 事件的回调就会切换回发起 IO 操作的协程中

**多协程让事件处理的异步逻辑可以同步方式编写**，解决了包含多个 IO 操作的任务事件处理中需要嵌套太多层回调的问题，以简单的协程切换代替了复杂的回调逻辑

**IO 多路复用事件处理可以避免 IO 数据准备的阻塞**，解决了主协程对阻塞于 IO 操作的协程，无法确定最佳切换时机的问题，配合协程切换实现了 IO 异步化 

对于实现高并发服务，瓶颈大多数时候在 IO 上面，包括内存 IO、磁盘 IO、网络 IO, 通过结合多协程 + IO 多路复用所实现的事件驱动模型，就能很好地解决这个瓶颈

## 实现
以下用例代码中的线程和进程都用对应执行函数来表示，以省略对象初始化和启动的重复代码

执行函数及其用到的相关变量，前者先定义则必须通过传参的方式来调用后者，前者后定义则可以直接调用后者，因为在子线程中变量会被共享，而在子进程中变量会被复制

### 多线程和多进程
Cpython 解释器中由于 GIL 的存在，多线程不适合计算密集型任务，只适合 IO 密集型任务，IO 密集型任务也可以通过多协程 + I/O 多路复用来解决，而计算密集型任务，则应该使用多进程，因为多进程编程在多处理器的环境下可以利用到所有处理器资源

#### threading
threading 模块在较低级的 _thread 模块基础上提供较高级的多线程接口，包括线程间同步状态的多种方式，如 **锁、信号量、条件变量、事件和栅栏**，[官方文档](https://docs.python.org/zh-cn/3/library/threading.html)

##### 线程管理
线程对象的获取：
``` py
from threading import Thread

# 有两种方法得到线程对象

# 1、直接初始化 Thread 类
# target 为可调用对象
# args、kwargs 分别是调用对象时传入的位置参数和关键字参数
# name 为线程名，默认为 Thread-N
# daemon 表示是否为 daemon 线程
thread = Thread(target=func, args=(), kwargs={}, name=None, daemon=None)

# 2、继承 Thread 类重写方法后，再进行初始化
class MyThread(Thread):
    def __init__(self, word):
        """
        初始化一些用于自定义任务的属性
        """
        super().__init__()
        self.word = word

    def run(self):
        """
        线程的执行入口，原本是调用 target(*args, **kwargs)
        可重写为自定义任务
        """
        print(f'my word is {self.word}')

thread = MyThread('example')
```

线程对象的常用方法和属性：
``` py
# 启动线程，线程是同时执行的，且由主线程创建
thread.start()

# 阻塞等待直到该线程执行结束或超时，默认永不超时
# 该方法并不会终止线程，需要紧接着通过 is_alive() 来判断线程是否超时
thread.join(timeout=None)

# 返回线程是否存活       
thread.is_alive()

# 设置线程是否为 daemon 线程，必须在线程启动前设置
# 也可以直接设置 daemon 属性
# 当主线程结束时，所有 daemon 线程会强制结束，若还有非 daemon 线程存活，则进程会等待所有线程结束时才结束
thread.setDaemon(True)
thread.daemon = True

# 返回线程名字，是只用于识别的字符串
thread.name

# 返回由解释器分配的线程标识符，若线程未开始则为 None
# 是一个非零整数，当线程结束后，该线程 ID 会被回收和复用
thread.ident

# 返回由内核分配的原生线程 ID，若线程未开始则为 None
# 是一个非负整数，当线程结束后，该线程 ID 会被回收和复用
# python 3.8 新属性
thread.native_id
```

常用的功能函数：
``` py
import threading

# 返回当前存活的线程数量
threading.active_count()

# 返回包含当前所有存活的线程对象的列表
threading.enumerate()

# 返回当前线程对应的线程对象
threading.current_thread()

# 返回当前线程的线程标识符
threading.get_ident()

# 返回当前线程的原生线程 ID，python 3.8 新方法
threading.get_native_id()

# 返回主线程对象
threading.main_thread()
```

##### 锁和信号量
锁的作用是限制资源被线程同时访问，修复竞态条件，以及保证操作的完整性，其使用方式：
``` py
from thrading import Lock, RLock

# 初始化线程锁对象
lock = Lock()

def thread_func():
    # 获取线程锁，没获得锁的线程需要阻塞等待
    # 可以设置阻塞超时，或者不阻塞，根据返回值判断是否获取到线程锁
    lock.acquire(blocking=True, timeout=-1)
    # 返回该锁是否已被获取
    lock.locked()
    pass
    # 释放线程锁
    lock.release()

# 初始化递归线程锁对象，一个线程可多次获取锁来实现锁中锁，但每个锁都需要释放
# 用于解决一个线程中使用多个线程锁，容易因为不同线程获取锁的顺序不同而导致死锁问题
rlock = RLock()

def thread_func():
    # 获取递归锁，没获得锁的线程需要阻塞等待
    # 可以设置阻塞超时，或者不阻塞，根据返回值判断是否获取到线程锁
    rlock.acquire(blocking=True, timeout=-1)
    rlock.acquire()
    pass
    rlock.acquire()
    pass
    # 释放递归锁
    rlock.release()
    rlock.release()
```

信号量的作用是限制资源同时访问的线程数，防止过度使用资源，其使用方式：
``` py 
from thrading import Semaphore, BoundSemaphore

# 初始化信号量对象
# value 设置计数器的初始值
semp = Semaphore(value=1)

# 初始化有界信号量对象，使用和信号量对象一致，当计数器的值大于初始值的时候触发异常
# 防止无限制地使用 release() 导致资源保护失败
bound_semp = BoundSemaphore(value=1)

def thread_func():
    # 获取信号量，其计数器 -1，当计数器为 0 时需要阻塞等待
    # 可以设置阻塞超时，或者不阻塞，根据返回值判断是否获取到信号量
    semp.acquire(blocking=True, timeout=None)
    pass
    # 释放信号量，其计数器 +1
    semp.release()
```

两种锁和信号量都支持上下文管理，在进入时获取锁，退出时释放锁：
``` py
with lock_or_semaphore_obj:
    pass
```

##### 条件变量和事件
条件表量的作用是通过锁在线程间同步某些共享状态从而进行活动，所有线程使用同一把锁，不允许同时进行活动，其使用方式：
``` py
from threading import Condition

# 初始化条件表量对象，需要和锁对象相关联，默认自动创建递归锁对象
# 可以多个条件变量对象共用一个锁对象
cv = Condition(lock=None)

# 调用底层锁对象来获取，条件变量也支持上下文管理，实现底层锁的获取释放
cv.acquire(blocking=True, timeout=-1)

# 调用底层锁对象来释放锁
cv.release()

# 释放锁（必须先获获取锁），阻塞直到其它线程使用条件表量进行通知，然后重新获得锁
# timout 可以设置等待超时，返回值为 False 则表示超时
wait(timeout=None)

# 通知指定数目个等待该条件变量的线程（必须先获取锁）
# 如果没有线程在等待，这是一个空操作
# 被唤醒的线程实际上不会立即返回其调用的 wait() ，而是继续阻塞直到有线程已经释放锁，因为 notify() 不会释放锁
cv.notify(n=1)

# 通知所有等待该条件变量的线程（必须先获取锁）
notify_all()

# 若指定条件为假则释放锁（必须先获得锁），阻塞直到其它线程使用条件表量进行通知且对指定条件为真，然后重新获得锁
# predicate 为返回布尔值的可调用对象
# timout 可以设置等待超时，返回值为 False 则表示超时
wait_for(predicate, timeout=None)


# 用例：设置和使用结果的两个线程
def set_thread():
    while True:
        with cv:
            set_result()
            cv.notify()

def get_thread():
    while True:
        with cv:
            cv.wait()
            get_result()


# 用例：为结果增加过期的标志
EXPRIED = 1

def get_expried()
    return EXPRIED

def set_thread():
    while True:
        with cv:
            set_result()
            global EXPRIED
            EXPRIED = 0
            cv.notify()

def get_thread():
    while True:
        with cv:
            cv.wait_for(get_expried)
            get_result()
            global EXPRIED
            EXPRIED += 1
```

事件的作用是让线程间可以直接通过事件通信，即一个线程发出事件通知，而其他线程则等待到事件后同时进行活动，其使用方式：
``` py
from thrading import Event

# 初始化事件对象
event = Event()

# 返回事件是否已设置标记
event.is_set()

# 为事件设置标记，所有正在等待这个事件的线程将被唤醒
event.set()

# 为事件清除标记
event.clear()

# 等待事件，若事件未设置标记则会阻塞，直到事件设置标记
# timout 可以设置等待超时，返回值为 False 则表示超时
event.wait(timeout=None)


# 用例：代表红路灯和汽车的两个线程
event = Event()

def traffic_light_thread():
    timer = 0
    while True:
        timer += 1
        if timer < 5:
            print('red light')
        elif timer == 5:
            event.set() 
        elif timer < 10:
            print('green light')          
        else:
            event.clear()
            timer = 0

def car():
    while True:
        if event.is_set():
            print('runing')
            time.sleep(1)
        else:
            print('stoping')
            event.wait()
```

##### 定时器和栅栏
定时器的作用是通过线程实现一个定时执行的任务，其使用方式：
``` py
from thrading import Timer

# 初始化定时器
# interval 为定时间隔秒数，func 为可调用对象，args、kwargs 为传入参数
timer = Timer(interval, func, args=None, kwargs=None)

# 启动定时器
timer.start()

# 取消定时器
timer.cancel()
```

栅栏的作用是应对固定数量的线程需要彼此相互等待的情况，当该线程等待数满足后，再同时释放所有线程，其使用方式：
``` py
from thrading import Barrier,BrokenBarrierError


# 初始化栅栏对象
# parties 为需要满足的等待线程数，timeout 为线程进行等待的默认超时
# action 是可调用对象，当所有线程被释放时，在其中一个线程中自动调用
# 若此时 action 调用失败，栅栏对象将进入损坏状态
# 栅栏对象进入损坏状态后，若仍有线程当前或未来还进行等待，则会触发 BrokenBarrierError 异常
barrier = Barrier(parties, action=None, timeout=None)

# 阻塞等待直到栅栏对象满足线程等待数或超时，超时后栅栏对象进入损坏状态
# 返回一个范围在 [0, barrier.parties] 的整数，每个线程的返回值不同，用于区分操作
barrier.wait(timeout=None)

# 重置栅栏为初始状态
# 重置时若仍有线程等待释放，这些线程将会收到 BrokenBarrierError 异常
barrier.reset()

# 使栅栏对象进入损坏状态
barrier.abort()

# 返回栅栏对象需要满足的等待线程数
barrier.parties

# 返回当前栅栏对象中阻塞等待的线程数量
barrier.n_waiting

# 返回栅栏对象是否为损坏状态
barrier.broken


# 用例：设置多个线程等待同时工作，工作前改变工作标志
work = False

def chang_flag():
    global work
    work = True

barrier = Barrier(2, chang_flag, timeout=10)

def work_thread():
    pass
    barrier.wait()
    pass
```

#### queue
queue 模块提供了多线程间传递消息的队列，在 threading 模块中是没有的，[官方文档](https://docs.python.org/zh-cn/3.7/library/queue.html#simplequeue-objects)

队列是有序的，用于进行数据存储和消费的数据容器，它可以：
- 解藕程序的功能
- 保证多线程的数据传递安全
- 提高程序运行效率 

队列对象的获取：
``` py
import queue

# 初始化普通队列对象
# maxsize 设置队列大小，默认无限制
q = queue.Queue(maxsize=0)

# 初始化后进先出（LIFO）队列对象
q = queue.LifoQueue(maxsize=0)

# 初始化优先级队列对象
# 插入数据时可以设置优先级，优先级的值越小越先被取出，取出的形式为  (priority_number, data) 元祖
q = queue.PriorityQueue(maxsize=0)

# 初始化无上限简单队列对象，没有任务追踪功能
# 即不提供 full()、join() 和 task_done() 方法
q = queue.SimpleQueue()
```

队列对象的方法：
``` py
# 插入数据，默认阻塞且永不超时
# 阻塞插入时若队列已满，则阻塞直到队列中的数据被消费掉
q.put(data, block=True, timeout=None)

# 非阻塞插入数据，若队列已满则触发 queue.Full 异常
# 相当于 put(data, (=False)
q.put_nowait(data)

# 取出数据，默认阻塞且不超时
# 阻塞取出时若队列为空，则阻塞直到队列有数据被插入
q.get(block=True, timeout=None)

# 非阻塞取出数据，若队列为空则触发 queue.Empty 异常
# 相当于 get(block=False)
q.get_nowait()

# 返回队列的当前大小
q.qsize()

# 返回队列是否为空
q.empty()
                        
# 返回队列是否已满
q.full()               

# 阻塞写入直到所有的数据都被取出且任务处理完毕
# 每当队列插入数据，未完成任务计数就会加 1，当未完成计数降为 0 时，join() 解除阻塞
q.join()

# 通知队列一个任务处理完毕，即队列的未完成任务计数减 1
# 若队列的未完成任务计数小于 0，则会触发 ValueError 异常
q.task_done()


# 用例：等待队列消费

# 插入队列，并阻塞队列直到消费完成
q = Queue()
for data in all_data:
    q.put(item)
q.join()

# worker 线程，用于消费队列
def worker_thread():
    while True:
        data = q.get()
        pass
        q.task_done()
        

# 用例：生产者消费者模式的实现，解藕并解决两者处理能力不平衡的问题

# 生产者线程
def producer_thread():
    num = 0
    while True:
        q.put(f'put data {num}' )
        num += 1

# 消费者线程，生产者和消费者同时执行
def comsumer_thread():
    while True:
        q.get(f'get data {num}' )
```

#### multiprocessing
multiprocessing 模块用于提供多进程的接口，有着和 threading 模块相似的接口，包括：
- 进程间同步状态的 **锁、信号量、条件变量、事件和栅栏**，是来自 threading 模块的原语等价物
- 进程间传递数据的 **管道和队列**
- 共享数据的 **共享类型和管理器**
- 高效使用进程的 **进程池** 

[官方文档](https://docs.python.org/zh-cn/3.7/library/multiprocessing.html#all-start-methods)

##### 进程管理
进程对象的获取：
``` py
from multiprocessing import Process

# 和 threadinng 的接口一样，有两种方法得到进程对象

# 1、初始化进程对象
# target 为可调用对象
# args、kwargs 分别是调用对象时传入的位置参数和关键字参数
# name 为线程名，默认为 Process-N
process = Process(target=None, name=None, args=(), kwargs={}, daemon=False)

# 2、继承 Process 类重写方法后，再进行初始化
class MyProcess(Process):
    def __init__(self, word):
        """
        初始化一些用于自定义任务的属性
        """
        super().__init__()
        self.word = word

    def run(self):
        """
        进程的执行入口，原本是调用 target(*args, **kwargs)
        可重写为自定义任务
        """
        print(f'my word is {self.word}')

process = MyProcess('example')
```

进程对象的常用方法和属性：
``` py
# 以下方法和属性与 Thread 线程对象保持一致：

process.start()
process.join(timeout=None)      
process.is_alive()
process.daemon = True    # 不允许 daemon 进程创建子进程
process.name


# 以下方法和属性是进程对象独有的：

# 返回由内核分配的原生进程 ID，若进程未开始则为 None
process.pid

# 返回进程的退出返回码，若线程未结束则为 None
process.exitcode

# 返回进程 byte 类型的身份验证密钥，可用于管理器对象的进程服务验证
# 是通过 os.urandom() 分配一个随机字符串
process.authkey

# 终止进程，在 Unix 上是使用 SIGTERM 信号完成的
process.terminate()

# 终止进程，与 terminate() 类似，但在 Unix 上使用 SIGKILL 信号
process.kill()

# 关闭进程，释放与之关联的所有资源
# 若进程仍在执行，会引发 ValueError
# 若进程成功关闭后，其大部分方法和属性将引发 ValueError
process.close()
```
> `start()` 、 `join()` 、 `is_alive()` 、 `terminate()` 和 `exitcode` 这些方法和属性只能由创建进程对象的进程调用

常用的功能函数：
``` py
import multiprocessing

# 返回当前进程所有存活的子进程列表
multiprocessing.active_children()

# 返回与当前进程对应的进程对象
multiprocessing.current_process()

# 返回系统的 CPU 数量
# 当前进程可用的 CPU 数量由 len(os.sched_getaffinity(0)) 方法获得
multiprocessing.cpu_count()

# 设置进程的启动方式，method 可设置 spawn、fork 和 forkserver 三种
# spawn：父进程启动一个新的 python 解释器进程
# fork：父进程使用 os.fork() 来产生 python 解释器分叉，Unix 中默认
# forkserver：启动进程服务器，父进程通过连接到服务器并请求分叉一个新进程
# 在程序中不应该被多次调用，设置和创建进程的代码要注意在 __name__='__main__' 条件下执行
multiprocessing.set_start_method(method)

# 返回上下文对象，上下文对象和 multiprocessing 模块拥有一样的接口，用于同步各个不同的进程启动方式
# method 为 None 表示默认
multiprocessing.get_context(method=None)

# 设置子进程时使用的 python 解释器路径
multiprocessing.set_executable()
```

##### 管道和队列
管道用于在两个进程间进行传递消息，不允许多个进程同时操作管道，其使用方式：
``` py
from multiprocessing import Pipe

# 创建管道，返回一对连接对象
# duplex 表示是否为双工管道，默认是，否则 conn1 仅能接收消息，conn2 仅能发送消息
conn1, conn2 = Pipe(duplex=True)

# 连接对象的使用：

# 发送对象到连接的另一端，必须是可以序列化且不能过大（接近 32MB，该值取决于操作系统）
conn1.send(obj)

# 返回接收到的对象，会一直阻塞直到接收到对象，若对端的连接关闭则触发 EOFError  异常
conn2.recv()

# 返回连接对象所使用的描述符
conn1.fileno()

# 关闭连接对象，连接对象被 gc 回收时会自动调用
conn1.close()

# 返回连接对象是否有可以读取的数据，默认立即返回，也可以指定进行阻塞等待，直到连接对象有可以读取的数据
# timeout 可以指定等待超时，若为 None 则永不超时
# 使用 multiprocessing.connection.wait() 可以一次性阻塞等待并轮询多个连接对象是否有可以读取的数据
conn2.poll([timeout])
```

队列用于多个进程间进行传递消息，同时是线程安全的，其使用方式：
``` py
from multiprocessing import Queue, JoinableQueue, SimpleQueue

# 初始化普通队列对象
# 使用方法和 queue.Queue 基本一致，除了没有 join() 和 task_done() 方法
# 当一个对象被插入队列中时，这个对象首先会被一个后台线程进行 pickle 序列化，并将结果通过一个管道传递到底层的队列
q = Queue(maxsize=0)

# 初始化可阻塞队列对象，是基于 Queue 的扩展
# 使用方法和 queue.Queue 基本一致，包括了 join() 和 task_done() 方法
q = JoinableQueue([maxsize])

# 初始化简化的无上限队列对象
# 只提供了 queue.Queue 中的 empty()、get()、put() 接口，且插入取出都只能是无超时阻塞的
q = SimpleQueue(maxsize=0)

# Queue 和 JoinableQueue 对象的一些额外方法：

# 指示当前进程不会再对队列进行插入数据，该方法在队列被 gc 回收时会自动调用
# 一旦所有缓冲区中的数据被写入管道之后，后台线程会退出
q.close()

# 等待后台线程退出，仅在调用了 close() 后使用，确保所有缓冲区中的数据都被写入管道
# 非队列创建的进程退出时会自动调用
q.join_thread()

# 防止进程退出时被 join_thread() 阻塞等待后台线程，用于立即退出，不在乎缓冲区的数据丢失
q.cancel_join_thread()
```

##### 共享类型和管理器
共享类型对象是一些分配来自共享内存的、可被子进程继承的 ctypes 对象，其使用方式：
``` py
# 更多的共享类型由 multiprocessing.sharedctypes 模块提供
from multiprocessing import Value, Array

# 初始化共享值对象
v = Value(typecode, value)

# 返回共享值对象的值，同时用于修改
v.value = new_value

# 类似 v.value += 1 的操作不具有原子性，要调用它的锁来保证其原子性
with v.get_lock():
    v.value += 1
     
# 初始化共享数组对象
l = Array(typecode, value_list)


# 用例：通过进程操作共享类型
n = Value('d', 1.01)
l = Array('i', range(5))

def work_process():
    pass
    n.value = 2
    l[2] = 3
```
> typecode 的类型对应如下：
> ![](media/15918614398226.jpg)

管理器的作用是通过控制一个服务进程，该进程可以保存数据并允许其他进程通过代理对象来对数据进行操作，实现本地甚至跨网络机器的进程间的数据共享，其使用方式：

**基础管理器**

``` py
from multiprocessing.manager import BaseManager

# 初始化基础管理器对象
# 基础管理器对象支持上下文协议，进行启动和关闭服务进程
# address 是管理器服务的监听地址，网络地址为 ('ip', port) 元组，默认为文件地址，即不对网络进行监听
# authkey 是 byte 类型的认证标识，用于检查连接服务的进程合法性，默认使用 current_process().authkey
# 当管理器对象被 gc 回收或者父进程退出时，服务进程会立即退出
manager = BaseManager(address=None, authkey=None)

# 为管理器对象启动一个服务进程
# initializer 是可调用对象，若不为 None , 则子进程启动时将调用 initializer(*initargs) 进行初始化
manager.start([initializer=None, initargs=())

# 阻塞启动服务，即 start() 所启动服务进程的运行方法
manager.get_server().serve_forever()

# 将本地管理器对象连接到一个远程的管理器服务进程，自身不需要启动
manager.connect()

# 停止管理器对象的服务进程，只能用于已通过 start() 启动的服务进程
manager.shutdown()

# 将一个类型或者可调用对象注册到管理器对象中成为共享类型，是个类方法
# typeid 是用于唯一表示该共享类型的字符串标识
# callable 是类型或者可调用对象，用于通过管理器对象创建共享类型对象
manager.register(typeid, callable)
```

**使用本地管理器**

``` py
from multiprocessing.manager import SyncManager
from multiprocessing import Manager

# 初始化同步管理器对象，该类是在父类 BaseManager 之上注册了很多类型作为共享类型
# 支持的共享类型有：list、dict、Namespace、Lock、RLock、Semaphore、BoundedSemaphore、Condition、Event、Barrier、Queue、Value 和 Array
manager = SyncManager(address=None, authkey=None)

# 初始化管理器对象，本质是已调用 start() 启动的 SyncManager 同步管理器对象
# 其支持的方法以及类型：list、dict、Namespace、Lock、RLock、Semaphore、BoundedSemaphore、Condition、Event、Barrier、Queue、Value 和 Array
manager = Manager(address=None, authkey=None)


# 用例：使用管理器对象来共享对象
manager = Manager()
d = manager.dict()
l = manager.list(range(10))

def f_process():
    d[1] = '1'
    d['2'] = 2
    d[0.25] = None
    l.reverse()
    
    
# 用例：注册自定义类型为共享类型，可以基于 SyncManager 扩展，或者基于 BaseManager 完全自定义
# 自定义类型必须是可序列化的
class MathsClass:
    def add(self, x, y):
        return x + y
    def mul(self, x, y):
        return x * y

# class MyManager(SyncManager):
class MyManager(BaseManager):
    pass

MyManager.register('Maths', MathsClass)
math = MyManager.Maths()

def f_process():
    math.add(3, 2)
    math.mul(2, 1)
```

**使用远程管理器**

``` py
# 可用于实现简单的进程分布式方案

# 本地服务进程通过网络提供服务
manager = SyncManager(address=('', 50000), authkey=b'abracadabra')
manager.start()
q = manager.Queue()
q.put('1')

# 本地服务进程连接到远程服务
# 本地的管理器对象需要远程服务的管理器对象一致
manager = SyncManager(address=('', 50000), authkey=b'abracadabra')
manager.connect()
q = manager.Queue()
q.put('2')
```

##### 进程池
进程池的作用是通过维护和控制一些工作进程，来执行提交给它的任务，其使用方法：
``` py
from multiprocessing import Pool

# 初始化进程池对象
# processes 是工作进程的数目，默认使用 os.cpu_count() 的值
# initializer 是可调用对象，当其不为 None 时，则每个工作进程将会在启动时调用 initializer(*initargs) 进行初始化
# maxtasksperchild 是一个工作进程的最大完成任务数，达到之后将会退出或被代替，目的是为了释放未使用的资源，默认无限制
pool = Pool(processes=None, initializer=None, initargs=(), maxtasksperchild=None)

# 提交任务，会阻塞直到任务被执行并返回结果
pool.apply(func=callable, args=(), kwds={})

# 批量提交任务，会阻塞直到任务被执行并返回结果迭代器，和内置的 map() 类似
# iterable 是表示每个任务的参数的可迭代对象，只支持单个参数
# chunksize 是将可迭代对象分割为许多任务块提交时，每块的任务数，对于很长的可迭代对象，设置大一些会执行更快，但内存消耗也更多
pool.map(func=callable, iterable=[], chunksize=1)

# 异步提交任务，立即返回一个 multiprocessing.pool.AsyncResult 结果对象 
# callback、error_callback 都是可调用对象，分别表示任务成功和失败的回调
# 回调函数应该立即执行完成，否则会阻塞负责处理结果对象的线程
pool.apply_async(func=callable, args=(), kwds={}, callback=None, error_callback=None)

# 异步批量提交任务，立即返回一个 multiprocessing.pool.AsyncResult 结果对象 
pool.map_async(func=callable, args=(), kwds={}, callback=None, error_callback=None)

# 阻止后续任务提交到进程池，当所有任务执行完成后，工作进程会退出
pool.close()

# 不必等待未完成的任务，立即停止工作进程，当进程池对象被 gc 回收时，会立即调用
pool.terminate()

# 等待所有工作进程退出
# 调用 join() 前必须先调用 close() 或者 terminate()
pool.join()


# multiprocessing.pool.AsyncResult 结果对象的方法：

# 阻塞直到返回执行结果或超时，若执行触发了异常也会通过该方法抛出
# timeout 表示等待超时，超时则触发 multiprocessing.TimeoutError 异常
rst = get(timeout=None)

# 阻塞直到执行完成
rst.wait(timeout=None)

# 返回执行是否已完成
rst.ready()

# 返回执行是否已完成且未触发异常，若未完成则触发 ValueError
successful()
```

#### concurrent.futures
concurrent.futures 模块提供底层为多线程或多进程的异步执行高层接口，封装了 threading 和 multiprocessing 模块，[官方文档](https://docs.python.org/zh-cn/3.7/library/concurrent.futures.html?highlight=concurrent.futures#module-concurrent.futures)

**任务提交**

``` py
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# 初始化以多线程为底层的线程池执行器对象
# max_workers 是最大工作线程数，默认为机器的处理器个数，若为 I/O 密集型任务可以设置为乘 5 的数
# thread_name_prefix 是线程命名前缀
# initializer 是可调用对象，当其不为 None 时，则每个工作线程将会在启动时调用 initializer(*initargs) 进行初始化
t_executor = ThreadPoolExecutor(max_workers=None, thread_name_prefix='', initializer=None, initargs=())

# 初始化以多进程为底层的进程池执行器对象
# max_workers 是最大工作进程数，默认为机器的处理器个数
# mp_context 是多进程上下文，无设置则使用默认
# initializer 是可调用对象，当其不为 None 时，则每个工作进程将会在启动时调用 initializer(*initargs) 进行初始化
p_exector = ProcessPoolExecutor(max_workers=None, mp_context=None，initializer=None, initargs=())


# 两个执行器对象拥有一致的接口
# 执行器对象支持上下文管理，在退出时自动调用 shutdown(wait=True) 等待所有任务完成并关闭

# 提交任务，返回一个代表结果的 concurrent.futures.Future 对象
# fn 是可调用对象，底层执行是调用 fn(*args **kwargs)
exector.submit(fn, *args, **kwargs)

# 批量提交任务，返回一个包含结果的有序迭代器
# fn 是可调用对象，底层执行是调用 fn(*iterables.__next__())
# timeout 是迭代该迭代器时阻塞等待每个结果返回的超时，默认永不超时
# chunksize 是将可迭代对象分割为许多任务块提交时，每块的任务数，对于很长的可迭代对象，设置大一些会执行更快，但内存消耗也更多，对线程池执行器对象无效
exector.map(func, *iterables, timeout=None, chunksize=1)
 
# 关闭执行器对象，释放所有资源
# wait 表示是否阻塞等待所有的任务执行完成
exector.shutdown(wait=True)
```

**结果处理**

``` py
# 期望对象的方法和属性：

# 尝试取消任务，并返回是否成功取消
# 如果任务正在执行或已结束则不能被取消
fu.cancel()

# 返回任务是否已被取消
f.cancelled()

# 返回任务是否正在执行
f.running()

# 返回任务是否已被取消或结束
f.done()

# 返回任务结果，若任务未完成则阻塞等待直到任务结束或者超时，默认永不超时
# 等待超时则触发 concurrent.futures.TimeoutError 异常
# 等待期间任务被取消则触发 concurrent.futures.CancelledError
# 若任务过程触发了异常，该方法会直接抛出异常
f.result(timeout=None)

# 返回由任务触发的异常，若任务未完成则阻塞等待直到任务结束或者超时，默认永不超时
# 如果调用正常完成则返回 None
f.exception(timeout=None)

# 添加一个任务完成后的回调
# fn 是可调用对象，当任务被取消或完成运行时，期望对象将会执行 fn(self) 进行回调
# 如果任务已完成或取消，会立即执行回调
f.add_done_callback(fn)


# 常用函数

from concurrent.futures import wait, as_completed

# 阻塞等待列表中所有期望对象直到满足返回条件或超时，返回一个 (done_fs, not_done_fs) 元组
# done_fs 包含正常结束或被取消的期望对象，not_done_fs 包含挂起的或正在运行的期望对象
# return_when 指定函数的返回条件，有以下三种：
# FIRST_COMPLETED：当任意一个期望对象结束或取消时返回
# FIRST_EXCEPTION：当任意一个期望对象因触发异常而结束时返回
# ALL_COMPLETED：当所有期望对象结束或取消时返回
wait(fs, timeout=None, return_when=ALL_COMPLETED)

# 返回一个包含所有期望对象的迭代器
# 迭代该迭代器时，将阻塞等待直到超时或任意一个期望对象完成或被取消，并返回该期望对象
# 若等待超时则触发 concurrent.futures.TimeoutError 异常
concurrent.futures.as_completed(fs, timeout=None)


# 用例：快速使用多线程执行任务
def task(arg):
    pass

with ThreadPoolExecutor(max_workers=4) as executor:
	fs = [executor.submit(task, i) for i in range(10)]
	
	# 任务每完成一个就处理一个
	for f in as_completed(fs):
	   try:
	       print(f'succeed:{f.result()}')
	   except:
	       print(f'failed:{f.exception}')

# 在执行器的上下文管理执行 shutdown() 后，任务全部完成后统一处理
for f in fs:
	   try:
	       print(f'succeed:{f.result()}')
	   except:
	       print(f'failed:{f.exception}')
```

### 事件驱动（多协程 + IO 多路复用）
#### asyncio
asyncio 模块是一个基于多协程 + IO 多路复用的事件驱动框架，使用 async/await 语法，实现基于协程的并发和 IO 异步化，[官方文档](https://docs.python.org/zh-cn/3/library/asyncio.html)

所提供的高级接口有：
- 协程和任务
- 协程间的同步和消息传递
- 协程的流控制
- 协程的子进程控制

##### 协程和任务
协程的定义和打包为任务执行：
``` py
import asyncio

# async def 用于定义一个协程函数，调用协程函数将返回一个协程对象
async def coro_func():
    pass
    # await 表示异步等待，即释放执行权直到可等待对象执行结束，并返回其结果
    # aw 是可等待对象，包括协程对象（coroutine）、任务对象（Task）和期望对象（Future）
    # 若 aw 是协程对象，
    rst = await aw
    pass

# 协程的执行是通过把协程对象打包为任务对象来完成的，任务对象的初始化过程会将协程对象打包后排入事件循环中执行
# 执行协程的三种方式：

# 1、执行协程对象并返回执行结果
# 总是新建并运行一个事件循环，来执行打包为任务对象的协程对象，并在结束时关闭，应该用于运行程序入口协程
# 当前线程存在其他运行中的事件循环时，会触发 async.RuntimeError 异常
# debug 表示是否开启调试模式
asyncio.run(coro, debug=False)

# 2、将协程对象打包为任务对象执行，返回该任务对象
# name 表示任务名
# 会通过 get_running_loop() 获取事件循环，来执行打包为任务对象的协程对象
asyncio.create_task(coro, name=None)

# 3、在协程函数中通过 await 异步等待协程对象
# 协程对象会自动打包为任务对象，并排入当前线程中正在运行的当前事件循环中执行
await coro


用例：包含三种协程的执行方式
async def step1():
    pass

async def step2():
    pass    

async def main():
    # 异步等待
    await step1()
    # 先打包为任务对象执行再异步等待
    task2 = asyncio.step2()
    await task2

asyncio.run(main())
```

任务是期望的子类，任务所提供的高级接口也建立在期望的低级接口之上，因此两种对象的方法和属性也基本一致：
``` py
# 任务对象被用来在事件循环中执行协程
# 任务对象可以通过高级接口 asyncio.create_task() 来创建，也可以通过低级接口 loop.create_task() 或 asyncio.ensure_future() 来创建

# 期望对象被用来保存异步函数的结果
# 期望对象属于不应该被暴露和直接使用的低级对象，通常使用 await 异步等待来直接返回对应结果


# 取消任务对象的执行，将在被打包的协程中触发  CancelledError 异常
# 若取消期间协程正在异步等待一个对象，那该对象也将被取消
# 该方法不能保证任务对象会被取消，因为在协程中可以通过异常捕捉来拒绝取消
task.cancel()

# 返回任务对象是否已取消，即打包的协程没有抑制 CancelledError 异常并且确实被取消
task.cancelled()

# 返回任务对象是否已完成，包括触发异常或被取消
task.done()

# 返回任务对象的结果
# 若任务对象已完成，其打包的协程的结果会被返回
# 若任务对象已取消则触发 CancelledError 异常，而未完成则触发 InvalidStateError 异常
task.result()

# 若任务对象已完成，其打包的协程的异常会被返回，若无触发异常则返回 None
# 类似 task.result()，若任务对象已取消未完成则触发异常
task.exception()

# 返回任务对象打包的协程对象
task.get_coro()

# 返回任务对象的名称
task.get_name()

# 设置任务对象的名称
task.set_name(value)
```

返回可等待对象的函数：
``` py
# 返回一个阻塞指定秒数的协程对象
# result 指定协程对象的结果
# loop 默认为 get_event_loop() 返回的事件循环
asyncio.sleep(delay, result=None, loop=None)

# 返回一个期望对象，等待所有可等待对象都执行完成，其结果是一个包含这些可等待对象的结果的有序列表
# aws 是可等待对象列表
# return_exceptions 默认为 False，当首个可等待对象触发异常时，期望对象的结果会抛出异常，但其他可等待对象不会被都取消
# 为 True 则异常会和成功结果一样，被聚合到列表中作为期望对象的结果
asyncio.gather(*aws, loop=None, return_exceptions=False)

# 返回一个期望对象，等待可等待对象执行完成，其结果是可等待对象的结果，但不允许进行取消
# aw 是可等待对象
# 即使包含它的协程被取消，可等待对象也不会被取消
asyncio.shield(aw, loop=None)

# 返回一个协程对象，等待指定可等待对象执行完成或超时，其结果是可等待对象的结果
# 若 aw 是协程对象，并被自动创建为任务
# timeout 若为 None 则永不超时，否则表示秒数
asyncio.wait_for(aw, timeout, loop=None)

# 返回一个协程对象，等待所有可等待对象达到返回条件或者超时，其结果是（done, pending）元祖
# done 表示已完成的可等待对象列表，pending 表示未完成的可等待对象列表
# 若 aws 中存在协程对象，不会被自动创建为任务，应自行先创建为任务对象再进行传入
# return_when 指定函数的返回条件，有以下三种：
# FIRST_COMPLETED：当任意一个期望对象结束或取消时返回
# FIRST_EXCEPTION：当任意一个期望对象因触发异常而结束时返回
# ALL_COMPLETED：当所有期望对象结束或取消时返回
asyncio.wait(aws, loop=None, timeout=None, return_when=ALL_COMPLETED)

# 返回一个期望对象，线程安全地在指定事件循环中执行协程对象，其结果是协程对象的执行结果
# 应在非事件循环所属的线程中调用
asyncio.run_coroutine_threadsafe(coro, loop)
```

常用的功能函数：
``` py
# 返回当前事件循环中运行中的任务对象，没有则返回 None
asyncio.current_task(loop=None)

# 返回当前事件循环中所有任务对象的集合
asyncio.all_tasks(loop=None)

# 返回当前线程中正在运行的当前事件循环对象，若不存在则触发 asyncio.RuntimeError 异常
# 该函数只能在协程或回调中调用
asyncio.get_running_loop()

# 返回当前线程正在运行的当前事件循环对象，若不存在且当前线程为主线程、未设置过当前事件循环对象，则自动创建并设置，但不会运行
# 该在协程或回调中调用相当于 asyncio.get_running_loop()
asyncio.get_event_loop()

# 创建并返回一个事件循环对象
asyncio.new_event_loop()

# 将指定事件循环对象设置为当前线程的当前事件循环对象
asyncio.set_event_loop(loop)
```

#### gevent
gevent 第三方模块进一步封装了 greenlet，实现了一个基于多协程 + IO 多路复用的事件驱动框架，实现协程进行 IO 操作时的自动切换，达到 IO 异步化的高效率并发，[官方文档](http://www.gevent.org/contents.html)

##### greenlet
利用生成器中的 yield 进行用户态的上下文切换，功能上实现了不便于进行创建和切换的协程，greenlet 第三方模块则通过 C 扩展提供了较高级的协程管理接口，[官方文档](https://greenlet.readthedocs.io/en/latest/)

``` py
import greenlet
# 初始化 greenlet 对象，相当于一个以函数为执行内容的协程
gr = greenlet.greenlet(func)

# greenlet 对象的方法：

# 切换到该 greenlet 对象
# 首次切换表示开始执行协程，可以传入执行参数
# 再次切换到该协程的上下文继续执行              
gr.switch(*args, **kwargs)


# 用例：实现两个互相切换的协程
def func1(*args):
    gr2.switch(*args)
    while True:
        gr2.switch()
        time.sleep(0.5)

def func2(*args):
    while True:
        gr1.switch()
        time.sleep(0.5)

gr1 = greenlet(func1)
gr2 = greenlet(func2)

gr1.swith('a', 'b') 
```

##### 使用方式
``` py
import gevent

# 为 IO 操作的相关模块和方法进行补丁，使协程进行 IO 操作时能自动切换，使得 IO 操作异步化
# 协程进行 IO 操作时，会自动向 IO 多路复用 selector 注册监听 IO 设备，并释放执行权，事件循环待通过 IO 多路复用得知 IO 操作的已完成数据准备阶段后，再切换回该协程
from gevent import monkey;monkey.patch_all()

# 返回一个 greenlet 对象，将以函数为执行内容的协程排入事件循环中执行，并传入执行参数
gr = gevent.spwan(func, *args, **kwargs)     

# 阻塞直到列表中的所有 greenlet 对象执行完毕或超时，返回执行完毕的 greenlet 对象列表
# timeout 是等待超时，默认永不超时
# raise_error 表示当 greenlet 对象触发异常，是否停止阻塞并抛出异常
gevent.joinall(grs, timeout=None, raise_error=False) 

# 使当前协程至少睡眠指定的秒数，会释放执行权
gevent.sleep(second) 


# 用例：实现并发的 socket 服务端
def handle_request(conn, cli_addr):
    while True:
        host, port = cli_addr
        data = conn.recv(1024)
        if not data: break
        respond = "your host is %s, your port is %s, your word is %s" % (host, port, data.decode())
        conn.send(respond.encode())

ss = socket.socket()
ss.bind(('', 6666))
ss.listen()
while True:
    conn,cli_addr = ss.accept()
    gevent.spawn(handle_request,conn,cli_addr)
```