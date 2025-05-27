---
title: nginx历史版本结构分析(一)
subtitle: socket编程，事件处理机制
date: 2024-04-27 11:42:11
categories:
- [组件,nginx,nginx历史版本]
tags:
- 组件
- nginx
---

## 概述

nginx是一个高性能web服务器，从2002到现在已经有20多年了，经过不断的迭代完善，功能越来越多也越来越强大，如果直接看最新的nginx版本会陷入较多的细节中。这里通过查找nginx最原始的版本，查看nginx的迭代，试图找到nginx的演化路径以及摸清楚nginx的主体功能实现。

这里分析的版本是nginx作者在2002-08-06发布的第一个版本，该版本还不是一个可运行版，但我们仍可以通过代码了解到nginx的一些设计。

## 基础

### socket编程

nginx作为一个web服务器，最主要的功能就是作为网络服务server端，为client访问提供服务，所以nginx绕不开网络编程。类unix系统下tcp编程大致遵循以下的调用链路。

```plaintext
       
                                                               TCP服务器
                                                            +--------------+
                                                            |   socket()   |
                                                            +--------------+
                                                                    |
                                                                   \|/         
                                                            +--------------+
                                                            |    bind()    |绑定众所周知的端口
                                                            +--------------+
                                                                    |
                                                                   \|/           
                                                            +--------------+
                                                            |   listen()   |
                                                            +--------------+
                                                                    |
                                                                   \|/
                                                            +--------------+
                                                            |   accept()   |
            TCP客户                                         +--------------+
       +--------------+                                             |
       |   socket()   |                                            \|/
       +--------------+                                    一直阻塞直到客户连接到达
              |                                                     |
             \|/                                                    |
       +--------------+                 连接建立                    |                  
       |   connect()  |-------------------------------------------->|                  
       +--------------+                     TCP三次握手             |
             |                                                      |
            \|/                                                    \|/              
       +--------------+            数据请求                 +--------------+                            
    +->|   write()    |------------------------------------>|   read()     |                            
    |  +--------------+                                     +--------------+                            
    |        |                                                     |(处理请求)
    |       \|/                                                   \|/               
    |  +--------------+           数据应答                  +--------------+                            
    +--|   read()     |<------------------------------------|   write()    |                            
       +--------------+                                     +--------------+                            
             |                                                     |
            \|/                                                   \|/
       +--------------+             文件结束通知            +--------------+                            
       |    close()   |------------------------------------>|    read()    |                            
       +--------------+                                     +--------------+                            
                                                                    |
                                                                   \|/
                                                             +--------------+                            
                                                             |   close()    |                            
                                                             +--------------+                            
```

### IO多路复用

在传统网络编程体系下，服务端一次只能处理一个请求，效率会比较低，所以一般采用IO多路复用的技术同时监听多个socket，当部分socket处于就绪状态时，监听返回并处理。

在nginx-0.0.1-2002-08-06版本中，nginx提供了两种IO多路复用技术：select和kqueue。这里我们选择基于select的技术来分析。select函数的原型如下：
``int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset, const struct timeval *timeout)``
函数的第二三四参数都是都是描述符集，分别表示select监听的读、写、异常描述符集，对这些描述符集的操作是通过宏定义实现：
``void FD_ZERO(fd_set *fdset) /*清除fd_set中的所有描述符*/``
``void FD_SET(int fd, fd_set *fdset) /*将fd设置到fdset描述符集中*/``
``void FD_CLR(int fd, fd_set *fdset) /*将fd从fdset中清除*/``
``void FD_ISSET(int fd, fd_set *fdset) /*判断fd在fdset中是否设置*/``

## nginx基础

### 事件

nginx是事件驱动的，事件驱动模型通过事件循环监听各类事件，在事件发生时调用相应的处理程序，而不是阻塞等待。事件的核心要素包括

* __事件源__。事件源是指能产生某种操作的一方，例如socket。如客户端有连接请求时，监听socket会产生连接事件；连接socket会产生读写事件，以及连接超时、关闭等都是事件。
* __事件循环__。通过循环遍历事件，持续监听并处理事件。
* __事件注册__。事件注册是告诉操作系统哪些事件是程序感兴趣的。
* __事件通知机制__。事件通知机制是当事件发生时，程序能以某种方式获取到。
* __事件处理器__。事件发生后要执行的动作。

事件驱动模型是一种使用比较广泛的模型，除了在高性能网络编程中有使用，桌面编程上也有应用。

nginx主要处理以下三类事件：

1. __连接请求事件__。nginx作为服务端监听在指定端口上，当有客户端连接请求时，nginx会从监听socket中返回一个表示连接的socket。
2. __读事件__。当客户端发送数据时，连接socket的读事件就绪，此时我们可以从socket中读取数据。
3. __写事件__。当需要发送数据时，可以通过连接socket将数据写入，传递给客户端。

#### nginx事件结构

在nginx中，使用结构体ngx_event_s表示事件。结构体定义如下：

