# Java并发包中的同步队列SynchronousQueue实现原理[转载](http://ifeve.com/java-synchronousqueue/)
作者：一粟

# 介绍
Java 6的并发编程包中的SynchronousQueue是一个没有数据缓冲的BlockingQueue，生产者线程对其的插入操作put必须等待消费者的移除操作take，反过来也一样。

不像ArrayBlockingQueue或LinkedListBlockingQueue，**SynchronousQueue内部并没有数据缓存空间**，你不能调用peek()方法来看队列中是否有数据元素，因为数据元素只有当你试着取走的时候才可能存在，不取走而只想偷窥一下是不行的，当然遍历这个队列的操作也是不允许的。**队列头元素是第一个排队要插入数据的线程，而不是要交换的数据**。数据是在配对的生产者和消费者线程之间直接传递的，并不会将数据缓冲数据到队列中。**可以这样来理解：生产者和消费者互相等待对方，握手，然后一起离开**。

# SynchronousQueue的一个使用场景是在线程池里。
Executors.newCachedThreadPool()就使用了SynchronousQueue，这个线程池根据需要（新任务到来时）创建新的线程，如果有空闲线程则会重复使用，线程空闲了60秒后会被回收。

# 实现原理
阻塞队列的实现方法有许多：

## 阻塞算法实现
阻塞算法实现通常在内部采用一个锁来保证多个线程中的put()和take()方法是串行执行的。采用锁的开销是比较大的，还会存在一种情况是线程A持有线程B需要的锁，B必须一直等待A释放锁，即使A可能一段时间内因为B的优先级比较高而得不到时间片运行。所以在高性能的应用中我们常常希望规避锁的使用。
```java
public class NativeSynchronousQueue<E> {
    boolean putting = false;
    E item = null;

    public synchronized E take() throws InterruptedException {

        while (item == null)
            wait();
        E e = item;
        item = null;
        notifyAll();
        return e;
    }

    public synchronized void put(E e) throws InterruptedException {
        if (e==null) 
            return;
        while (putting)
            wait();
        putting = true;
        item = e;
        notifyAll();
        while (item!=null)
            wait();
        putting = false;
        notifyAll();
    }
}
```
## 信号量实现
经典同步队列实现采用了三个信号量，代码很简单，比较容易理解：
```java
public class SemaphoreSynchronousQueue<E> {
    E item = null;
    Semaphore sync = new Semaphore(0);
    Semaphore send = new Semaphore(1);
    Semaphore recv = new Semaphore(0);
 
    public E take() throws InterruptedException {
        recv.acquire();
        E x = item;
        sync.release();
        send.release();
        return x;
    }

    public void put (E x) throws InterruptedException{
        send.acquire();
        item = x;
        recv.release();
        sync.acquire();
    }
}
```
在多核机器上，上面方法的同步代价仍然较高，操作系统调度器需要上千个时间片来阻塞或唤醒线程，而上面的实现即使在生产者put()时已经有一个消费者在等待的情况下，阻塞和唤醒的调用仍然需要。

## Java 5实现
```java
public class Java5SynchronousQueue<E> {
    ReentrantLock qlock = new ReentrantLock();
    Queue waitingProducers = new Queue();
    Queue waitingConsumers = new Queue();

    static class Node extends AbstractQueuedSynchronizer {
        E item;
        Node next;
        Node(Object x) { item = x; }
        void waitForTake() { /* (uses AQS) */ }
           E waitForPut() { /* (uses AQS) */ }
    }


    public E take() {
        Node node;
        boolean mustWait;
        qlock.lock();
        node = waitingProducers.pop();
        if(mustWait = (node == null))
           node = waitingConsumers.push(null);
         qlock.unlock();

        if (mustWait)
           return node.waitForPut();
        else
            return node.item;
    }
 
    public void put(E e) {
         Node node;
         boolean mustWait;
         qlock.lock();
         node = waitingConsumers.pop();
         if (mustWait = (node == null))
             node = waitingProducers.push(e);
         qlock.unlock();

         if (mustWait)
             node.waitForTake();
         else
            node.item = e;
    }
}
```
Java 5的实现相对来说做了一些优化，只使用了一个锁，使用队列代替信号量也可以允许发布者直接发布数据，而不是要首先从阻塞在信号量处被唤醒。

## Java6实现
Java 6的SynchronousQueue的实现采用了一种性能更好的无锁算法 — 扩展的“Dual stack and Dual queue”算法。性能比Java5的实现有较大提升。竞争机制支持公平和非公平两种：非公平竞争模式使用的数据结构是后进先出栈(Lifo Stack)；公平竞争模式则使用先进先出队列（Fifo Queue），性能上两者是相当的，一般情况下，Fifo通常可以支持更大的吞吐量，但Lifo可以更大程度的保持线程的本地化。

