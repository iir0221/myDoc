# cookies

cookie被保存在客户端

每个cookie存储的信息量是有限制的。针对某个域名，浏览器保存的cookie数量也是有限的。

cookie通过http header传输

servlet开发者可以控制cookie的存活时间。

cookie有一个缺点，用户可以通过设置浏览器拒绝接受cookie。

与cookie有关的类为
```java
javax.servlet.http.Cookie
```
以及HttpServletRequest 和 HttpServletResponse interfaces中的部分方法.

## 创建cookie

```java
Cookie cookie = new Cookie(name, value);
```
## 设置存活时间
```java
cookie.setMaxAge(int maxAge); 
```
Cookie默认的maxAge值为-1。

如果maxAge为负数，则表示该Cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，关闭窗口后该Cookie即失效。maxAge为负数的Cookie，为临时性Cookie，不会被持久化，不会被写到Cookie文件中。Cookie信息保存在浏览器内存中，因此关闭浏览器该Cookie就消失了。

如果maxAge为0，则表示删除该Cookie。Cookie机制没有提供删除Cookie的方法，因此通过设置该Cookie即时失效实现删除Cookie的效果。失效的Cookie会被浏览器从Cookie文件或者内存中删除。

## 设置访问路径
```java
cookie.setPath(String url);
```
默认只有设置该 cookie 的 URI 及其子路径可以访问客户端中存储的cookie。

例：

cookie设置在如下面servlet中：http://localhost:8080/JavaWeb/servlet/CookieDemo，并没有指定path，则

/JavaWeb/servlet 就是当前 Cookie 的 path 。
若访问的地址的 URI 包含着 cookie 的路径，即 URI.startWith(cookie 的路径 ), 为 true ，则服务器可访问存储在客户端的cookie。

例：

如果浏览器存的 cookie 的路径是 /JavaWeb，则 

http://localhost:8080/JavaWeb/servlet/CookieDemo：可访问

http://localhost:8080/JavaWeb/CookieDemo：可访问

例：

如果浏览器存的 cookie 的路径是 /JavaWeb/servlet/，则 

http://localhost:8080/JavaWeb/servlet/CookieDemo：可访问 

http://localhost:8080/JavaWeb/CookieDemo：不可访问

如果一个 cookie 的路径设置成了 /JavaWeb ，意味着浏览器访问当前应用下的所有资源时都会带着该 cookie 给服务器。

## 设置domain
```java
cookie.setDomain(String domain);
```

## read cookie
例：读取一个名为maxRecords的cookie
```java
Cookie[] cookies = request.getCookies();
Cookie maxRecordsCookie = null;
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if (cookie.getName().equals("maxRecords")) {
            maxRecordsCookie = cookie;
            break;
        }
    }
}
```

## delete cookie

