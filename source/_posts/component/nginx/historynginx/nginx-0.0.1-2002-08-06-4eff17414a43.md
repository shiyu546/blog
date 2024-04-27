---
title: nginx历史版本结构分析
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
            TCP客户                                          +--------------+
       +--------------+                                             |
       |   socket()   |                                            \|/
       +--------------+                                    一直阻塞直到客户连接到达
              |                                                    |
             \|/                                                   |
       +--------------+                   连接建立                  |                  
       |   connect()  |------------------------------------------->|                  
       +--------------+                     TCP三次握手             |
             |                                                     |
            \|/                                                   \|/              
       +--------------+            数据请求                  +--------------+                            
    +->|   write()    |------------------------------------>|   read()     |                            
    |  +--------------+                                     +--------------+                            
    |        |                                                     |(处理请求)
    |       \|/                                                   \|/               
    |  +--------------+           数据应答                   +--------------+                            
    +--|   read()     |<------------------------------------|   write()    |                            
       +--------------+                                     +--------------+                            
             |                                                     |
            \|/                                                   \|/
       +--------------+             文件结束通知             +--------------+                            
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


### 事件

在nginx中，事件指的是web服务器中发生的各种活动，在该版本中共有三种事件类型：

1. read event读事件
2. write event写事件
3. timer event，时钟事件

读和写都可以归类于Network I/O event，包括从客户端读取数据，向客户端写数据，接收新的连接请求等等。nginx抽象了一个ngx_event_t结构体表示事件。

```c {.line-numbers}

typedef struct ngx_event_s       ngx_event_t;
struct ngx_event_s {
    void            *data;

    int            (*event_handler)(ngx_event_t *ev);
    int            (*close_handler)(ngx_event_t *ev);
    void            *context;
    char            *action;

    ngx_event_t     *prev;     /* queue in select(), poll() */
    ngx_event_t     *next;

    int            (*timer_handler)(ngx_event_t *ev);
    ngx_event_t     *timer_prev;
    ngx_event_t     *timer_next;

    u_int            timer_delta;
    u_int            timer;

    ngx_log_t       *log;

    int              available; /* kqueue only:                              */
                                /*   accept: number of sockets that wait     */
                                /*           to be accepted                  */
                                /*   read:   bytes to read                   */
                                /*   write:  available space in buffer       */
                                /* otherwise:                                */
                                /*   accept: 1 if accept many, 0 otherwise   */

    /* flags - int are probably faster on write then bits ??? */
    unsigned         listening:1;
    unsigned         write:1;

    unsigned         ready:1;
    unsigned         timedout:1;
    unsigned         process:1;
    unsigned         read_discarded:1;

    unsigned         unexpected_eof:1;

#if (HAVE_DEFERRED_ACCEPT)
    unsigned         accept_filter:1;
#endif
#if (HAVE_KQUEUE)
    unsigned         eof:1;
    int              errno;
#endif
};
```

- line3: data是一个void类型的指针，赋值时一般指向事件的连接信息。
- line10-11: 节点包含有两个指向相同类型的指针，说明event结构可以构成一个链表。
- line30-end: 这些都是一些标识表示事件的信息。

## 分析

### main函数入口

再来看nginx的main函数，为了更聚焦整体逻辑，这里把部分平台相关的代码给屏蔽掉了。

```c {.line-numbers}

int main(int argc, char *const *argv)
{
    char addr_text[22];
    ngx_socket_t fd;
    ngx_listen_t ls;

    ngx_log.log_level = NGX_LOG_DEBUG;
    ngx_pool.log = &ngx_log;
    ngx_addr.sin_port = htons(8000);
    ngx_addr.sin_family = AF_INET;

    fd = ngx_listen((struct sockaddr *) &ngx_addr, -1, &ngx_log, addr_text);

    ngx_server.buff_size = 1024;
    ngx_server.handler = ngx_http_init_connection;

    /* daemon */

    ls.fd = fd;
    ls.server = &ngx_server;
    ls.log = &ngx_log;

    /* fork */
    ngx_worker(&ls, 1, &ngx_pool, &ngx_log);
}

struct sockaddr_in ngx_addr = {0, AF_INET, 0, 0, 0};
```

