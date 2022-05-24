# socket技术详解

Socket  编程：

Socket编程也就是面向服务器响应客户端请求之前，必须进行一些初步的设置流程来为之后的工作做准备。首先会创建一个通信端点，它能够使服务器监听请求。可以把服向务器当作公司的前台，或者应答公司主线呼叫的总计接线员。一旦电话号码和设备安装成功且接线员到达时，服务就可以开始了。
客户端所做的就比较简单了，只是创建它的单一通信点，然后建立一个服务器的连接，然后客户端就可以发出请求，该请求包含任何必要的数据交换。一旦请求被服务器处理，且客户端收到结果或某种确认信息此次通信就会结束。


如何创建一个Socket 套接字：

socket(family,type[,protocal]) 使用给定的地址族、套接字类型、协议编号（默认为0）来创建套接字。

| socket类型                           | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| socket.AF_UNIX                       | 只能够用于单一的Unix系统进程间通信                           |
| socket.AF_INET                       | 服务器之间网络通信                                           |
| socket.AF_INET6                      | IPv6                                                         |
| socket.SOCK_STREAM                   | 流式socket , for TCP                                         |
| socket.SOCK_DGRAM                    | 数据报式socket , for UDP                                     |
| socket.SOCK_RAW                      | 原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而SOCK_RAW可以；其次，SOCK_RAW也可以处理特殊的IPv4报文；此外，利用原始套接字，可以通过IP_HDRINCL套接字选项由用户构造IP头。 |
| socket.SOCK_SEQPACKET                | 可靠的连续数据包服务                                         |
| 创建TCP Socket：                     | s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)           |
| 创建UDP Socket：                     | s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)            |
| **socket函数**                       | **描述**                                                     |
| **服务端socket函数**                 |                                                              |
| s.bind(address)                      | 将套接字绑定到地址, 在AF_INET下,以元组（host,port）的形式表示地址. |
| s.listen(backlog)                    | 开始监听TCP传入连接。backlog指定在拒绝连接之前，操作系统可以挂起的最大连接数量。该值至少为1，大部分应用程序设为5就可以了。 |
| s.accept()                           | 接受TCP连接并返回（conn,address）,其中conn是新的套接字对象，可以用来接收和发送数据。address是连接客户端的地址。 |
| **客户端socket函数**                 |                                                              |
| s.connect(address)                   | 连接到address处的套接字。一般address的格式为元组（hostname,port），如果连接出错，返回socket.error错误。 |
| s.connect_ex(adddress)               | 功能与connect(address)相同，但是成功返回0，失败返回errno的值。 |
| s.recv(bufsize[,flag])               | 接受TCP套接字的数据。数据以字符串形式返回，bufsize指定要接收的最大数据量。flag提供有关消息的其他信息，通常可以忽略。 |
| s.send(string[,flag])                | 发送TCP数据。将string中的数据发送到连接的套接字。返回值是要发送的字节数量，该数量可能小于string的字节大小。 |
| s.sendall(string[,flag])             | 完整发送TCP数据。将string中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据。成功返回None，失败则抛出异常。 |
| s.recvfrom(bufsize[.flag])           | 接受UDP套接字的数据。与recv()类似，但返回值是（data,address）。其中data是包含接收数据的字符串，address是发送数据的套接字地址。 |
| s.sendto(string[,flag],address)      | 发送UDP数据。将数据发送到套接字，address是形式为（ipaddr，port）的元组，指定远程地址。返回值是发送的字节数。 |
| s.close()                            | 关闭套接字。                                                 |
| s.getpeername()                      | 返回连接套接字的远程地址。返回值通常是元组（ipaddr,port）。  |
| s.getsockname()                      | 返回套接字自己的地址。通常是一个元组(ipaddr,port)            |
| s.setsockopt(level,optname,value)    | 设置给定套接字选项的值。                                     |
| s.getsockopt(level,optname[.buflen]) | 返回套接字选项的值。                                         |
| s.settimeout(timeout)                | 设置套接字操作的超时期，timeout是一个浮点数，单位是秒。值为None表示没有超时期。一般，超时期应该在刚创建套接字时设置，因为它们可能用于连接的操作（如connect()） |
| s.gettimeout()                       | 返回当前超时期的值，单位是秒，如果没有设置超时期，则返回None。 |
| s.fileno()                           | 返回套接字的文件描述符。                                     |
| s.setblocking(flag)                  | 如果flag为0，则将套接字设为非阻塞模式，否则将套接字设为阻塞模式（默认值）。非阻塞模式下，如果调用recv()没有发现任何数据，或send()调用无法立即发送数据，那么将引起socket.error异常。 |
| s.makefile()                         | 创建一个与该套接字相关连的文件                               |

<h2>Socket实现流程：</h2>

![image-20211227153716824](C:\Users\F1241138\AppData\Roaming\Typora\typora-user-images\image-20211227153716824.png)