> ```c
>  //file: src/event/ngx_event.h
>  typedef struct ngx_event_s       ngx_event_t;
>  struct ngx_event_s {
>      void            *data;
> ```
>
>> data指向与该事件相关的上下文数据，通常是 ngx_connection_t（连接）或其他模块自定义结构。
>
> ```c
>     int            (*event_handler)(ngx_event_t *ev);
>     int            (*close_handler)(ngx_event_t *ev);
>```
>
>> event_handler是事件处理函数指针，即事件处理器。close_handler是事件关闭时调用的处理函数。
>
>```c
>     void            *context;
>     char            *action;
>```
>
>> action用于记录当前正在执行的操作名称，用于日志打印或调试.
>
>```c
>     ngx_event_t     *prev;     /* queue in select(), poll() */
>     ngx_event_t     *next;
>```
>
>> prev和next是指向自身的指针，说明event可以当作节点构成链表。
>
>```c
>     int            (*timer_handler)(ngx_event_t *ev);
>     ngx_event_t     *timer_prev;
>     ngx_event_t     *timer_next;
> 
>     u_int            timer_delta;
>     u_int            timer;
> 
>     ngx_log_t       *log;
> 
>     int              available; /* kqueue only:                              */
>                                 /*   accept: number of sockets that wait     */
>                                 /*           to be accepted                  */
>                                 /*   read:   bytes to read                   */
>                                 /*   write:  available space in buffer       */
>                                 /* otherwise:                                */
>                                 /*   accept: 1 if accept many, 0 otherwise   */
> 
>     /* flags - int are probably faster on write then bits ??? */
>     unsigned         listening:1;
>     unsigned         write:1;
> 
>     unsigned         ready:1;
>     unsigned         timedout:1;
>     unsigned         process:1;
>     unsigned         read_discarded:1;
> 
>     unsigned         unexpected_eof:1;
> 
> #if (HAVE_DEFERRED_ACCEPT)
>     unsigned         accept_filter:1;
> #endif
> #if (HAVE_KQUEUE)
>     unsigned         eof:1;
>     int              errno;
> #endif
> };
> ```
>
>> 最后是一些标识字段，用于标识事件类型或者处理状态。

每当有事件发生时，都会生成对应的事件结构体对象，并加入到事件队队列中去。

#### 事件队列

事件队列以链表的形式串联起所有的事件对象，由于IO多路复用技术不同，对事件队列的处理方式也不一样，所以事件队列是实现相关的。

在select模式中，文件`event/modules/ngx_select_module.c`有一个类型为`ngx_event_t`的静态变量`event_queue`，即表示事件队列。

事件队列是一个双向链表，初始化见`ngx_select_init`方法:

>```c
>    event_queue.prev = &event_queue;
>    event_queue.next = &event_queue;
>```
>
>> 事件队列初始化时通过prev和next指针指向自身形成了环。

```plaintext
+------------+      +--------------+           
|           \|/    \|/             |
|   +-------------------------+    |
+---| prev |          |  next |----+
    +-------------------------+
        event_queue
       event_queue初始化      
```

事件队列中事件对象的增加和删除分别见方法`ngx_select_add_event`和`ngx_select_del_event`,

>```c
>    ngx_event_t *ev
>
>    ev->prev = &event_queue;
>    ev->next = event_queue.next;
>    event_queue.next->prev = ev;
>    event_queue.next = ev;
>```
>
>> 添加事件采用头插法插入事件对象。

```plaintext
                          +--------------+-------------------------------------+ 
                         \|/                                                   |
          +-------------------------+           +-------------------------+    |
          | prev |          |  next |<----------| prev |          |  next |----+
          +-------------------------+           +-------------------------+
              |                                                /|\
              +-------------------------------------------------+
              event_queue                                   ev
                              event_queue初始化     
```

>```c
>    ngx_event_t *ev;
>    if (ev->prev)
>        ev->prev->next = ev->next;
>
>    if (ev->next) {
>        ev->next->prev = ev->prev;
>        ev->prev = NULL;
>    }
>```
>
>> 删除事件将事件对象从链表中移除。

### socket连接

socket是内核为程序提供的网络通信通道的句柄，用于发送和接收数据。socket既可以是监听socket，也可以是连接socket。socket本质上是一个文件描述符(一个非负整数)，代表了一个连接。

nginx对能处理的连接数设置了上限，在`src/event/ngx_event.c`的`ngx_worker`方法中，`max_connections`表示能同时处理的连接数，值为512.同时初始化了三个全局变量数组:

```c
    ngx_connection_t *ngx_connections;
    ngx_event_t *ngx_read_events, *ngx_write_events;

    ngx_read_events = ngx_alloc(sizeof(ngx_event_t) * max_connections, log);
    ngx_write_events = ngx_alloc(sizeof(ngx_event_t) * max_connections, log);
    ngx_connections = ngx_alloc(sizeof(ngx_connection_t) * max_connections, log);
```

