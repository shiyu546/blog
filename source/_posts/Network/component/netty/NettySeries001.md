---
title: Netty系列01 Future和Executor
date: 2024-10-30 09:27:00
mathjax: true
categories:
- [网络编程,netty]
tags:
- 原理
- 实现
description: "netty系列基础部分：Future和Executor <br>
- 描述了Future和Executor接口以及部分实现"
---

## 背景

Netty基于java NIO构建了异步事件驱动的网络框架，要了解netty除了NIO和事件模型之外，还需要了解一些java基础功能，这些功能大部分由concurrent包提供，用来解决一些基础的并发编程问题。下面逐步介绍这些功能以期对netty能有一个更好的理解。

## 功能

### Executor接口

Executor接口代码如下:

```JAVA .{line-numbers}
//source file
public interface Executor {
    void execute(Runnable command);
}
```

Executor提供了一种执行任务的机制，将任务的提交与执行解耦。execute()的执行是同步还是异步取决于Executor的实现：

```JAVA .{line-numbers}
//source file
 class DirectExecutor implements Executor {  
    public void execute(Runnable r) {
        //在当前线程执行  
        r.run();    
    }
 }

 class ThreadPerTaskExecutor implements Executor {    
    public void execute(Runnable r) {
        //在新线程中执行
        new Thread(r).start();    
    } 
 }
```

### Future接口

Future表示异步执行的结果，初看起来这句话比较难以理解，考虑下面的场景：
> 现在我们执行一个任务，任务共分为A，B，C，D，E五个步骤，按照A->B->C->D->E的顺序执行，如果要提高任务的执行效率，第一个想到的是将任务并行执行，即A，B，C，D，E同时执行，但假如任务间存在依赖关系，假如D依赖B的执行结果，那我们必须拿到B的结果后再执行D；再假如B在线程t执行，D在主线程执行，则D在主线程中要等待B执行的结果。这种等待其它线程的执行结果就是等待异步执行的结果,也就是D等待的东西(对象)，用Future表示。

```JAVA .{line-numbers}
//source file
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

Future提供了三类方法，一类是get()方法用于获取异步执行的结果，当任务没有执行完成时会阻塞当前线程；第二类是判断任务的状态，包括isDone()和isCancelled()方法；第三类是取消任务执行的cancel()方法。

### ExecutorService接口

由于Executor只提供了execute()方法，无法对任务做更多的控制，所以jdk提供了Executor的实现ExecutorService。

```JAVA .{line-numbers}
//source file
public interface ExecutorService extends Executor {
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> Future<T> submit(Callable<T> task);
    <T> Future<T> submit(Runnable task, T result);
    Future<?> submit(Runnable task);
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```

9-11行ExecutorService提供了submit()方法，该方法接收一个task并返回一个Future，为异步执行提供了基础。ExecutorService还提供了shutdown()方法和一些task状态判断方法，丰富了Executor的功能。

### FutureTask实现

了解了Future接口的功能之后，再来看Future是怎么实现的，这里我们选取了Future的实现FutureTask来分析，先看FutureTask的类图:
{% asset_img nettyseries001_001.jpg 摆放方式 %}
FutureTask同时实现了Runnable和Future接口，所以FutureTask既可以当作task投入Executor，也可以当作异步等待结果对象，获取任务的执行结果。

Future接受一个Callable作为构造函数参数，该参数是具体的业务执行逻辑所在。FutureTask还提供了state字段，用来标记task的状态，state共有7种取值，
>NEW          = 0
COMPLETING   = 1
NORMAL       = 2
EXCEPTIONAL  = 3
CANCELLED    = 4
INTERRUPTING = 5
INTERRUPTED  = 6

当创建FutureTask对象时，state会初始化为NEW状态，表示任务创建。任务开始执行时，会调用run()方法，run()方法先判断state的状态，并调用Callable执行具体的逻辑并获取结果，然后设置返回结果。

```JAVA .{line-numbers}
//source file
public void run() {
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }

    protected void set(V v) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = v;
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
            finishCompletion();
        }
    }

private volatile Thread runner;

