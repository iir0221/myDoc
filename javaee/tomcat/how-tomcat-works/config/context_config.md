# <span id="context">Tomcat context.xml配置详解</span>
[转载自]http://blog.csdn.net/nlznlz/article/details/77623379
## 在tomcat 5.5之前：

Context体现在/conf/server.xml中的Host里的<Context>元素，它由Context接口定义。每个<Context>元素代表了运行在虚拟主机上的单个Web应用。

<Context>元素：

```xml
<Context path="/kaka" docBase="kaka" debug="0" reloadbale="true">    
```
* path：即要建立的虚拟目录，,注意是/kaka，它指定访问Web应用的URL入口，如http://localhost:8080/kaka/****。
* docBase：为实际目录在硬盘上的位置（应用程序的路径或者是WAR文件存放的路径）。
* reloadable：如果这个属性设为true，Tomcat服务器在运行状态下会监视在WEB-INF/classes和Web-INF/lib目录CLASS文件的改变，如果监视到有class文件被更新，服务器自动重新加载Web应用，这样我们可以在不重起tomcat的情况下改变应用程序。

一个Host元素中嵌套任意多的Context元素。每个Context的路径必须是惟一的，由path属性定义。另外，你必须定义一个path=“”的context，这个Context称为该虚拟主机的缺省web应用，用来处理那些不能匹配任何Context的Context路径的请求。

 
## 在tomcat 5.5之后：

不推荐在server.xml中进行配置，而是在/conf/context.xml中进行独立的配置。因为server.xml是不可动态重加载的资源，服务器一旦启动了以后，要修改这个文件，就得重启服务器才能重新加载。而context.xml文件则不然，tomcat服务器会定时去扫描这个文件。一旦发现文件被修改（时间戳改变了），就会自动重新加载这个文件，而不需要重启服务器。

```xml
<Context path="/kaka" docBase="kaka" debug="0" reloadbale="true" privileged="true">  
  
<WatchedResource>WEB-INF/web.xml</WatchedResource>  
  
<WatchedResource>WEB-INF/kaka.xml</WatchedResource> 监控资源文件，如果web.xml || kaka.xml改变了，则自动重新加载改应用。  
  
<Resource name="jdbc/testSiteds" 表示指定的jndi名称  
auth="Container" 表示认证方式，一般为Container  
type="javax.sql.DataSource"  
maxActive="100" 连接池支持的最大连接数  
maxIdle="30" 连接池中最多可空闲maxIdle个连接  
maxWait="10000" 连接池中连接用完时,新的请求等待时间,毫秒  
username="root" 表示数据库用户名  
password="root" 表示数据库用户的密码  
driverClassName="com.mysql.jdbc.Driver" 表示JDBC DRIVER  
url="jdbc:mysql://localhost:3306/testSite" /> 表示数据库URL地址  
  
</Context>  
```
附： context.xml的三个作用范围：

* tomcat server级别：
    在/conf/context.xml里配置。如果你在这个地方配置、那么这个配置文件将会被所有的webApp共享 

* Host级别：
    在/conf/Catalina/${hostName}里添加context.xml，继而进行配置。这个配置将会被这个主机上的webapp共享。

* web app 级别：
    在/conf/Catalina/${hostName}里添加${webAppName}.xml，继而进行配置。tomcat 服务器文件中的 $CATALINA_BASE/webapps 目录下的所有文件夹都是一个应用。这个时候不需要自己动手配置，服务器默认将文件夹名映射成虚拟目录名称。还可以通过 $CATALINA_BASE/webapps/{App}/META-INF/context.xml 来配置，这个是在web应用中自己添加的，配置和其它一样。