每一个数组长度都设置成了max_connections大小.每当有socket建立时，就以socket为索引，保存对应事件到数组对应位置。

`ngx_connection_t`是连接的抽象，结构体定义如下：

> ```c
> typedef struct ngx_connection_s  ngx_connection_t;
> 
> struct ngx_connection_s {
>     ngx_socket_t      fd;
> ```
>
>> fd是socket描述符，用于标识连接，既可以是连接socket，也可以是监听socket。
>
> ```c
>     void            *data;
> ```
>
>> data是自定义数据结构，用于保存与连接相关的上下文数据等，不同的模块中data保存的数据不同。
>
> ```c
> #ifdef NGX_EVENT
>     ngx_event_t      *read;
>     ngx_event_t      *write;
> #endif
> ```
>
>> read和write指向对应socket的读写事件对象，分别表示socket的读写事件。
>
> ```c
>     ngx_log_t        *log;
>     ngx_server_t     *server;
>     ngx_server_t     *servers;
>     ngx_pool_t       *pool;
> };
> ```

`ngx_read_events`和`ngx_write_events`则分别表示socket的读写事件对象数组。socket的值索引其对应数组中的事件对象。

### IO多路复用选择

nginx支持多种IO多路复用技术，包括select、kqueue等，选择哪一种是由配置决定的。在`src/event/ngx_event.h`中，`USE_KQUEUE`宏决定了以下四个宏的取值:

```c
#if (USE_KQUEUE)
#define ngx_init_events      ngx_kqueue_init
#define ngx_process_events   ngx_kqueue_process_events
#define ngx_add_event        ngx_kqueue_add_event
#define ngx_del_event        ngx_kqueue_del_event
#else
#define ngx_init_events     (ngx_event_init[ngx_event_type])
#define ngx_process_events   ngx_event_actions.process
#define ngx_add_event        ngx_event_actions.add
#define ngx_del_event        ngx_event_actions.del
#endif
```

非kqueue条件下，一般使用else中的宏定义，这其中涉及到三个全局变量`ngx_event_init`、`ngx_event_type`和`ngx_event_actions`,前两者定义在`src/event/ngx_event.c`中，

>```c
>   #if 1
>   ngx_event_type_e ngx_event_type = NGX_SELECT_EVENT;
>   #else
>   ngx_event_type_e ngx_event_type = NGX_KQUEUE_EVENT;
>   #endif
>
>   /* ngx_event_type_e order */
>   static void (*ngx_event_init[])(int max_connections, ngx_log_t *log) = {
>    ngx_select_init,
>   #if (HAVE_POLL)
>       ngx_poll_init,
>   #endif
>   #if (HAVE_KQUEUE)
>       ngx_kqueue_init
>   #endif
> };
>```
>
> ```c
> //src\event\ngx_event.h
> typedef enum {
>    NGX_SELECT_EVENT = 0,
>   #if (HAVE_POLL)
>       NGX_POLL_EVENT,
>   #endif
>   #if (HAVE_KQUEUE)
>       NGX_KQUEUE_EVENT,
>   #endif
> } ngx_event_type_e ;
> ```
>
>> ngx_event_init是一个函数指针数组，在上面的宏定义中，`ngx_init_events`宏的取值根据`ngx_event_type`的值来确定。`ngx_event_type`的取值是枚举类型`NGX_SELECT_EVENT`,由枚举定义中得知该值为0，所以`ngx_init_events`宏的值为`ngx_select_init`函数指针。

`ngx_event_actions`定义了一组底层事件模型的操作函数指针集合，本质上是对具体事件处理机制（如 epoll、kqueue、select、poll 等）的一层抽象。

```c
typedef struct {
    int  (*add)(ngx_event_t *ev, int event, u_int flags);
    int  (*del)(ngx_event_t *ev, int event);
    int  (*process)(ngx_log_t *log);
/*
    int  (*read)(ngx_event_t *ev, char *buf, size_t size);
    int  (*write)(ngx_event_t *ev, char *buf, size_t size);
*/
} ngx_event_actions_t;
```

`ngx_event_actions`的初始化发生在`ngx_init_events`宏调用中，当采用select模式时，调用的是`ngx_select_init`方法，该方法初始化`ngx_event_actions`如下：

```c
    ngx_event_actions.add = ngx_select_add_event;
    ngx_event_actions.del = ngx_select_del_event;
    ngx_event_actions.process = ngx_select_process_events;
```

所以在select模式下余下的三个宏执行的就是select中对应的方法。

## 分析

### 执行流程

当nginx服务启动时，会监听在指定端口上，然后生成一个连接事件对象放入事件队列，并将事件对应的socket描述符fd放入`select`对应的fd_set中，然后循环监听fd_set,阻塞等待事件就绪。当有fd就绪时，执行对应的事件对象的事件处理函数。通过循环监听fd_set，nginx得以不断处理各种事件。

### main函数入口

main函数是nginx启动时的入口，主要功能是监听在指定端口上，然后转入worker中执行。