- 结构
  - line5: ngx_listen_t表示监听的socket配置信息，该结构体主要含有三个字段：表示socket描述符的fd，指向ngx_server_t结构体的指针，指向ngx_log_t结构体的指针。

    ```c {.line-numbers}

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

  - line8: ngx_pool是一个ngx_pool_t类型的全局变量，主要是用来做内存管理。

    ```c {.line-numbers}

        typedef struct ngx_pool_s  ngx_pool_t;
        struct ngx_pool_s {
            char              *last;
            char              *end;
            ngx_pool_t        *next;
            ngx_pool_large_t  *large;
            ngx_log_t         *log;
        };
    ```

  - line9: ngx_addr是一个sockaddr_in类型的全局变量，该变量是tcp实现部分提供的结构体，用来当作socket函数的参数。
  - 部分结构体的关系如下：

    ```c {.line-numbers}

        +------------+
        |  int  fd   |
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
                                                            | func handler  |                     ngx_pool_t
                                                            +---------------+
                                                            |  buff_size    |
                                                            +---------------+
                                                              ngx_server_t
    ```

- 流程
  - 整个main函数调用链路如下：

    ```plaintext
        
        main()
          |
          |
          o--------->ngx_listen()
          |
          |
          o--------->ngx_worker()
    ```

  - line12: ngx_listen()函数主要完成启动tcp服务端程序的一系列操作：socket()->bind()->listen(),最终返回监听的套接字socket.
  - line24: ngx_worker()函数主要事件处理，具体的tcp处理都发生在这里。

### ngx_listen函数

函数主要流程是调用启动TCP服务端的那一系列函数：socket()->bind()->listen()，最终函数返回了socket()函数返回的文件描述符s。

```c {.line-numbers}

ngx_socket_t ngx_listen(struct sockaddr *addr, int backlog,
                        ngx_log_t *log, char *addr_text)
{
    ngx_socket_t   s;
    int            reuseaddr = 1;
#if (WIN32)
    unsigned long  nb = 1;
#endif

    if ((s = socket(AF_INET, SOCK_STREAM, 0)) == -1)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno, "socket failed");

    if (setsockopt(s, SOL_SOCKET, SO_REUSEADDR,
                   (const void *) &reuseaddr, sizeof(int)) == -1)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
                     "ngx_listen: setsockopt (SO_REUSEADDR) failed");

#if (WIN32)
    if (ioctlsocket(s, FIONBIO, &nb) == -1)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
                     "ngx_listen: ioctlsocket (FIONBIO) failed");
#else
    if (fcntl(s, F_SETFL, O_NONBLOCK) == -1)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
                     "ngx_listen: fcntl (O_NONBLOCK) failed");
#endif

    if (bind(s, (struct sockaddr *) addr, sizeof(struct sockaddr_in)) == -1)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
                     "ngx_listen: bind to %s failed", addr_text);

    if (listen(s, backlog) == -1)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_socket_errno,
                     "ngx_listen: listen to %s failed", addr_text);

    return s;
}
```

### ngx_worker函数

ngx_worker函数完成了剩余的事件处理工作，ngx_worker的函数调用关系如下：

```plaintext

    ngx_worker()
        |
        |
        o--------------->ngx_init_events()
        |
        |
        o--------------->ngx_add_event()
        |
        |
        o--------------->ngx_process_events()
```

```c {.line-numbers}

