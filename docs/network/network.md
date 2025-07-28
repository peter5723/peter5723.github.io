# 计算机网络介绍

## 第一章 总览

传输的信息包基本单元：packet（分组，包）

端
接入网
网络核心——ISP（因特网服务提供商）

最大的网：因特网/互联网（internet）

分布式

网络编程接口 API

协议

客户机-服务器

P2P

局域网——以太网

无线局域网 802.11

分组（packet）交换


网络协议自顶向下分层：

- 应用层：网络应用程序和应用层协议：HTTP、SMTP、FTP、DNS。应用层的信息分组（packet）称为报文（message）
- 运输层：应用程序端点之间传输应用层报文。协议：TCP、UDP。运输层的分组称为报文段（segment）
- 网络层：将称为数据报（datagram）的网络层分组从一台主机移动到另一台主机。运输层协议向网络层递交运输层报文段和目的地址。网络层协议包含 IP 协议与路由的选路协议。
- 链路层：网络层将数据下发给链路层，链路层沿着路径将数据报传递给下一个节点。链路层协议包括以太网、wifi。链路层分组称为帧（frame）。
- 物理层：将帧中的比特一个一个实际地传输到下一个节点。

发送时：应用层报文传给运输层。运输层收取报文附上首部信息，构成运输层报文段。（首部信息包括差错检测比特等等）。运输层向网络层传递报文段，网络层增加目的系统地址等信息，形成网络层数据报。数据报传给链路层，链路层也依据协议封装成链路帧。最后，通过物理层执行实际的传输。

每一层的分组（packet）包含两种字段：首部字段和有效载荷。

在接收端，数据报重新还原成报文段。

上面的描述见下图，就非常直白了：

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net1.png)

网络每一层都对上一层封装（Encapsulation）好。只要按照协议将数据交给下一层即可，这层无需担心下一层怎么传输。

## 第二章 应用层

### 2.1 例子

我们用 Python 可以简单地写一个服务器客户端应用程序。可以在两台连接的计算机上跑，看会发生什么。

```python
# serve.py
# TCP服务器端
import socket

# 创建TCP套接字
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 绑定本地地址和端口
server_socket.bind(('192.168.223.1', 8888))
# 监听连接（最大连接数5）
server_socket.listen(5)
print("服务器已启动，等待客户端连接...")

while True:
    # 接受客户端连接
    client_socket, addr = server_socket.accept()
    print(f"客户端 {addr} 已连接")

    # 接收客户端数据（最大1024字节）
    data = client_socket.recv(1024).decode('utf-8')
    print(f"收到消息: {data}")

    # 发送响应
    response = f"已收到你的消息: {data}"
    client_socket.send(response.encode('utf-8'))

    # 关闭连接
    client_socket.close()
```

```python
# client.py
# TCP客户端
import socket

# 创建TCP套接字
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 连接服务器
client_socket.connect(('192.168.223.1', 8888))

# 发送消息
message = "Hello, Server!"
client_socket.send(message.encode('utf-8'))

# 接收响应
response = client_socket.recv(1024).decode('utf-8')
print(f"服务器响应: {response}")

# 关闭连接
client_socket.close()
```

### 2.2 HTTP 协议

客户机-服务器

进程通信：交换报文

进程通过一个称为套接字（socket）的软件接口来交换报文。socket 在英文中是插口的意思。我们可以将之理解为一个“门”。报文从这个门出来，走到另一个门。套接字就是应用程序和网络之间的应用程序编程接口（API），应用程序通过套接字来使用网络。通过套接字可以控制应用层端的所有东西，但是运输层则很少，仅限于选择运输层协议（如 TCP），设置一些运输层参数。

可供应用程序运输的服务协议主要有 TCP 和 UDP。TCP 连接全双工，传输可靠。目前大多数应用程序如电子邮件、web、文件传输都使用 TCP 作为运输层协议。

通信寻址：IP 地址标识唯一主机，端口号标识主机进程。

Web 应用：用户打开 Web 浏览器（比如 Firefox），打开一个站点，发送请求。请求按照 HTTP 协议构造的报文发送。Web 服务器接受请求，按 HTTP 协议返回文件（HTML 格式），用户就可以浏览网页。

HTTP：超文本传输协议。HTTP 协议由两部分组成：客户机和服务器程序，交换 HTTP 报文进行会话。

Web 术语：

- Web 页面：由 HTML 文件、JPEG 图片、程序、视频等对象（object）一起构成。通过对象的 URL 地址对对象进行引用。URL（Uniform Resource Locator，统一资源定位符）是用于完整地描述互联网上网页或其他资源位置的一组字符。一个 URL 地址包括协议、域名或IP地址、端口号、路径等组成部分。例如 "http://192.168.1.1/index.html"。
- Web 浏览器：实现了客户机部分，发送请求，接收数据，展示给用户。
- Web 服务器：实现了服务器部分，存储 Web 对象，每个对象由 URL 寻址，发送给请求的用户。

