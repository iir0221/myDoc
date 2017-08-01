# Servlet

##annotations
```java
@WebServlet(name = "MyServlet", urlPatterns = { "/my" })
```
* name：servlet class的名字
* urlPatterns：访问路径
* 访问http://localhost:8080/appName/my

##ServletConfig
通过init方法，获取ServletConfig并赋值给成员变量
```java
private transient ServletConfig servletConfig;

  @Override
  public void init(ServletConfig servletConfig)
          throws ServletException {
      this.servletConfig = servletConfig;
  }
```
##输出流
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
* response.getWriter()获取java.io.PrintWriter实例，处理字符流，默认的编码是ISO-8859-1。
* response.getOutputStream()获取javax.servlet.ServletOutputStream(继承java.io.OutputStream)实例，处理字节流。



##ContentType
用于定义网络文件的类型和网页的编码，决定浏览器将以什么形式、什么编码读取这个文件
setContentType






