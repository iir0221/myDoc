# java.util.concurrent中的超时
提供了两种处理超时的方法
## 异常通知超时
java.util.concurrent.Future
```java
/**
    * Waits if necessary for the computation to complete, and then
    * retrieves its result.
    *
    * @return the computed result
    * @throws CancellationException if the computation was cancelled
    * @throws ExecutionException if the computation threw an
    * exception
    * @throws InterruptedException if the current thread was interrupted
    * while waiting
    */
V get() throws InterruptedException, ExecutionException;
```

java.util.concurrentExchanger
```java
/**
    * Waits for another thread to arrive at this exchange point (unless
    * the current thread is {@linkplain Thread#interrupt interrupted}),
    * and then transfers the given object to it, receiving its object
    * in return.
    *
    * <p>If another thread is already waiting at the exchange point then
    * it is resumed for thread scheduling purposes and receives the object
    * passed in by the current thread.  The current thread returns immediately,
    * receiving the object passed to the exchange by that other thread.
    *
    * <p>If no other thread is already waiting at the exchange then the
    * current thread is disabled for thread scheduling purposes and lies
    * dormant until one of two things happens:
    * <ul>
    * <li>Some other thread enters the exchange; or
    * <li>Some other thread {@linkplain Thread#interrupt interrupts}
    * the current thread.
    * </ul>
    * <p>If the current thread:
    * <ul>
    * <li>has its interrupted status set on entry to this method; or
    * <li>is {@linkplain Thread#interrupt interrupted} while waiting
    * for the exchange,
    * </ul>
    * then {@link InterruptedException} is thrown and the current thread's
    * interrupted status is cleared.
    *
    * @param x the object to exchange
    * @return the object provided by the other thread
    * @throws InterruptedException if the current thread was
    *         interrupted while waiting
    */
@SuppressWarnings("unchecked")
public V exchange(V x) throws InterruptedException {
    Object v;
    Object item = (x == null) ? NULL_ITEM : x; // translate null args
    if ((arena != null ||
            (v = slotExchange(item, false, 0L)) == null) &&
        ((Thread.interrupted() || // disambiguates null return
            (v = arenaExchange(item, false, 0L)) == null)))
        throw new InterruptedException();
    return (v == NULL_ITEM) ? null : (V)v;
}
```
java.util.concurrent.CyclicBarrier 
```java
/**
    * Waits until all {@linkplain #getParties parties} have invoked
    * {@code await} on this barrier.
    *
    * <p>If the current thread is not the last to arrive then it is
    * disabled for thread scheduling purposes and lies dormant until
    * one of the following things happens:
    * <ul>
    * <li>The last thread arrives; or
    * <li>Some other thread {@linkplain Thread#interrupt interrupts}
    * the current thread; or
    * <li>Some other thread {@linkplain Thread#interrupt interrupts}
    * one of the other waiting threads; or
    * <li>Some other thread times out while waiting for barrier; or
    * <li>Some other thread invokes {@link #reset} on this barrier.
    * </ul>
    *
    * <p>If the current thread:
    * <ul>
    * <li>has its interrupted status set on entry to this method; or
    * <li>is {@linkplain Thread#interrupt interrupted} while waiting
    * </ul>
    * then {@link InterruptedException} is thrown and the current thread's
    * interrupted status is cleared.
    *
    * <p>If the barrier is {@link #reset} while any thread is waiting,
    * or if the barrier {@linkplain #isBroken is broken} when
    * {@code await} is invoked, or while any thread is waiting, then
    * {@link BrokenBarrierException} is thrown.
    *
    * <p>If any thread is {@linkplain Thread#interrupt interrupted} while waiting,
    * then all other waiting threads will throw
    * {@link BrokenBarrierException} and the barrier is placed in the broken
    * state.
    *
    * <p>If the current thread is the last thread to arrive, and a
    * non-null barrier action was supplied in the constructor, then the
    * current thread runs the action before allowing the other threads to
    * continue.
    * If an exception occurs during the barrier action then that exception
    * will be propagated in the current thread and the barrier is placed in
    * the broken state.
    *
    * @return the arrival index of the current thread, where index
    *         {@code getParties() - 1} indicates the first
    *         to arrive and zero indicates the last to arrive
    * @throws InterruptedException if the current thread was interrupted
    *         while waiting
    * @throws BrokenBarrierException if <em>another</em> thread was
    *         interrupted or timed out while the current thread was
    *         waiting, or the barrier was reset, or the barrier was
    *         broken when {@code await} was called, or the barrier
    *         action (if present) failed due to an exception
    */
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```
java.util.concurrent.CountDownLatch
```java
/**
* Causes the current thread to wait until the latch has counted down to
* zero, unless the thread is {@linkplain Thread#interrupt interrupted}.
*
* <p>If the current count is zero then this method returns immediately.
*
* <p>If the current count is greater than zero then the current
* thread becomes disabled for thread scheduling purposes and lies
* dormant until one of two things happen:
* <ul>
* <li>The count reaches zero due to invocations of the
* {@link #countDown} method; or
* <li>Some other thread {@linkplain Thread#interrupt interrupts}
* the current thread.
* </ul>
*
* <p>If the current thread:
* <ul>
* <li>has its interrupted status set on entry to this method; or
* <li>is {@linkplain Thread#interrupt interrupted} while waiting,
* </ul>
* then {@link InterruptedException} is thrown and the current thread's
* interrupted status is cleared.
*
* @throws InterruptedException if the current thread is interrupted
*         while waiting
*/
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
## 返回值通知超时
java.util.concurrent.BlockingQueue
```java
/**
    * Inserts the specified element into this queue if it is possible to do
    * so immediately without violating capacity restrictions, returning
    * {@code true} upon success and {@code false} if no space is currently
    * available.  When using a capacity-restricted queue, this method is
    * generally preferable to {@link #add}, which can fail to insert an
    * element only by throwing an exception.
    *
    * @param e the element to add
    * @return {@code true} if the element was added to this queue, else
    *         {@code false}
    * @throws ClassCastException if the class of the specified element
    *         prevents it from being added to this queue
    * @throws NullPointerException if the specified element is null
    * @throws IllegalArgumentException if some property of the specified
    *         element prevents it from being added to this queue
    */
