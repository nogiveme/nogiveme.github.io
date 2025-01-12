---
layout: wiki
title: C++网络编程
wiki: Notes
menu_id: notes
order: 704
---

*Unix哲学——万物皆文件*

## 套接口

### 流式套接口`stream socket`

*可靠的，面向连接的套接口*

### 报式套接口`datagram socket`

*无连接的套接口*

### 注意

* *TCP协议允许在传输层对数据进行分段，而UDP则不会，UPD会在发送前在主机上完成分段操作*

## 字节序和数据结构

### 字节序

* **大端序**：低字节存高位数据，高字节存低位数据
* **小端序**：低字节存地位数据，高字节存高位数据
* **网络序**：网络中采用大端序

### 数据结构

* **套接口描述符**：本质上就是文件描述符，符合Linux的设计哲学，类型是整型。

* `struct addrinfo`

  ```C
  struct addrinfo
  {
    int ai_flags;                 /* Input flags.  */
    int ai_family;                /* Protocol family for socket.  */
    int ai_socktype;              /* Socket type.  */
    int ai_protocol;              /* Protocol for socket.  */
    socklen_t ai_addrlen;         /* Length of socket address.  */
    struct sockaddr *ai_addr;     /* Socket address for socket.  */
    char *ai_canonname;           /* Canonical name for service location.  */
    struct addrinfo *ai_next;     /* Pointer to next in list.  */
  };
  ```

  * `ai_flag`：用于标识 `getaddrinfo()` 函数返回结果的标志字段，可以设置为以下常量之一或它们的按位或组合。其中AI前缀表示**Address Information**

    * `AI_PASSIVE`：用于指定用于套接字的地址是通配地址，适用于服务器端程序，表示服务器端将监听所有可用的网络接口。
    * `AI_CANONNAME`：用于指定返回的主机名是否是规范名，如果设置了该标志，则 `getaddrinfo()` 会将主机名转换为其规范名，否则返回的主机名可能是别名。
    * `AI_NUMERICHOST`：用于指定主机名必须是一个 IP 地址字符串，而不是一个主机名，如果指定了该标志，则 `getaddrinfo()` 不会尝试解析主机名。
    * `AI_NUMERICSERV`：用于指定服务名必须是一个端口号字符串，而不是一个服务名，如果指定了该标志，则 `getaddrinfo()` 不会尝试查找服务名。
    * `AI_ADDRCONFIG`：用于指定只返回与本地系统的地址族相匹配的地址，例如 IPv4 地址族的系统将只返回 IPv4 地址。
    * `AI_V4MAPPED`：用于指定如果没有找到与查询参数完全匹配的 IPv6 地址，那么`getaddrinfo()`将尝试返回一个 IPv4 映射的 IPv6 地址。

  * `ai_family`：表示协议类别，`AF_INET`表示`IPV4`，`AF_UNSPEC`表示自动判定

  * `ai_addr`

    ```C
    struct sockaddr {
        unsigned short sa_family; // address family, AF_xxx
        char sa_data[14]; // 14 bytes of protocol address
    };
    ```

    **通常采用它的等价结构体**

    ```C
    struct sockaddr_in {
        short int sin_family; // Address family, AF_INET
        unsigned short int sin_port; // Port number
        struct in_addr sin_addr; // Internet address
        unsigned char sin_zero[8]; // 與 struct sockaddr 相同的大小
    };
    ```

    * `sin_addr`

      ```C
      // Internet address (a structure for historical reasons)
      struct in_addr {
          uint32_t s_addr; // that's a 32-bit int (4 bytes)
      };
      ```

* **函数**

  * `inet_pton(family, "IP address", &(struct sa.in_addr))`：将字符串形式的IP地址转化为字节流存储
  * `inet_ntop(family, &(struct sa.in_addr), buffer'address,len)`：将字节流形式的IP地址转化为字符串存储

## 系统调用

### `getaddrinfo(...)`

*前身是用来做DNS查询的`gethostbyname()`*

