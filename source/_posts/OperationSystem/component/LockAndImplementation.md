---
title: Lock及其实现(一)
date: 2024-06-25 15:05:25
categories:
- [操作系统,组件,Lock]
tags:
- 操作系统
- Lock
description: "从锁的基本原理到实现方式"
---

## 基本原理

### 什么是锁

在并发编程中，存在着多个运行实体(进程或线程)同时访问一个资源的情况，如果该资源一次只允许一个进程使用，那我们称这种资源为临界资源，访问临界资源的代码段就叫临界区(Critical Section)。为了保证临界区的正常访问，我们需要某种机制保证一次只有一个进程进入临界区，这种机制就是锁。

锁通常由锁对象本身(Lock)和施加在锁上的加锁(tryLock)和解锁(releaseLock)操作构成，当多个进程访问临界区时，需要先实施加锁操作，加锁操作保证只有一个进程能获得锁，其余进程则阻塞在获取锁上，当进程执行完临界区代码，则执行解锁操作，锁释放之后则其余的阻塞进程中有一个能获取锁。

``` plaintext
//锁及其操作
Lock lock;

//加锁保证多个进程同时获取锁时只有一个进程能得到锁
void tryLock(lock);

//释放锁则可以让其余进程再次获取锁
void releaseLock(lock);
```

## 锁的实现

### 基础条件

在软件层面上，通过纯软件的方案实现锁是一件比较复杂的事情，所以目前主流的方案是通过硬件指令的支持来实现锁。在x86架构下，一些指令例如xchg等实现了原子操作，我们可以基于这些原子操作来实现锁。

### 工程实现

#### linux中的spinlock

##### spinlock结构

在linux6.0中，spinlock_t定义在*include/linux/spinlock_types.h*中，如下所示。

```c {.line-numbers}
//include/linux/spinlock_types.h
#ifndef CONFIG_PREEMPT_RT

/* Non PREEMPT_RT kernels map spinlock to raw_spinlock */
typedef struct spinlock {
 union {
  struct raw_spinlock rlock;

#ifdef CONFIG_DEBUG_LOCK_ALLOC
# define LOCK_PADSIZE (offsetof(struct raw_spinlock, dep_map))
  struct {
   u8 __padding[LOCK_PADSIZE];
   struct lockdep_map dep_map;
  };
#endif
 };
} spinlock_t;

#else /* !CONFIG_PREEMPT_RT */
//ignore preempt_rt case
#endif /* CONFIG_PREEMPT_RT */
```

宏定义CONFIG_PREEMPT_RT定义操作系统是否转换为实时操作系统(RTOS),这里我们不考虑实时操作系统，所以不考虑spinlock_t在RTOS下的实现,同时省略了spinlock_t在RTOS下的实现。CONFIG_DEBUG_LOCK_ALLOC是一个调试选项，一般在调试内核的时候开启，正式环境下一般都是关闭的，所以正式环境下spinlock_t等价于含有一个raw_spinlock类型的结构体变量。

raw_spinlock定义在*include/linux/spinlock_types_raw.h*中，

```c {.line-numbers}
//include/linux/spinlock_types_raw.h
typedef struct raw_spinlock {
  arch_spinlock_t raw_lock;
#ifdef CONFIG_DEBUG_SPINLOCK
  unsigned int magic, owner_cpu;
  void *owner;
#endif
#ifdef CONFIG_DEBUG_LOCK_ALLOC
  struct lockdep_map dep_map;
#endif
} raw_spinlock_t;
```

CONFIG_DEBUG_SPINLOCK是调试选项，默认为否，则raw_spinlock包含了一个类型为arch_spinlock_t类型的变量，该类型与cpu架构有关，这里我们考虑x86下该结构体的实现,可以看到，arch_spinlock_t是atomic_t类型的别名。

```c {.line-numbers}
//include/asm-generic/spinlock_types.h
typedef atomic_t arch_spinlock_t;
```

