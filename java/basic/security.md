# Security

## Classload

### Class.forName的类加载器
* Class.forName(String className)使用的类加载器是调用者的类加载器，不一定是System类加载器
* Class.forName(String name,boolean initialize,ClassLoader loader)可以指定类加载器，推荐通过这种方式来获取类


### 当前线程上下文类加载器
* 主线程的类加载器是系统的类加载器
* 当新线程创建时，它的上下文类加载器会被设置为创建该线程的上下文类加载器
* 因此，如果没有任何特殊操作，所有线程的上下文类加载器都是系统类加载器
* 当然也可以设置某个线程的类加载器
```java
Thread t = Thread.currentThread();
t.setContextClassLoader(loader);
```

## Classloader作为命名空间
一个类是有它的限定名+Classloader来决定的，因此在一个虚拟机中，可以由两个类拥有同样的限定名