#### 监听端口

监听端口第一步需要创建socket，然后调用`bind()`将socket绑定在指定地址上，调用`listen()`监听端口。以上代码在`src/core/ngx_listen.c`文件的`ngx_listen()`方法中。

> ```c
> ngx_socket_t ngx_listen(struct sockaddr *addr, int backlog,
>                         ngx_log_t *log, char *addr_text)
> {
>     ngx_socket_t   s;
>     int            reuseaddr = 1;
> #if (WIN32)
>     unsigned long  nb = 1;
> #endif
> 
>     if ((s = socket(AF_INET, SOCK_STREAM, 0)) == -1)
>         ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno, "socket failed");
>```
>
>> 调用`socket()`函数返回一个socket，socket的第一和第二参数分别表示协议族和socket类型，这里的AF_INET表示协议族为ipv4，SOCK_STREAM表示字节流套接字。
>
>```c
>     if (setsockopt(s, SOL_SOCKET, SO_REUSEADDR,
>                    (const void *) &reuseaddr, sizeof(int)) == -1)
>         ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
>                      "ngx_listen: setsockopt (SO_REUSEADDR) failed");
> 
> #if (WIN32)
>     if (ioctlsocket(s, FIONBIO, &nb) == -1)
>         ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
>                      "ngx_listen: ioctlsocket (FIONBIO) failed");
> #else
>     if (fcntl(s, F_SETFL, O_NONBLOCK) == -1)
>         ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
>                      "ngx_listen: fcntl (O_NONBLOCK) failed");
> #endif
>```
>
>> `fcntl()`通过*O_NONBLOCK*参数设置socket为非阻塞模式。
>
>```c
>     if (bind(s, (struct sockaddr *) addr, sizeof(struct sockaddr_in)) == -1)
>         ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
>                      "ngx_listen: bind to %s failed", addr_text);
> 
>     if (listen(s, backlog) == -1)
>         ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
>                      "ngx_listen: listen to %s failed", addr_text);
> 
>     return s;
> }
> ```
>
>> `bind()`的*addr*参数指示了绑定的端口，通过和`listen()`一起调用，使得socket监听在指定端口上。

#### 创建ngx_listen_t结构体对象

当创建socket成功后，nginx会将socket和类型为`ngx_server_t`的结构体对象封装进`ngx_listen_t`结构体对象，并传入给`ngx_worker()`方法。两者结构体定义如下：

```c
// src/core/ngx_server.h
typedef struct {
    int          log_level;
    ngx_pool_t  *pool;
    int        (*handler)(void *data);
    int          buff_size;
} ngx_server_t;


typedef struct {
    ngx_socket_t  fd;

    ngx_log_t    *log;
    ngx_server_t *server;

    unsigned      shared:1;
#if (HAVE_DEFERRED_ACCEPT)
    unsigned      accept_filter:1;
#endif
} ngx_listen_t;
```

>> `ngx_server_t`包含一个*handler*函数指针。`ngx_listen_t`则包含一个socket描述符fd和一个指向ngx_server_t结构体的指针。

结构体之间的关系如下，其中忽略了`ngx_listen_t`中的标志位字段。

```c {.line-numbers}

    +------------+
    |  int  fd   | (socket描述符)
    +------------+                           +------------+
    |    log     |-------------------------->|            |
    +------------+                           +------------+
    |   server   |-------------+                 ngx_log_t
    +------------+             |
     ngx_listen_t              |                                                                                 
                               |                        +---------------+
                               +----------------------> | int log_level |
                                                        +---------------+                 +--------------+
                                                        |    pool       |---------------->|              |
                                                        +---------------+                 +--------------+ 
                                                        | func* handler |                     ngx_pool_t
                                                        +---------------+
                                                        | int buff_size |
                                                        +---------------+
                                                          ngx_server_t
```

创建完毕后，将`ngx_listen_t`对象*ls*传递到`ngx_worker()`方法中做下一步处理。

#### 整个执行流程

以下是`main()`中的整个执行过程。

> ```c {.line-numbers}
> struct sockaddr_in ngx_addr = {0, AF_INET, 0, 0, 0};
> //file: src/core/nginx.c
> int main(int argc, char *const *argv)
> {
>     char addr_text[22];
>     ngx_socket_t fd;
>     ngx_listen_t ls;
> 
>     ngx_log.log_level = NGX_LOG_DEBUG;
>     ngx_pool.log = &ngx_log;
>     ngx_addr.sin_port = htons(8000);
>     ngx_addr.sin_family = AF_INET;
> 
>     fd = ngx_listen((struct sockaddr *) &ngx_addr, -1, &ngx_log, addr_text);
> ```
>
>> 先创建了`sockaddr_in`结构体的全局对象*ngx_addr*，并调用`ngx_listen()`方法创建socket描述符*fd*。
>
> ```c
>     ngx_server.buff_size = 1024;
>     ngx_server.handler = ngx_http_init_connection;
> 
>     /* daemon */
>     ls.fd = fd;
>     ls.server = &ngx_server;
>     ls.log = &ngx_log;
> 
>     /* fork */
>     ngx_worker(&ls, 1, &ngx_pool, &ngx_log);
> }
> ```
>
>> 为*ls*对象属性赋值，并传递给`ngx_worker()`方法.