* ```C
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <netdb.h>
  
  int getaddrinfo(const char *node, // 例如： "www.example.com" 或 IP
                  const char *service, // 例如： "http" 或 port number
                  const struct addrinfo *hints,
                  struct addrinfo **res);
  ```

  `hints->flags`：设置为`AI_PASSIVE`代表将绑定至本机IP，作为监听套接口

  `res`：返回结果

  **示例代码**

  ```C
  int status;
  struct addrinfo hints;
  struct addrinfo *servinfo; // 将指向结果
  
  memset(&hints, 0, sizeof hints); // 确保 struct 为空
  hints.ai_family = AF_UNSPEC; // 不用管是 IPv4 或 IPv6
  hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
  
  // 准备好连接
  status = getaddrinfo("www.example.net", "3490", &hints, &servinfo);
  
  // servinfo 现在指向有一个或多个 struct addrinfos 的链表
  
  我一直说 serinfo 是一个链表，它有各种的地址资料。让我们写一个能快速 demo 的程序，来呈现这个资料。这个小程序 [18] 会打印出你在命令行中所指定的主机 IP address：
  ** showip.c -- 顯示命令列中所給的主機 IP address
  */
  #include <stdio.h>
  #include <string.h>
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <netdb.h>
  #include <arpa/inet.h>
  #include <netinet/in.h>
  
  int main(int argc, char *argv[])
  {
    struct addrinfo hints, *res, *p;
    int status;
    char ipstr[INET6_ADDRSTRLEN];
  
    if (argc != 2) {
      fprintf(stderr,"usage: showip hostname\n");
      return 1;
    }
  
    memset(&hints, 0, sizeof hints);
    hints.ai_family = AF_UNSPEC; // AF_INET 或 AF_INET6 可以指定版本
    hints.ai_socktype = SOCK_STREAM;
  
    if ((status = getaddrinfo(argv[1], NULL, &hints, &res)) != 0) {
      fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(status));
      return 2;
    }
  
    printf("IP addresses for %s:\n\n", argv[1]);
  
    for(p = res;p != NULL; p = p->ai_next) {
      void *addr;
      char *ipver;
  
      // 取得本身地址的指针
       // 在 IPv4 与 IPv6 中的栏位不同：
      if (p->ai_family == AF_INET) { // IPv4
        struct sockaddr_in *ipv4 = (struct sockaddr_in *)p->ai_addr;
        addr = &(ipv4->sin_addr);
        ipver = "IPv4";
      } else { // IPv6
        struct sockaddr_in6 *ipv6 = (struct sockaddr_in6 *)p->ai_addr;
        addr = &(ipv6->sin6_addr);
        ipver = "IPv6";
      }
  
      // convert the IP to a string and print it:
      inet_ntop(p->ai_family, addr, ipstr, sizeof ipstr);
      printf(" %s: %s\n", ipver, ipstr);
    }
  
    freeaddrinfo(res); // 释放链表
  
    return 0;
  }
  ```

  **结果**

  ```C
  $ showip www.example.net
  IP addresses for www.example.net:
  
    IPv4: 192.0.2.88
  
  $ showip ipv6.example.com
  IP addresses for ipv6.example.com:
  
    IPv4: 192.0.2.101
    IPv6: 2001:db8:8c00:22::171
  ```

### `socket(...)`

*是一个系统调用，返回套接口描述符*

* ```C
  #include <sys/types.h>
  #include <sys/socket.h>
  
  int socket(int domain, int type, int protocol);
  ```

  `domain`：协议家族，`PF_INET`或者`PF_INET6`

  `type`：`socket`的种类

  `protocal`：传输层协议

  ```C
  int s;
  struct addrinfo hints, *res;
  
  ...
  // 运行查询
  getaddrinfo("www.example.com", "http", &hints, &res);
  s = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
  ```

### `bind(...)`

*将socket绑定到指定的IP地址和端口号上*

* ```C
  #include <sys/types.h>
  #include <sys/socket.h>
  
  int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
  ```

  `socfd`：套接字描述符

  `myaddr`：`getaddrinfo(...)`返回的`res`中的`res->ai_addr`

  `addrlen`：`getaddrinfo(...)`返回的`res`中的`res->ai_addrlen`

  **示例代码**

  ```C
  struct addrinfo hints, *res;
  int sockfd;
  
  // 首先，用 getaddrinfo() 载入地址结构：
  
  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC; // use IPv4 or IPv6, whichever
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE; // fill in my IP for me
  
  getaddrinfo(NULL, "3490", &hints, &res);
  
  // 建立一个 socket：
  
  sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
  
  // 将 socket bind 到我们传递给 getaddrinfo() 的 port：
  
  bind(sockfd, res->ai_addr, res->ai_addrlen);
  ```

### `connect(...)`

