---
title: golang并发学习笔记
date: 2018-10-13 06:28:05
tags: [golang, 并发]
---

golang的并发模型叫做CSP（communicating sequential process），称为通信顺序进程模型，模型由独立并发执行的实体组成（go块），模型之间的通信通过channel来实现。因此，golang的并发模型哲学是：万物皆通信！！golang的核心概念主要是：
1. channel
2. go块

## channel

- channel可以单独创建，在进程之间传递
- channel是`线程安全`队列，任何持有channel引用的任务(go块)都可以读写channel
- channel默认是无缓冲区的，也就是channel本身是同步的，一端写数据操作必然会阻塞直到channel的数据被别的地方读取
- channel可以关闭，向关闭的channel读数据会读到的默认值，向关闭的channel写数据会导致panic！！

        func main() {
            ch := make(chan int)

            go func() {
                ch <- 20
                close(ch)
                ch <- 30 //panic: send on closed channel
            }()
            println(<-ch) // 20
            println(<-ch) // 0(默认值)
        }

- 有缓冲区的channel，根据缓冲区已满时候的策略，可以分为
    - 阻塞型：写入阻塞
    - 弃用新值：新值写入被抛弃
    - 移除旧值：太旧的数据被channel抛弃

- channel和队列很像
    在 `golang`里，channel可以用 `for i := range channelName {}`循环获取channel信息

## goroutine


### 线程模型的缺点

java和c++的并发模型都是线程模型，它的好处是直接对硬件的抽象，大多数语言，包括python，它的线程模型都是操作系统线程，但是坏处是使用复杂。

但是线程模型有三个危害
> 1. 竞态条件
> 2. 死锁
> 3. 内存可见性问题
>  引用自《七周七并发编程模型》



    public class Test {
        static boolean ready = false;   // 竞态条件一：共享变量
        static int data = 0

        static Thread t1 = new Thread() {
            public void run() {
                data = 10;  // 竞态条件二：会有并行实体(线程)修改变量
                ready = true;
            }
        };

        static Thread t2 = new Thread() {
            public void run() {
                if (ready) {    // 竞态条件三：一个未处理完成另外一个处理可能会介入
                    System.out.Println("data is :" + data)
                } else {
                    System.out.Println("no data")
                }
            }
        };

        public static void main(String[] args) throw InterruptedException{
            t1.start();
            t2.start();

            t1.join();
            t2.join();
        }
    }

尽管线程模型问题很多，但是线程模型是其他模型的基础，比如nodejs的异步io模型，本质上也是基于线程池技术实现的，java的nio底层实现也是基于线程池。

线程池是多线程模型的改良，线程的启动和运行都有一定的开销，为了避免直接创建线程，才有了线程池，线程池方便了线程的复用，但是涉及线程通信的时候，如果线程被阻塞，那这个线程的资源永远都被占用者，线程池就显得鸡肋了。nodejs的决绝方法是限制程序员的代码风格，使之变成`事件驱动`的形式。

### goroutine调度机制和状态机

所谓事件驱动是指node.js会把所有的异步操作使用事件机制解决，有个线程在不断地循环检测事件队列。

node.js中所有的逻辑都是事件的回调函数，所以node.js始终在事件循环中，程序入口就是事件循环第一个事件的回调函数。事件的回调函数中可能会发出I/O请求或直接发射（ emit）事件，执行完毕后返回事件循环。事件循环会检查事件队列中有没有未处理的事件，直到程序结束，因此，node.js 是单线程，异步非阻塞

node的这种方式有几个问题：
1. CPU密集型任务存在短板
2. 无法利用CPU的多核
3. 代码变得难以阅读
4. 回调函数保存数据需要经常用到全局变量

golang本质上也是使用了事件驱动的机制，但是这个过程对我们是透明的，主要解决了第三个问题，原理是把每个go块(`go func(){}()`)当层成了一个状态机，当go块从channel里读写，遇到阻塞的时候，go块进入暂停状态，让出线程控制权，代码可以继续进行的时候，状态扭转，go块继续运行(可能不在原来的线程上了) 

#### goroutine和线程

- 每个os线程有一个固定大小的栈内存（通常2MB）
- goroutine的栈空间大小不固定，开始通常是2KB，按需扩展，最大可达1GB
- go运行时候有自己的goroutine调度算法，称为m：n调度，m个goroutine运行在n个线程上

同是调度算法，为何go的调度算法如此优秀？

其实不然，和操作系统线程调度器对比，主要不同在于：
1. os内核调度上下文切换开销大，go调度器只需要调度一个go程序自己的goroutine，更容易hold住
2. os调度是硬件时钟中断触发的，goroutine调度的触发是channel读写阻塞或者`time.Sleep()`来实现的，因此不需要切换到内核态。

>goroutine没有标识，线程是有自己的标识的，因此可以方便的实现一个线程局部变量(`map[thread_symbol]object`),在web服务器上，线程局部变量通常会被用来存储http请求信息。在goroutine上没有这种机制，鼓励更简单的编程风格。
> ——《go程序设计语言》