### ngx_worker函数

`ngx_worker()`函数的流程如下：

1. 事件处理初始化，主要是初始化事件队列以及全局事件处理对象*ngx_event_actions*；
2. 初始化了三个全局变量数组:`ngx_connections,ngx_read_events,ngx_write_events`，数组长度为*max_connections*，数组每一个下标与一个socket对应；
3. 初始化`ngx_connections`和`ngx_read_events`数组中监听socket的值所对应的下标的对象；
4. 将上一步初始化的读事件对象加入事件队列，并将对应的socket加入select的读描述符集中；
5. 进入无限循环，循环中阻塞在select()方法上，并在有socket就绪时，执行socket对应的事件的事件处理函数。

```plaintext

    ngx_worker()
        |
        |
        o------------->ngx_init_events()
        |
        |    
        o------------->ngx_add_event()
        |
        |
        o------------->ngx_process_events()
                                |
                                |
                                o-------------->event_handle()[ngx_event_accept]
                                                     |
                                                     |
                                                     o----------->server->handler()[ngx_http_init_connection]
```

#### 初始化

ngx_worker首先调用宏`ngx_init_events`初始化相关对象，配置不同宏调用的事件处理机制(select,poll,kqueue)不一样，在`select()`中初始化对象包括事件队列、时间队列以及事件处理机制中具体执行的方法。

以下是`ngx_select_module.c`中的初始化方法。

> ```c
> void ngx_select_init(int max_connections, ngx_log_t *log)
> {
> #if (WIN32)
>     if (max_connections > FD_SETSIZE)
>         ngx_log_error(NGX_LOG_EMERG, log, 0,
>                       "ngx_select_init: maximum number of descriptors "
>                       "supported by select() is %d",
>                       FD_SETSIZE);
> #else
>     if (max_connections >= FD_SETSIZE)
>         ngx_log_error(NGX_LOG_EMERG, log, 0,
>                       "ngx_select_init: maximum descriptor number"
>                       "supported by select() is %d",
>                       FD_SETSIZE - 1);
> #endif
> 
>     FD_ZERO(&master_read_fds);
>     FD_ZERO(&master_write_fds);
>```
>
>> *master_read_fds*和*master_write_fds*是两个全局变量，分别表示select监听的读描述符集和写描述符集。
>
> ```c
>     event_queue.prev = &event_queue;
>     event_queue.next = &event_queue;
> 
>     timer_queue.timer_prev = &timer_queue;
>     timer_queue.timer_next = &timer_queue;
> 
>     ngx_event_actions.add = ngx_select_add_event;
>     ngx_event_actions.del = ngx_select_del_event;
>     ngx_event_actions.process = ngx_select_process_events;
> ```
>
>> 事件队列初始，定时器队列初始化，事件操作对象的初始化。
>
> ```c
> #if (WIN32)
>     max_read = max_write = 0;
> #else
>     max_fd = -1;
> #endif
> }
> ```

#### socket数组初始化

事件队列及相关处理函数初始化后，nginx初始化了三个全局数组，每一个数组对应位置的元素都与一个socket关联，数组大小限制了nginx能同时处理的连接数。

|                    |      0              |        1             |        ...           | max_connections-1      |
| :---               |    :----:           |          ---:        |          ---:        |           ---:         |
| ngx_connection_t*  | ngx_connections[0]  | ngx_connection_t[1]  | ngx_connection_t[k]  | ngx_connection_t[n-1]  |
| ngx_event_t*       | ngx_read_events[0]  | ngx_read_events[1]   | ngx_read_events[k]   | ngx_read_events[n-1]   |
| ngx_event_t*       | ngx_write_events[0] | ngx_write_events[1]  | ngx_write_events[k]  | ngx_write_events[n-1]  |

#### 将监听socket封装成事件对象

`ngx_worker()`接收一个指向`ngx_listen_t`结构体的指针，所以实际接收参数是一个数组，数组中每一个元素都包含了一个监听socket。接下来根据socket的值初始化连接数组、读事件数组中对应下标的值，并在读事件中设置了事件处理函数。

```c
void ngx_worker(ngx_listen_t *sock, int n, ngx_pool_t *pool, ngx_log_t *log)
{
    int i, fd;
    //...

    /* for each listening socket */
    for (i = 0; i < n; i++)
    {
        fd = sock[i].fd;

        ngx_memzero(&ngx_read_events[fd], sizeof(ngx_event_t));
        ngx_memzero(&ngx_write_events[fd], sizeof(ngx_event_t));
        ngx_memzero(&ngx_connections[fd], sizeof(ngx_connection_t));

        ngx_connections[fd].fd = fd;
        ngx_connections[fd].server = sock[i].server;
        ngx_connections[fd].read = (void *)&ngx_read_events[fd].data;
        ngx_read_events[fd].data = &ngx_connections[fd];
        ngx_read_events[fd].log = ngx_connections[fd].log = sock[i].log;
        ngx_read_events[fd].data = &ngx_connections[fd];
        ngx_read_events[fd].event_handler = &ngx_event_accept;
        ngx_read_events[fd].listening = 1;

        ngx_read_events[fd].available = 0;
        //...
    }
}
```