private Object outcome;
```

1. 3-5行 判断了state的状态是否为NEW，并通过一个原子操作判断runner(Thread对象)是否为null，为null则设置runner的值为当前线程，原子操作的代码操作含义如下代码片段,因返回值判断了runner是否为null，所以这里实际上是判断了FutureTask的状态，初始状态下，state=NEW,runner=null,通过该操作将runner设置为当前线程。

> ```JAVA .{line-numbers}
> //code segment
> if(runner==null){
>     runner=Thread.currentThread();
> }
> return (runner==null);
> ```

2. 8-19行 调用callable执行了具体的任务逻辑，如果执行成功，将ran设置为true。
3. 35-41行 set()方法，先原子的判断state是否等于NEW，等于则设置为COMPLETING，表示任务已经执行完，但outcome的值还没设置。然后设置outcome的值，并在38行将state的值设置为NORMAL。

> compareAndSwapInt和putOrderedInt都是UNSAFE中的方法，都是native方法，表示实现与操作系统有关，两者更新对象属性值时都有一个offset参数，表示属性在对象中的偏移量。

接下来调用了finishCompletion()方法。该方法获取FutureTask中的waiters属性，waiters是一个链表，每个节点表示一个阻塞的线程，通过for循环从节点q中获取等待的线程，并调用`LockSupport.unpark(t)`唤醒该线程。

```JAVA .{line-numbers}
//code segment
static final class WaitNode {
        volatile Thread thread;
        volatile WaitNode next;
        WaitNode() { thread = Thread.currentThread(); }
}
private volatile WaitNode waiters;

private void finishCompletion() {
        // assert state > COMPLETING;
        for (WaitNode q; (q = waiters) != null;) {
//通过原子的设置waiters为null，只允许一个线程进入if方法
            if (UNSAFE.compareAndSwapObject(this, waitersOffset, q, null)) {
                for (;;) {
                    Thread t = q.thread;
                    if (t != null) {
                        q.thread = null;
//unpark()与park()成对出现，前者唤醒线程，后者则阻塞等待，直到被唤醒。
                        LockSupport.unpark(t);
                    }
                    WaitNode next = q.next;
                    if (next == null)
                        break;
                    q.next = null; // unlink to help gc
//链式调用，如果q有后继节点，不断的将q设置为后继节点，遍历阻塞在该链表上的线程列表
                    q = next;
                }
                break;
            }
        }

        done();

        callable = null;        // to reduce footprint
    }
```

从上述FutureTask的run()方法执行可以看出，task执行会将结果保存到FutureTask的outcome属性中，并唤醒阻塞在链表上的线程。大致猜测get的操作包括判断outcome的值，并根据值是否就绪阻塞在链表上。下面是get()方法的实现：

```JAVA .{line-numbers}
//source file
public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }

private int awaitDone(boolean timed, long nanos)
        throws InterruptedException {
        final long deadline = timed ? System.nanoTime() + nanos : 0L;
        WaitNode q = null;
        boolean queued = false;
        for (;;) {
            if (Thread.interrupted()) {
                removeWaiter(q);
                throw new InterruptedException();
            }

            int s = state;
            if (s > COMPLETING) {
                if (q != null)
                    q.thread = null;
                return s;
            }
            else if (s == COMPLETING) // cannot time out yet
                Thread.yield();
            else if (q == null)
                q = new WaitNode();
            else if (!queued)
                queued = UNSAFE.compareAndSwapObject(this, waitersOffset,
                                                     q.next = waiters, q);
            else if (timed) {
                nanos = deadline - System.nanoTime();
                if (nanos <= 0L) {
                    removeWaiter(q);
                    return state;
                }
                LockSupport.parkNanos(this, nanos);
            }
            else
                LockSupport.park(this);
        }
    }
```

1. 4行 判断state的值，`s <= COMPLETING`表示s处于NEW或COMPLETING状态，这两种状态下task执行结果都没有保存到outcome中，执行awaitDone()方法。
2. awaitDone()主体由一个无限for循环中的多个if-else构成，分析state分别在NEW，COMPLETING和>COMPLETING状态下的执行流：
   + state为NEW时，执行到第三个if，新建WaitNode q；再次循环执行到第四个if，采用cas操作将q插入到waiters链表的表头；再次循环则执行else操作`LockSupport.park(this)`阻塞。
   + state为COMPLETING时，执行第二个if，此时结果已计算完毕但还没有赋值，直接让当前线程yield(),下次再试。
   + state>COMPLETING时,表示结果已经执行并赋值(无论是正常还是中断异常)，执行第一个if返回state。
3. 接下来调用report()方法获取值，正常获取则返回值，异常则抛出异常。

> ```JAVA .{line-numbers}
> //code segment
> private V report(int s) throws ExecutionException {
>         Object x = outcome;
>         if (s == NORMAL)
>             return (V)x;
>         if (s >= CANCELLED)
>             throw new CancellationException();
>         throw new ExecutionException((Throwable)x);
>     }
> ```

#### 线程的阻塞等待与唤醒

FutureTask中实现异步等待执行结果的关键是线程阻塞在等待结果这个条件上，并采用LockSupport类的park()与unpark()成对使用来阻塞与唤醒线程。两个方法的实现如下:

```JAVA .{line-numbers}
//source file
public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }

