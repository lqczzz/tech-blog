---
title: unix网络编程笔记-tcp编程基础
date: 2018-10-21 14:00:26
tags: tcp/ip
---

## 关于《unix网络编程》

unix网络编程这书分为两册：
1. 一册讲socket编程，700多页，31章
2. 二册讲进程之间的通信，400页，16章

看目录可以看出，最基础最核心的知识在：
- chapter1: 简介
- chapter2: tcp／udp和stcp(这玩意不管)
- chapter3: 套接字编程简介
- chapter4: 基本tcp套接字编程
- chapter5: tcp客户／服务端程序实例
- chapter30: 客户／服务程序设计范式

感觉这几章学完了就差不多够了，其他章节需要再读～


## 阅读前提

假设你对OSI七层模型有所了解

## chapter0:简介

### 从现代交换技术说起

《现代交换技术》是通信专业的必修课，嗯嗯，好歹我也是通信专业的学生，就先把知识脉络拓展一下～

现代交换技术的分类：
1. 电路交换
2. 分组交换

计算机网络的协议用的就是分组交换技术，我们发送的信息会像快递包裹一样一个个的传送到接收方。而电路交换很简单，就是每个通信实体都连接到交换机上，而交换机使用交换的方法，让实体之间可以很方便地通信，现在最广泛的应用就是电话网络了。
从打电话也可以看出来，电路交换一定是：
1. 面向连接；(分组交换则不一定，如udp协议)
2. 同步时分复用；
3. 信息传送无差错控制；

### chapter1:分组交换协议

![分组交换协议](/images/net/分组交换协议.png)
1. PDU: 协议数据单元，即对等实体(处于同一层)之间的交换单元信息
2. SDU: 下一层承载上一层数据的单元，比如tcp层传输的tcp报文(报文头+报文体)数据在tcp层就是一个PDU，传给ip层之后，ip层认为它是SDU（ip层在tcp报文之外加入ip报文头，类似俄罗斯套娃）

不同协议之间的不同完全取决于协议头（废话～）

### tcp／ip简介
1. 一般认为web服务器程序是长时间运行的程序，即所谓的守护程序
2. 用户进程定义应用协议，tcp和ip协议的转换和包装在内核协议栈中，由操作系统提供支持
![](/images/net/1.1.png)
3. tcp是没有记录边界的字节流协议
    
    - tcp应用进程之间是没有长度限制的字节流，udp进程交换的数据长度不能超过udp发送缓冲区大小的单个记录(record)
    - tcp协议：应用程序一次次输出操作写到socket的数据经过顺序分割，得到分节(segment)，*数据量太大的时候，我们无法确保一次read到所有的数据，所以必须要把read编写在某个循环中*
    - tcp没有边界,所以tcp服务需要自己实现，提供一个表示长度的协议头
4. ip报文的SDU最大是65535，所以tcp一次发送的报文大小不会超过64k
    对于平常实用的`conn.Write([]byte)`，我们是不用考虑这些，操作系统会对这类阻塞写操作进行自动分片并且不用考虑缓冲区写满的情况
5. 套接字编程是应用层进入传输层的接口
    - 这样设计由两个理由：
        1. 应用层对通信细节很少关心，而底下四层对应用协议不关心，只关心如何通信
        2. 应用层常构成用户进程，地下四层作为操作系统和内核的机制，存在与内核态
    - socket可以绕过tcp和udp直接实用ipv4/ipv6，这种socket称为原始套接字(raw socket),很少用到，在整本书里面第28章介绍了它的两个用途：
        1. ping
        2. traceroute
    因此不打算深入了解了

![osi和网际协议的对应关系](/images/net/osi和网际协议的对应关系.png)
6. `netstat`和`ifconfig`可以很方便的查看网络的细节

#### 案例分析

##### bug
这里记录一个工作中遇到的bug：

    // 没有for循环读取数据
    func request(conn net.Conn, buffer bytes.Buffer, command []byte) error {

        // 读协议头，得到body的长度
        recvBuf := make([]byte, 4)
        resHead := binary.LittleEndian.Uint32(recvBuf)
        
        // 指定读取数据的大小，读取数据，bug:读取不完整
        var resBody bytes.Buffer
        recvBuf = make([]byte, resHead)
        length, err := conn.Read(recvBuf)
        if err != nil {
            return err
        }
        return nil
    }

bug分析：

##### 原因

1. 原因1:

![bug](/images/net/bug_coreserver.png)
socket上的read和write(操作系统的系统调用)不同于通常的文件读写，可能的到的字节数比预期的要少，原因在于内核缓冲区可能数据不够(read)或者缓冲区已经满了(non block write),上面的主要问题是read的时候缓冲区的数据不够，在项目中，由于网络原因，当我们
  
    var recvData = make([]byte, Size)
    conn.Read(recvData)

这样获取数据，由于网络不稳定，可能缓冲区的数据足够，可能不够，所以出现了调用20次成功一次的情况
既然如此,为什么go实现`conn.Read()`为什么不帮我们阻塞去等待数据的到来呢