>> 首先是初始化连接对象，然后读事件对象，两者之间通过指针相互引用。并在读事件中设置了事件处理函数`ngx_event_accept()`,同时设置监听标志位*listening*为1，可使用标识*available*为0.

#### 将socket对应的读事件加入事件队列

接下来将调用宏定义`ngx_add_event`将读事件加入事件队列中。在初始化时，如果配置的IO复用模型是select，则会调用`src/event/modules/ngx_select_module.c`中的`ngx_select_add_event()`方法.该方法会先根据事件类型获取对应的全局描述符集，并将事件对应的socket描述符加入到描述符集中，同时将事件插入到事件队列中。

> ```c {.line-numbers}
> int ngx_select_add_event(ngx_event_t *ev, int event, u_int flags)
> {
>     fd_set *fds;
>     ngx_connection_t *cn = (ngx_connection_t *) ev->data;
> ```
>
> > 在`ngx_worker`中，`read_event`的*data*指针指向的是对应的`ngx_connection_t`对象。这里通过connection对象取出对应的socket描述符。
>
> ```c
>     if (event == NGX_TIMER_EVENT) {
>         ngx_add_timer(ev, flags);
>         return 0;
>     }
> 
>     ngx_assert((flags != NGX_ONESHOT_EVENT), return -1, ev->log,
>                "ngx_select_add_event: NGX_ONESHOT_EVENT is not supported");
> 
>     fds = ngx_select_get_fd_set(cn->fd, event, ev->log);
> ```
>
>> `ngx_select_get_fd_set`根据event的类型取出对应的描述符集。
>
> ```c
>     if (fds == NULL)
>         return -1;
> 
>     ev->prev = &event_queue;
>     ev->next = event_queue.next;
>     event_queue.next->prev = ev;
>     event_queue.next = ev;
> 
>     FD_SET(cn->fd, fds);
> ```
>
>> 这里做了两件事：将事件加入事件队列；将socket描述符加入描述符集，为`select()`监听socket做准备。

#### 循环监听并处理事件

当描述符集和事件队列都准备好后，程序进入死循环，执行监听->socket就绪->处理socket对应事件的处理函数->监听的循环流程.此过程由执行宏定义`ngx_process_events`实现，该宏定义也是由IO复用实现相关的，选择select模式时执行的是`src/event/modules/ngx_select_module.c`中的`ngx_select_process_events()`方法。

> ```c {.line-numbers}
> int ngx_select_process_events(ngx_log_t *log)
> {
>     int                ready, found;
>     u_int              timer, delta;
>     ngx_event_t       *ev, *nx;
>     ngx_connection_t  *cn;
>     struct timeval     tv, *tp;
> 
>     work_read_fds = master_read_fds;
>     work_write_fds = master_write_fds;
> 
>     if ((ready = select(max_fd + 1, &work_read_fds, &work_write_fds, NULL, tp))
>                == -1) {
>         ngx_log_error(NGX_LOG_ALERT, log, ngx_socket_errno,
>                      "ngx_select_process_events: select failed");
>         return -1;
>     }
> ```
>
> > `select()`会阻塞，当有描述符集中的socket就绪时，函数返回，ready返回就绪的socket数量。
>
> ```c
>     for (ev = event_queue.next; ev != &event_queue; ev = ev->next) {
>         cn = (ngx_connection_t *) ev->data;
>         found = 0;
> 
>         if (ev->write) {
>             if (FD_ISSET(cn->fd, &work_write_fds)) {
>                 ngx_log_debug(log, "ngx_select_process_events: write %d" _
>                               cn->fd);
>                 found = 1;
>             }
> 
>         } else {
>             if (FD_ISSET(cn->fd, &work_read_fds)) {
>                 ngx_log_debug(log, "ngx_select_process_events: read %d" _
>                               cn->fd);
>                 found = 1;
>             }
>         }
> 
>         if (found) {
>             ev->ready = 1;
>             if (ev->event_handler(ev) == -1)
>                 ev->close_handler(ev);
> 
>             ready--;
>         }
>     }
> }
> ```
>
>> 通过遍历事件队列，判断事件对应的描述符是否就绪，如果就绪，则执行事件的事件处理函数。在`ngx_worker`中，监听socket对应的事件的事件处理函数是`ngx_event_accept`，该函数实现在`src\event\ngx_event_accept.c`中。

#### ngx_worker()整个执行流程

`ngx_worker()`的代码如下：

