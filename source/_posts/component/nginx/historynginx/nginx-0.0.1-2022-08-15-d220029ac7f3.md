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

nginx http处理由几个函数指针串联起来，在主函数main.c中，ngx_server_t的handler函数指针指向了ngx_http_init_connection函数，在ngx_event_accept.c中，会调用handler指针指向的函数。

ngx_http_init_connection函数中，event对象的event_handler函数指针会指向ngx_http_init_request函数，event的event_handler会在ngx_event.c的ngx_process_events中调用(不同的IO多路复用调用的函数不同，以select为例，调用的就是ngx_select_process_events方法)，从而执行ngx_http_init_request方法。

ngx_http_init_request完成http消息处理，通过为ngx_http_request_t的state_handler函数指针赋不同的值，调用不同的函数完成http消息的处理过程。

### 版本

本文基于nginx-d220029ac7f3,上传于2002-08-15版本进行分析。

## 基础

### nginx内存池ngx_pool_t

nginx自定义了一个表示内存空间的数据结构ngx_pool_t，ngx_pool_t内存池数据结构如下：

```c {.line-numbers}
//file: src/core/ngx_alloc.h
typedef struct ngx_pool_s  ngx_pool_t;
struct ngx_pool_s {
    char              *last;
    char              *end;
    ngx_pool_t        *next;
    ngx_pool_large_t  *large;
    ngx_log_t         *log;
};

ngx_pool_t *ngx_create_pool(size_t size, ngx_log_t *log)
{
    ngx_pool_t *p;

    ngx_test_null(p, ngx_alloc(size, log), NULL);

    p->last = (char *) p + sizeof(ngx_pool_t);
    p->end = (char *) p + size;
    p->next = NULL;
    p->large = NULL;
    p->log = log;

    return p;
}

void *ngx_alloc(size_t size, ngx_log_t *log)
{
    void *p;

    p = malloc(size);
    if (p == NULL)
        ngx_log_error(NGX_LOG_EMERG, log, ngx_errno,
                     "ngx_alloc: malloc %d bytes failed", size);
    return p;
}
```

结合内存分配函数ngx_create_pool来看，内存池ngx_pool_t p分配了一块大小为size的内存，p->last指向ngx_pool_t的末尾，p->end指向内存池的末尾。同时从next可以看出ngx_pool_t构成了内存池链表，large则指向大块内存。

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

## 分析

### ngx_http_init_connection函数

```c {.line-numbers}
//file:src/http/ngx_http_event.c
int ngx_http_init_connection(ngx_connection_t *c)
{
    ngx_event_t *ev;

    ev = c->read;
    ev->event_handler = ngx_http_init_request;
    ev->log->action = "reading client request line";

    ngx_log_debug(ev->log, "ngx_http_init_connection: entered");

    /* XXX: ev->timer ? */
    if (ngx_add_event(ev, NGX_TIMER_EVENT, ev->timer) == -1)
        return -1;
    /**...*/
}
```

- line7: init_connection的主要作用是设置event对象的event_handler函数指针，该函数指针会在event有事件发生时在ngx_process_events函数中调用。

### ngx_http_init_request函数

该函数主要完成一些数据结构的初始化，包括connection对象的pool，request对象的buffer等。

```c {.line-numbers}
//file:src/http/ngx_http_event.c
int ngx_http_init_request(ngx_event_t *ev)
{
    ngx_connection_t *c = (ngx_connection_t *)ev->data;
    ngx_http_request_t *r;

    ngx_log_debug(ev->log, "ngx_http_init_request: entered");

    ngx_test_null(c->pool, ngx_create_pool(16384, ev->log), -1);
    ngx_test_null(r, ngx_pcalloc(c->pool, sizeof(ngx_http_request_t)), -1);

    c->data = r;
    r->connection = c;

    ngx_test_null(r->pool, ngx_create_pool(16384, ev->log), -1);
    ngx_test_null(r->buff, ngx_palloc(r->pool, sizeof(ngx_buff_t)), -1);
    ngx_test_null(r->buff->buff,
                  ngx_pcalloc(r->pool, sizeof(c->server->buff_size)), -1);

    r->buff->pos = r->buff->last = r->buff->buff;
    r->buff->end = r->buff->buff + c->server->buff_size;

    r->state_handler = ngx_process_http_request_line;

    ev->event_handler = ngx_http_process_request;
    ev->close_handler = ngx_http_close_request;
    c->write->close_handler = ngx_http_close_request;
    return ngx_http_process_request(ev);
}
```

