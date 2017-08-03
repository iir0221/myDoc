# -XX options

## -XX:PretenureSizeThreshold

虚拟机提供了一个-XX:PretenureSizeThreshold参数，令大于这个设置值的对象直接在老年代中分配。这样做的目的是避免在Eden区及两个Survivor区之间发生大量的内存拷贝（复习一下：新生代采用复制算法收集内存）。

* PretenureSizeThreshold参数只对Serial和ParNew两款收集器有效，Parallel Scavenge收集器不认识这个参数，Parallel Scavenge收集器一般并不需要设置。如果遇到必须使用此参数的场合，可以考虑ParNew加CMS的收集器组合。

The maximum size of an object HotSpot JVM may allocate in young generation is nearly as large as the size of Eden (YoungGen minus two Survivor spaces).

That's how the allocation rougly looks like:

Use thread local allocation buffer (TLAB), if tlab_top + size <= tlab_end
This is the fastest path. Allocation is just the tlab_top pointer increment.
If TLAB is almost full, create a new TLAB in Eden and retry in a fresh TLAB.
If TLAB remaining space is not enough but is still to big to discard, try to allocate an object directly in Eden. Allocation in Eden is also a pointer increment (eden_top + size <= eden_end) using atomic operation, since Eden is shared between all threads.
If allocation in Eden fails, a minor collection typically occurs.
If there is not enough space in Eden even after Young GC, an attempt to allocate directly in Old generation is made.

* 另外，PretenureSizeThreshold is not used unless specified explicitly in the command line. But even if set, it is checked only in slow allocation path. JIT-compiled code first tries to allocate in TLAB or in Eden before falling to slow path.



http://stackoverflow.com/questions/24618467/size-of-huge-objects-directly-allocated-to-old-generation#comment38190239_24620205

## -XX:GCTimeRatio

通过-XX:GCTimeRatio=<value>我们告诉JVM吞吐量要达到的目标值。 更准确地说，-XX:GCTimeRatio=N指定目标应用程序线程的执行时间(与总的程序执行时间)达到N/(N+1)的目标比值。 例如，通过-XX:GCTimeRatio=9我们要求应用程序线程在整个执行时间中至少9/10是活动的(因此，GC线程占用其余1/10)。 基于运行时的测量，JVM将会尝试修改堆和GC设置以期达到目标吞吐量。 -XX:GCTimeRatio的默认值是99，也就是说，应用程序线程应该运行至少99%的总执行时间。


不明确：此参数是否只针对Parallel GC有效？


## -XX:MaxGCPauseMillis
通过-XX:GCTimeRatio=<value>告诉JVM最大暂停时间的目标值(以毫秒为单位)。 在运行时，吞吐量收集器计算在暂停期间观察到的统计数据(加权平均和标准偏差)。 如果统计表明正在经历的暂停其时间存在超过目标值的风险时，JVM会修改堆和GC设置以降低它们。 需要注意的是，年轻代和年老代垃圾收集的统计数据是分开计算的，还要注意，默认情况下，最大暂停时间没有被设置。
如果最大暂停时间和最小吞吐量同时设置了目标值，实现最大暂停时间目标具有更高的优先级。 当然，无法保证JVM将一定能达到任一目标，即使它会努力去做。 最后，一切都取决于手头应用程序的行为。
当设置最大暂停时间目标时，我们应注意不要选择太小的值。 正如我们现在所知道的，为了保持低暂停时间，JVM需要增加GC次数，那样可能会严重影响可达到的吞吐量。 这就是为什么对于要求低暂停时间作为主要目标的应用程序(大多数是Web应用程序)，我会建议不要使用吞吐量收集器，而是选择CMS收集器。


不明确：此参数是否只针对Parallel GC有效？