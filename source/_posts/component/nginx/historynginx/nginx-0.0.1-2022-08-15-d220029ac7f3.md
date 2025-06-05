---
title: nginx历史版本结构分析(二)
subtitle: http消息处理
date: 2024-05-03 17:07:11
categories:
- [组件,nginx,nginx历史版本]
tags:
- 组件
- nginx
---

## 概述

上一节中分析了nginx的整体执行流程。流程如下：

1. 首先监听在指定端口上，并创建监听socket和对应的事件，将事件投入事件队列，当有客户端请求连接时，socket就绪，执行事件处理函数。
2. 监听socket对应的事件处理函数从监听socket创建连接socket，并生成对应的事件对象,随后执行了连接事件的事件处理函数`ngx_http_init_connection`。
3. 该事件处理函数设置了连接socket的事件处理函数(为NULL)，并将事件加入了事件队列。

在nginx-d220029ac7f3版本中，连接socket的事件处理函数已经设置，所以这里分析以下相应的执行方法。

## 基础

### nginx内存池ngx_pool_t

在处理连接socket的事件时，nginx使用了自定义的存储管理功能。nginx使用了结构体ngx_pool_t来表示内存结构，ngx_pool_t定义如下：

```c {.line-numbers}
typedef struct ngx_pool_large_s  ngx_pool_large_t;
struct ngx_pool_large_s {
    ngx_pool_large_t  *next;
    void              *alloc;
};

typedef struct ngx_pool_s  ngx_pool_t;
struct ngx_pool_s {
    char              *last;
    char              *end;
    ngx_pool_t        *next;
    ngx_pool_large_t  *large;
    ngx_log_t         *log;
};
```

>> *last* 指向的是当前已分配的内存块的末尾地址；
>> *end* 指向的是内存池的末尾地址；
>> *next* 表示内存块通过指针连接成了链表，构成了内存池链表；
>> *large* 则指向分配的大块内存，large本身也是一个链表结构。
>> *large->alloc* 则是large大块内存指向的分配的内存地址。

```plaintext
               ngx_pool_t p --------------+
                                         \|/
----        ----                   +--------------+                         +-----------+              
 /|\        /|\               +----|    last      |            linked_list  |           |          
  |          |                |    +--------------+          +------------->|ngx_pool_t |                     
  |          |             +-------|     end      |          |              |           |         
  |          |             |  |    +--------------+          |              |           |         
  |       ngx_pool_t       |  |    |    next      |----------+              +-----------+          
  |          |             |  |    +--------------+                                   
 size        |             |  |    |    large     |----------+                                     
  |          |             |  |    +--------------+          |            +--------------+            
  |         \|/            |  |    |     log      |          |            |              |              
  |        ----            |  |    +--------------+          +----------->|              |        
  |                        |  +--->|              |            large_list | large_block  | 
  |                        |       |              |                       |              | 
  |                        |       |              |                       |              |  
  |                        |       |              |                       +--------------+   
 \|/                       |       |              |                    
 ----                      |       +--------------+               
                           |------>                        

ngx_pool_t结构
```

nginx内存分配的层次结构如下，从下至上分别是库函数层，库函数封装层，内存池结构初始化层以及内存分配函数层。

```plaintext
                                +-----------------------------------+
                                |   ngx_palloc()  ngx_pcalloc()     |
                           +------------------------------------------------+
                           |  ngx_create_pool()      ngx_destroy_pool()     |
                      +----------------------------------------------------------+
库函数封装层           |   ngx_alloc()                          ngx_calloc()      |
                 +--------------------------------------------------------------------------+
库函数层         |                           malloc()                                       |
                 +--------------------------------------------------------------------------+
```

#### ngx_alloc和ngx_calloc函数

`ngx_alloc()`和`ngx_calloc()`都是对`malloc()`库函数的封装，两者的区别是`ngx_calloc()`会对分配的内存进行初始化，即初始化内存块中的每个字节为0。

#### ngx_create_pool函数

`ngx_create_pool()`函数用于创建一个内存池，该函数首先调用`ngx_alloc()`函数分配一个内存块，内存块的起始内存部分分配给p，并将*last*设置为除p占用空间的起始位置，end设置为内存块的末尾。

> ```c
> ngx_pool_t *ngx_create_pool(size_t size, ngx_log_t *log)
> {
>     ngx_pool_t *p;
> 
>     ngx_test_null(p, ngx_alloc(size, log), NULL);
> ```
>
>> *p*指向了分配的内存块地址，所以内存块应当容纳一个结构体`ngx_pool_t`,但这里分配的内存明显大于结构体的大小，结构体会占据内存块的起始部分，剩余的内存则当作内存池由程序来申请使用。这里*last*指向的就是剩余内存块的起始位置，*end*指向了内存块的末尾。
>
> ```c
>     p->last = (char *) p + sizeof(ngx_pool_t);
>     p->end = (char *) p + size;
>     p->next = NULL;
>     p->large = NULL;
>     p->log = log;
> 
>     return p;
> }
> ```