> ```c
> void ngx_worker(ngx_listen_t *sock, int n, ngx_pool_t *pool, ngx_log_t *log)
> {
>     int i, fd;
> 
>     /* per group */
>     int max_connections = 512;
> 
>     ngx_init_events(max_connections, log);
> 
>     ngx_read_events = ngx_alloc(sizeof(ngx_event_t) * max_connections, log);
>     ngx_write_events = ngx_alloc(sizeof(ngx_event_t) * max_connections, log);
>     ngx_connections = ngx_alloc(sizeof(ngx_connection_t) * max_connections, log);
> 
>     /* for each listening socket */
>     for (i = 0; i < n; i++)
>     {
>         fd = sock[i].fd;
> 
>         ngx_memzero(&ngx_read_events[fd], sizeof(ngx_event_t));
>         ngx_memzero(&ngx_write_events[fd], sizeof(ngx_event_t));
>         ngx_memzero(&ngx_connections[fd], sizeof(ngx_connection_t));
> 
>         ngx_connections[fd].fd = fd;
>         ngx_connections[fd].server = sock[i].server;
>         ngx_connections[fd].read = (void *)&ngx_read_events[fd].data;
> 
>         ngx_read_events[fd].data = &ngx_connections[fd];
>         ngx_read_events[fd].log = ngx_connections[fd].log = sock[i].log;
>         ngx_read_events[fd].data = &ngx_connections[fd];
>         ngx_read_events[fd].event_handler = &ngx_event_accept;
>         ngx_read_events[fd].listening = 1;
> 
>         ngx_read_events[fd].available = 0;
> 
> #if (HAVE_DEFERRED_ACCEPT)
>         ngx_read_events[fd].accept_filter = sock->accept_filter;
> #endif
>         ngx_add_event(&ngx_read_events[fd], NGX_READ_EVENT, 0);
>     }
> ```
>
>> 上面的过程初始化了socket对应的事件对象，并将读事件加入了事件队列中。
>
> ```c
>     
> 
>     while (1)
>     {
>         ngx_log_debug(log, "ngx_worker cycle");
> 
>         ngx_process_events(log);
>     }
> }
> ```
>
>> 循环，`select()`监听在描述符集上，并调用相应的事件处理函数。

### 连接处理函数

`ngx_worker()`执行之后，服务端阻塞在`select()`上，等待客户端的请求。当有客户端的请求过来时，`select()`返回，对应的socket描述符就绪，nginx遍历事件队列，找到就绪的事件并执行其事件处理函数。监听socket对应的事件处理函数为`ngx_event_accept()`。

`ngx_event_accept()`执行以下功能：

1. 执行`accept()`从监听socket中获取一个连接socket；
2. 从连接socket填充连接数组对应下标的连接、读写事件对象；
3. 执行连接对象的server字段的`handler`处理函数.

> ```c {.line-numbers}
> int ngx_event_accept(ngx_event_t *ev)
> {
>     ngx_err_t           err;
>     ngx_socket_t        s;
>     struct sockaddr_in  addr;
>     int addrlen = sizeof(struct sockaddr_in);
>     ngx_connection_t *cn = (ngx_connection_t *) ev->data;
>                 
>     ev->ready = 0;
>   
>     do {
>         if ((s = accept(cn->fd, (struct sockaddr *) &addr, &addrlen)) == -1) {
>             return 0;
>         }
> ```
>
>> `accept()`从监听socket中返回一个连接socket，程序既可以从该socket读取客户端发送的数据，也可以通过该socket发送数据给客户端。
>
> ```c
>         ngx_memzero(&ngx_read_events[s], sizeof(ngx_event_t));
>         ngx_memzero(&ngx_write_events[s], sizeof(ngx_event_t));
>         ngx_memzero(&ngx_connections[s], sizeof(ngx_connection_t));
> 
>         ngx_read_events[s].data = ngx_write_events[s].data
>                                                          = &ngx_connections[s];
>         ngx_connections[s].read = &ngx_read_events[s];
>         ngx_connections[s].write = &ngx_write_events[s];
> 
>         ngx_connections[s].fd = s;
>         ngx_read_events[s].unexpected_eof = 1;
>         ngx_write_events[s].ready = 1;
> 
>         ngx_write_events[s].timer = ngx_read_events[s].timer = 10000;
> 
>         ngx_write_events[s].timer_handler =
>             ngx_read_events[s].timer_handler = ngx_event_close;
> 
>         ngx_write_events[s].close_handler =
>             ngx_read_events[s].close_handler = ngx_event_close;
> 
>         ngx_connections[s].server = cn->server;
>         ngx_connections[s].servers = cn->servers;
>         ngx_connections[s].log =
>             ngx_read_events[s].log = ngx_write_events[s].log = ev->log;
>     
> 
>         cn->server->handler(&ngx_connections[s]);
> ```
>
>> `cn->server->handler`在main.c中设置，见`ngx_server.handler = ngx_http_init_connection`,
>> 所以这里在设置好连接socket对应的连接数组对象后，执行的是`ngx_http_init_connection`。
>> 这里`ngx_read_events`的`event_handler`暂未设置值，所以这里还未完成。
>
> ```c
>     } while (ev->available);
>   
>     return 0;
> }
> ```