ngx_http_request_t r中的r->buff和r->buff->buff都从ngx_pool_t分配的内存中获取内存块，同时初始化buff结构中的一些变量。

ngx_http_init_request方法中调用了ngx_http_process_request函数，该函数主要从socket连接中读取数据并保存到buff中。然后调用request的state_handler函数指针。由上面init函数可知state_handler指向了ngx_process_http_request_line函数，所以接下来调用了ngx_process_http_request_line函数。

```c {.line-numbers}
//file:src/http/ngx_http_event.c
int ngx_http_process_request(ngx_event_t *ev)
{
    int n;
    ngx_connection_t *c = (ngx_connection_t *)ev->data;
    ngx_http_request_t *r = (ngx_http_request_t *)c->data;

    n = ngx_event_recv(ev, r->buff->last, r->buff->end - r->buff->last);

    if (!ev->read_discarded)
    {
        r->buff->last += n;

        /* state_handlers are called in following order:
            ngx_process_http_request_line()
            ngx_process_http_request_header() */
        do
        {
            if ((r->state_handler)(r) < 0)
                return -1;
        } while (r->buff->pos < r->buff->last);
    }
    return 0;
}
```

### ngx_process_http_request_line函数

```c {.line-numbers}
//file src/http/ngx_http_event.c
static int ngx_process_http_request_line(ngx_http_request_t *r)
{
    int n;

    if  ((n = ngx_read_http_request_line(r)) == 1)
    {
        *r->uri_end = '\0';
        ngx_log_debug(r->connection->log, "HTTP: %d, %d, %s" _
                                              r->method _ r->http_version _ r->uri_start);
        r->state_handler = ngx_process_http_request_header;
        r->connection->read->action = "reading client request headers";
    }

    return n;
}
```

- line6: 函数主要调用了ngx_read_http_request_line来处理数据。该方法使用状态模式来解析http请求头的数据，通过遍历接收到的数据提取http请求头并放入相应的request字段。

   ```c {.line-numbers}
   int ngx_read_http_request_line(ngx_http_request_t *r)
    {
        char  ch;
        char *buff = r->buff->buff;
        char *p = r->buff->pos;
        enum {
            rl_start = 0,
            rl_space_after_method,
            rl_spaces_before_uri,
            rl_after_slash_in_uri,
            rl_check_uri,
            rl_uri,
            rl_http_09,
            rl_http_version,
            rl_first_major_digit,
            rl_major_digit,
            rl_first_minor_digit,
            rl_minor_digit,
            rl_almost_done,
            rl_done
        } state = r->state;

        while (p < r->buff->last && state < rl_done) {
            ch = *p++;

            switch (state) {

            /* HTTP methods: GET, HEAD, POST */
            case rl_start:
            /**... */
                state = rl_space_after_method;
                break;
            }
        }  

        if (state == rl_done) {
            r->http_version = r->http_major * 1000 + r->http_minor;
            r->state = rl_start;
            return 1;
        } else {
            r->state = state;
            return 0;
        }   
    }
   ```

## 总结

nginx http消息处理模块通过ngx_event_t的event_handler、state_handler等函数指针串联起来，在nginx_http_parse.c模块处理，处理内容包括http requeline，http header等。该处理逻辑基于http0.9的RFC文档RFC-1945中规定的http协议格式逐个处理。
