# 运行时数据区域

根据Java虚拟机规范，Java虚拟机所管理的内存包括以下几个运行时数据区域：
![](/image/runtime-data-areas.jpg)

* 程序计数器
 * 唯一没有定义任何OutOfMemoryError的区域
* 虚拟机栈
 * 请求深度大于虚拟机所允许的深度（死循环等），StackOverflowError
 * 无法申请足够的内存，OutOfMemoryError
* 本地方法栈
 * 请求深度大于虚拟机所允许的深度（死循环等），StackOverflowError
 * 无法申请足够的内存，OutOfMemoryError
* 堆
 * 无法申请足够的内存，OutOfMemoryError
* 方法区
 * 无法申请足够的内存，OutOfMemoryError


    
* 直接内存
 * 并不是JVM规范中定义的
 * NIO类使用Native函数库直接分配堆外内存。然后通过堆中的DirectByteBuffer对象对这块内存引用，这块内存就是直接内存