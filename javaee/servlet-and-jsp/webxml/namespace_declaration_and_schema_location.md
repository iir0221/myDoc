# Namespace Declaration and Schema Location

## Servlet 3.1 deployment descriptor


<center>
  <font color=green>
    <br/><br/>
    Java EE 7 XML schema，namespace是</br>
    http://xmlns.jcp.org/xml/ns/javaee
    <br/><br/>
  </font>
 </center>

web.xml
```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
		 http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
</web-app>
```


## Servlet 3.0 deployment descriptor


<center>
  <font color=green>
    <br/><br/>
    Java EE 5,Jave EE 6 XML schema, namespace 是 </br>
    http://java.sun.com/xml/ns/javaee
    <br/><br/>
  </font>
 </center>
 
 web.xml
```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
	      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	      version="3.0">
</web-app>
```

## Servlet 2.5 deployment descriptor
   
 web.xml
```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
	      http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	      version="2.5">
</web-app>
```
## Servlet 2.4 deployment descriptor

<center>
  <font color=green>
    <br/><br/>
    J2EE 4 XML schema, namespace 是 </br>
    http://java.sun.com/xml/ns/j2ee
    <br/><br/>
  </font>
 </center>
 
 web.xml
```xml
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
	      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	      xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
	      http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
	      version="2.4">

  <display-name>Servlet 2.4 Web Application</display-name>
</web-app>
```
## Servlet 2.3 deployment descriptor

<center>
  <font color=green>
    <br/><br/>
    J2EE 3 使用DTDs schema
    <br/><br/>
  </font>
 </center>
 
 web.xml
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Servlet 2.3 Web Application</display-name>
</web-app>
```
## reference：

[web.xml deployment descriptor examples](http://www.mkyong.com/web-development/the-web-xml-deployment-descriptor-examples/)
