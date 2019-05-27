---
title: "tinyHttp"
date: 2019-05-15T18:41:25+08:00
tags:
- Thinyhttpd
#thumbnailImage: //example.com/image.jpgt
---



最近在看TinyHttp的源码。记录了一些，比较模糊的概念。

<!--more-->

### intptr_t

具体代码在linux平台的/usr/include/stdint.h头文件中。

````c
/* Types for `void *' pointers.  */
#if __WORDSIZE == 64
# ifndef __intptr_t_defined
typedef long int		intptr_t;
#  define __intptr_t_defined
# endif
typedef unsigned long int	uintptr_t;
#else
# ifndef __intptr_t_defined
typedef int			intptr_t;
#  define __intptr_t_defined
# endif
typedef unsigned int		uintptr_t;
````

应该是为了提高平台的可移植性。比如在64位的操作系统中，在指针的大小是8字节，但是在32位操作系统为4字节。

```c
//C code , arg 是 void * 
int clinet = (intptr_t) arg；
```

### strcasecmp

无视大小写，比较字符串

```c
if( strcasecmp(method,"GET") );//get、GET和gEt都可以
```

### stat 

通过文件名filename获取文件信息，并保存在buf所指的结构体stat中

> 定义函数:    int stat(const char *file_name, struct stat *buf);

```c
struct stat {
    dev_t         st_dev;       //文件的设备编号
    ino_t         st_ino;       //节点
    mode_t        st_mode;      //文件的类型和存取的权限
    nlink_t       st_nlink;     //连到该文件的硬连接数目，刚建立的文件值为1
    uid_t         st_uid;       //用户ID
    gid_t         st_gid;       //组ID
    dev_t         st_rdev;      //(设备类型)若此文件为设备文件，则为其设备编号
    off_t         st_size;      //文件字节数(文件大小)
    unsigned long st_blksize;   //块大小(文件系统的I/O 缓冲区大小)
    unsigned long st_blocks;    //块数
    time_t        st_atime;     //最后一次访问时间
    time_t        st_mtime;     //最后一次修改时间
    time_t        st_ctime;     //最后一次改变时间(指属性)
};
```



###	 TCP粘包问题