HTTP 协议用 TCP 作为其运输层协议。客户机发起与服务器的 TCP 连接，从套接字接口发送 HTTP 请求报文，接收 HTTP 响应报文。服务器从套接字接口接收 HTTP 请求报文，发送 HTTP 响应报文。

报文被发送后，就由 TCP 控制。TCP 对 HTTP 封装好，提供可靠传输服务。HTTP 无需担心任何事情，都是 TCP 和更下层的工作。HTTP 只需到 TCP 发送和接收数据即可。

HTTP 服务器不保存关于客户机的任何信息，是“无状态协议”。

我们看下面这张图，展示了传输的过程：

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net2.png)

就是一个所谓的**三次握手**过程：

1. 客户机向服务器发送一个小 TCP 报文段
2. 服务器返回一个小 TCP 报文段进行确认和响应
3. 客户机向服务器返回确认，且发送一个 HTTP 请求报文

这样完成握手以后，服务器才向客户机发送文件。

HTTP 报文格式：

请求报文：

```
<请求行>：方法、路径、协议版本
<请求头部字段>：包含多个键值对，如 Host、connection 等等。
空行（CRLF） ：表示头部结束。
<请求正文（可选）>
```

例如：
```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr

```

响应报文：
```
<状态行>：协议版本、状态码、状态短语
<响应头部字段>：服务器信息等内容
空行（CRLF） ：表示头部结束。
<响应正文（可选）>：返回客户端的实际内容，如 HTML 页面。
```

```
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html

(data data data data data ...)
```

浏览器按 F12 打开开发者页面就可以轻松地看到 http 报文。

http 是无状态协议。但是 http 可以使用 cookie 技术来让服务器跟踪用户状态。简单说：第一次到达一个使用 cookie 的站点，站点会产生一个唯一识别码，添加到后端数据库中。接着站点返回一个响应报文，包含 Set-cookie: 这个首部表项和识别码。用户端浏览器接收到这个报文，就往其管理的 cookie 文件中添加一行，包含站点服务器的主机名和识别码。之后用户再访问站点时，浏览器发送的请求报文都会包含这个识别码。服务器就可以根据这个识别码，检索数据库，来跟踪用户的状态，如访问活动。


Web 缓存器

电子邮件协议：SMTP 发送，POP3/IMAP 接收


### 2.3 DNS 协议

互联网主机识别：主机名（hostname）或者 IP 地址。进行转换的目录服务：域名系统（Domain Name System，DNS）。DNS 协议正常由其他应用层协议使用，用于将主机名转化成 IP 地址。举例，用户输入 `www.baidu.com` 时，发生的事情：

- 用户主机运行 DNS 应用的客户机端
- 浏览器将主机名 `www.baidu.com` 传给 DNS 应用的客户机端
- DNS 客户机向 DNS 服务器发送一个包含主机名的请求
- DNS 客户机收到 DNS 服务器的回答报文
- 浏览器接收到来自 DNS 的 IP 地址，向该 IP 地址定位的 HTTP 服务器发送一个 TCP 连接

用 wireshark 抓包，过滤 DNS 协议，即可查看 DNS 服务器地址以及 DNS 报文。

实验室的电脑不知道怎么回事，抽风了抓不出来，回寝室用自己的笔记本电脑就可以成功抓包 DNS 服务了。首先用 `nslookup <hostname>` 就可以找到域名对应的 IP 地址。`nslookup` 指令是走 DNS 服务查询的。我们开启 wireshark 抓包，过滤 DNS 协议，打开一个命令行输入 `nslookup bing.com`，就可以看到如下图的结果。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net3.png)

我们用 `ipconfig /all` 查看本机的 IP 地址是 `10.192.252.13`，DNS 服务器是 `10.10.0.21` 与 `10.10.2.21`，这是使用浙大校网使用的 DNS 服务器。这与上面抓包的结果一致。我们打开 DNS 服务器的响应报文，如下所示：

```
Frame 109837: 532 bytes on wire (4256 bits), 532 bytes captured (4256 bits) on interface \Device\NPF_{B8D67097-65C7-4EE3-90AD-955585FA85A3}, id 0
Ethernet II, Src: H3CTechnolog_ff:80:10 (00:0f:e2:ff:80:10), Dst: Intel_25:25:88 (9c:b1:50:25:25:88)
Internet Protocol Version 4, Src: 10.10.0.21, Dst: 10.193.252.13
User Datagram Protocol, Src Port: 53, Dst Port: 50849
Domain Name System (response)
    Transaction ID: 0x0002
    Flags: 0x8180 Standard query response, No error
    Questions: 1
    Answer RRs: 2
    Authority RRs: 13
    Additional RRs: 13
    Queries
    Answers
        bing.com: type A, class IN, addr 150.171.27.10
        bing.com: type A, class IN, addr 150.171.28.10
    Authoritative nameservers
    Additional records
    [Request In: 109836]
    [Time: 0.002144000 seconds]
```