boolean offer(E e);

/**
    * Retrieves and removes the head of this queue, waiting up to the
    * specified wait time if necessary for an element to become available.
    *
    * @param timeout how long to wait before giving up, in units of
    *        {@code unit}
    * @param unit a {@code TimeUnit} determining how to interpret the
    *        {@code timeout} parameter
    * @return the head of this queue, or {@code null} if the
    *         specified waiting time elapses before an element is available
    * @throws InterruptedException if interrupted while waiting
    */
E poll(long timeout, TimeUnit unit)
    throws InterruptedException;
```

java.util.concurrent.Semaphore
```java
/**
    * Acquires a permit from this semaphore, only if one is available at the
    * time of invocation.
    *
    * <p>Acquires a permit, if one is available and returns immediately,
    * with the value {@code true},
    * reducing the number of available permits by one.
    *
    * <p>If no permit is available then this method will return
    * immediately with the value {@code false}.
    *
    * <p>Even when this semaphore has been set to use a
    * fair ordering policy, a call to {@code tryAcquire()} <em>will</em>
    * immediately acquire a permit if one is available, whether or not
    * other threads are currently waiting.
    * This &quot;barging&quot; behavior can be useful in certain
    * circumstances, even though it breaks fairness. If you want to honor
    * the fairness setting, then use
    * {@link #tryAcquire(long, TimeUnit) tryAcquire(0, TimeUnit.SECONDS) }
    * which is almost equivalent (it also detects interruption).
    *
    * @return {@code true} if a permit was acquired and {@code false}
    *         otherwise
    */
public boolean tryAcquire() {
    return sync.nonfairTryAcquireShared(1) >= 0;
}
```
java.util.concurrent.locks.Lock 

```java
/**
    * Acquires the lock only if it is free at the time of invocation.
    *
    * <p>Acquires the lock if it is available and returns immediately
    * with the value {@code true}.
    * If the lock is not available then this method will return
    * immediately with the value {@code false}.
    *
    * <p>A typical usage idiom for this method would be:
    *  <pre> {@code
    * Lock lock = ...;
    * if (lock.tryLock()) {
    *   try {
    *     // manipulate protected state
    *   } finally {
    *     lock.unlock();
    *   }
    * } else {
    *   // perform alternative actions
    * }}</pre>
    *
    * This usage ensures that the lock is unlocked if it was acquired, and
    * doesn't try to unlock if the lock was not acquired.
    *
    * @return {@code true} if the lock was acquired and
    *         {@code false} otherwise
    */
boolean tryLock();
```