#### ngx_palloc函数

`ngx_palloc()`从内存池*p*中申请一块大小为size的内存块，并返回该内存块的地址。根据申请的内存块大小，nginx会选择从内存池中申请内存块还是从大块内存中申请内存块。

> ```c {.line-numbers}
> void *ngx_palloc(ngx_pool_t *pool, size_t size)
> {
>     void              *m;
>     ngx_pool_t        *p, *n;
>     ngx_pool_large_t  *large, *last;
> 
>     if (size <= NGX_MAX_ALLOC_FROM_POOL) {
> ```
>
>> 该NGX_MAX_ALLOC_FROM_POOL宏定义如下：
>> ``#define NGX_MAX_ALLOC_FROM_POOL (8192 - sizeof(ngx_pool_t))``
>> 以该宏定义的大小为界，申请的内存块小于宏定义的值时，从内存池pool中分配内存。
>
> ```c
>         for (p = pool, n = pool->next; /* void */; p = n, n = n->next) {
>             if ((size_t) (p->end - p->last) >= size) {
>                 m = p->last;
>                 p->last += size;
> 
>                 return m;
>             }
> 
>             if (n == NULL)
>                 break;
>         }
> ```
>
>> pool本身是一个链表，这里遍历链表，直到找到一块内存能满足分配需求，并更新last指针，返回内存块地址。
>
> ```c
>         /* alloc new pool block */
>         ngx_test_null(n, ngx_create_pool(p->end - (char *) p, p->log), NULL);
>         p->next = n;
>         m = n->last;
>         n->last += size;
>         return m;
> ```
>
>> 如果pool链表没有内存块满足要求，则新分配一个内存块，这里的size为`p->end - (char *) p`使得链表的每一个节点内存块大小相等。
>
> ```c
>     /* alloc large block */
>     } else {
>         large = NULL;
>         last = NULL;
> 
>         if (pool->large) {
>             for (last = pool->large; /* void */; last = last->next) {
>                 if (last->alloc == NULL) {
>                     large = last;
>                     last = NULL;
>                     break;
>                 }
> 
>                 if (last->next == NULL)
>                     break;
>             }
>         }
> 
>         if (large == NULL) {
>             ngx_test_null(large, ngx_palloc(pool, sizeof(ngx_pool_large_t)),
>                           NULL);
>         }
> 
>         ngx_test_null(p, ngx_alloc(size, pool->log), NULL);
> 
>         if (pool->large == NULL) {
>             pool->large = large;
> 
>         } else if (last) {
>             last->next = large;
>         }
> 
>         large->alloc = p;
> 
>         return p;
>     }
> }
> ```
>
>> 当申请内存大小大于指定值时，由内存池的large成员来管理大块内存。large是一个链表结构，large->alloc指向大块内存的起始地址，large->next指向下一个大块内存。从`if (pool->large)`条件判断，大块内存只会在链表的第一个节点上分配。

### http请求对象

`ngx_http_request_t`是NGINX HTTP模块中的核心数据结构，表示一个完整的HTTP请求的上下文.用途如下：

* 用来表示并管理客户端请求的状态；
* 承载解析状态（如 method、URI、版本号、header）；
* 关联底层连接、内存池、缓冲区；
* 在整个请求生命周期中传递，供 handler 使用；

> ```c
> typedef struct ngx_http_request_s ngx_http_request_t;
> 
> struct ngx_http_request_s {
>     int    method;
> 
>     int    http_version;
>     int    http_major;
>     int    http_minor;
> ```
>
>> *method* 表示请求方式，有GET、HEAD、POST三种类型的取值。
>>
>>    ```
>>    #define NGX_HTTP_GET   1
>>    #define NGX_HTTP_HEAD  2
>>    #define NGX_HTTP_POST  3
>>    ```
>>
>> *http_version* 表示HTTP版本,http版本由"\<major\>.\<minor\>"格式组成，major和minor分别存储到http_major和http_minor中。http_version的计算逻辑是http_major * 1000 + http_minor.
>
> ```c
>     char  *uri;
>     ngx_http_request_t *main;
> ```
>
>> *uri* 表示http请求的uri。
>
> ```c
>     ngx_connection_t *connection;
>     ngx_buff_t  *buff;
>     ngx_pool_t  *pool;
> ```
>
>> buff和pool用来存储并操作从socket读取的数据，buff含有两个指针pos和last，分别指向缓冲区已经读取的数据的位置和缓冲区中有效数据位置。
>
> ```c
>     /* internal */
>     unsigned  unusual_uri:1;
>     unsigned  complex_uri:1;
> 
>     int    state;
>     char  *uri_start;
>     char  *uri_end;
>     char  *uri_ext;
> ```
>
>> *state* 状态标识，解析数据时用来标识当前的状态；
>> *uri_start* *uri_end* *uri_ext* 分别表示用来记录uri的起始位置、结束位置和扩展名位置。
>
> ```c
>     char  *args_start;
>     char  *header_name_start;
>     char  *header_name_end;
>     char  *header_start;
>     char  *header_end;
> #ifdef NGX_EVENT
>     int  (*state_handler)(ngx_http_request_t *r);
> #endif
> };
> ```