```plaintext
//struct spinlock_t
+------------------------------------+
|                                    |
|  raw_spinlock rlock                |----------------->+------------------------------+
+-(union)----------------------------+                  |                              |   [typedef]
| [cond]|                            |                  |  arch_spinlock_t raw_lock    |------------->+-------------------------------------+
| struct| u8 __padding[LOCK_PADSIZE] |                  +------------------------------+              |  typedef atomic_t arch_spinlock_t;  |
|       +----------------------------+                  |[cond]       int magic        |              +-------------------------------------+
|       |   lockdep_map dep_map      |                  +------------------------------+              
+------------------------------------+                  |[cond]       int owner_cpu    |
                                                        +------------------------------+
                                                        |[cond]   lockdep_map dep_map  |
                                                        +------------------------------+
```

spinlock内部包含了一个union，如果条件满足，spinlock内部包含一个raw_spinlock的结构体或者是一个匿名结构体，其内部包含一个数组和lockdep_map结构体。raw_spinlock包含一个架构相关的变量arch_spinlock_t和几个根据条件满足与否而包含的变量，在x86架构下，arch_spinlock_t就是一个atomic_t类型的别名。

##### spinlock的操作

###### spinlock初始化

spinlock由spin_lock_init宏定义初始化，宏定义如下：

```c {.line-numbers}
//include/linux/spinlock.h
# define spin_lock_init(_lock)   \
do {      \
  spinlock_check(_lock);   \
  *(_lock) = __SPIN_LOCK_UNLOCKED(_lock); \
} while (0)

/**spinlock_check实现 */
static __always_inline raw_spinlock_t *spinlock_check(spinlock_t *lock)
{
  return &lock->rlock;
}
```

spinlock_check用来检查_lock类型是否正确。__SPIN_LOCK_UNLOCKED宏定义则见下，扩展开来就是初始化_lock指针指向的spinlock_t结构体。

```c {.line-numbers}
//include/linux/spinlock_types.h
#define __SPIN_LOCK_UNLOCKED(lockname) \
  (spinlock_t) __SPIN_LOCK_INITIALIZER(lockname)

#define __SPIN_LOCK_INITIALIZER(lockname) \
  { { .rlock = ___SPIN_LOCK_INITIALIZER(lockname) } }

#define ___SPIN_LOCK_INITIALIZER(lockname)  \
  {          \
  .raw_lock = __ARCH_SPIN_LOCK_UNLOCKED,  \
  SPIN_DEBUG_INIT(lockname)    \
  SPIN_DEP_MAP_INIT(lockname) }

//under x86
#define __ARCH_SPIN_LOCK_UNLOCKED ATOMIC_INIT(0)
```

```plaintext
//
spin_lock_init() "include/linux/spinlock.h"
      |
      | [define] 
      o----->spinlock_check()
      |
      o----->__SPIN_LOCK_UNLOCKED()
                    |
                    | [define] 
                    o----->__SPIN_LOCK_INITIALIZER()
                                    |
                                    | [define] 
                                    o----->{ { .rlock = ___SPIN_LOCK_INITIALIZER(lockname) } }
```

通过宏定义展开，*(_lock)赋值如下，所以_lock是一个指向spinlock_t类型的指针，采用结构体字面量形式为该指针赋值，并初始化rlock.raw_lock值为0.

```c {.line-numbers}
*(_lock) = (spinlock_t) {
                {
                    .rlock = {
                        .raw_lock = ATOMIC_INIT(0),
                        SPIN_DEBUG_INIT(_lock)
                        SPIN_DEP_MAP_INIT(_lock)
                    }
                }
            }
```

总结下来，初始化操作就是初始化一个指向spinlock_t结构体的指针。

###### spinlock加锁操作

spinloc加锁函数调用spin_lock方法，该方法位于文件*/include/linux/spinlock.h*中。

```c {.line-numbers}
//file /include/linux/spinlock.h
static __always_inline void spin_lock(spinlock_t *lock)
{
  raw_spin_lock(&lock->rlock);
}

#define raw_spin_lock(lock) _raw_spin_lock(lock)
```