可以知道服务器的端口号是53，本机的端口号是50849，传输的方式是 UDP，返回的结果在 `answer` 条目中，其中一个 IP 地址是 `150.171.27.10`，就是 `bing.com` 对应的 IP 地址。这与 `nslookup` 的输出结果是一致的。当然，出于多种原因，直接在浏览器中输入解析的 IP 地址是一般是打不开对应网页的，[原因在这里，大概是反向代理，我暂时不研究](https://blog.csdn.net/gui951753/article/details/83070180)

这个链接较好，可以看看。[DNS 实验](https://zhuanlan.zhihu.com/p/335814524)

在做实验的时候，发现现在 baidu ping 不通了，有说 baidu 是关闭了 ICMP 协议导致的，但是我 telnet 也连接不到。[baidu ping
不到](https://www.cnblogs.com/a-s-m/p/11167669.html)

上面动手实操了一下，下面我们继续学习原理。

应用程序调用 DNS 客户机端，指明主机名，主机 DNS 接收到后，向网络中发送 DNS 查询报文。一般请求和回答报文均使用 UDP 经端口 53 发送。经过若干时间，主机 DNS 从服务器接收到一个 DNS 回答报文，即可解析出 IP 地址。

看起来，DNS 是一个黑盒子。那么黑盒子内部如何？

DNS 是一个典型的分布式应用。有三种类型的 DNS 服务器：根 DNS 服务器，顶级域服务器（TLD）和权威 DNS 服务器。以及用于代理的本地 DNS 服务器。

下面图片比较清楚地展示了域名系统，域名系统与各级别 DNS 服务器相对应。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net4.png)

以 `www.baidu.com` 为例，`com` 是顶级域名，`baidu.com` 对应一个组织的权威域名服务器， `www` 是主机名，因特网中默认的主机名就叫 `www`。在 DNS 查询时，主机先将请求发送到本地 DNS 服务器，本地 DNS 服务器将该请求传到根 DNS 服务器，根 DNS 服务器注意到 `com` 前缀，于是向本地 DNS 服务器返回负责 `com` 的 TLD 的查询列表。本地 DNS 服务器向这些 TLD 发送查询报文。其中一个 TLD 服务器找到了 `baidu.com` 负责的权威 DNS 服务器，并用该权威 DNS 服务器的地址进行响应。最后，本地 DNS 服务器向该权威 DNS 服务器发送请求，寻找名为 `www` 的主机名。权威 DNS 服务器返回 `www` 的 IP 地址给本地服务器，本地服务器再将结果返还给我的主机，这样就完成了一整个 DNS 解析流程。

这样查询开销大，DNS 也有缓存技术来减少开销。

## 第三章 运输层 transport-layer


### 3.1 概述

首先仍然是概要：运输层为应用层的程序提供了“逻辑通信”功能。这里的“逻辑”是封装的意思。不同主机的应用层程序通过运输层连接好像直接相连一般，而不需要考虑实际的实现如何。运输层协议是在端系统中实现的。发送方的运输层将发送程序的报文转化成运输层分组，这个分组称为运输层报文段。然后将报文段传给网络层，网络层将其封装成网络层分组（称为数据报），向目的地发送。接收方的网络层提取出运输层报文段，向上交给运输层。

运输层为不同主机上的进程提供了逻辑通信，而网络层则提供了主机之间的逻辑通信。这句话，看自顶向下方法中的邮政服务例子就容易理解了。

运输层服务有两种：UDP 和 TCP。

首先介绍网络层。因特网网络层协议，有一个 IP 协议。IP 为主机之间提供逻辑通信。IP 的服务模型是尽力而为交付服务，是不可靠服务，数据可能丢失。每台主机至少有一个网络层地址，即 IP 地址。

再介绍运输层的具体任务。运输层协议的基本任务是将两个端系统之间的 IP 交付服务扩展成两个端系统上的进程之间的交付服务。主机间交互扩展成进程间交付的两个动作称为 transport-layer multiplexing and demultiplexing （多路复用和多路分解）。运输层协议可以在报文段的首部添加查错检测字段提供完整性检查。UDP 只进行数据交付和差错检测，也是不可靠服务，不保证数据完整性。TCP 提供附加服务：可靠数据传输和拥塞控制。

### 3.2 多路复用和多路分解

然后讨论一下多路复用和多路分解。考虑这样的问题，我有四个网络应用进程在运行，我的计算机的运输层从底层的网络层接收数据时，**如何将这些数据定位到其所需到达的进程？**

回想套接字。每个进程有一个或多个套接字和运输层交互。运输层是将数据交付给套接字的。每个套接字都有一个唯一标识符。所以关键是怎样将运输层报文段交接给合适的套接字。每个运输层报文段都有几个字段。在接收端，运输层检查这些字段，标识出接收套接字，然后将报文段定向到该套接字。将运输层报文段的数据交付到正确的套接字的工作叫多路分解（demultiplexing）。反过来，从源主机的不同套接字中收集数据块，并为每个数据块封装首部信息从而形成报文段，将报文段传给网络层的工作称为多路复用（multiplexing）。

再具体一点，多路复用的要求是：1、套接字有唯一标识符。2、每个报文段有特殊字段来指示该报文段所要交付的套接字。这个标识符以及特殊字段就是端口号。报文段的端口字段包括源端口号和目的端口号字段（其他字段后面讲述）。端口号是一个 16 bit 的数字，值为 0 到 65535。0 到 1023 的端口号被保留，用于基本的应用层协议（如 HTTP 是 80）。

这样，我们就可以对问题给出回答：

**运输层的多路分解服务实现**：主机上的每一个套接字分配一个端口号，报文段到达主机时，运输层检查报文段中的目的端口号，再将之定向到对应的套接字，然后报文段中的数据通过套接字进入其所连接的进程。

UDP 的客户端服务器例子：

```python
# service
def service_one():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind(('127.0.0.1', 9001))
    print("Service One is running on port 9001")

    while True:
        data, addr = sock.recvfrom(65535)
        print(f"Service One received from {addr}: {data.decode()}")
        sock.sendto(f"[Service One] Echo: {data.decode()}".encode(), addr)

# client
def client_service_one():
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.sendto(b"Hello from Service One", ('127.0.0.1', 9001))
    data, _ = sock.recvfrom(65535)
    print("Response from Service One:", data.decode())
```

可以模拟，抓包一下，抓包要选择 `Npcap Loopback Adapter` 接口，对应 `127.0.0.1` 本地回环流量。

然后我们再看一下 TCP 的代码：

```python
#service
def service_one():
    # 创建TCP套接字
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 绑定地址和端口
    sock.bind(('127.0.0.1', 9001))
    # 开始监听
    sock.listen(5)
    print("Service One (TCP) is running on port 9001")

    while True:
        client_socket, addr = sock.accept()
        print(f"Service One: 客户端 {addr} 已连接")

        data = client_socket.recv(65535)
        print(f"Service One received: {data.decode()}")

        response = f"[Service One] Echo: {data.decode()}".encode()
        client_socket.sendall(response)
        client_socket.close()
#client
def client_service_one():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('127.0.0.1', 9001))
    sock.sendall(b"Hello from Service One (TCP)")

    data = sock.recv(65535)
    print("Response from Service One:", data.decode())
    sock.close()
```

我们可以看到 UDP 和 TCP 的区别。UDP 是直接发送，直接接收的。TCP 多了一步 `sock.connect(('127.0.0.1', 9001))`。TCP 是先再两台主机之间建立连接才交流数据，而 UDP 不需要。所以说 TCP 是面向连接的，UDP 是无连接的。

### 3.3 UDP 协议

UDP （User Datagram Protocol，用户数据报协议）是非常简单的协议——它只做了运输层协议必须做的最少工作：多路复用/分解、差错检测。

UDP 的工作过程：

UDP takes messages from the application process, attaches source and destination port number fields for the multi
plexing/demultiplexing service, adds two other small fields, and passes the resulting segment to the network layer.
The network layer encapsulates the transport-layer segment into an IP datagram and then makes a best-effort attempt to deliver the segment to the receiving host.
If the segment arrives at the receiving host, UDP uses the destination port number to deliver the segment's data to the correct application process.

UDP 在发送报文段之前，发送方和接收方的实体之间没有进行握手建立连接，所以说 UDP 是无连接的。

DNS 服务一般就使用 UDP 协议。

用 wireshark 抓一下包，就可以看 UDP 报文段结构：

```
Frame 248: 54 bytes on wire (432 bits), 54 bytes captured (432 bits) on interface \Device\NPF_Loopback, id 0
Null/Loopback
Internet Protocol Version 4, Src: 127.0.0.1, Dst: 127.0.0.1
User Datagram Protocol, Src Port: 51604, Dst Port: 9001
    Source Port: 51604
    Destination Port: 9001
    Length: 30
    Checksum: 0x2cd7 [unverified]
    [Checksum Status: Unverified]
    [Stream index: 7]
    [Timestamps]
    UDP payload (22 bytes)
Data (22 bytes)
```

UDP 报文段结构由报文头和报文组成。报文头包含四个字段：源端口号、目的端口号、报文长度、校验和。报文就是应用层数据。

简单介绍一下校验和。发送方将所有字节相加（不考虑溢出），再对结果取反，得到一个校验和。接收方接收时，将所有字节相加，加上校验和，结果应当是所有二进制位均为 1。否则就说明这个分组出现了差错。

UDP 提供了差错检测，但其无法进行差错恢复。

### 3.4 可靠数据传输

这个链接很好：[可靠数据传输原理详细图解](https://blog.csdn.net/Sqrt1230/article/details/121269131)

​​可靠数据传输（Reliable Data Transfer）​​，下称 rdt。

我们简单介绍一下，分析一下解决问题的思路。


首先，假设运输层的下层不出任何差错。那我们什么都不用干。这是 rdt1.0。

但是，底层信道大概率会出现比特差错，即 01 互换。我们引入自动重传请求（ARQ）协议。接收者在接收数据后，进行差错检测，提供反馈给发送者，回送肯定确认分组（ACK）或者否定确认分组（NAK）。发送方接收到确认信息后，重传这个分组。发送方在等待 ACK 时，不做其他事情，这样的协议被称为停等（stop and wait）协议。

但是可能出现这样的问题：ACK 出错了怎么办？

简单的解决方法是：在发送的分组中添加一个1 bit序号字段，如果是新包就变字段，如果是重传就不变。第一个包是 0，第二个包是新包则是 1，如果是第一个包的重传则是 0，以此类推。101010 就是一直新的数据包，10010 第三次是第二次的重传。这是 rdt2.0。

现在再加入新的问题：如果丢包，怎么解决？如果分组丢失，当然连 ACK 都接收不到了。最简单的方法是加入一个定时器计时。发送方发送一个分组时启动一个倒计数定时器，如果在倒计数定时器倒计时结束之前收到了ACK响应，则中断定时器，进入准备接收来自上层的下一次调用。如果倒计数定时器超时，则发送方认为分组丢失，向接收方重传该分组，并且重新启动定时器。这是 rdt3.0。


rdt3.0 可以说比较可靠。但是它是停等协议，效率极低。简单的解决方法称为“流水线”：发送方发送多个分组而无需等待确认，每个分组一个个处理。流水线中差错恢复两种基本方法：回退 N 步和选择重传。


### 3.5 TCP

TCP 全称为 Transmission Control Protocol，即传输控制协议。TCP 是运输层面向连接的可靠运输协议。其运用的原理包括差错检测、重传、累积确认、定时器等。

我们来复习一下 TCP 连接是怎样建立的。一个进程想要与另一个进程建立连接。发起连接请求的进程称为客户进程，另一个进程成为服务器进程。python 代码 `sock.connect(('127.0.0.1', 9001))` 就实现了这样的请求，建立连接。建立连接的过程就是三次握手：the client first
sends a special TCP segment;
the server responds with a second special TCP segment;
and finally the client responds again with a third special segment.
The first two segments carry no payload, that is, no application-layer data; the third of these
segments may carry a payload.

建立连接后，两个进程间就可以发送数据了。TCP 连接是点对点的，单个发送方与单个接收方之间建立。

TCP 连接组成包括：一台主机上的缓存、变量和与进程连接的套接字。

客户进程通过套接字传递数据流到 TCP。TCP 将这些数据引导到发送缓存中。接下来 TCP 就会不时地从发送缓存中取出一块数据，传递到网络层。TCP 可从缓存中取出并放入报文段中的数据数量受限于最大报文段长度 (Maximum Segment Size, MSS)。TCP 为每块客户数据配上一个 TCP 首部，从而形成多个 TCP 报文段 (TCP segment)。这些报文段被下传给网络层， 网络层将其分别封装在网络层IP数据报中。 然后这些IP数据报被发送到网络中。 当TCP在另一端接收到一个报文段后，该报文段的数据就被放入该 TCP 连接的接收缓存中。 应用程序从此缓存中读取数据流。 该连接的每一端都有各自的发送缓存和接收缓存。


然后，我们看一下 TCP 报文段的结构。可见 TCP 首部有固定的 20 字节。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net5.png)


`curl` 是一个比较方便的测试 web 的工具。我们在命令行中输入 `curl -v www.baidu.com `，它会向百度发送 HTTPS 请求，百度会返回网页。我们输入这个指令，用 wireshark 抓包，过滤 TCP 请求，先看看 TCP 报文段的结构。下面是第一次握手的 TCP 报文段。

```
Transmission Control Protocol, Src Port: 51999, Dst Port: 80, Seq: 0, Len: 0
    Source Port: 51999
    Destination Port: 80
    [Stream index: 7]
    [Conversation completeness: Incomplete, DATA (15)]
    [TCP Segment Len: 0]
    Sequence Number: 0    (relative sequence number)
    Sequence Number (raw): 619352399
    [Next Sequence Number: 1    (relative sequence number)]
    Acknowledgment Number: 0
    Acknowledgment number (raw): 0
    1000 .... = Header Length: 32 bytes (8)
    Flags: 0x002 (SYN)
    Window: 65535
    [Calculated window size: 65535]
    Checksum: 0xa281 [unverified]
    [Checksum Status: Unverified]
    Urgent Pointer: 0
    Options: (12 bytes), Maximum segment size, No-Operation (NOP), Window scale, No-Operation (NOP), No-Operation (NOP), SACK permitted
    [Timestamps]
```

TCP 报文段由首部字段和数据字段组成。上面这个是首部字段，第一次握手的话，没有数据字段。


TCP 首部首先是源端口号和目的端口号。源端口号就是 `curl` 进程的端口，目的端口号是 HTTP，默认是 80。然后就是 32 bit 的序列号字段（Sequence Number）和 32 bit 的确认号字段（Acknowledgment Number），用来实现可靠传输服务。然后是 6 bit 的标志字段（flag），ACK 表示成功接收报文段，RST、SYN、FIN 用于连接建立和拆除。
再是 16 bit 的接收窗口字段（window），用于流量控制，表示接收方愿意接受的字节数量。然后是校验和字段。
再是选项字段（Option），包括最大报文段长度（MSS）、时间戳等。

TCP 首部最重要的两个字段是序列号和确认号字段，是可靠传输服务的关键部分。TCP 将数据当成字节流处理。一个报文段的序列号是该报文段**首字节**的字节流编号。假设传输的数据有 500 000 字节，MSS（报文段最大长度）为 1000 字节，则这个数据将被划分成 500 个报文段，第一个报文段的序号是 0，第二个是 1000，以此类推。

确认号是这样的：TCP 是全双工，即主机 A 向 B 发送数据的时候，也同时接收来自主机 B 的数据。从主机 B 到达 A 的每个报文段中都有一个序号用于从 B 流向 A 的数据。主机 A 填充进报文段的确认号是主机 A 期望从主机 B 收到的下一字节的序号。举例：主机 A 从 B 中收到了编号为 0 到 535 的所有字节，并正要发送一个报文段给主机 B，那么 A 就会在确认号字段中填上 536。

知道这些，我们可以看看 TCP 的可靠数据传输的原理大致如何。

主机 A 向主机 B 发送一个大文件。发送基本的事件有三个：上层应用数据接收数据，封装成报文段，启动定时器。如果定时器到时间没有收到 ACK，则默认丢包，进行重传。

如下图所示：

```
发送端                          接收端
   |                                 |
   |---[SEQ=0, LEN=1000]----------->|    # 发送段
   |<--[ACK=1001]--------------------|    # 确认收到
   |                                 |
   |---[SEQ=1000, LEN=1000]-------->|    # 发送下一数据段
   |<--[ACK=2001]--------------------|    # 继续确认
```

若某段丢失：

```
   |---[SEQ=2000, LEN=1000]-------->|    # 发送段
   |                                 |
   |<-- (未响应)                     |
   |                                 |
   |------(等待超时)---------------->|
   |---[SEQ=2000, RETRANSMITTED]--->|    # 重传该段
   |<--[ACK=3001]--------------------|    # 接收方确认收到
```

下面的图片，抓取了 `curl -v www.baidu.com` 客户机与服务器进行数据交换的过程。观察客户机发送报文的 ack 和 服务器发送的报文的 seq。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net6.png)

最后我们来看看建立和断开 TCP 连接的过程。

三次握手建立连接：

```
Client                  Server
   |                         |
   |----[SYN, SEQ=x]-------->|
   |<--[SYN-ACK, SEQ=y, ACK=x+1]--|
   |----[ACK, SEQ=x+1, ACK=y+1]-->|
```

1. 客户机端的 TCP 先向服务器端的 TCP 发送一个特殊的 TCP 报文段。该报文段不含应用层数据，SYN 比特置为 1。客户端选择了一个初始序列号 x。
2. SYN 报文段的 IP 数据报到达服务器主机，服务器提取出 TCP SYN 报文段，为该 TCP 连接分配 TCP 缓存和变量，向用户发送允许连接的 SYNACK 报文段。服务器选择了一个初始序列号 y。
3. 收到 SYNACK 报文段后，客户机给连接分配缓存和变量，并给服务器发送一个 ACK 报文段，对服务器的报文段进行确认。之后，客户机与服务器就可以相互发送数据了。客户端的序列号加 1。

四次挥手断开连接：

```
Client                  Server
   |                         |
   |----[FIN, SEQ=u]-------->|
   |<--[ACK, SEQ=v, ACK=u+1]--|
   |<--[FIN, SEQ=w]----------|
   |----[ACK, SEQ=u+1, ACK=w+1]-->|
```

1. 客户机想要关闭连接，先发送一个 FIN 报文段
2. 服务器接收到，回送一个 ACK 报文段表示确认
3. 服务器发送一个 FIN 报文段，表示可以终止连接
4. 客户机发送 ACK 报文段，关闭连接


下面的图片，展示了用 `curl` 进行连接的三次握手以及后续传输数据的过程。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net7.png)

