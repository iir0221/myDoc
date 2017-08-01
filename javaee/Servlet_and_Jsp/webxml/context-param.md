# <context-param>d
用于声明**应用（Application）范围**的初始化参数。它用于向 ServletContext提供键值对，即应用程序上下文信息。我们的listener, filter等在初始化时会用到这些上下文中的信息。

例如
```xml
<web-app>
    <context-param>
        <param-name>Country</param-name>
        <param-value>India</param-value>
    </context-param>
    <context-param>
        <param-name>Age</param-name>
        <param-value>24</param-value>
    </context-param>
</web-app>
```
在servlet中，通过如下方式根据key获取value：
```java
getServletContext().getInitParameter("Country");
getServletContext().getInitParameter("Age");
```

## context-param与ServletContext.setAttribute()对比
servletContext.setAttribute()是动态的，可以在运行时被设置或重置。

init-parameter是静态的，不能在app生命周期内改变。



## context-param与init-param对比


context-param是web.xml的元素，其初始化的键值对是Application范围的，对所有servlet有效。

init-param是servlet元素的子元素，其初始化的键值对是servlet范围的，只对配置的servlet有效。

## reference：

[init-param and context-param](http://stackoverflow.com/questions/28392888/init-param-and-context-param)

[Why use ServletContext.setAttribute()?](http://stackoverflow.com/questions/11046717/why-use-servletcontext-setattribute/11046800#11046800)

[A web.xml Deployment Descriptor Elements](https://docs.oracle.com/middleware/12212/wls/WBAPP/web_xml.htm#WBAPP502)

