---
title: 服务器最大支持的TCP连接数
date: 2024-02-01 09:03:12
tags: TCP最大连接 进程最大TCP连接
---

本章内容
>- 一个服务端进程最大能支持多少条 TCP 连接？
>- 一台服务器最大能支持多少条 TCP 连接？
<!--more-->
很多同学第一反应就是端口的限制，端口号最多是 65536个，那就最多只能支持 65536 条 TCP 连接。

实际上这是不对的

今天都带大家分析一波这两个问题。

# 一个服务端进程最多能支持多少条 TCP 连接？
首先我们要知道 TCP 连接本质上在内核里就是一个 socket 对象。

```c
struct socket {  
    ....
    //INET域专用的一个socket表示, 提供了INET域专有的一些属性，比如 IP地址，端口等
    struct sock             *sk;  
    //TCP连接的状态：SYN_SENT、SYN_RECV、ESTABLISHED.....
    short                   type;  
    ....
};  

struct inet_sock {  
...
  __u32    daddr;   //IPv4的目标地址。  
  __u16    dport;   //目标端口。   
  __u32    saddr;   //源地址。  
  __u16    sport;   //源端口。  
...
};  
```
这个 socket 对象也就是一个数据结构，里面包含了 TCP 四元组的信息：源IP、源端口、目标IP、目标端口。

![TCP_四元组](image.png)

所以， 只要确认了【源IP、源端口、目标IP、目标端口】这四个信息，就能在内核中找到这个 socket 对象，也就能确定一条 TCP 连接。

一个服务端进程通常是监听 1 个端口号（当然也可能监听多个端口号，这里不考虑），比如我的图解网站的 nginx 服务，就监听了 443 端口。

![Alt text](image-1.png)
你们看图解网站的时候，实际上就是通过 nginx 服务把网页数据发送给你们的。

然后，服务端进程除了会固定监听某个一个端口之外，也通常会绑定 0.0.0.0 IP 地址。

这个IP地址是特殊的， 0.0.0.0 指的是本机上的所有IPV4地址，如果一个主机有两个 IP 地址，192.168.1.1 和 10.1.2.1，并且该主机上的一个服务监听的地址是0.0.0.0，那么通过两个 IP 地址都能够访问该服务。

所以一个服务端进程，意味着他的 IP地址和端口号是固定的（0.0.0.0:443）。

也就是当客户端与服务端建立一条 TCP 连接的时候，这个 TCP 连接的四元组信息中服务端的 IP地址和端口号是固定的，能产生变化的就是客户端的 IP 地址和端口号了。

因此，一个服务端进程最大能支持的 TCP 连接个数的计算公式如下：

![Alt text](image-2.png)

对 IPv4，客户端的 IP 数最多为 2 的 32 次方，客户端的端口数最多为 2 的 16 次方。

那么一个服务端进程理想情况下，最大的 TCP 连接数约为 2 的 48 次方（2^32 (ip数) * 2^16 (端口数），这数值是非常夸张的了，约等于两百多万亿！

当然，服务端进程最大能支持的 TCP 连接数远不能达到理论上限，还会受到文件描述符、内存大小资源的限制，毕竟 socket 在 Linux 的视角其实就是文件资源，而且一个 socket 对象也会占用一定的内存资源。

因此，会受以下因素影响：

>- 文件描述符限制，每个 TCP 连接都是一个文件，如果文件描述符被占满了，会发生 Too many open files。Linux 对可打开的文件描述符的数量分别作了三个方面的限制：
    >>- 系统级：当前系统可打开的最大数量，通过 cat /proc/sys/fs/file-max 查看；
    >>- 用户级：指定用户可打开的最大数量，通过 cat /etc/security/limits.conf 查看；
    >>- 进程级：单个进程可打开的最大数量，通过 cat /proc/sys/fs/nr_open 查看；

>- 内存限制，每个 TCP 连接都要占用一定内存，操作系统的内存是有限的，如果内存资源被占满后，会发生 OOM。

# 一台服务器最大最多能支持多少条 TCP 连接？
前面分析是一个服务端进程理的情况，理论上能最大支持约为 2 的 48 次方（2^32 (ip数) * 2^16 (端口数），约等于两百多万亿！

那到了一台服务器的视角就会有一点不一样。

一台服务器是可以有多个服务端进程的，每个服务端进程监听不同的端口，比如：ssh的22，Redis的6339，当然所有65535个端口你都可以用来监听一遍。
![Alt text](image-3.png)
当然所有65535个端口你都可以用来监听一遍，这样理论上线就到了2的32次方（ip数）×2的16次方（port数）×2的16次方（服务器port数）个，感兴趣你可以算一下，这个基本相当于无穷个了。

不过理想和实际总是会有差距的！

因为Linux每维护一条TCP连接都要花费资源，处理连接请求，保活，数据的收发时需要消耗一些CPU，维持TCP连接主要消耗内存。

我们题目的问题是考虑最大多少个连接，所以我们先不考虑数据的收发，那么TCP在静止的状态下，就不怎么消耗CPU了，主要消耗内存，而Linux上内存是有限的。

首先，我们要知道一条处于 ESTABLISH 状态的 TCP 连接具体占用多大内存？

**一个 TCP 对象占用的大小，等于它所包含的一些数据结构占用大小的总和，也是就把上面这些数据结构的大小累加起来，就是一个 TCP 连接占用的大小了。**

这里直接给大家一个结论，**一条处于 ESTABLISH 状态的 TCP 连接占用的大小是 3.44 KB（0.81K+2.19K+0.19K+0.25K）**。
![TCP对象内存开销总结](image-4.png)

也就是，每一条静止状态的TCP连接大约需要吃 3.44K 的内存。

那么 **8 GB 物理内存的服务器，最大能支持的 TCP 连接数=8GB/3.44KB=2,438,956（约240万）**！

当然， 实际过程中的 TCP 连接，肯定不是静止状态的，还会进行发送数据和接收数据了，那么这些过程还是会额外消耗更多的内存资源的，并发很难达到百万级别。

## 总结
    一个服务端进程最多能支持多少条 TCP 连接？

如果在不考虑服务器的内存和文件句柄资源的情况下，理论上一个服务端进程最多能支持约为 2 的 48 次方（2^32 (ip数) * 2^16 (端口数），约等于两百多万亿！

但是在实际中是支持不了这个数值的，每个 TCP 连接都是一个文件，会占用文件句柄资源，也会占用一定的内存空间。

    一台服务器最大最多能支持多少条 TCP 连接？

一台服务器是可以有多个服务端进程的，每个服务端进程监听不同的端口，当然所有65535个端口你都可以用来监听一遍。

当然所有65535个端口你都可以用来监听一遍，这样理论上线就到了2的32次方（ip数）×2的16次方（port数）×2的16次方（服务器port数）个，这个基本相当于无穷个了。

但是 Linux每维护一条TCP连接都要花费内存资源的，每一条静止状态（不发送数据和不接收数据）的 TCP 连接大约需要吃 3.44K 的内存，那么 8 GB 物理内存的服务器，最大能支持的 TCP 连接数=8GB/3.44KB=2,438,956（约240万）。

实际过程中的 TCP 连接，还会进行发送数据和接收数据了，那么这些过程还是会额外消耗更多的内存资源的，并发很难达到百万级别。





___________
#文章来源

[腾讯三面：一台服务器，最大支持的TCP连接数是多少？
](https://mp.weixin.qq.com/s/l9ggXLAEHp4LTjd2Qsyqtg)