为了节省开销，默认情况下，HTTP/1.1 会复用 TCP 连接（Connection: keep-alive），避免频繁握手。这点我们查看抓包中的 http 协议就可以确认。因此 `curl` 执行完毕后没有发送 FIN 报文段。

TCP 除了可靠传输，还有拥塞控制，来处理网络拥塞问题。

## 第四章 网络层

### 4.1 概述

网络层实现主机到主机的通信服务。


网络层的两个功能，转发（forward）和选路（route）。

- 转发：一个分组到达某个路由器的输入链路时，这个路由器必须将该分组移动到适当的输出链路。——对一个路由器而言
- 选路：当分组从发送方流向接收方的时候，网络层必须决定这些分组所采用的路由或者路径。——对整个传输而言

一台执行分组转发功能的机器，称为路由器（router）

网络服务模型。因特网的网络层提供的服务称为尽力而为服务：既不保证定时，也不保证顺序，也不保证不丢失。

### 4.2 路由器

路由器完成网络层的转发功能，将分组从一台路由器的入链路传送到适当的出链路。

每台路由器有一个转发表。路由器通过检查到达的分组的首部中某一个字段的值，然后用这个值在路由器的转发表中索引查询，查询的结果是分组将被转发出去的那个输出链路接口。

选路算法决定了一个路由器中转发表的值。