### ngx_http_init_connection连接执行过程

`accept()`在执行了accept操作后，执行了server->handler的处理函数，即执行了`ngx_http_init_connection`方法，该方法将`ngx_connection_t`关联的事件加入事件队列。

> ```c
> int ngx_http_init_connection(ngx_connection_t *c)
> {
>     ngx_event_t *ev;
> 
>     ev = c->read;
> ```
>
>> 在`ngx_event_accept()`方法中，*c*是连接socket对应的connection对象，`c->read`指向的是socket对应的读事件对象。所以这里实际上是获取的连接socket对应的读事件对象。
>
> ```c
>     /*
>         ev->event_handler = ngx_http_init_request;
>     */
>     ev->event_handler = NULL;
>     ev->log->action = "reading client request line";
> 
>     ngx_log_debug(ev->log, "ngx_http_init_connection: entered");
> 
>     /* XXX: ev->timer ? */
>     if (ngx_add_event(ev, NGX_TIMER_EVENT, ev->timer) == -1)
>         return -1;
> 
> #if (HAVE_DEFERRED_ACCEPT)
>     if (ev->ready)
>         return ngx_http_init_request(ev);
>     else
> #endif
> #if (NGX_CLEAR_EVENT)
>         return ngx_add_event(ev, NGX_READ_EVENT, NGX_CLEAR_EVENT);
> #else
>     return ngx_add_event(ev, NGX_READ_EVENT, NGX_ONESHOT_EVENT);
> #endif
> }
> ```
>
>> 将连接socket对应的读事件对象加入事件队列。同时我们可以看到读事件的事件处理函数为空，即这里的功能尚未完成。

## 总结

nginx事件处理模块在TCP服务端程序处理框架下，将整个过程划入了不同的模块。包括event事件处理，IO复用技术相关的select模块，事件accept模块，http event模块等。

```plaintext

    +------------------------------+                  +------------------------------+                 +-----------------------------+     
    |   1.server.handler init      |                  |    2.events init             |---------------->|                             |   
    |                    |         |                  |    3.event join event_queue  |---------------->|      5.eventhandler         |----+    
    |                    |         |----------------->|    4.event process           |---------------->|                             |    |
    |                    |         |                  |                              |                 |                             |    |
    +--------------------|---------+                  +------------------------------+                 +-----------------------------+    |
                  main.c |                                       event module                                     select module           |
                         |                                     (ngx_event.c)                                (ngx_select_module.c)         |
                         |                                                                                                                |
                         |                                                                                                                |
                         |                                                                                                                |
                         |              +------------------------------+                        +------------------------------+          |
                         |              |                              |                        |                              |          |
                         +------------> |  7.add event to event_queue  |<-----------------------|         6.accept handler     |<---------+
                                        |                              |                        |                              |
                                        +------------------------------+                        +------------------------------+
                                                 http_event module                                         accept module
                                                 (ngx_http_event.c)                                    (ngx_event_accept.c)
```

在event module模块中，nginx定义了一个event对象ngx_event_t类型的数组ngx_x_events，数组上限由max_connections决定，默认使512。对于监听的socket、连接的soket，nginx按照
``f(x)=x``哈希函数确定socket对应的事件在ngx_x_events数组中的位置，并初始化。例如监听的socket描述符fd=5,则该fd对应的事件对象初始化在ngx_x_events[5]。

整个nginx请求处理采用事件驱动机制，处理都抽象成了事件对象，nginx维护了一个全局的事件循环链表event_queue，listen socket fd以事件对象投入到了event_queue中。每当该fd有事件发生时，通过accept生成了连接socket fd，并加入到event_queue中，nginx循环遍历event_queue，这样当连接fd有消息到达时，nginx就能通过读取该event获取消息。

```plaintext

    +--------------------+
    |                    |
   \|/    event_loop    /|\
    |                    |
    +--------------------+                                  event type
+--------------------------------------+   single event    +------------+    listen          +--------+    accept
|         |        |        |          |------------------>|  listen    |  ----------------> |        | ----------------+
|  event  |  event | ...    |  event   |                   |  socket fd |                    |        |      generate   |
|------+------------------------+------|                   +------------+                    +--------+                 |
| event|                        |event |                                                                                |
|------|                        |------|                                                         event                  |
|     .|      event_queue       |.     |                   add to event loop                 +------------+             |
|     .|                        |.     |<----------------------------------------------------|  accept    |<------------+
|     .|                        |.     |                                                     |  socket fd |   
|      |                        |      |                                                     +------------+                  
|------+------------------------+------|
|  event  |                  | event   |
|         |     ...          |         |
+--------------------------------------+

监听socket event产生连接socket并加入到事件队列过程
```
