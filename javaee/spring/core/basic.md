# 基本概念
## spring设计原则
### 1.使用POJO进行轻量级和最小侵入式开发
#### POJO
* pure old java object
* 有无参构造函数和getter,setter方法
#### 非侵入编程
* 一个类在spring应用和非spring应用中都可以工作

### 2.通过依赖注入和基于接口编程实现松耦合
#### 依赖注入 spring IoC container
* 控制反转（inversion of control-IOC）和依赖注入（dependency injection-DI）
    * 这两个概念在spring中是一样的
    * 主要目的：解耦和
        * 相比于继承，组合可以降低耦合度
        * 自己管理对象的应用，会导致高度耦合，即紧耦合
        * 耦合是必须的，但不应当与某个具体的类发生耦合，而是要求这些类实现同一个接口，即松耦合
        * 紧耦合不易进行单元测试，松耦合可以轻松进行单元测试，例如使用Mock框架
    * 装配
        * 创建应用对象之间协作关系的行为
        * xml显示配置
        * java显示配置
        * 隐士bean发现机制和自动装配
        * 不同的装配方式，使用不同的context来装载

### 3.基于切面和惯例进行声明式编程
#### 面向切面编程AOP（aspect-oriented programming,AOP）
* 将各个功能分离为可重用组件
* 横切关注点
    * 跨越系统多个组件的系统服务
    * 例如，日志，事务管理，安全
    * AOP使上述服务模块化，以声明的方式将他们应用到需要的组件中，使得安全，事物，日志等关注点与核心业务逻辑分离    

### 4.通过切面和模板减少样式代码
#### 样板代码
* 例如：JDBC，JMS，JNDI，REST都涉及大量重复的样板代码
* spring通过诸如JdbcTemplate等消除样板代码

## spring framework runtime
![](./image/spring-framework.png)