路由器结构图。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net8.png)

路由器由 4 个部分组成。

- 输入端口：输入端口做好几件事情。首先要执行将输入的物理链路接到路由器的物理层功能。然后是链路层功能，与链路层交互（协议、拆分）。还有查找与转发功能，使得交换结构部分的分组能够出现在适当的输出端口。还要将控制分组（control packet）（携带选路协议信息的分组）从输入端口转发到选路处理器。
- 交换结构：交换结构将路由器的输入端口连接到它的输出端口。
- 输出端口：输出端口接收交换结构转发给它的分组，将这些分组传输给输出链路。
- 选路处理器：选路处理器执行选路协议，维护选路信息和转发表。


下面看看输入端口的结构。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net9.png)

在其中包括线路端接、数据链路处理（协议、拆分）、查找输出端口并转发（可能要排队）、送至交换结构四个步骤。

交换结构可通过三种方式完成。内存交换、一根总线交换、多个总线纵横交换。

输出端口结构和输入端口结构基本一致，只是次序反过来。

如果输入速率大于输出速率，则等待处理的分组放在缓冲区中排队。

### 4.3 网际协议（The Internet Protocol，IP）

现在我们学习大名鼎鼎的 IP 协议。

因特网网络层由三个部分组成

- IP 协议，确定网络层中主机的编址规则、数据报的格式与分组处理的规则
- 选路协议（如 BGP），决定数据报从源到目的地所经过的路径
- 差错报告设施，用于报告数据差错和对某些网络层信息请求进行响应，如 ICMP 协议。ping 就是基于 ICMP 协议来检测网络。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net10.jpg)