### 数据接收缓冲区ngx_buff_t

`ngx_buff_t`是NGINX中用来管理HTTP请求的缓冲区，其结构体如下:

> ```c
> typedef struct {
>     char *buff;
> ```
>
>> buff指向分配的内存。
>
> ```c
>     char *pos;
>     char *last;
>     char *end;
> } ngx_buff_t;
> ```
>
>> pos指向缓冲区中已经读取的数据位置，last指向缓冲区中还有效的数据位置，end指向缓冲区的末尾位置。读取数据时，last指针前移，消费数据时，pos指针前移。

## 分析

### ngx_http_init_connection函数

在上一节中，`ngx_http_init_connection`函数设置了连接socket对应的事件的事件处理函数，并调用`ngx_add_event`函数将事件添加到事件队列中。其中事件处理函数设置为空，这一节设置成了`ngx_http_init_request`。

### ngx_http_init_request函数

该函数执行如下过程：

1. 初始化一个`ngx_http_request_t`对象r；
2. 分配r->buff->buff,并设置位置指针；
3. 设置r的state_handler为ngx_process_http_request_line，即处理HTTP请求行；
4. 变更事件的事件处理函数为ngx_http_process_request.

执行`ngx_http_init_request`函数之后，事件处理函数指向`ngx_http_process_request`,用于处理客户端发送的数据。

```plaintext
    客户端                             服务端
    connection---------------> ngx_http_init_request
                                        |               
                                        |               
                                       \|/              
                                ngx_http_process_request
                                           |
                                           |
    write--------------------------------->|                    
                                           |
                                          \|/
```

> ```c
> int ngx_http_init_request(ngx_event_t *ev)
> {
>     ngx_connection_t   *c = (ngx_connection_t *) ev->data;
>     ngx_http_request_t *r;
> 
>     ngx_log_debug(ev->log, "ngx_http_init_request: entered");
> 
>     ngx_test_null(c->pool, ngx_create_pool(16384, ev->log), -1);
>     ngx_test_null(r, ngx_pcalloc(c->pool, sizeof(ngx_http_request_t)), -1);
> 
>     c->data = r;
>     r->connection = c;
> ```
>
>> 初始化了http请求对象ngx_http_request_t，并利用connection属性保存连接对象。
>
> ```c
>     ngx_test_null(r->pool, ngx_create_pool(16384, ev->log), -1);
>     ngx_test_null(r->buff, ngx_palloc(r->pool, sizeof(ngx_buff_t)), -1);
>     ngx_test_null(r->buff->buff,
>                   ngx_pcalloc(r->pool, sizeof(c->server->buff_size)), -1);
> 
>     r->buff->pos = r->buff->last = r->buff->buff;
>     r->buff->end = r->buff->buff + c->server->buff_size;
> ```
>
>> r->buff是一个类型为`ngx_buff_t`指针的字段，这里分配了一段内存给buff。
>
> ```c
>     r->state_handler = ngx_process_http_request_line;
> 
>     ev->event_handler = ngx_http_process_request;
>     ev->close_handler = ngx_http_close_request;
>     c->write->close_handler = ngx_http_close_request;
>     return ngx_http_process_request(ev);
> }
> ```
>
>> 执行这里后，该事件的事件处理函数已经改变，为`ngx_http_process_request`。

### ngx_http_process_request执行过程

`ngx_http_process_request`主要过程为：

1. 从socket读取数据并存入r->buff->buff中；
2. 执行`state_handler`函数指针指向的函数。

> ```c
> int ngx_http_process_request(ngx_event_t *ev)
> {
>     int n;
>     ngx_connection_t *c = (ngx_connection_t *) ev->data;
>     ngx_http_request_t *r = (ngx_http_request_t *) c->data;
> 
>     ngx_log_debug(ev->log, "http process request");
> 
>     n = ngx_event_recv(ev, r->buff->last, r->buff->end - r->buff->last);
> ```
>
>> 调用`recv()`函数读取数据，将数据存入r->buff->buff中。
>
> ```c
>     if (!ev->read_discarded) {
>         r->buff->last += n;
> 
>         /* state_handlers are called in following order:
>             ngx_process_http_request_line()
>             ngx_process_http_request_header() */
> 
>         do {
>             if ((r->state_handler)(r) < 0)
>                 return -1;
>         } while (r->buff->pos < r->buff->last);
>     }
> 
>     return 0;
> }
> ```
>
>> 执行`r->state_handler`函数，从上面的初始化过程得知函数指针指向`ngx_process_http_request_line`.

