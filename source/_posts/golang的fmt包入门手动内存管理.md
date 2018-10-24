---
title: 从golang的fmt包入门手动内存管理
date: 2018-10-17 22:01:43
tags: timeline重构任重道远
---

## 矫情的话

在做feed流开发的时候，我负责timeline的业务开发，刚开始设计的时候我以为也就是个业务代码开发，能有啥难度。结果开发完了之后，被leader疯狂吐槽。代码组织不好，这些都能通过对业务的深入理解，去重新设计，但是说到一个内存管理的问题，我是完全没想到的，以前没有接触过高并发场景，不知道在高并发场景下，依赖语言自身的gc会导致内存的频繁申请和回收。

痛定几周之后决定思痛，要参考学习优秀的代码
于是我想，哪里会有优秀的涉及到内存管理的代码呢！

官方库！！

然后想，timeline涉及网络io，io才会有大量的内存分配和回收的场景！！

直接看net包？太尼玛复杂了
ok，看fmt包

## fmt包源码摘要和笔记

> 来自fmt/print.go

    // Fprintf根据w的不同，调用w的write方法，很容易做到打印日志到不同地方
    func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
        p := newPrinter()
        p.doPrintf(format, a)
        n, err = w.Write(p.buf)
        p.free()
        return
    }

    // Printf调用了Fprintf，打印的地方是os.Stdout
    func Printf(format string, a ...interface{}) (n int, err error) {
        return Fprintf(os.Stdout, format, a...)
    }

    func Sprintf(format string, a ...interface{}) string {
        p := newPrinter()
        p.doPrintf(format, a)
        s := string(p.buf)
        p.free()
        return s
    }

这里至少有三点可以学:
1. 包本身就是模块化的一种方式，对外提供的函数不一定非得属于某个对象
2. 接口作为参数的好处：封装变化
    这里，变化指的是[]byte的去向，比如os.Stdout
3. `p := newPrinter()`这里采用了临时对象池来实现内存的管理

看下去：

    // pp is used to store a printer's state and is reused with sync.Pool to avoid allocations.
    type pp struct {
        buf buffer
        // 省略
    }

    var ppFree = sync.Pool{
        New: func() interface{} { return new(pp) },
    }

    // newPrinter allocates a new pp struct or grabs a cached one.
    func newPrinter() *pp {
        p := ppFree.Get().(*pp)
        p.panicking = false
        p.erroring = false
        p.fmt.init(&p.buf)
        return p
    }

    // free saves used pp structs in ppFree; avoids an allocation per invocation.
    func (p *pp) free() {
        p.buf = p.buf[:0]   // 清空slice
        p.arg = nil 
        p.value = reflect.Value{}
        ppFree.Put(p)   // 放回对象池里
    }



一个sync.Pool对象就是一组临时对象的集合。Pool是协程安全的。
Pool用于存储那些被分配了但是没有被使用，而未来可能会使用的值，以减小垃圾回收的压力。

fmt包总是需要使用一些[]byte之类的对象，golang建立了一个临时对象池，存放着这些对象，如果需要使用一个[]byte，就去Pool里面拿，如果拿不到就分配一份。
这比起不停生成新的[]byte，用完了再等待gc回收来要高效得多

## sync.Pool测试

    // 一个[]byte的对象池，每个对象为一个[]byte
    var bytePool = sync.Pool{
        New: func() interface{} {
            b := make([]byte, 1024)
            return &b
        },
    }

    func main() {
        a := time.Now().Unix()
        // 不使用对象池
        for i := 0; i < 1000000000; i++ {
            obj := make([]byte, 1024)
            _ = obj
        }
        b := time.Now().Unix()
        // 使用对象池
        for i := 0; i < 1000000000; i++ {
            obj := bytePool.Get().(*[]byte)
            _ = obj
            bytePool.Put(obj)
        }
        c := time.Now().Unix()
        fmt.Println("without pool ", b-a, "s")
        fmt.Println("with    pool ", c-b, "s")
    }

> 来自：[go的临时对象池--sync.Pool
](https://www.jianshu.com/p/2bd41a8f2254)

测试效果：

    // 数据量更大更明显
    without pool  21 s
    with    pool  16 s

the end...