#### IPv4 数据报格式

然后我们看看 IPv4 协议的数据报格式。IPv4 的数据报格式如下图所示：

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net11.png)

随便打开 wireshark 抓包看看：

```
Internet Protocol Version 4, Src: 114.55.100.69, Dst: 10.193.179.172
    0100 .... = Version: 4
    .... 0101 = Header Length: 20 bytes (5)
    Differentiated Services Field: 0x04 (DSCP: LE, ECN: Not-ECT)
    Total Length: 86
    Identification: 0x2645 (9797)
    010. .... = Flags: 0x2, Don't fragment
    ...0 0000 0000 0000 = Fragment Offset: 0
    Time to Live: 53
    Protocol: TCP (6)
    Header Checksum: 0x8a6f [validation disabled]
    [Header checksum status: Unverified]
    Source Address: 114.55.100.69
    Destination Address: 10.193.179.172
```

然后我们解释一下各个部分的内容。

- 版本：这里是 4，表示 IPv4
- 首部长度：确定数据报的数据部分从哪里开始。如果没有选项，则是 20 字节。（以 32 位为单位）
- 服务类型（TOS）：指定数据报的处理方式，例如优先级、延迟、吞吐量等。现代网络中通常使用区分服务代码点（DSCP）和显式拥塞通知（ECN）来扩展此字段。
- 总长度：首部加上数据的长度。该字段长 16 位，所以理论最大长度为 65535 字节。
- 标识、标志、片偏移：用于分片。DF 标志禁止分片。
- 生存时间（TTL）：8 位，表示数据报的最大跳数（hop count）。每经过一个路由器，TTL值减1，当TTL为 0 时丢弃数据报，防止数据报在网络中无限循环。
- 协议：指示这个 IP 数据报的数据部分应该交给哪一个运输层协议。如上面的数据报中该值是 6，则交给 TCP 协议。
- 首部校验和：用于检验数据报中的比特错误。
- 源 IP 地址
- 目的 IP 地址
- 选项
- 填充：使首部长度为 32 位的整数倍。
- 数据（有效载荷）