代码实现里的Dual Queue或Stack内部是用链表(LinkedList)来实现的，其节点状态为以下三种情况：

持有数据 – put()方法的元素
持有请求 – take()方法
空
这个算法的特点就是任何操作都可以根据节点的状态判断执行，而不需要用到锁。

其核心接口是Transfer，生产者的put或消费者的take都使用这个接口，根据第一个参数来区别是入列（栈）还是出列（栈）。

01
/**
02
    * Shared internal API for dual stacks and queues.
03
    */
04
   static abstract class Transferer {
05
       /**
06
        * Performs a put or take.
07
        *
08
        * @param e if non-null, the item to be handed to a consumer;
09
        *          if null, requests that transfer return an item
10
        *          offered by producer.
11
        * @param timed if this operation should timeout
12
        * @param nanos the timeout, in nanoseconds
13
        * @return if non-null, the item provided or received; if null,
14
        *         the operation failed due to timeout or interrupt --
15
        *         the caller can distinguish which of these occurred
16
        *         by checking Thread.interrupted.
17
        */
18
       abstract Object transfer(Object e, boolean timed, long nanos);
19
   }
TransferQueue实现如下(摘自Java 6源代码)，入列和出列都基于Spin和CAS方法：

view sourceprint?
01
/**
02
    * Puts or takes an item.
03
    */
04
   Object transfer(Object e, boolean timed, long nanos) {
05
       /* Basic algorithm is to loop trying to take either of
06
        * two actions:
07
        *
08
        * 1. If queue apparently empty or holding same-mode nodes,
09
        *    try to add node to queue of waiters, wait to be
10
        *    fulfilled (or cancelled) and return matching item.
11
        *
12
        * 2. If queue apparently contains waiting items, and this
13
        *    call is of complementary mode, try to fulfill by CAS'ing
14
        *    item field of waiting node and dequeuing it, and then
15
        *    returning matching item.
16
        *
17
        * In each case, along the way, check for and try to help
18
        * advance head and tail on behalf of other stalled/slow
19
        * threads.
20
        *
21
        * The loop starts off with a null check guarding against
22
        * seeing uninitialized head or tail values. This never
23
        * happens in current SynchronousQueue, but could if
24
        * callers held non-volatile/final ref to the
25
        * transferer. The check is here anyway because it places
26
        * null checks at top of loop, which is usually faster
27
        * than having them implicitly interspersed.
28
        */
29
 
30
       QNode s = null; // constructed/reused as needed
31
       boolean isData = (e != null);
32
 
33
       for (;;) {
34
           QNode t = tail;
35
           QNode h = head;
36
           if (t == null || h == null)         // saw uninitialized value
37
               continue;                       // spin
38
 
39
           if (h == t || t.isData == isData) { // empty or same-mode
40
               QNode tn = t.next;
41
               if (t != tail)                  // inconsistent read
42
                   continue;
43
               if (tn != null) {               // lagging tail
44
                   advanceTail(t, tn);
45
                   continue;
46
               }
47
               if (timed &amp;&amp; nanos &lt;= 0)        // can't wait
48
                   return null;
49
               if (s == null)
50
                   s = new QNode(e, isData);
51
               if (!t.casNext(null, s))        // failed to link in
52
                   continue;
53
 
54
               advanceTail(t, s);              // swing tail and wait
55
               Object x = awaitFulfill(s, e, timed, nanos);
56
               if (x == s) {                   // wait was cancelled
57
                   clean(t, s);
58
                   return null;
59
               }
60
 
61
               if (!s.isOffList()) {           // not already unlinked
62
                   advanceHead(t, s);          // unlink if head
63
                   if (x != null)              // and forget fields
64
                       s.item = s;
65
                   s.waiter = null;
66
               }
67
               return (x != null)? x : e;
68
 
69
           } else {                            // complementary-mode
70
               QNode m = h.next;               // node to fulfill
71
               if (t != tail || m == null || h != head)
72
                   continue;                   // inconsistent read
73
 
74
               Object x = m.item;
75
               if (isData == (x != null) ||    // m already fulfilled
76
                   x == m ||                   // m cancelled
77
                   !m.casItem(x, e)) {         // lost CAS
78
                   advanceHead(h, m);          // dequeue and retry
79
                   continue;
80
               }
81
 
82
               advanceHead(h, m);              // successfully fulfilled
83
               LockSupport.unpark(m.waiter);
84
               return (x != null)? x : e;
85
           }
86
       }
87
   }
参考文章
Javadoc of SynchronousQueue
Scalable Synchronous Queues
Nonblocking Concurrent Data Structures with Condition Synchronization
原创文章，转载请注明： 转载自并发编程网 – ifeve.com本文链接地址: Java并发包中的同步队列SynchronousQueue实现原理