# IOC Container
* 声明bean(注册bean到IOC Container中)
* 依赖注入
# 主要接口
## BeanFactory & ApplicationContext
* 接口org.springframework.beans.factory.BeanFactory
    * 最基础的IOC Container,提供管理bean的基础能力
* 接口org.springframework.context.ApplicationContext
    * 继承beanFactory，对其进行扩展，包括整合AOP特性，消息处理，事件发布，以及应用级别的context，例如用于webapp的WebApplicationContext
* 对于大多数的应用场景，无需显示的实例化ApplicationContext，比如在springmvc中，只需在配置文件中配置即可。


## BeanDefinition
* 接口org.springframework.beans.factory.config.BeanDefinition
    * 是让IOC Container起作用的主要数据结构，BeanDefinition描述了一个bean实例，包括属性值，构造函数参数值以及其他信息。IOC功能围绕对BeanDefinition的处理完成

## Resource
* 接口org.springframework.core.io.Resource
    * 资源描述符接口，

## 工具类接口
* org.springframework.beans.factory.support.BeanDefinitionReader

# Register Beans into IOC Container
## Resource定位(即找到配置)
* Resource指