来自 [asklw](<https://blog.csdn.net/asklw/article/details/79244386>)

> - TCP（transport control protocol，传输控制协议）是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的socket，因此，发送端为了将多个发往接收端的包，更有效的发到对方，使用了优化方法（Nagle算法），将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样，接收端，就难于分辨出来了，必须提供科学的拆包机制。 **即面向流的通信是无消息保护边界的。**
>
> - UDP（user datagram protocol，用户数据报协议）是无连接的，面向消息的，提供高效率服务。不会使用块的合并优化算法，, 由于UDP支持的是一对多的模式，所以接收端的skbuff(套接字缓冲区）采用了链式结构来记录每一个到达的UDP包，在每个UDP包中就有了消息头（消息来源地址，端口等信息），这样，对于接收端来说，就容易进行区分处理了。 **即面向消息的通信是有消息保护边界的。**

由于TCP无消息保护边界, 需要在消息接收端处理消息边界问题。也就是为什么我们以前使用UDP没有此问题。 反而使用TCP后，出现少包的现象

来自 lzk_baggk

> 在TCP有流水线之后，本来应该是发送方不停的发送数据，来提高信道的使用率。但是发送端可能并没有这多的数据，而且大量的小报文会减低有用信息在实际传输过程中的比例，因为需要为每个小报文都加一个TCP头。所以就有了Nagle算法。具体是因为什么。我还没有完全的想清楚。所以这里先暂时记录一下自己的想法

### 大小端与网络字节序

​	大小端是网络字节数，是网络编程中，需要考虑的问题。首先，发送方要将本主机的字序换成网络的字节序，然后再交给目的主机，之后目的直接换成自己的字节序。

#### 什么是大端模式、小端模式

​	“大端”和”小端”表示**多字节值**的哪一端存储在该值的起始地址处;小端存储在起始地址处,即是小端字节序;大端存储在起始地址处,即是大端字节序;具体的说： 

   + 大端字节序（Big Endian）：最高有效位存于最低内存地址处，最低有效位存于最高内存处； 

   + 小端字节序（Little Endian）：最高有效位存于最高内存地址，最低有效位存于最低内存处。 

     如下图：当以不同的存储方式，存储数据为0x12345678时： 
     　　![大小端模式示意图](/images/bigend.jpg)

#### 网络字节序

　　网络上传输的数据都是字节流,对于一个多字节数值,在进行网络传输的时候,先传递哪个字节?也就是说,当接收端收到第一个字节的时候,它将这个字节作为高位字节还是低位字节处理,是一个比较有意义的问题; 
　　UDP/TCP/IP协议规定:**把接收到的第一个字节当作高位字节看待**,这就要求发送端发送的第一个字节是高位字节;而在发送端发送数据时,发送的第一个字节是该数值在内存中的起始地址处对应的那个字节,也就是说,该数值在内存中的起始地址处对应的那个字节就是要发送的第一个高位字节(即:高位字节存放在低地址处);由此可见,多字节数值在发送之前,在内存中因该是以大端法存放的; 
　　**所以说,网络字节序是大端字节序；**

### sockaddr和sockaddr_in详解

truct sockaddr和struct sockaddr_in这两个结构体用来处理网络通信的地址。

#### 一、sockaddr

**sockaddr**在头文件`#include <sys/socket.h>`中定义，sockaddr的缺陷是：sa_data把目标地址和端口信息混在一起了，如下：

```c
struct sockaddr {  
     sa_family_t sin_family;//地址族
　　  char sa_data[14]; //14字节，包含套接字中的目标地址和端口信息               
};
```

#### 二、sockaddr_in

**sockaddr_in**在头文件`#include<netinet/in.h>或#include <arpa/inet.h>`中定义，该结构体解决了sockaddr的缺陷，把port和addr 分开储存在两个变量中，如下： 

````c
struct sockaddr_in{
    sa_family_t 		sin_family;		//地址族(Address Family)
    uint16_t			sin_port;		//16位 TCP/UDP端口号
    struct in_addr		sin_addr;		//32位IP地址
    char				sin_zero[0];	//不使用
};

struct in_addr{
    In_addr_t			s_addr;			//32位IPv4的地址
}
````

sin_port和sin_addr都必须是网络字节序（NBO），一般可视化的数字都是主机字节序（HBO）。

来自lzk_baggk

> sockaddr 与 sockaddr_in都是用来网络通信的，在socket在刚刚被设计的时候，并不是为了TCP/IP设计的，而是希望未来可以兼容更多的网络协议。
>
> 使用面向对象的思想，可以这样认为，sockaddr是一个基类，sockaddr_in是sockaddr的一个派生类，专门用在internet网络中。sockaddr_in命名的那个in，应该也就是internet的意思了。

### getsockeopt与setsockopt

主要的作用是获取、设置影响套接口的选项。

原型：

```c
#include <sys/socket.h>

int getsockopt(int sockfg,int level,int optname,void *optval,socklen_t *optlne);
int setsockopt(int sockfg,int level,int optname,const void *optval,socklent_t *optlen);
```

我这里主要介绍的参数是

level:SOL_SOCKET	optname = SO_REUSEADDR;

#### SO_REUSEADDR

SO_REUSEADDR的作用主要包括两点
 **1、改变了通配绑定时处理源地址冲突的处理方式**，其具体的表现方式为：未设置SO_REUSEADDR时，socketA先绑定到0.0.0.0:21，后socketB绑定192.168.0.1:21将失败，不符合规则3。但在设置SO_REUSEADDR后socketB将绑定成功。并且这个设置对于socketA（通配绑定）和socketB（特定绑定）的绑定是顺序无关的。

| SO_REUSEADDR | 先绑定socketA  | 后绑定socketB  | 绑定socketB的结果  |
| ------------ | -------------- | -------------- | ------------------ |
| 无关         | 192.168.0.1:21 | 192.168.0.1:21 | Error (EADDRINUSE) |
| 无关         | 192.168.0.1:21 | 10.0.0.1:21    | OK                 |
| 无关         | 10.0.0.1:21    | 192.168.0.1:21 | OK                 |
| disable      | 0.0.0.0:21     | 192.168.1.0:21 | Error (EADDRINUSE) |
| disable      | 192.168.1.0:21 | 0.0.0.0:21     | Error (EADDRINUSE) |
| enable       | 0.0.0.0:21     | 192.168.1.0:21 | OK                 |
| enable       | 192.168.1.0:21 | 0.0.0.0:21     | OK                 |
| 无关         | 0.0.0.0:21     | 0.0.0.0:21     | Error (EADDRINUSE) |

对于linux，要使这个设置达到预期效果，对于绑定的顺序的有要求的，即在设置了SO_REUSEADDR，须先进行特定绑定，后进行通配绑定，后者才能成功；如果先进行通配绑定，后面的绑定（端口相同情况下）地址只要和通配绑定中的一个相同都将失败。

**2、改变了系统对处于TIME_WAIT状态的socket的看待方式**，要理解这个句话，首先先简单介绍以下什么是处于TIME_WAIT状态的socket？

> socket通常都有发送缓冲区，当调用send()函数成功后，只是将数据放到了缓冲区，并不意味着所有数据真正被发送出去。对于TCP socket，在加入缓冲区和真正被发送之间的时延会相当长。这就导致当close一个TCP socket的时候，可能在发送缓冲区中保存着等待发送的数据。为了确保TCP的可靠传输，TCP的实现是close一个TCP socket时，如果它仍然有数据等待发送，那么该socket会进入TIME_WAIT状态。这种状态将持续到数据被全部发送或者发生超时（这个超时时间通常被称为Linger Time，大多数系统默认为2分钟）。

在未设置SO_REUSEADDR时，内核将一个处于TIME_WAIT状态的socketA仍然看成是一个绑定了指定ip和port的有效socket，因此此时如果另外一个socketB试图绑定相同的ip和port都将失败（不满足规则3），直到socketA被真正释放后，才能够绑定成功。如果socketB设置SO_REUSEADDR（仅仅只需要socketB进行设置），这种情况下socketB的绑定调用将成功返回，但真正生效需要在socketA被真正释放后。（这个地方的理解可能有点问题，待后续验证一下）。总结一下：**内核在处理一个设置了SO_REUSEADDR的socket绑定时，如果其绑定的ip和port和一个处于TIME_WAIT状态的socket冲突时，内核将忽略这种冲突，即改变了系统对处于TIME_WAIT状态的socket的看待方式。**

### getsockname,getpeername

原型

```c
int getsockname(int sockfg,struct sockaddr *localaddr,socklen_t *addrlen);
int getpeername(int sockfg,struct sockaddr *peeraddr,socklen_t *addrlen);
```

这两个函数主要是获得内核自动分配的端口号与IP号。

### dup

都可以用来复制一个现有的文件描述符

````C
#include<unistd.h>
int dup(int fd);
int dup2(int fd,int fd2);
````

若成功，放回新的文件描述符；出错 -1；

