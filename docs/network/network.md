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