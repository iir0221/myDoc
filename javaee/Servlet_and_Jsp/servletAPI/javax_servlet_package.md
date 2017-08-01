# javax.servlet Package

**<center>javax.servlet</center>**

![](image/javax_servlet.png)
## ServletRequest
**servlet Container将servletRequest实例传递给servlet的service方法。**

```java
public int getContentLength()
// 获取Content type
public java.lang.String getContentType()
// 获取get或post方法传递的value
public java.lang.String getParameter(java.lang.String name)
public java.lang.String getProtocol()
```

## ServletResponse
**servlet Container将servletResponse实例传递给servlet的service方法。**
* response.getWriter()获取java.io.PrintWriter实例，处理字符流，默认的编码是ISO-8859-1。
* response.getOutputStream()获取javax.servlet.ServletOutputStream(继承java.io.OutputStream)实例，处理字节流。
* response.setContextType()设置content type（content type用于定义网络文件的类型和网页的编码，决定浏览器将以什么形式、什么编码读取这个文件）。
```java
@Override
public void service(ServletRequest request,
        ServletResponse response) throws ServletException,
        IOException {
    String servletName = servletConfig.getServletName();
    response.setContentType("text/html");
    PrintWriter writer = response.getWriter();
    writer.print("<!DOCTYPE html>"
            + "<html>"
            + "<body>Hello from " + servletName 
            + "</body>"
            + "</html>");
}
```

##ServletConfig
**servlet Container初始话servlet时，通过init方法，将ServletConfig实例传递给servlet。**

通过init方法，获取ServletConfig并赋值给成员变量

```java
private transient ServletConfig servletConfig;

  @Override
  public void init(ServletConfig servletConfig)
          throws ServletException {
      this.servletConfig = servletConfig;
  }
```
**servletConfig可获取初始化参数（init-param）给servlet。**
```java
java.lang.String getInitParameter(java.lang.String name)
java.util.Enumeration<java.lang.String> getInitParameterNames()
java.lang.String servletConfig.getServletName();
// 获取servletContext实例
javax.servlet.ServletContext servletConfig.getServletContext();
```
例
```java
@WebServlet(name = "ServletConfigDemoServlet", 
    urlPatterns = { "/servletConfigDemo" },
    initParams = {
        @WebInitParam(name="admin", value="Harry Taciak"),
        @WebInitParam(name="email", value="admin@example.com")
    }
)
public class ServletConfigDemoServlet implements Servlet {
    private transient ServletConfig servletConfig;

    @Override
    public ServletConfig getServletConfig() {
        return servletConfig;
    }

    @Override
    public void init(ServletConfig servletConfig) 
            throws ServletException {
        this.servletConfig = servletConfig;
    }

    @Override
    public void service(ServletRequest request, 
            ServletResponse response)
            throws ServletException, IOException {
        ServletConfig servletConfig = getServletConfig();
        String admin = servletConfig.getInitParameter("admin");
        String email = servletConfig.getInitParameter("email");
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.print("<!DOCTYPE html>"
                + "<html>"
                + "<body>" 
                + "Admin:" + admin 
                + "<br/>Email:" + email
                + "</body></html>");
    }

    @Override
    public String getServletInfo() {
        return "ServletConfig demo";
    }
    
    @Override
    public void destroy() {
    }    
}
```
## ServletContext
一个web app，只有一个context。分布式环境下，一个web app部署在多个container中，则一个JVM有一个context。

**两种方法获取ServletContext实例：**
```java
// 通过request获取
request.getServletContext();
// 通过servletConfig实例获取
servletConfig.getServletContext();
```
**存储在servletContext中的对象被称为attributes**

**ServletContext提供如下方法处理attributes**
```java
java.lang.Object getAttribute(java.lang.String name)
java.util.Enumeration<java.lang.String> getAttributeNames()
void setAttribute(java.lang.String name, java.lang.Object object)
void removeAttribute(java.lang.String name)
```

## GenericServlet
实现了Servlet和ServletConfig接口，主要完成如下任务：

可通过getServletConfig方法获取servletConfig实例

提供servlet接口除service外所有方法的默认实现。

包装servletConfig的方法。

例
```java
@WebServlet(name = "GenericServletDemoServlet", 
    urlPatterns = { "/generic" },
    initParams = {
        @WebInitParam(name="admin", value="Harry Taciak"),
        @WebInitParam(name="email", value="admin@example.com")
    }
)
public class GenericServletDemoServlet extends GenericServlet {
    
    private static final long serialVersionUID = 62500890L;

    @Override
    public void service(ServletRequest request, 
            ServletResponse response)
            throws ServletException, IOException {
        ServletConfig servletConfig = getServletConfig();
        String admin = servletConfig.getInitParameter("admin");
        String email = servletConfig.getInitParameter("email");
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.print("<!DOCTYPE html>"
                + "<html><head></head><body>" 
                + "Admin:" + admin
                + "<br/>Email:" + email
                + "</body></html>");
    }
}
```



## annotations
配置servlet&servlet-mapping
```java
@WebServlet(name = "MyServlet", urlPatterns = { "/my" })
```
* name：servlet class的名字
* urlPatterns：访问路径
* 访问http://localhost:8080/appName/my


配置init-param
```java
@WebServlet(name = "ServletConfigDemoServlet", 
    urlPatterns = { "/servletConfigDemo" },
    initParams = {
        @WebInitParam(name="admin", value="Harry Taciak"),
        @WebInitParam(name="email", value="admin@example.com")
    }
)
```