例：删除一个名为userName的cookie
```java
Cookie cookie = new Cookie("userName", "");
cookie.setMaxAge(0);
response.addCookie(cookie);
```
例
在PreferenceServlet中设置cookie
```java
@WebServlet(name = "PreferenceServlet", 
        urlPatterns = { "/preference" })
public class PreferenceServlet extends HttpServlet {
    private static final long serialVersionUID = 888L;

    public static final String MENU = 
            "<div style='background:#e8e8e8;"
            + "padding:15px'>"
            + "<a href='cookieClass'>Cookie Class</a>&nbsp;&nbsp;"
            + "<a href='cookieInfo'>Cookie Info</a>&nbsp;&nbsp;"
            + "<a href='preference'>Preference</a>" + "</div>";

    @Override
    public void doGet(HttpServletRequest request,
            HttpServletResponse response) throws ServletException,
            IOException {
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.print("<!DOCTYPE html>"
                + "<html><head>" + "<title>Preference</title>"
                + "<style>table {" + "font-size:small;"
                + "background:NavajoWhite }</style>"
                + "</head><body>"
                + MENU
                + "Please select the values below:"
                + "<form method='post'>"
                + "<table>"
                + "<tr><td>Title Font Size: </td>"
                + "<td><select name='titleFontSize'>"
                + "<option>large</option>"
                + "<option>x-large</option>"
                + "<option>xx-large</option>"
                + "</select></td>"
                + "</tr>"
                + "<tr><td>Title Style & Weight: </td>"
                +"<td><select name='titleStyleAndWeight' multiple>"
                + "<option>italic</option>"
                + "<option>bold</option>"
                + "</select></td>"
                + "</tr>"
                + "<tr><td>Max. Records in Table: </td>"
                + "<td><select name='maxRecords'>"
                + "<option>5</option>"
                + "<option>10</option>"
                + "</select></td>"
                + "</tr>"
                + "<tr><td rowspan='2'>"
                + "<input type='submit' value='Set'/></td>"
                + "</tr>"
                + "</table>" + "</form>" + "</body></html>");

    }

    @Override
    public void doPost(HttpServletRequest request,
            HttpServletResponse response) throws ServletException,
            IOException {

        String maxRecords = request.getParameter("maxRecords");
        String[] titleStyleAndWeight = request
                .getParameterValues("titleStyleAndWeight");
        String titleFontSize =
                request.getParameter("titleFontSize");
        response.addCookie(new Cookie("maxRecords", maxRecords));
        response.addCookie(new Cookie("titleFontSize", 
                titleFontSize));

        // delete titleFontWeight and titleFontStyle cookies first
        // Delete cookie by adding a cookie with the maxAge = 0;
        Cookie cookie = new Cookie("titleFontWeight", "");
        cookie.setMaxAge(0);
        response.addCookie(cookie);
        cookie = new Cookie("titleFontStyle", "");
        cookie.setMaxAge(0);
        response.addCookie(cookie);

        if (titleStyleAndWeight != null) {
            for (String style : titleStyleAndWeight) {
                if (style.equals("bold")) {
                    response.addCookie(new 
                            Cookie("titleFontWeight", "bold"));
                } else if (style.equals("italic")) {
                    response.addCookie(new Cookie("titleFontStyle",
                            "italic"));
                }
            }
        }

        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.println("<!DOCTYPE html>"
                + "<html><head>" + "<title>Preference</title></head>"
                + "<body>" + MENU
                + "Your preference has been set."
                + "<br/><br/>Max. Records in Table: " + maxRecords
                + "<br/>Title Font Size: " + titleFontSize
                + "<br/>Title Font Style & Weight: ");

        // titleStyleAndWeight will be null if none of the options
        // was selected
        if (titleStyleAndWeight != null) {
            writer.println("<ul>");
            for (String style : titleStyleAndWeight) {
                writer.print("<li>" + style + "</li>");
            }
            writer.println("</ul>");
        }
        writer.println("</body></html>");
    }
}
```
读取cookie：maxRecords
```java
@WebServlet(name = "CookieClassServlet", 
        urlPatterns = { "/cookieClass" })
public class CookieClassServlet extends HttpServlet {
    private static final long serialVersionUID = 837369L;
    
    private String[] methods = {
            "clone", "getComment", "getDomain",
            "getMaxAge", "getName", "getPath",
            "getSecure", "getValue", "getVersion",
            "isHttpOnly", "setComment", "setDomain",
            "setHttpOnly", "setMaxAge", "setPath",
            "setSecure", "setValue", "setVersion"
    };

    @Override
    public void doGet(HttpServletRequest request,
            HttpServletResponse response) throws ServletException,
            IOException {

        Cookie[] cookies = request.getCookies();
        Cookie maxRecordsCookie = null;
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                if (cookie.getName().equals("maxRecords")) {
                    maxRecordsCookie = cookie;
                    break;
                }
            }
        }
        
        int maxRecords = 5; // default
        if (maxRecordsCookie != null) {
            try {
                maxRecords = Integer.parseInt(
                        maxRecordsCookie.getValue());
            } catch (NumberFormatException e) {
                // do nothing, use maxRecords default value
            }
        }
        
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.print("<!DOCTYPE html>"
                + "<html><head>" + "<title>Cookie Class</title></head>"
                + "<body>" + PreferenceServlet.MENU
                + "<div>Here are some of the methods in " +
                		"javax.servlet.http.Cookie");
        writer.print("<ul>");
        
        for (int i = 0; i < maxRecords; i++) {
            writer.print("<li>" + methods[i] + "</li>");
        }
        writer.print("</ul>");
        writer.print("</div></body></html>");
    }
}
```
读取多个cookie用来控制css样式
```java
@WebServlet(name = "CookieInfoServlet", urlPatterns = { "/cookieInfo" })
public class CookieInfoServlet extends HttpServlet {
    private static final long serialVersionUID = 3829L;

    @Override
    public void doGet(HttpServletRequest request,
            HttpServletResponse response) throws ServletException,
            IOException {

        Cookie[] cookies = request.getCookies();
        StringBuilder styles = new StringBuilder();
        styles.append(".title {");
        if (cookies != null) {
            for (Cookie cookie : cookies) {
                String name = cookie.getName();
                String value = cookie.getValue();
                if (name.equals("titleFontSize")) {
                    styles.append("font-size:" + value + ";");
                } else if (name.equals("titleFontWeight")) {
                    styles.append("font-weight:" + value + ";");
                } else if (name.equals("titleFontStyle")) {
                    styles.append("font-style:" + value + ";");
                }
            }
        }
        styles.append("}");
        response.setContentType("text/html");
        PrintWriter writer = response.getWriter();
        writer.print("<!DOCTYPE html>"
                + "<html><head>" + "<title>Cookie Info</title>"
                + "<style>" + styles.toString() + "</style>"
                + "</head><body>" + PreferenceServlet.MENU 
                + "<div class='title'>"
                + "Session Management with Cookies:</div>");
        writer.print("<div>");

        // cookies will be null if there's no cookie
        if (cookies == null) {
            writer.print("No cookie in this HTTP response.");
        } else {
            writer.println("<br/>Cookies in this HTTP response:");
            for (Cookie cookie : cookies) {
                writer.println("<br/>" + cookie.getName() + ":"
                        + cookie.getValue());
            }
        }
        writer.print("</div>");
        writer.print("</body></html>");
    }
}
```