#### IPv4 编址

下面我们来看看 IPv4 编址。

主机和物理链路之间有一个接口（interface）连接。而路由器的任务是从链路上接收数据报并将这些数据报从其他链路转发，故路由器有多个接口，每个接口对应一条链路。IP 协议要求每台主机和路由器的接口都有自己的 IP 地址。

每个 IP 地址长度是 32 位，按字节分开。举例 IP：`192.168.30.2`。这种写法称为点分十进制。


一些特殊 IP 地址：

- `127.0.0.1`（本地回环地址）
- `0.0.0.0`（默认路由）
- `255.255.255.255`（广播地址）

由于 IPv4 地址资源有限，用子网划分（subnet）允许将一个大的网络划分为多个小的子网，以提高地址利用率。一个 IPv4 地址可以分为网络部分和主机部分。这种分配策略称为 CIDR（无类别域间路由）。

一个地址的形式为 `a.b.c.d/x`。`x` 指示 IP 地址的网络部分，地址前 `x` 位称为地址的前缀（prefix）。一个组织内部设备的 IP 地址的前缀都是相同的。在选路时，组织外部的路由器只需考虑地址前缀即可。

一个地址的后 `32-x` 位是主机部分，用于区分组织的内部设备（主机）。 举例，`x=24`，则后 8 位用于区分不同的设备。

