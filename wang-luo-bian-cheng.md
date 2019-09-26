# 网络编程

我们需要理解基本的`客户端-服务端`编程模型，以及如何编写使用因特网提供的服务的`客户端-服务端`程序。

最后，我们将把所有这些概念结合起来，开发一个小的但功能齐全的`Web`服务器，能够为真实的`Web`浏览器提供静态的和动态的文本和图形内容。

## 客户端-服务器编程模型

每个网络应用程序都是基于`客户端 - 服务器模型`的

**客户端和服务器都是进程.而不是机器或主机.**

* 采用这种模型，一个应用是由一个`服务器`进程 和一个或多个客户端`进程`组成。
  * `服务器`管理某种资源，并且通过操作这种资源为它的客户端提供某种服务。
    * `WEB`服务器，代表`客户端`检索，执行磁盘内容。
    * `FTP`服务器，为`客户端`进行存储和检索。
    * `电子邮件`服务器，为`客户端`进行读和更新。
  * `客户端-服务器`模型中的基本操作是`事务(transaction)`.
    * 一个`客户端-服务器`事务由四步组成
      * 客户端需要服务的时候，向服务器发送请求，发送一个`事务`。
      * 服务器收到请求后，解释它，并以适当方式操作它的资源。
      * 服务器给客户端发送一个`响应`，并等待下一个请求。
      * 客户端收到`响应`并处理它。
    * **事务仅仅是客户端和服务器执行的一系列步骤.**

![](.gitbook/assets/ping-mu-kuai-zhao-20190925-xia-wu-8.03.20.png)

## 网络

`客户端`和`服务端`通常运行在不同的主机上，并且通过`计算机网络`的硬件和软件资源来通信。

* 对于一个主机而言，`网络`只是又一种`I/O`设备，作为**数据源**和**数据接收方**。

