# Java Runtime.addShutdownHook()方法

Java Runtime.addShutdownHook()方法用法实例教程。允许应用程序接口与应用程序运行的环境中.


## 描述
java.lang.Runtime.addShutdownHook(Thread hook) 方法注册一个新的虚拟机关闭挂钩。 Java虚拟机的关机响应于两种类型的事件：

* 所有**非守护线程**运行结束或**System.exit()**方法被调用

* 响应一个用户中断，如键入**Ctrl+ C,kill -15**，或一个**全系统的事件，如用户注销或系统关闭**.

**一个关闭钩子是一个**简单初始化还没有启动的**线程**。当虚拟机开始进入关闭阶段时，虚拟机将以不确定的顺序启动所有已经注册的关闭钩子，当所有的钩子线程结束时，如果finalization-on-exit 启用，JVM将接着运行所有还没被调用的finalizer。最后，虚拟机停止。注意，**在JVM的关闭阶段，守护线程将持续运行，如果JVM的关闭阶段是通过调用exit方法进入的，非守护的线程也会在JVM的关闭阶段持续运行**。

一旦进入关闭工序，JVM将只能通过调用halt方法停止，这个方法将强制结束虚拟机。

一旦关闭工序开始，虚拟机不能再被注册新的ShutDown hook，也不能撤销之前注册的钩子。这两个操作都会导致抛出一个IllegalStateException错误。

ShutDown钩子运行在Java虚拟机整个生命周期的一个很微妙的时刻，虚拟机生命周期的最后阶段，因此在编写程序时应该十分小心。这些钩子线程应该都是线程安全的，并且要避免任何可能的死锁的发生。钩子线程不应该依赖于绑定的服务，这些绑定的服务包括，注册到虚拟机的其他的钩子线程和在JVM已经开启关闭工序时钩子线程自己。使用其他基于线程的服务，像AWT的事件分发线程，可能导致死锁。

关闭钩子线程中未捕获的错误的处理方式跟其他的线程一样，通过调用ThreadGroup#uncaughtException方法，这个方法的默认实现是打印错误堆栈信息（System#err），然后结束线程；这个方法不会导致虚拟机结束或终止。

在罕见的情况下，虚拟机可能会abort，也就是说，没有干净的关闭而直接停止运行。这种情况会发生在当虚拟机被外部的力量结束时。例如，Unix中的SIGKILL信号或者window平台中的TerminateProcess被调用。虚拟机也可能在一个native的方法出错时abort，例如，毁坏的内部数据结构或者尝试访问不存在的内存。如果虚拟机abort，将不能保证关闭钩子线程被完整运行。

上边这一段是Runtime#addShutDownHook方法的注释。简单总结一下，钩子线程的启动时机：

* 虚拟机自己结束时会调用，比如程序运行完成，或者用户跟虚拟机交互之后，虚拟机接收到退出信号，自己结束
* 外部力量结束虚拟机时，不能保证钩子线程启动，比如在windows平台中，启动任务管理器，直接将这个Java虚拟机关闭进程，这种情况下不能保证钩子线程一定会被调用

## 声明
以下是声明java.lang.Runtime.addShutdownHook()方法
```java
public void addShutdownHook(Thread hook)
```
## 参数
hook -- 一个初始化但尚未启动的线程对象

## 返回值
此方法不返回一个值。

## 异常
IllegalArgumentException -- 如果指定的钩已被注册，或如果它可以判定钩已经运行或已被运行

IllegalStateException -- 如果虚拟机已经是在关闭的过程中

SecurityException -- 如果存在安全管理器并且它拒绝的RuntimePermission（“shutdownHooks”）

## 实例
>在JVM的关闭阶段，守护线程将持续运行，如果JVM的关闭阶段是通过调用exit方法进入的，非守护的线程也会在JVM的关闭阶段持续运行
```java
package daemon;

/**
 * Created by xinyuan.zhang on 11/8/17.
 */
public class Test {
    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(
            new Thread() {
                public void run() {
                    System.out.println("shutdown hook");
                }
            }
        );

        Thread userThread = new Thread() {
            public void run() {
                System.out.println(Thread.currentThread().getName()+ " start");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+ " end");
            };
        };

        Thread daemonThread = new Thread() {
            public void run() {
                System.out.println(Thread.currentThread().getName()+ " start");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+ " end");
            }
        };

        userThread.setName("User Thread");
        userThread.start();
        daemonThread.setName("Daemon Thread");
        daemonThread.setDaemon(true);
        daemonThread.start();
        System.out.println(Thread.currentThread().getName()+ " end");
    }
}
```
结果
```
User Thread start
Daemon Thread start
main end
User Thread end
shutdown hook
```
```java

/**
 * Created by xinyuan.zhang on 11/8/17.
 */
public class Test {


    public static void main(String[] args) {

        Runtime.getRuntime().addShutdownHook(
            new Thread() {
                public void run() {
                    System.out.println("shutdown hook");
                }
            }
        );

        Thread userThread = new Thread() {
            public void run() {
                System.out.println(Thread.currentThread().getName()+ " start");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+ " end");
            };

        };

        Thread daemonThread = new Thread() {
            public void run() {
                System.out.println(Thread.currentThread().getName()+ " start");
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName()+ " end");
            }
        };


        userThread.setName("User Thread");
        userThread.start();
        daemonThread.setName("Daemon Thread");
        daemonThread.setDaemon(true);
        daemonThread.start();

        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName()+ " end");
        System.exit(0);

    }

}
```
结果
```
User Thread start
Daemon Thread start
main end
shutdown hook
```
参考:
* [JAVA Runtime.addShutdownHook()方法{拿到线程句柄，在程序关闭之前调用释放资源}](http://blog.csdn.net/bbaiggey/article/details/53634477)

* [Java Runtime.addShutdownHook()方法](http://www.yiibai.com/javalang/runtime_addshutdownhook.html)
## finalization-on-exit
## 守护线程