public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }

private static void setBlocker(Thread t, Object arg) {
        // Even though volatile, hotspot doesn't need a write barrier here.
        UNSAFE.putObject(t, parkBlockerOffset, arg);
    }

class Thread implements Runnable {
    volatile Object parkBlocker;
}   
```

park()方法设置了线程对象的parkBlocker为调用对象，并在阻塞唤醒时清除parkBlocker的值，具体的阻塞逻辑落在了UNSAFE.park()方法上，同时看unpark()方法，调用了UNSAFE.unpark()方法，线程阻塞和唤醒的逻辑最终落在了UNSAFE类的park()和unpark()方法中。

这两个方法都是native方法，具体实现与操作系统有关(相关代码在虚拟机中)，在linux中，相关的实现如下,省去一些细节后，park本质上采用的是pthread的pthread_cond_wait阻塞在指定的条件变量上，而unpark则调用pthread_cond_signal唤醒指定的条件变量。

```cpp .{line-number}
void os::PlatformEvent::park() {       // AKA "down()"
//...抛去了一些细节，留下主体部分
  if (v == 0) {
//_cond条件变量，提供了一种机制让线程等待指定的事件发生
     int status = pthread_mutex_lock(_mutex);
     while (_Event < 0) {
        status = pthread_cond_wait(_cond, _mutex);
     }
     status = pthread_mutex_unlock(_mutex);
  }
}

void os::PlatformEvent::unpark() {
//...
  int status = pthread_mutex_lock(_mutex);
  int AnyWaiters = _nParked;
  if (AnyWaiters != 0 && WorkAroundNPTLTimedWaitHang) {
    AnyWaiters = 0;
    pthread_cond_signal(_cond);
  }
  status = pthread_mutex_unlock(_mutex);
}
```

#### FutureTask Demo

当FutureTask执行run方法时，会判断state的状态值，所以即使同一个futureTask并行执行多次，也只有一个会执行，但get方法可以使线程阻塞在waiters链表上，所以可以多个线程等待同一个task的执行结果。

>这里的例子展示了多个线程阻塞在get方法上，主线程中执行run方法唤醒阻塞的线程。
>
> ```JAVA .{line-numbers}
> public static void main(String[] args) throws ExecutionException, InterruptedException {
>         MyFutureTask task = new MyFutureTask(new Callable() {
>             @Override
>             public Object call() throws Exception {
>                 System.out.println("线程" + Thread.currentThread().getName() + "执行");
>                 Thread.sleep(1000);
>                 return Thread.currentThread().getName();
>             }
>         });
>         ExecutorService executor = new ThreadPoolExecutor(10, 10,
>                 60, TimeUnit.SECONDS, new LinkedBlockingQueue<>());
>         for (int i = 0; i < 5; i++) {
>             executor.execute(new MyRunnable(task));
>         }
>         Thread.sleep(5000);
>         task.run();
>         System.out.println("主线程执行返回");
>         executor.shutdown();
>     }
> 
>     public static class MyRunnable implements Runnable {
>         private MyFutureTask task;
> 
>         public MyRunnable(MyFutureTask task) {
>             this.task = task;
>         }
> 
>         @Override
>         public void run() {
>             try {
>                 String name = (String) task.get();
>                 System.out.println("线程" + Thread.currentThread().getName() + "获取到了执行结果" + name);
>             } catch (InterruptedException e) {
>                 throw new RuntimeException(e);
>             } catch (ExecutionException e) {
>                 throw new RuntimeException(e);
>             }
> 
>         }
>     }
> ```
>
> 执行结果如下:
>> 线程main执行
>> pool-1-thread-5->pool-1-thread-3->pool-1-thread-2->pool-1-thread-4->pool-1-thread-1->
>> 线程pool-1-thread-1获取到了执行结果main
>> 主线程执行返回
>> 线程pool-1-thread-2获取到了执行结果main
>> 线程pool-1-thread-4获取到了执行结果main
>> 线程pool-1-thread-5获取到了执行结果main
>> 线程pool-1-thread-3获取到了执行结果main
> 第二行输出了waiters链表的节点顺序，从中可以看到线程等待的入栈顺序。

## 小结

concurrent包中提供了同步工具包，各种工具不仅构成了java同步的基础，也为netty实现提供了工具类。这里主要介绍了Future接口和Executor接口，以及分析Future的实现FutureTask。接下来我们将了解netty中是如何使用这些类的。