![](http://i.imgur.com/UWEx308.png)

* 对于物理上而言，网络是一个按照地理远近组成的层次系统。
  * 最低层是`LAN(Local Area Network,局域网)`:在一个建筑或校园范围内。
    * 迄今为止，最流行的`LAN`技术是`以太网(Ethernet)`.
      * 由`Xerox PARC`公司在20世纪70年代中期提出。
        * `以太网`被证明是适应力极强的，从`3 MB/s`到`10 GB/s`。
      * 一个`以太网段(Ethernet segment)`
        * ![](http://i.imgur.com/P4ZTZdh.png)
        * 包括一些`电缆(通常是双绞线)`和一个叫做`集线器`的小盒子。
          * 每根`电缆`都有相同的最大位带宽
            * 典型的是`100MB/s`或者`1GB/S`.
            * 一端连接在主机的`适配器`，一端连接到集线器的`一个端口`。
          * `集线器`不加分辨地将从一个端口收到的每个位复制到其他所有端口上。
            * 因此每台主机都能看到每个位。
        * `以太网段`通常跨越一些小的区域。
          * 例如某建筑物的一个房间或一个楼层。

扩展介绍`以太网`

每个`以太网适配器(网卡)`都有一个全球唯一的`48`位地址，它存储在这个适配器的`ROM`上\(`MAC`\)。

* 一台主机可以发送一段`位`，称为`帧(frame)`，到这个`网段`内其他任何主机。
  * 每个`帧`包括
    * 一些固定数量的`头部(header)`位
      * 用于表示此`帧`的源，和目的地址以及此`帧`的长度。
    * 此后就是数据位的`有效载荷`。
  * 每个主机适配器都能看到这个`帧`，但是只有目的主机实际读取它。

使用一些`电缆`和叫做`网桥(bridge)`的小盒子，多个`以太网段`可以连接称较大的局域网，称为`桥接以太网(bridged Ethernet)`。

![](http://i.imgur.com/fN9znxV.png)

* 一些`电缆`连接网桥与网桥，或者 网桥与集线器。
  * 这些电缆的带宽可以是不同的。

![](http://i.imgur.com/HiQyXb7.png)

在层次的更高级别，多个不兼容的局域网可以通过叫做`路由器(router)`的特殊计算机连接起来，组成一个`internet(互联网络)`

> Internet和internet  
>   
> 我们总是用小写字母的`internet`表示一般概念，大写的`Internet`表示一种具体实现，如全球IP因特网。

* `WAN`\(`Wide-Area Network`，广域网\)

![](http://i.imgur.com/oJUwlJs.png)

互联网至关重要的**特性**是:

* 它能由采用完全不同和**不兼容**技术的各种局域网和广域网组成。

`Q`:如何能够让某台`源主机`跨过所有这些不兼容的网络发送数据位到另一台`目的主机`呢?

`A`:解决办法是一层运行在每台主机和路由器上的`协议软件`，消除不同网络之间的差异。

* 这个软件实现一种`协议`:控制主机和路由器如何协调工作来实现数据传输。
  * 必须提供两种基本能力:
    * `命名机制`
      * 每台主机会被分配至少一个`互联网地址(internet address)`,这个地址唯一标识了这台主机。
    * `传送机制`
      * `协议`通过定义一种把数据位捆扎成不连续的片\(称为`包`\)的方式。
        * 一个`包`是由`包头`和`有效载荷`组成的。
          * `包头`
            * `包的大小`
            * `源主机`和`目的主机`地址
          * `有效载荷`包括从源主机发出的数据位

![](http://i.imgur.com/Nnjj0dR.png)

一个`客户端`运行在主机`A`上，主机`A`与`LAN1`相连，它发送了一串数据字节到运行在主机B上的服务器端，主机B则连接在`LAN2`上。有如下8个步骤。

1. 运行在主机`A`上的客户端进行系统调用，从客户端的`虚拟地址空间`拷贝到`内核缓冲区`。
2. 主机`A`上的`协议软件`通过在数据前附加互联网络`包头`和`LAN1`帧头，创建了一个`LAN1`的帧。
   * `互联网包头`寻址到互联网主机B。\(最终目的\)
   * `LAN1帧头`寻址到`路由器`。\(中转站\)
   * 封装
     * `LAN1帧`的**有效载荷**是`互联网络包`。
     * `互联网络包`的**有效载荷**是实际的用户数据。
     * 这种`封装`是基本的网络互联方法之一。
3. `LAN1`适配器拷贝该`帧`到网络上。
4. `帧`到达路由器，路由器的`LAN1适配器`从电缆上读取它，并传送到协议软件中。
5. 路由器从`互联网包头中`提取处目的互联网络地址，用它作为路由器的索引，确定向哪里转发这个包。
   * 路由器剥落旧的`LAN1`的帧头，加上寻址到主机`B`的新的`LAN2`帧头，并把得到的帧传送到适配器。
6. 路由器的`LAN2适配器`拷贝该`帧`到网络
7. `帧`到达主机B时，它的适配器从电缆上读到此帧，并将它传送到协议软件。
8. 最后，主机B上的协议软件剥落包头和帧头。服务器进行一个读取这些数据的`系统调用`。

当然，在这里，我们掩盖了许多非常艰难的问题。

* 如果不同的网络有不同`帧`大小的最大值，该怎么办。
* 路由器如何知道往哪里转发`帧`。
* 网络拓扑变化的时候，如何通知路由器。
* 包丢失了，会如何?

虽然如此，我们也能大概了解到互联网络思想的精髓。

## 全球 IP 因特网

![](http://i.imgur.com/T3BNTQy.png)

每台因特网主机都运行实现`TCP/IP`协议 \(Transmission Control Protocol/Intelnet Protocol,传输控制协议/互联网络协议\)的软件，几乎所有计算机系统都支持这个`协议`。

* 因特网的客户端和服务端混合使用`套接字接口`函数和`Unix I/O`函数来进行通信。
  * `套接字函数`典型地是作为会陷入内核的`系统调用`来实现的，并调用各种内核模式的`TCP/IP`函数。

`TCP/IP`协议实际上一个`协议族`，每一个协议提供不同的功能。

* 例
  * `IP`协议提供基本的命名方法，和传递机制。
    * 这种`传递机制`能够从一台因特网主机往其他主机发送包，也叫做`数据报(datagram)`
    * `IP`机制从某种意义上是不可靠的，如果数据报在网络丢失或重复，并不会试图恢复。
      * `UDP(Unreliable Datagram Protocol,不可靠数据报协议)`稍微扩展了`IP`协议。
        * 这样，`包`可以在`进程`间，而不是`主机`间传送。
  * `TCP`是一个构建在`IP`之上的复杂协议，提供了进程间可靠地`全双工(双向)`的连接。

为了简化讨论

* 我们将`TCP/IP`看作是一个单独的整体协议。
* 不讨论它的内部工作，只讨论`TCP`和`IP`为应用程序提供的基本功能。
* 不讨论`UDP`

从程序员的角度，我们可以把因特网看作世界范围内主机的集合，满足一下特性。

* 主机集合被映射为一组`32`位的`IP`地址。
* 这组`IP`地址可以被映射为一组称为`因特网域名(Internet domain name)`的标示符。
* 因特网主机上的进程能够通过`连接`和任何其他主机上的进程通信。

### IP地址

一个`IP`地址就是一个32位无符号整数。网络程序将`IP`地址存放在一个`IP地址结构`中, 并且是大端表示法.

```c
/* Internet address structure */
#include<arpa/inet.h>
struct in_addr{
    unsigned int s_addr;
}

大端表示法存储.
IP : 168.122.34.1
s_addr = 2826576385
  十六进制 =  0xA8 7A 22 01
    二进制 = 1010 1000 0111 1010 0010 0010 0000 0001
```

> 为什么要用结构来存放标量`IP`地址  
>   
> 是早期的不幸产物，但是现在更改太迟了。

**Unix提供下面这样的函数在网络\(大端\)和主机字节顺序\(小端\)之间实现转换.**

```c
#include <arpa/inet.h>
typedef unsigned int    uint32_t;
typedef unsigned short  uint16_t;

uint32_t htonl(uint32_t hostlong);        //本 转 网 IP    h  to n l
uint16_t htons(uint16_t hostshort);       //本 转 网 端口   h  to n s

uint32_t ntohl(uint32_t netlong);        //网 转 本 IP     n to h l
uint16_t ntohs(uint16_t netshort);       //网 转 本 端口    n to h s
```

 

IP地址通常是以一种称为`点分十进制表示法`来表示的

* 这里，每个`字节`\(8位\)都是由它的十进制表示\(0~255\)，并且用句点和其他字节间分开。
* 在Linux系统上，你能够使用`hostname`命令来确定你自己主机的点分十进制:

  ```text
    linux> hostname -i
    10.174.204.145
  ```

* **可以使用`inet_pton`和`inet_ntop`函数来实现两者之间互相转换。**

```c
#include <arpa/inet.h>
int inet_pton(AF_INET, const char* src, void* dst);
        点分十进制字符串 转无符号int网络字节序(大端).
              src: 需要转换的十进制字符串
              dst: 接受转换完成的网络字节序, uin32_t 类型,一般使用 struct in_addr 来接收.
          返回值: 成功返回1, src参数非法则返回0, 出错返回-1
        
const char* inet_ntop(AF_INET, const void* src, char* dst, socklen_t size);
        无符号int网络字节序转点分十进制字符串.
             src: 网络字节序地址, uint32_t类型, 一般还是 struct in_addr
             dst: 接受转换完成后的点分十进制地址字符串.
             size: 表示的是 dst 数组的大小.
          返回值: 成功指向点分十进制字符串的指针(指向dst), 出错返回 NULL
```

### 因特网域名

方便人们记忆的对于`IP`的映射就是`域名`。

`域名集合`形成了一个层次结构，每个域名编码了它在层次中的位置.

![](http://i.imgur.com/3XVt4jl.png)

* 叶子结点反向到根的`路径`就是`域名`。
* 层次结构第一层 : 未命名的根结点
* 层次结构第二层 : `一级域名(first-level domain name)`
  * 由非盈利组织`ICANN`\(Internet Corporation for Assigned Names and Numbers,因特尔分配名字数字协会\)定义。
  * 常见的一级域名:`com`,`edu`,`gov`,`org`和`net`。
* 层次结构第三层: `二级域名(second-level)`
  * 例如:`cmu.edu`。
  * 这些域名是由`ICANN`的各个授权代理按照先到先服务的基础分配的。
  * 一旦一个组织得到一个二级域名，那么它就可以在这个子域中创建任何新的域名了。



因特网定义了`域名集合`和`IP`地址直接的映射。

* `HOSTS.TXT`
  * 直到`1988`年，这个映射都是通过一个叫做`HOSTS.TXT`的文本文件来手工维护的。
* `DNS`:
  * 之后，通过分布世界范围内的数据库\(`DNS`，`Domain Name System`,域名系统\)，来维护的。
  * `DNS`数据库由上百万的`主机条目结构(host entry structure)`组成的。
    * 定义了一组`域名`\(一个官方名字和一个别名\)和一组`IP`地址之间的映射。

{% hint style="info" %}
每台因特网主机都有本地定义的域名  localhost  , 这个域名总是映射为 **回送地址 `127.0.0.1`**

#### 可以使用命令  nslookup   来进行查看,和解析,\(会出现无法解析的情况\)

* Linux&gt;   nslookup  localhost
  * 输出:      Address:  127.0.0.1
* Linux&gt;   nslookup  www.baidu.com
  * 输出:      Address: 61.135.169.125
{% endhint %}



### 因特网连接

`Internet`服务端和客户端通过在`连接`上发送和接收`字节流`来通信。

* 从连接一对进程的意义上而言，连接是`点对点`的。
* 从数据可以同时双向流动的角度来说，它是`全双工`的。
* 并且从由源进程发出的字节流最终被目的进程按照发送的数据接收来说，它是`可靠`的

一个`套接字`是`连接的`一个端点。

* 每个套接字都有相应的`套接字地址`。
  * 是由一个`IP`地址和一个16位的整数`端口`组成的，用`地址:端口`来表示。
* 当客户端发起一个连接请求时，**客户端**套接字地址中的`端口`由内核**自动分配**的。
  * 称为`临时端口`
* **然后**，服务器套接字地址中的`端口`通常是某个**知名的端口**，和这个服务相对应的。
  * 例如:
    * **Web服务器**通常使用端口`80`
    * **电子邮件服务器**使用端口`25`
  * 在`Unix`机器上，文件`/etc/services` 包含一张这台机器提供的服务和他们的知名端口号的综合列表。

一个`连接`是由它两端的**套接字地址**唯一确定的。

* 这对套接字地址叫做`套接字对(socket pair)`,由下列元组来表示:
  * `(cliaddr:cliport,servaddr:servport)`

![](http://i.imgur.com/1SceWbw.png)

## 套接字接口

`套接字接口(socket interface)`是一组函数，他们和`Unix I/O`函数结合起来，用以创建网络应用。

给出一个典型的**客户端-服务器事务**的上下文中`套接字接口`概述，以此导向。

![](http://i.imgur.com/kyVfcmT.png)

![](http://i.imgur.com/rFbybKQ.png)



### 套接字地址结构

不同的角度:

* 从`Unix`内核角度来看，一个套接字就是通信的一个`端点`。
* 从`Unix`程序来看，套接字就是一个有相应描述符的打开文件。

#### 套接字地址结构

```c
IP地址结构, IP和端口号走势以网络字节顺序(大端)存放的.
#include <sys/socket.h>

struct sockaddr_in{                    /* 保存套接字地址 (网络字节序)*/
    uint16_t        sin_family;        /* 协议族,IP4给 AF_INET, IP6给AF_INET6 */
    uint16_t        sin_port;          /* 端口号 */
    struct in_addr  sin_addr;          /* 用来存储IPv4地址的结构体(网络字节序),定义在IP*/
    unsigned char   sin_zero[8];       /* 结构长度垫到 sizeof（struct sockaddr）相同*/
};

struct in_addr {        // 用来存储 IPv4 的地址
    unsigned int  s_addr ;          /* internet地址, 32位IPv4 地址 (网络字节序) */
};

struct sockaddr{                /* 通用套接字地址结构, connect(), bind(), accept() */
    uint16_t  sa_family;        /* 协议族,IP4给 AF_INET, IP6给AF_INET6 */
    char      sa_adta[14];      /* 14 字节的协议地址, (有可能会更长)  */
};
```

`Internet`的套接字地址\(`Internet-sytle`\)存放在类型为`sockaddr_in`的16字节结构中。

* `sin_family`成员是`AF_INET`，ipv4还是ipv6。
* `sin_port`成员是一个16位的端口号。
* `sin_addr`成员就是一个32位的`IP`地址。
  * `IP`地址和端口号总是以网络字节顺序\(大端法\)存放的。
* `sin_zero` 是填充，使得`sockaddr_in`和`sockaddr`一样大。

`sockaddr_in`给程序员操作的，`sockaddr`交由套接字函数使用的，两者可以直接强制转换。

```c
typedef  struct  sockaddr SA;   
    #为了强转方便,那么这么来定义,以方便下面代码的书写和阅读.
```



### socket 函数 \(客户端+服务器\)

客户端和服务端使用`socket`函数来创建一个`套接字描述符(socket descriptor)`

跟`open`差不多

```c
#include<sys/types.h>
#include<sys/socket.h>

int socket(int domain, int type, int protocol);
      doman 参数:  IP协议   AF_INET        ipv4网域  
                           AF_INET6       ipv6网域
			   	           AF_UNIX        本地套接字
      type  参数:  指定采用的哪种协议, SOCK_STREAM   表示采用TCP 流式协议,
                                    SOCK_DGRAM   表示采用UDP 报式协议.
                                    SOCK_RAW     允许对底层协议的直接访问, 用于新网络协议测试
      protocol  参数: 给0 就好了, 表示按给定的域和套接字类型选择默认协议.
 返回值:  是一个文件描述符, (就是个套接字)可以使用IO 文件操作来进行读写.  否则-1
        创建出来的返回值就是套接字.
    范例:  int lfd = socket(AF_INET, SOCK_STREAM, 0);
```

我们总是带这样的参数调用`socket`函数:

```c
clientfd = Socket(AF_INET,SOCK_STREAM,0);
```

* `AF_INET`表面我们在使用IPV4协议。
* `SOCK_STREAM`表示这个套接字是**`TCP 流式协议。`**
* `socket`返回的`clientfd`描述符，仅仅是部分打开，还不能用于读写。
  * 如何完成打开套接字的工作，取决于我们是客户端还是服务器。
  * 下一节描述我们是客户端时如何打开套接字。

### conect 函数\(客户端\)

客户端通过调用`connect`函数来建立和服务器的连接

```c
#include<sys/socket.h>

创建与指定外部端口的连接 (一般是客户端用来连接服务端的函数)
int connect(int sockfd, const struct sockaddr* addr, socklen_t addrlen);
                sockfd  参数: 套接字
                addr    参数: 服务器端的IP和端口
                addrlen 参数: 第二个参数的长度
        返回值: 成功返回9 ,否则-1, 并且把error设置一个值.

    范例:    struct sockaddr_in server;
            connect(lfd, (struct sockaddr*)&server, sizeof(server));
```

`connect`函数试图于套接字地址为`serv_addr`的服务器建立一个因特网连接.

* 其中`addrlen`是`sizeof(sockaddr_in)`.
* `connect`函数会**阻塞**，一直到连接成功建立或是发生错误。
* 如果成功，`sockfd`描述符就可以读写了。
  * 并且得到链接是由套接字对`(x:y,serv_addr.sin_addr,serv_addr.sin_port)`刻画的。
    * 其中`x`是客户端IP地址，而`y`表示临时端口。
    * 它唯一地确立了客户端主机上的客户端进程。

###  bind 函数,  listen函数, accept函数 \(服务器\)

#### bind

剩下的套接字函数`bind`，`listen`和`accept`被服务器用来和客户端建立链接。

```c
#include<sys/socket.h>
将本地的IP 和端口 与创建出的套接字绑定 (客户端不需要绑定, 只有服务端需要)
int bind( int scokfd, const struct sockaddr* addr, socklen_t addrlen);
    sockfd  参数: 创建出的文件描述符
    addr    参数: 端口和IP  (是个结构体,下面有),可以使用 sockaddr_in 强转 sockaddr 来使用.
    addrlen 参数: sockaddr 结构体的长度.
 返回值: 成功返回 0
        失败返回-1, 并且把error设置一个值
        
 范例:   struct sockaddr_in server;  
         bind(lfd, (struct sockaddr*)&server, sizeof(server)); 
```

`bind`函数告诉内核将`my_addr`中的服务器套接字地址和套接字描述符`sockfd`联系起来。

* 参数`addrlen`就是`sizeof(sockaddr_in)`

#### listen

客户端是发起连接请求的主动实体。服务器是等待来自客户端连接请求的被动实体。

* **默认情况**下，内核会认为`socket`函数创建的描述符对应于`主动套接字(active socket)`.
  * 它存在于一个连接的客户端。
* 服务器调用`listen`告诉内核，描述符是被**服务器**而不是客户端使用的

```c
#include<sys/socket.h>
设置监听同时连接到服务器的客户端的个数
 int listen(int sockfd, int backlong);
                sockfd  参数: socket函数创建出来的文件描述符
                backlong参数: 同时连接的个数,填写同时连接的最大值 128 就可以了
        返回值: 成功返回 0
               失败返回-1, 并且把error设置一个值

    范例:   listen(fd, 20);
```

`listen`函数将`sockfd`从一个`主动套接字`转化为一个`监听套接字(listenning socket)`。

* 该套接字可以接收来自客户端的连接请求。
* `backlog`参数暗示了内核在开始拒绝连接请求之前，应该放入队列中等待的未完成连接请求的数量。
  * `backlog`参数的确切含义要求对`TCP/IP`协议的理解，这超出了我们的讨论的范围。
  * 通常我们会把它设置成一个较大的值，比如`1024`。

#### accept

```c
#include <sys/socketc.h>
阻塞等待客户端连接请求,并接受连接 (阻塞函数)
int accept(int sockfd, struct sockaddr *addr, socklen_t* addrlen);
         sockfd  参数: 文件描述符, 使用socket 创建出的 监听套接字描述符                      
         addr    参数: 存储客户端的端口和IP, 传出参数.
        addrlen  参数: 传入传出参数. (可能是addr 的大小),这个值必须单独拿出来,会有用,绝对不能sizeof()

        返回值: 返回的是一个套接字, 对应客户端
                服务器端与客户端进程通信使用 accept 的返回值对应的套接字.
              如果失败则返回 -1 ,并且把errno 设置一个值.
            
* 如果该函数等待连接阻塞时,被信号中断了,那么errno会被设置成 EINTR ,并且返回 -1.那么解决方式是:
   int ffd = accept(lfd, (struct sockaddr*)&client, &len);   // 出现信号,然后去处理,回来之后就解除阻塞了
   while(ffd == -1 && errno == EINTR ){
        ffd = accept(lfd, (struct sockaddr*)&client, &len);   // 再来一次就可以了,
    }                                                         // 如果再次被信号中断,那也出不去循环.
    范例:    struct sockaddr_in client;
            socklen_t  len = sizeof(client);
            int cfd = accept(lfd,(struct sockaddr*)&client, &len);
```

* 用于从已完成连接`队列`队头返回下一个已完成连接。
  * 如果已完成连接为空，那么进程进入阻塞\(假定套接字为默认的阻塞方式\)
* 返回三个值
  * `已连接标示符`
  * `客户端地址`
  * `客户度地址长度`

`监听描述符`和`已连接描述符`之间的区别是很多人迷惑。

* `监听描述符`是作为**客户端连接请求**的一个端点。
  * 它被创建一次，并存在于**服务器**的整个生命周期。
* `已连接描述符`是客户端和服务器之间已经建立起来的**连接**的一个端点。
  * 服务器每次接收连接请求时都会创建一次。
  * 它只存在于服务器为一个客户端服务的过程中。

![](.gitbook/assets/ping-mu-kuai-zhao-20190926-shang-wu-10.46.33.png)

### 主机和服务的转换

Linux 提供了一些强大的函数 \( 称为 **`getaddrinfo`** 和 **`getnameinfo`** \) 实现二进制套接字地址结构和主机名, 主机地址, 服务名 和 端口号的字符串表示之间的相互转化.  和套接字地址一起使用时,这些函数能使我们编写独立于任何特定版本的IP协议的网络程序.

#### 都会用到的数据结构

```c
struct addrinfo {
    int              ai_flags;     /* 提示参数标志,是一个掩码位, AI_xxx */
    int              ai_family;    /* IP4或IP6 , 用作 socket() 第一个参数 */
    int              ai_socktype;  /* 表示采用通讯协议,用作 socket() 第二个参数 */
    int              ai_protocol;  /* 表示按给定的域和套接字类型选择默认协议,用作 socket() 第三个参数 */
    char            *ai_canonname; /* 无需设置, 是主机域名 */
    size_t           ai_addrlen;   /* 无需设置, ai_addr结构体的大小,用作connect()和bind()的第三个参数 */
    struct sockaddr *ai_addr;      /* 无需设置, 套接字结构地址指针,用作connect()和bind()的第二个参数*/
    struct addrinfo *ai_next;      /* 无需设置, 指向链接列表中的下一个项目 */
};
需要设置的是三个变量: ai_flags, ai_family, ai_socktype.
    ai_flags :  AI_ADDRCONFIG 当前主机是IP4时,getaddrinfo() 就返回IP4地址,IP6同理, 如果不设置, 那么IP4和IP6都返回
                AI_CANONNAME   告诉函数getaddrinfo(), 第一个参数是 域名 而不是IP地址.
                AI_NUMERICSERV  告诉函数 getaddrinfo() 和 getnameinfo(), 参数service是端口号而不是服务名
                AI_PASSIVE     告诉getaddrinfo() 默认返回套接字地址会被服务器用作监听套接字, 而且host参数必须是NULL
    ai_family:  AF_INET    IP4地址
                AF_INET6   IP6地址
  ai_socktype:  SOCK_STREAM    TCP流式协议
                SOCK_DGRAM     UDP报式协议
                SOCK_RAW       允许地产协议的直接访问, 用于新网络协议测试

服务器设置该结构示例:     TCP协议, 使用端口号, IP4 地址
    struct addrinfo  addr, *serv;  /* 用来初始化和接收 getaddrinfo 参数和返回值的 */
    int  err = 0;
    memset(&addr, 0, sizeof(struct addrinfo));
    addr.ai_flags = AI_ADDRCONFIG | AI_NUMERICSERV | AI_PASSIVE;
    addr.ai_family = AF_INET;
    addr.socktype = SOCK_STREAM;
    if( (err = getaddrinfo(NULL,"80",&addr, &serv)) != 0)
        fprintf(strerr,"%s\n",gai_streror(err));
    int fd = socket( serv->ai_family, serv->socktype, serv->ai_protocol);
    bind( fd, serv->ai_addr, serv->ai_addrlen);
    listen( fd, 1024);

    struct sockaddr client_in;   /* 这个用来存放客户端信息的结构,不是 addrinfo */
    size_t  clietn_in_len;
    char hostname[16] = { [1 ... 16-2]='\0','\0'};  /* 存放客户端IP地址 */
    char service[8] = { [1 ... 8-2]='\0','\0'};     /*存放客户端 端口号 */
    
    int cfd = accpet(fd, &client_in, &client_in_len);   /* 注意参数 */
    getnameinfo(&clent_in, client_in_len, hostname, sizeof(hostname),
                service, sizeof(service), 0);   /*获取客户端的IP地址和端口号*/
    printf("客户端IP = %s , 端口= %s \n", hostname, service );
    freeaddrinfo(&serv);        /* 一定要释放getaddrinfo 得到的数据结构 */
    

客户端设置示例:     TCP协议, 使用端口号, IP4地址
    struct addrinfo  addr, *serv;
    int  err = 0;
    memset(&addr, 0, sizeof(struct addrinfo));
    addr.ai_flags = AI_ADDRCONFIG | AI_NUMERICSERV ;  /* 这里体现了和服务器的区别*/
    addr.ai_family = AF_INET;
    addr.ai_socktype = SOCK_STREAM;
    if( (err = getaddrinfo("1.1.1.1","80",&addr, &serv)) != 0)  /* IP 不为NULL了 */
        fprintf(strerr,"%s\n",gai_streror(err));
    int fd = socket( serv->ai_family, serv->socktype, serv->ai_protocol);
    conect( fd, serv->ar_addr, serv->ai_addrlen);   /*连接服务器*/
    freeaddrinfo(&serv);        /* 一定要释放getaddrinfo 得到的数据结构 */    
    
```

#### getaddrinfo 函数

```c
将  主机名, 主机地址, 服务名 和 端口号 的字符串表示转换成套接字地址结构.

#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo( 
         const char* host,                /*服务器给NULL,客户端给 服务器的IP地址或域名*/
         const char* service,             /*端口号或服务名*/
         const struct addrinfo* hints,    /*一个数据结构,包含IP类型,协议类型,函数行为*/
         struct addrinfo** result         /*这个是返回的 addrinfo 结构的链表 */
         );
    参数:  host: 服务器给 NULL , 客户端给 服务器的IP地址或者域名
       service: 服务器写开放和监听端口号字符串, 客户端♏️服务器开发和监听的端口号字符串或 http
      addrinfo: 一个被 memset() 初始化 并且设置完成的 addrinfo 数据结构.
        result: 函数初始化并返回的一个二级指针,使用这个返回指针来进行操作,里面是初始化完成的结构.

返回值: 如果成功返回0, 错误则返回非零的错误代码.使用 gai_strerror(错误码); 来输出查看.
```

#### getnameinfo 函数

```text
将 套接字地址结构转换成相应的 主机和服务名字符串. ( 和getaddrinfo() 作用相反 ),一般用在服务器端.
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int  getnameinfo(
    const struct sockaddr* sa,   /* 通过 accect 得到的第二个参数的数据结构,里面包含IP和端口号*/
    socklen_t   salen,           /* sockaddr 的结构大小 */
    char*   host,
    size_t  hostlen,
    char*   service,
    size_t  servlen,
    int   flage
    );

```

