*连接远端主机*

* ```C
  #include <sys/types.h>
  #include <sys/socket.h>
  
  int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
  ```

  `socfd`：套接字描述符

  `serv_addr`：`getaddrinfo(...)`返回的`res`中的`res->ai_addr`

  `addrlen`：`getaddrinfo(...)`返回的`res`中的`res->ai_addrlen`

  ***注意***：`kernel` 会帮我们选择一个 `local port`，因此一般客户端连接的时候不用`bind(...)`

  **示例代码**

  ```C
  struct addrinfo hints, *res;
  int sockfd;
  
  // 首先，用 getaddrinfo() 载入 address structs：
  
  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_STREAM;
  
  getaddrinfo("www.example.com", "3490", &hints, &res);
  
  // 建立一个 socket：
  
  sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
  
  // connect!
  connect(sockfd, res->ai_addr, res->ai_addrlen);
  ```

### `listen(...)`

*更改此套接口的状态为监听状态*

* ```C
  int listen(int sockfd, int backlog);
  ```

  `backlog`：代表连接此套接口队列的最大长度

  **一般调用顺序**

  ```C
  getaddrinfo();
  socket();
  bind();
  listen();
  /* accept() 从这里开始 */
  ```

### `accept(...)`

*创建一个新的`socket`来处理进行对话*

* ```C
  #include <sys/types.h>
  #include <sys/socket.h>
  
  int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  ```

  `addr`：存放连接？？？

  **示例代码**

  ```C
  #include <string.h>
  #include <sys/types.h>
  #include <sys/socket.h>
  #include <netinet/in.h>
  
  #define MYPORT "3490" // 使用者将连接的 port
  #define BACKLOG 10 // 在队列中可以有多少个连接在等待
  
  int main(void)
  {
  　　struct sockaddr_storage their_addr;
  　　socklen_t addr_size;
  　　struct addrinfo hints, *res;
  　　int sockfd, new_fd;
  
  　　// !! 不要忘了帮这些调用做错误检查 !!
  
  　　// 首先，使用 getaddrinfo() 载入 address struct：
  
  　　memset(&hints, 0, sizeof hints);
  　　hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，都可以
  　　hints.ai_socktype = SOCK_STREAM;
  　　hints.ai_flags = AI_PASSIVE; // 帮我填上我的 IP 
  
  　　getaddrinfo(NULL, MYPORT, &hints, &res);
  
  　　// 产生一个 socket，bind socket，并 listen socket：
  
  　　sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
  　　bind(sockfd, res->ai_addr, res->ai_addrlen);
  　　listen(sockfd, BACKLOG);
  
  　　// 现在接受一个进入的连接：
  
  　　addr_size = sizeof their_addr;
  　　new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &addr_size);
  
  　　// 准备好与 new_fd 这个 socket descriptor 进行沟通！
  　　...
  ```

### `send(...) & recv(...)`

*和远端主机通信*

* ```C
  int send(int sockfd, const void *msg, int len, int flags);
  ```

  `msg`：发送的信息

  `len`：发送的长度

  `flags`：一般设置为0即可

  **注意**：`send(...)`只会尽量将资料送出，并认为你之後会再次送出剩下没送出的部分，因此它不能保证送完数据

  **示例代码**

  ```C
  char *msg = "Beej was here!";
  int len, bytes_sent;
  
  len = strlen(msg);
  bytes_sent = send(sockfd, msg, len, 0);
  ```

* ```C
  int recv(int sockfd, void *buf, int len, int flags);
  ```

  `buf`：接收消息的缓冲区

  `len`：发送的长度

  `flags`：一般设置为0即可

  **注意**： `recv(...) `会返回 0，这表示远端主机已经关闭了连接

## 高等技术

### 阻塞

*阻塞就是线程在某一条语句中sleep*

* ```c
  #include <unistd.h>
  #include <fcntl.h>
  
  sockfd = socket(PF_INET, SOCK_STREAM, 0);
  fcntl(sockfd, F_SETFL, O_NONBLOCK);
  ```

  `fcntl`：使阻塞函数成为非阻塞函数

### `select(...)`

*同时监听多个套接口*

* ```C
  #include <sys/time.h>
  #include <sys/types.h>
  #include <unistd.h>
  
  int select(int numfds, fd_set *readfds, fd_set *writefds,
             fd_set *exceptfds, struct timeval *timeout);
  ```

  