接下来`ngx_process_http_request_line`调用了`ngx_read_http_request_line`读取并解析缓冲区的数据，然后修改了*state_handler*的函数指针，指向了`ngx_process_http_request_header`。

```c
static int ngx_process_http_request_line(ngx_http_request_t *r)
{
    int n;

    if ((n = ngx_read_http_request_line(r)) == 1) {
        *r->uri_end = '\0';
        ngx_log_debug(r->connection->log, "HTTP: %d, %d, %s" _
                     r->method _ r->http_version _ r->uri_start);
        r->state_handler = ngx_process_http_request_header;
        r->connection->read->action = "reading client request headers";
    }

    return n;
}
```

#### ngx_read_http_request_line

处理客户端发送的数据发生在这里，`ngx_read_http_request_line`函数很长，主要是从缓冲区中读取数据，并按照http的协议解析数据。

HTTP首先接收的数据是一串Request-Line,格式如下：

`` Request-Line = Method SP Request-URI SP HTTP-Version CRLF ``

* Method: 请求方法，GET、POST、HEAD、PUT、DELETE、OPTIONS、TRACE、CONNECT都是允许的方法，在该版本的http协议中，只包含GET、POST、HEAD三种。
* SP : 空格，CRLF : 回车换行,这些都是控制字符。
* Request-URI: 请求的资源，可以是一个URL，也可以是一个本地文件。
* HTTP-Version: HTTP协议版本，HTTP协议由主版本和次要版本组成，例如HTTP/1.1。

> ```C
> int ngx_read_http_request_line(ngx_http_request_t *r)
> {
>     char  ch;
>     char *buff = r->buff->buff;
>     char *p = r->buff->pos;
>     enum {
>         rl_start = 0,
>         rl_space_after_method,
>         rl_spaces_before_uri,
>         rl_after_slash_in_uri,
>         rl_check_uri,
>         rl_uri,
>         rl_http_09,
>         rl_http_version,
>         rl_first_major_digit,
>         rl_major_digit,
>         rl_first_minor_digit,
>         rl_minor_digit,
>         rl_almost_done,
>         rl_done
>     } state = r->state;
> ```
>
>> state采用状态的模式，遍历缓冲区，并按照Request-Line的格式解析缓冲的内容。在下面的循环中，会根据state的值，逐步解析出method、uri、http_version等信息。
>
> ```c
>     while (p < r->buff->last && state < rl_done) {
>         ch = *p++;
> 
> /*
> printf("\nstate: %d, pos: %x, end: %x, char: '%c' buf: %s",
>        state, p, r->buff->last, ch, p);
> */
> 
>         /* GCC complie it as jump table */
> 
>         switch (state) {
> 
>         /* HTTP methods: GET, HEAD, POST */
>         case rl_start:
>             switch (ch) {
>             case 'G':
>                 if (p + 1 >= r->buff->last)
>                     return 0;
> 
>                 if (*p != 'E' || *(p + 1) != 'T')
>                     return NGX_HTTP_INVALID_METHOD;
> 
>                 r->method = NGX_HTTP_GET;
>                 p += 2;
>                 break;
> 
>             case 'H':
>                 if (p + 2 >= r->buff->last)
>                     return 0;
> 
>                 if (*p != 'E' || *(p + 1) != 'A' || *(p + 2) != 'D')
>                     return NGX_HTTP_INVALID_METHOD;
> 
>                 r->method = NGX_HTTP_HEAD;
>                 p += 3;
>                 break;
> 
>             case 'P':
>                 if (p + 2 >= r->buff->last)
>                     return 0;
> 
>                 if (*p != 'O' || *(p + 1) != 'S' || *(p + 2) != 'T')
>                     return NGX_HTTP_INVALID_METHOD;
> 
>                 r->method = NGX_HTTP_POST;
>                 p += 3;
>                 break;
> 
>             default:
>                 return NGX_HTTP_INVALID_METHOD;
>             }
> 
>             state = rl_space_after_method;
>             break;
>             //...
> }
> ```

## 总结

nginx http消息处理模块通过ngx_event_t的event_handler、state_handler等函数指针串联起来，在nginx_http_parse.c模块处理，处理内容包括http requeline，http header等。该处理逻辑基于http1.0的RFC文档RFC-1945中规定的http协议格式逐个处理。