我们可以使用子网掩码（subnet mask）来区分网络和主机部分，比如子网掩码是 `255.255.255.0`，即表示前 24 位是网络 IP。

网络地址相同的接口可以构成一个子网。下面图中，3 台路由器构成了 6 个子网。

![](https://cdn.jsdelivr.net/gh/peter5723/imagehost/img/net12.png)

#### 获取 IP 地址，DHCP 协议

然后我们看看在因特网中，主机和组织如何获取 IP 地址。

首先，一个组织获得从一个 ISP 那里获得一块地址。然后，这个主机为组织内的主机和路由器接口分配独立的 IP 地址。我们有的时候会自己手动配（比如在局域网连开发板的时候），但是在因特网中连接的话，一般就使用 DHCP 协议来给我们自动分配好。

DHCP 协议，全称 Dynamic Host Configuration Protocol，动态主机配置协议，自动为网络中的主机分配 IP 地址。它是一个基于 UDP 的应用层协议，是客户端-服务器模型。

为一个新主机，其通过四个步骤完成地址分配，称为 DORA。

1. Discover（发现）：这个新主机首先要寻找可用的 DHCP 服务器。客户机在 UDP 分组中向端口 67 发送 `DHCP Discover` 报文。但是它没有 IP 怎么办？不知道服务器地址怎么办？解决方法是使用源地址 `0.0.0.0`，目的地址使用 `255.255.255.255`，这是个广播地址，可以将这个分组传送到所有与这个子网连接的子网。
2. Offer（提供）：DHCP 服务器收到了 `DHCP Discover` 报文，然后它用一个 `DHCP Offer` 报文对客户机响应，当然，目标地址仍然需要用 `255.255.255.255` 进行广播。在报文中有可供客户机选择的 IP 地址、子网掩码等参数。
3. Request（请求）：服务器可能不止一个，客户端会收到许多 Offer，它选择其中一个 Offer，再广播 `DHCP Request` 报文，确认请求这个 IP。
4. ACK（确认）：服务器确认，发送 `DHCP ACK` 报文，完成分配。

#### NAT

NAT，全称 Network Address Translation，网络地址转换。为啥需要？在一个子网中，可能有非常多的设备连在一起，访问互联网。那 IPv4 的话只有 32 位，可能分配不过来。于是就引入了 NAT 技术。子网中的内部主机 IP 地址可能类似于 `10.0.0.1`，`10.0.0.2`，`10.0.0.3` 等等，它们和 NAT 路由器 `10.0.0.4` 相连。NAT 路由器还有一个对外的 IP 地址，比如 `138.76.29.7`。NAT 路由器将数据报的源私有 IP 替换成路由器的公有 IP，并记录转换映射。路由器接收外部服务器的响应后，再根据映射表将响应发送到对应的主机，就完成了一整个通讯过程。

#### IPv6


### 4.4 选路

选路是网络层的核心功能之一，负责确定数据报从源主机到目的主机所经过的路径（Path）。网络层根据选路算法计算最优路径，并维护转发表（Forwarding Table），指导路由器如何转发分组。

一台主机通常与一个路由器直接相连，这个路由器称为第一跳（first-hop）路由器或默认路由器。数据从路由器到路由器之间传一次称为一跳。源主机的第一跳路由器称为源路由器，目的主机的第一跳路由器称为目的路由器。

现在选路协议有 RIP、OSPF、BGP 等。

### 4.5 proxy，vpn

我们学了网络层，就能弄明白一些术语了。一是 proxy，即代理。代理是一种中介服务器，位于客户端和目标服务器之间，负责转发请求和响应。目标服务器看到的当然是代理服务器的 IP。有点像 NAT，当然完全不同。

二是 vpn，全称 Virtual Private Network，虚拟专用网络。其在公共网络（如互联网）上创建一条加密的“隧道”，将用户的设备与远程网络（如企业内网）安全连接。用户的所有流量全部加密，通过 vpn 服务器传输，故其本质上也是一种 proxy。公司和学校常用 vpn。

## 第五章 链路层


### 5.1 概述

先介绍一些术语。

网络层提供了两台主机间的通信服务。而分组是如何通过构成端到端通信路径的每一段链路，就是链路层协议考虑的事情。

本章中，主机和路由器均称为节点（node）。相邻两个路径的通信信道称为链路（link）。为了将一个从源主机传输到目的主机，数据报必须通过
端到端路径上的每段路径传输。

链路层数据单元称为帧（frame）。

通过特定的链路时，传输节点将数据报封装在链路层帧中，并将帧发送到链路上，然后接收节点接收该帧，提取出数据报。

同样，我们先考虑链路层提供什么服务。