```plaintext
//spinlock
/**
*图示说明：<>表示条件；
*         <()>表示条件成立或者不成立，y表示成立，n表示不成立
*         [define] 表示源是宏定义
 */
spin_lock() "inlcude/linux/spinlock.h"
    |
    |
    o----->raw_spin_lock() "inlcude/linux/spinlock.h"
                |
                | [define]
                o----->_raw_spin_lock()
                            |
                            |
                            |<CONFIG_SMP || CONFIG_DEBUG_SPINLOCK>
       +-------------------<o>------------------------------------+
       |                                                          |
       | <(y)>                                                    |<(n)>
       |                                                          | [define]
       |                                                          o----->__LOCK() "include/linux/spinlock_api_up.h"
       |  
       | <CONFIG_INLINE_SPIN_LOCK>
 +----<o>     
 |     |      
 |     |<(y)> 
 |     | [define]     
 |     o----->__raw_spin_lock() "include/linux/spinlock_api_smp.h"
 |
 |<(n)>
 |
 o----->__raw_spin_lock() "kernel/locking/spinlock.c"

```

从上可以得知，根据配置的条件不同，spinlock加锁操作有不同的实现，第一个是根据系统架构是SMP还是UP，实现不同；第二个是根据是否配置CONFIG_INLINE_SPIN_LOCK，_raw_spin_lock判断是宏定义还是函数。这里我们只讨论SMP架构下spinlock的加锁操作，最终调用了__raw_spin_lock()方法。以下是__raw_spin_lock()方法的实现:

```c {.line-numbers}
//file:/include/linux/spinlock_api_smp.h
static inline void __raw_spin_lock(raw_spinlock_t *lock)
{
  preempt_disable();
  spin_acquire(&lock->dep_map, 0, 0, _RET_IP_);
  LOCK_CONTENDED(lock, do_raw_spin_trylock, do_raw_spin_lock);
}
```

- line4. preempt_disable()用来关闭抢占。
- line6. LOCK_CONTENDED是一个宏定义，根据是否配置CONFIG_LOCK_STAT定义有些区别，但核心定义如下：
``#define LOCK_CONTENDED(_lock, try, lock) lock(_lock)``
所以__raw_spin_lock()实际上调用了do_raw_spin_lock()方法。

do_raw_spin_lock()又调用了arch_spin_lock()方法，arch_spin_lock()是处理器相关的，不同的处理器架构实现不一样，这里我们考虑x86下的实现。

> x86下采用了一种ticket机制实现了相对公平的自旋锁，大致原理如下：锁维护了一个锁全局自增的原子变量ticket和锁变量，初始时两者值相同，每个线程想要获得锁之前都会去拿到一个ticket的值，拿到之后ticket自增。由于ticket原子自增，不同的线程拿到不同的ticket值，且先获取的ticket值小，后获取的ticket值大。这时如果锁变量的值与线程拿到的ticket值相同，则线程获取锁，反之则等待；当线程释放锁时，锁变量值自增，这样拿到与锁变量相同的ticket的线程就能够运行，如此保证了线程按拿到ticket的顺序执行。整个原理与日常叫号排队类似，办事时先每人拿一个号，如果叫到的号是你手上的号，那就轮到你了。

具体到实现这里，ticket和锁变量存放在一个u32变量里，ticket取高16位，锁变量取低16位。

- line11. atomic_fetch_add是原子操作，该操作将lock的值加上第一个参数，并返回lock原来的值。
- line23. atomic_cond_read_acquire是宏定义，通过一系列架构相关的判断，核心功能在smp_cond_load_relaxed宏定义中实现，在这里我们可以看到循环操作。
- line42. spin_lock的核心，spin_lock自旋本质上就是循环等待条件为真，这里就是等待ticket和lock低16位值相等。
- line46. 由于自旋的忙等待机制，如果锁的竞争比较大，将会浪费很多cpu时间，所以这里采用了cpu_relax()方法，这是一种让出cpu的机制，视具体cpu架构不同，实现不同。

