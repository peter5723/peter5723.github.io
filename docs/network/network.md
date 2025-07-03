# 计算机网络介绍

## 第一章 总览

传输的信息包基本单元：packet（分组，包）

端
接入网
网络核心——ISP（因特网服务提供商）

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

首先仍然是概要：运输层为应用层的程序提供了“逻辑通信”功能。这里的“逻辑”是封装的意思。不同主机的应用层程序通过运输层连接好像直接相连一般，而不需要考虑实际的实现如何。运输层协议是在端系统中实现的。发送方的运输层将发送程序的报文转化成运输层分组，这个分组称为运输层报文段。然后将报文段传给网络层，网络层将其封装成网络层分组（称为数据报），向目的地发送。接收方的网络层提取出运输层报文段，向上交给运输层。

运输层为不同主机上的进程提供了逻辑通信，而网络层则提供了主机之间的逻辑通信。这句话，看自顶向下方法中的邮政服务例子就容易理解了。

运输层服务有两种：UDP 和 TCP。

首先介绍网络层。因特网网络层协议，有一个 IP 协议。IP 为主机之间提供逻辑通信。IP 的服务模型是尽力而为交付服务，是不可靠服务，数据可能丢失。每台主机至少有一个网络层地址，即 IP 地址。

再介绍运输层的具体任务。运输层协议的基本任务是将两个端系统之间的 IP 交付服务扩展成两个端系统上的进程之间的交付服务。主机间交互扩展成进程间交付的两个动作称为 transport-layer multiplexing and demultiplexing （多路复用和多路分解）。运输层协议可以在报文段的首部添加查错检测字段提供完整性检查。UDP 只进行数据交付和差错检测，也是不可靠服务，不保证数据完整性。TCP 提供附加服务：可靠数据传输和拥塞控制。