//file: src/event/ngx_event.c
void ngx_worker(ngx_listen_t *sock, int n, ngx_pool_t *pool, ngx_log_t *log)
{
    int i, fd;

    /* per group */
    int max_connections = 512;

    /**
     *ngx_init_events兜兜转转调用了ngx_select_init()方法，视选择的IO复用库而定,
     初始化了ngx_event_actions结构。
    */
    ngx_init_events(max_connections, log);

    ngx_read_events = ngx_alloc(sizeof(ngx_event_t) * max_connections, log);
    ngx_write_events = ngx_alloc(sizeof(ngx_event_t) * max_connections, log);
    ngx_connections = ngx_alloc(sizeof(ngx_connection_t) * max_connections, log);

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

#if (HAVE_DEFERRED_ACCEPT)
        ngx_read_events[fd].accept_filter = sock->accept_filter;
#endif
        ngx_add_event(&ngx_read_events[fd], NGX_READ_EVENT, 0);
    }

    while (1)
    {
        ngx_log_debug(log, "ngx_worker cycle");

        ngx_process_events(log);
    }
}
ngx_connection_t *ngx_connections;
ngx_event_t *ngx_read_events, *ngx_write_events;
```

- line12: ngx_init_events是一个宏定义，替换之后是取全局变量ngx_event_init的一个元素，ngx_event_init是一个指针函数指针的数组，根据不同的IO多路复用类型调用不同的函数，以select复用技术为例，调用了src/event/modules/ngx_select_module.c中的ngx_select_init函数。

  ```c {.line-numbers}

   //全局指针数组
   static void (*ngx_event_init[])(int max_connections, ngx_log_t    *log) = {
           ngx_select_init,
       #if (HAVE_POLL)
           ngx_poll_init,
       #endif
       #if (HAVE_KQUEUE)
           ngx_kqueue_init
       #endif
   };
   
   //变量ngx_event_type的定义
   #if 1
   ngx_event_type_e ngx_event_type = NGX_SELECT_EVENT;
   #else
   ngx_event_type_e ngx_event_type = NGX_KQUEUE_EVENT;
   #endif
   
   //宏定义，取得是数组ngx_event_init下标ngx_event_type，根据上面的定义，   取得是指向ngx_select_init函数的函数指针
   #define ngx_init_events     (ngx_event_init[ngx_event_type])
  ```

  ngx_select_init函数实现如下：

  ```c {.line-numbers}

    static fd_set master_read_fds;
    static fd_set master_write_fds;
    static fd_set work_read_fds;
    static fd_set work_write_fds;
    static ngx_event_t event_queue;
    static ngx_event_t timer_queue;
    void ngx_select_init(int max_connections, ngx_log_t *log)
    {

        if (max_connections >= FD_SETSIZE)
            ngx_log_error(NGX_LOG_EMERG, log, 0,
                          "ngx_select_init: maximum descriptor number"
                          "supported by select() is %d",
                          FD_SETSIZE - 1);
    
        FD_ZERO(&master_read_fds);
        FD_ZERO(&master_write_fds);
    
        event_queue.prev = &event_queue;
        event_queue.next = &event_queue;
    
        timer_queue.timer_prev = &timer_queue;
        timer_queue.timer_next = &timer_queue;
    
        ngx_event_actions.add = ngx_select_add_event;
        ngx_event_actions.del = ngx_select_del_event;
        ngx_event_actions.process = ngx_select_process_events;
    
        max_fd = -1;
    }
  ```

  - line16-17: master_read_fds和master_write_fds是全局变量，这两行将初始化这两个全局描述符集。
  - line19-20: event_queue是一个类型为ngx_event_t的全局变量，在nginx中，所有的事件都用ngx_event_t结构体表示，同时ngx_event_t还是一个双向链表，含有指向前驱和后继的prev和next指针，这里将event_queue的前驱和后继都初始化为指向自己，从而构成了一个环。同理也初始化了timer_queue链表。

    ```plaintext

    +------------+      +--------------+           
    |           \|/    \|/             |
    |   +-------------------------+    |
    +---| prev |          |  next |----+
        +-------------------------+
            event_queue
        event_queue初始化      
    ```

  - line25: ngx_event_actions是一个类型为ngx_event_actions_t的全局变量，该结构体含有几个函数指针，这里分别初始化指向了相应的函数。

- line15-37: ngx_read_events,ngx_write_events都是类型为ngx_event_t指针的全局变量，分配了max_connections大小的空间，所以我们可以把这两个变量看作数组类型，for循环中获取了监听配置中的监听socket描述符fd，并初始化了数组中fd下标的内容。
- line42: ngx_add_event是一个宏定义，宏定义展开后调用了ngx_select_add_event函数:

    ```c {.line-numbers}
    
    int ngx_select_add_event(ngx_event_t *ev, int event, u_int flags)
    {
        fd_set *fds;
        ngx_connection_t *cn = (ngx_connection_t *)ev->data;
    
        if (event == NGX_TIMER_EVENT)
        {
            ngx_add_timer(ev, flags);
            return 0;
        }
    
        fds = ngx_select_get_fd_set(cn->fd, event, ev->log);
        if (fds == NULL)
            return -1;
    
        /**插入event_queue链表，插入位置在event_queue指向节点的下一个节点*/
        ev->prev = &event_queue;
        ev->next = event_queue.next;
        event_queue.next->prev = ev;
        event_queue.next = ev;
    
        FD_SET(cn->fd, fds);
    
        if (max_fd != -1 && max_fd < cn->fd)
            max_fd = cn->fd;
        return 0;
    }
    ```

  - line12: 该函数根据事件类型获取相应的全局描述符集，这里ngx_worker中传的参数是NGX_READ_EVENT，所以这里返回的是全局读描述符集master_read_fds。
  - line17-20: 将初始化的fd下标事件插入到event_queue链表中，事件队列构成了一个环(ring)。

    ```plaintext

                            +--------------+-------------------------------------+ 
                           \|/                                                   |
            +-------------------------+           +-------------------------+    |
            | prev |          |  next |<----------| prev |          |  next |----+
            +-------------------------+           +-------------------------+
                |                  |                             /|\
                +-------------------------------------------------+
                event_queue                                   ev
        event_queue初始化     
    ```

  - line22: 将fd配置到描述符集中，这样该描述符集就可以通过判断fd的状态来确定是否有事件到达。
- line49: 和ngx_add_event一样也是宏定义，调用了ngx_select_process_events函数：

    ```c {.line-numbers}

    int ngx_select_process_events(ngx_log_t *log)
    {
        int ready, found;
        u_int timer, delta;
        ngx_event_t *ev, *nx;
        ngx_connection_t *cn;
        struct timeval tv, *tp;

        work_read_fds = master_read_fds;
        work_write_fds = master_write_fds;

        if (timer_queue.timer_next != &timer_queue)
        {
            timer = timer_queue.timer_next->timer_delta;
            tv.tv_sec = timer / 1000;
            tv.tv_usec = (timer % 1000) * 1000;
            tp = &tv;

            delta = ngx_msec();
        }
        else
        {
            timer = 0;
            tp = NULL;
            delta = 0;
        }

        if ((ready = select(max_fd + 1, &work_read_fds, &work_write_fds, NULL, tp))== -1)
        {
            ngx_log_error(NGX_LOG_ALERT, log, ngx_socket_errno,
                          "ngx_select_process_events: select failed");
            return -1;
        }

        if (timer)
        {
            if (delta >= timer)
            {
                for (ev = timer_queue.timer_next;
                     ev != &timer_queue && delta >= ev->timer_delta;
                     /* void */)
                {
                    delta -= ev->timer_delta;
                    nx = ev->timer_next;
                    ngx_del_timer(ev);
                    if (ev->timer_handler(ev) == -1)
                        ev->close_handler(ev);
                    ev = nx;
                }
            }
            else
            {
                timer_queue.timer_next->timer_delta -= delta;
            }
        }

        for (ev = event_queue.next; ev != &event_queue; ev = ev->next)
        {
            cn = (ngx_connection_t *)ev->data;
            found = 0;

            if (ev->write)
            {
                if (FD_ISSET(cn->fd, &work_write_fds))
                {
                    ngx_log_debug(log, "ngx_select_process_events: write %d" _
                                           cn->fd);
                    found = 1;
                }
            }
            else
            {
                if (FD_ISSET(cn->fd, &work_read_fds))
                {
                    ngx_log_debug(log, "ngx_select_process_events: read %d" _
                                           cn->fd);
                    found = 1;
                }
            }

            if (found)
            {
                ev->ready = 1;
                if (ev->event_handler(ev) == -1)
                    ev->close_handler(ev);

                ready--;
            }
        }
        return 0;
    }
    ```

  - line28: 调用select阻塞在事件监听上。
  - line57-89: 循环遍历event_queue链表，判断事件对象上的socket fd是否有事件发生，如果有则调用事件对象的函数指针event_handler，处理事件。在ngx_worker中，监听的socket fd的event_handler指向了ngx_event_accept函数，所以当监听事件发生时，会调用ngx_event_accept方法。

    ```c {.line-numbers}

    int ngx_event_accept(ngx_event_t *ev)
    {
        ngx_err_t           err;
        ngx_socket_t        s;
        struct sockaddr_in  addr;
        int addrlen = sizeof(struct sockaddr_in);
        ngx_connection_t *cn = (ngx_connection_t *) ev->data;

        ev->ready = 0;
    
        do {
            if ((s = accept(cn->fd, (struct sockaddr *) &addr, &addrlen)) == -1) {
            }

            ngx_memzero(&ngx_read_events[s], sizeof(ngx_event_t));
            ngx_memzero(&ngx_write_events[s], sizeof(ngx_event_t));
            ngx_memzero(&ngx_connections[s], sizeof(ngx_connection_t));

            ngx_read_events[s].data = ngx_write_events[s].data
                                                             = &ngx_connections[s];
            ngx_connections[s].read = &ngx_read_events[s];
            ngx_connections[s].write = &ngx_write_events[s];

            ngx_connections[s].fd = s;
            ngx_read_events[s].unexpected_eof = 1;
            ngx_write_events[s].ready = 1;

            ngx_write_events[s].timer = ngx_read_events[s].timer = 10000;

            ngx_write_events[s].timer_handler =
                ngx_read_events[s].timer_handler = ngx_event_close;

            ngx_write_events[s].close_handler =
                ngx_read_events[s].close_handler = ngx_event_close;

            ngx_connections[s].server = cn->server;
            ngx_connections[s].servers = cn->servers;
            ngx_connections[s].log =
                ngx_read_events[s].log = ngx_write_events[s].log = ev->log;

        #if (HAVE_DEFERRED_ACCEPT)
                if (ev->accept_filter)
                    ngx_read_events[s].ready = 1;
        #endif

            cn->server->handler(&ngx_connections[s]);
        } while (ev->available);
    
        return 0;
    }
    ```

    - line12: 当监听socket有请求到来时，accept()返回连接socket fd。这里将连接fd放入events数组中。
    - line46: 调用了ngx_server_t中的handler函数，在nginx.c中，server handler初始化为ngx_http_init_connection.该函数将连接socket fd加入到了event_queue队列中。

      ```c {.line-numbers}

        int ngx_http_init_connection(ngx_connection_t *c)
        {
            ngx_event_t  *ev;
        
            ev = c->read;
            ev->event_handler = NULL;
            ev->log->action = "reading client request line";

            /* XXX: ev->timer ? */
            if (ngx_add_event(ev, NGX_TIMER_EVENT, ev->timer) == -1)
                return -1;

            return ngx_add_event(ev, NGX_READ_EVENT,/*视配置而定*/);
        }
      ```

### 总结

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