```c {.line-numbers}
//include/linux/spinlock.h
static inline void do_raw_spin_lock(raw_spinlock_t *lock) __acquires(lock)
{
  __acquire(lock);
  arch_spin_lock(&lock->raw_lock);
  mmiowb_spin_lock();
}

static __always_inline void arch_spin_lock(arch_spinlock_t *lock)
{
  u32 val = atomic_fetch_add(1<<16, lock);
  u16 ticket = val >> 16; 
  if (ticket == (u16)val)
    return; 
  /*
   * atomic_cond_read_acquire() is RCpc, but rather than defining a
   * custom cond_read_rcsc() here we just emit a full fence.  We only
   * need the prior reads before subsequent writes ordering from
   * smb_mb(), but as atomic_cond_read_acquire() just emits reads and we
   * have no outstanding writes due to the atomic_fetch_add() the extra
   * orderings are free.
   */
  atomic_cond_read_acquire(lock, ticket == (u16)VAL);
  smp_mb();
}

#define atomic_cond_read_acquire(v, c) smp_cond_load_acquire(&(v)->counter, (c))

#ifndef smp_cond_load_acquire
#define smp_cond_load_acquire(ptr, cond_expr) ({  \
  __unqual_scalar_typeof(*ptr) _val;   \
  _val = smp_cond_load_relaxed(ptr, cond_expr);  \
  smp_acquire__after_ctrl_dep();    \
  (typeof(*ptr))_val;     \
})
#endif

#ifndef smp_cond_load_relaxed
#define smp_cond_load_relaxed(ptr, cond_expr) ({    \
  typeof(ptr) __PTR = (ptr);        \
  __unqual_scalar_typeof(*ptr) VAL;      \
  for (;;) {            \
    VAL = READ_ONCE(*__PTR);      \
    if (cond_expr)          \
      break;          \
    cpu_relax();          \
  }              \
  (typeof(*ptr))VAL;          \
})
#endif
```

###### spinlock释放锁操作

同加锁一样，spinlock的释放锁操作如下：

```c {.line-numbers}
//include/linux/spinlock.h
static __always_inline void spin_unlock(spinlock_t *lock)
{
  raw_spin_unlock(&lock->rlock);
}
```

具体的调用流程如下：

```plaintext
//释放锁操作流程
spin_unlock() "include/linux/spinlock.h"
      |
      |
      o----->raw_spin_unlock() "include/linux/spinlock.h"
                    |
                    |
                    | [define]
                    o----->_raw_spin_unlock() "include/linux/spinlock.h"
                                    |
                                    |
                                    |<CONFIG_SMP || CONFIG_DEBUG_SPINLOCK>
       +---------------------------<x>----------------------------+
       |                                                          |
       | <(y)>                                                    |<(n)>
       |                                                          | [define]
       | <CONFIG_UNINLINE_SPIN_UNLOCK>                            o----->__UNLOCK "include/linux/spinlock_api_up.h"
  +---<x>-------+
  |             |
  |             |<(y)>
  |             o----->__raw_spin_unlock() "kernel/locking/spinlock.c"
  |<(n)>                        |
  | [define]                    |
  |                             +----------------------------------------+
  |                                                                      |
  o----->__raw_spin_unlock() "include/linux/spinlock_api_smp.h"          |
                 |                                                       |
                 |                                                       |
                 +-------------------------------------------------------+
                 |
                 o----->spin_release()
                 |
                 o----->do_raw_spin_unlock() "include/linux/spinlock.h"
                 |               |
                 |               |
                 |               +---------------+
                 |                               |
                 |                               o----->mmiowb_spin_unlock() 
                 |                               |
                 |                               o----->arch_spin_unlock()
                 |                               |
                 |                               o----->__release()
                 |
                 o----->preempt_enable()
```

和加锁操作类似，释放锁也根据CONFIG_SMP和CONFIG_DEBUG_SPINLOCK配置不同而实现不同，这里我们只考虑在SMP架构下的实现。通过调用链以及宏定义spin_unlock()->raw_spin_unlock()->_raw_spin_unlock()->__raw_spin_unlock()->do_raw_spin_unlock()->arch_spin_unlock(),释放锁最终调用了架构相关的方法arch_spin_unlock()。我们考虑x86下arch_spin_unlock()方法的实现:

```c {.line-numbers}
//include/asm-generic/spinlock.h
static __always_inline void arch_spin_unlock(arch_spinlock_t *lock)
{
  u16 *ptr = (u16 *)lock + IS_ENABLED(CONFIG_CPU_BIG_ENDIAN);
  u32 val = atomic_read(lock);

  smp_store_release(ptr, (u16)val + 1);
}
```

释放锁的操作就是将锁变量的值加1，与加锁操作时采用ticket机制对应上。锁变量加1，持有该锁变量值的ticket的线程将得到运行。

###### 总结

linux中的spinlock遵循了自旋锁的机制，在公平性上引入了ticket机制，同时通过cpu_relax()等策略尽可能防止cpu空转。