很遗憾`Read`没有这样的能力，go也没有提供类似c的`Readn`这样的接口

> If some data is available but not len(p) bytes, Read conventionally returns what is available instead of waiting for more.
> 来自 go io包Read接口的注释

2. 原因2:

    不知道服务器端发送的逻辑(也不应该依赖它)，可能是
        
        for {
            conn.Write() // 手动分片
        }
    也可能是：
        
        conn.Write([]整个数据)


##### 解决

套一层for循环

    // 修改成for循环读取数据，bug解决
    func request(conn net.Conn, buffer bytes.Buffer, command []byte) error {

        // 读协议头，得到body的长度
        recvBuf := make([]byte, 4)
        resHead := binary.LittleEndian.Uint32(recvBuf)
        
        // 指定读取数据的大小，读取数据，bug:读取不完整
        var resBody bytes.Buffer
        recvBuf = make([]byte, resHead)
        for resBody.Len() < int(resHead) {
            length, err := conn.Read(recvBuf)
            if err != nil {
                return err
            }

            resBody.Write(recvBuf[:length])
        }
        return nil
      }


## chapter2:传输层：tcp/udp/sctp

主要讲了UDP／TCP／SCTP三种协议，SCTP日常用的少，以后再了解，重点讲了TCP编程，部分笔记来自第三章(方便总结)

### TCP/UDP协议族
![总图](/images/net/2.1总图.png)

1. ipv4/ipv6对上层协议提供了分组递送的能力，不具有可靠性(丢包可能)
2. tcp是面向连接的流式套接字(stream socket)，关心确认／超时／重传的细节
    - 需要三次握手建立连接
    - 源端数据发送需要对端确认，一段时间内(超时时间:RTT)收不到确认应答则重传，多次重传失败则终止传输
        - RTT(round-trip time)一次客户端和服务器端往返时间
    - 流量控制：接收方可以告诉发送方下一次我能接受的数据量，防止接收方缓冲区溢出
    - tcp是全双工的
3. udp是一种无连接的数据包报套接字(datagram socket)：
    - 不保证是否到达
    - 不保证到达顺序
    - 没有自动重传
    - 没有超时概念
    - 每个数据包都都有报文头标示长度等

### 三次握手和四次挥手

#### 三次握手
![基本socket函数for tcp](/images/net/基本的tcp_socket函数.png)

上图来自第五章，展示了基本的一个tcp客户端和服务端的socket系统调用函数的关系，具体每个系统调用的作用在下面总结。这里关心的是三次握手触发的时机:*服务端调用了accept，客户端调用connect主动打开*

![三次握手](/images/net/tcp三次握手.png)

#### 四次挥手
![四次挥手](/images/net/tcp四次挥手.png)
1. 主动关闭方(客户端)发送fin分节，意思是我该说的说完了，服务器收到立马回复说我收到了,然后这个分节放到服务端的缓冲区的末尾，等待应用程序处理
2. 应用程序处理完了，服务端也需要发一个fin告诉客户端我也完事了
3. 在服务端发送这两个分节的过程中，服务端仍然可以向客户端发送数据
4. 缓冲队列没有数据，服务端也不需要发送数据的时候，服务端会合并发送`ack m+1`和`fin n`分节，这时候就是三次挥手了。
5. 主动关闭方(客户端)响应了服务端的fin分节之后，会再等一段时间，进入`time_wait`状态，
6. tcp是全双工的，任何一方都可以关闭，通常是客户端关闭

#### tcp状态转换
![tcp状态转换](/images/net/tcp状态转换.png)

![tcp连接分组交换](/images/net/tcp连接分组交换.png)

#### time_wait状态

1. 可靠的实现全双工连接的终止：
    如果最后一个ack n+1没有发送给服务端，服务端会重新发送FIN N，这种情况**至少**花费一次来回(>=2MSL)，因此time_wait需要有至少2MSL的时间间隔
2. 允许老的重复分节消逝，主要是防止新的连接如果用了同样的ip和端口，被认为和上一次是同一个连接

#### socket pair

socket pair即`(src_ip, src_port, dest_ip, dest_port)`唯一确认一个tcp连接

![并发服务器](/images/net/并发服务器.png)

如上图，当两个客户端连接同一个socket的时候，无法通过服务端socket的ip和port唯一确认一个连接。详细原因看chapter4


## chapter3:套接字编程简介

1. 网际协议采用大端字节传递多字节数（网络字节序）
    - 大端字节序：高位内存地址对应高序字节
    - 小端字节序：低位内存地址对应高序字节

## chapter4:基本tcp套接字编程

1. `socket`函数

        // 执行网络io前的第一步：socket()
        #include<sys/socket.h>
        int socket(int family, int type, int protocol)

    - family
        ![](/images/net/socket_family.png)
    - type
        ![](/images/net/socket_type.png)
    - protocol
        ![](/images/net/socket_protocol.png)

2. `connect`函数
3. `bind`函数
4. `listen`函数
5. `accept`函数

## chapter5:tcp客户／服务程序示例

## chapter30:客户／服务器程序设计范式


> 《unix网络编程卷一》
> 《图解tcp/ip》
