---
title: 原子指令CAS注记
date: 2025-01-14 15:13:44
categories:
- [操作系统,组件,Lock]
tags:
- 操作系统
- CAS
description: "原子指令CAS"
---

## CAS原子指令

```c++ .{line-numbers}
//code segment
inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
```

上述代码片段采用了内联汇编的方式，以原子指令的方式实现了CAS。其伪代码如下:

```c .{line-numbers}
//code
if(*dest == compare_value){
    int temp = *dest;
    *dest = exchange_value;
    return temp;
}else{
    return *dest;
}
```

将dest指向的内容的值与compare_value比较，如果相等，dest赋值为exchange_value的值，并返回赋值前dest指向的内容；如果不等，不做任何操作，直接返回dest指向的内容。
