# 并发基础
## 线程安全

* 同一个对象
    * 无状态：没有属性
        * 无状态对象一定是线程安全的
    * 可变状态：只要属性有可能发生变化
        * 可变状态是非线程安全的
        * 原因：竞态条件
            * 竞态条件产生的情况如：
                * 读-修改-写
                * 延时初始化（单例）
                * 如果不存在则add
        * 避免竞态条件产生：保证原子性，可见性，有序性
            * 可变属性只有一个
                * 使用Atomic包提供的类
            * 可变属性多个
                * 加锁  

## 锁
* synchronized
    * 作用于方法时，锁为调用该方法的对象
    * 作用于静态方法时，锁为静态方法所在的Class对象

* 例子
   
   一个对象有如下三个方法：
    ```java
    public synchronized void method1();
    public synchronized void method2();
    public void method3();
    ```
    线程1调用method1,获取对象的锁
    
    线程2调用method1,阻塞，因为锁被线程1持有

    线程3调用method2,阻塞，因为锁被线程1持有

    线程4调用method3,非阻塞，因为调用method3,无需获取锁

## 对象的共享
### 内存可见性
* 为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制。
* 在没有同步的情况下，编译器，处理器，运行时都可能对操作的执行顺序进行一些意想不到的调整。
* 例子

    ```java
    public class Test{
        private int value;
        public int getValue(){
            return value;
        }
        public void setValue(int value) {
            this.value = value;
        }
    }
    ```
    上面这个方法是非线程安全的，调用get方法的线程，不一定能及时读取到调用set的线程设置的value值。


## Volatile、synchronized两者的区别联系


* 1.volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
* 2.volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的。
* 3.volatile仅能实现变量的修改可见性，不能保证原子性（线程A修改了变量还没结束时,另外的线程B可以看到已修改的值,而且可以修改这个变量,而不用等待A释放锁,因为Volatile 变量没上锁）；而synchronized则可以保证变量的修改可见性和原子性。
* 4.volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞和上下文切换。
* 5.<font color=#7FFFD4 >volatile标记的变量不会被编译器优化（禁止指令重排）；synchronized标记的变量可以被编译器优化。</font>
* 6.在使用volatile关键字时要慎重，并不是只要简单类型变量使用volatile修饰，对这个变量的所有操作都是原子操作。当变量的值由自身决定时，如n=n+1、n++ 等，volatile关键字将失效。只有当变量的值和自身无关时对该变量的操作才是原子级别的，如n = m + 1，这个就是原级别的。所以在使用volatile关键时一定要谨慎，如果自己没有把握，可以使用synchronized来代替volatile。
* 7.“锁是昂贵的”，谨慎使用锁机制。