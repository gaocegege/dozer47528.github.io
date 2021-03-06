---
title: 解决 MHA 切换后导致应用无可用连接的问题
author: Dozer
layout: post
permalink: /2015/07/mha.html
categories:
  - 编程技术
tags:
  - MySQL
  - MHA
  - c3p0
  - Socket
  - TCP
---

### 问题来源

环境：MHA + MySQL + c3p0

最近在一次 MHA 切换的时候，发现应用中的 c3p0 老连接一直在等待，而新进来的请求全部被 c3p0 拒绝了。

然后又过了很长时间，老的连接断开，然后恢复了正常。整个过程极其诡异。

<!--more-->

&nbsp;

### MHA 是什么

[mysql-master-ha](https://code.google.com/p/mysql-master-ha/): Master High Availability Manager and tools for MySQL

详细的我就不班门弄斧了，就简单地说一下它的作用和大致原理。

MHA 会为多个写库分配一个虚 IP，当一个写库故障的时候，MHA 会利用 [ARP](https://zh.wikipedia.org/zh/%E5%9C%B0%E5%9D%80%E8%A7%A3%E6%9E%90%E5%8D%8F%E8%AE%AE) 协议欺骗交换机，让交换机把所有数据包发送到新的机器上。而应用层配置的 IP 不需要变。

应用层只是感觉网络瞬间断了一下，但是又很快连了上去。

&nbsp;

### 重现问题

以上是正常的 MHA 切换流程，但是为什么我们应用的老连接一直不断开呢？然后它过了一段时间又断开了呢？

一看到现象，我就感觉这事和 TCP [KeepAlive](https://en.wikipedia.org/wiki/Keepalive#TCP_keepalive) 有关，MHA 的切换其实就是一次网络瞬断。

于是我便开始在本机模拟这次“故障”了。既然这里的核心是网络瞬断，那么不用 MHA 也可以，只要我本地的程序连上远程的 MySQL 服务器，然后拔掉网线瞬间断网就行了。

&nbsp;

#### 操作步骤
1. 初始化程序，c3p0 连接池设置成1个
2. 进行一次查询`select 1`，底层创建了一个连接
3. 拔网线
4. 再进行一次查询`select 1`
5. 插回网线
6. 再进行一次查询`select 1`

但是，c3p0 在第4个步骤的时候立刻报错了，而且插上了网线后又恢复了正常。

而我们的线上应用在这里是卡住的，直到触发 TCP KeepAlive 后才把连接断开的。

&nbsp;

#### 问题在哪？

再仔细回想一下，TCP KeepAlive 是为了解决网络瞬断后，无数据包发送便无法发现 TCP 连接已经断开的问题。

而这里的第4步其实是发送了数据的，虽然底层不知道连接已经断了，但是在发送的时候，还是可以知道连接已断的。

也就是说我们模拟的不对，线上的情况略有不同。

后来经过 DBA 的提醒，又尝试了另一种方案。

&nbsp;

#### 再一次模拟

1. 初始化程序，c3p0 连接池设置成1个
2. 进行一次查询`select 1`，底层创建了一个连接
3. 拔网线
4. 再进行一次查询`select sleep(60)`
5. 插回网线
6. 程序一直在等待…

是的，重现了！一定要在客户端发送完请求，然后等待服务器回应的时候断网，这样才会出现问题。

而我们的线上应用连接池不止一个，只有所有的连接池都发送完请求，并等待的时候做了 MHA 切换，才会让整个程序一直无可用连接。

此时线上只要有任何一个连接不在等待，那么在 MHA 切换后，它会立刻报错，并且重新建立连接。应用只会慢一点，但是不会拿不到任何一个连接。

这也是为什么这个问题在线上并不是一直出现，因为要在高 QPS，原来的 MySQL 服务器很慢的情况下才会出现。

&nbsp;

### 如何解决

解决这个问题的关键就是要让你的程序知道 TCP 连接已断。

其实 TCP 底层就可以通过 KeepAlive 机制来发现问题。但问题是，这个配置在 Linux 中默认是 2 小时。应用程序绝对不可能接受这么长时间的。而且这个配置是系统全局的，配置的太低可能会影响其他程序。

还有另一个方案就是通过 MySQL JDBC 的一个参数来解决：`jdbc:mysql://localhost:3306/mysql?socketTimeout=60`，这个参数默认是`0`，也就是意味着无限期等待…

但配置了这个参数，如果你的应用层有超过60秒的请求，它就挂了…

这里有一篇文章写的很好，给大家看一下：[传送门](http://www.cubrid.org/blog/dev-platform/understanding-jdbc-internals-and-timeout-configuration/)

里面详解了 JDBC 中的各种超时配置。

总之，如果你用了 MHA ，一定要注意一下这个问题，要么加上`socketTimeout`，要么配置 TCP KeepAlive。