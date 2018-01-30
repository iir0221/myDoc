# Passing and getting parameters
JSF提供多种强大灵活的通信方式


JSf提供多种方式，在Facelet，managedBean，UIComponent间传递参数 

# 使用 context parameters通信
**在web.xml中，通过<context-param>定义的context parameters将被保存在隐式对象initParam中。**
```xml
<context-param>
    <param-name>javax.faces.PROJECT_STAGE</param-name>
    <param-value>Development</param-value>
</context-param>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.1" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd">
    <context-param>
        <param-name>javax.faces.PROJECT_STAGE</param-name>
        <param-value>Development</param-value>
    </context-param>
    <context-param>
        <param-name>number.one.in.ATP</param-name>
        <param-value>Rafael Nadal</param-value>
    </context-param>
    <servlet>
        <servlet-name>Faces Servlet</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>Faces Servlet</servlet-name>
        <url-pattern>/faces/*</url-pattern>
    </servlet-mapping>
    <session-config>
        <session-timeout>
            30
        </session-timeout>
    </session-config>
    <welcome-file-list>
        <welcome-file>faces/index.xhtml</welcome-file>
    </welcome-file-list>
</web-app>
```
**在ManagedBean中通过FacesContext实例获取initParam实例**

ContextParamBean.java　　　　
```java
@Named
@RequestScoped
public class ContextParamBean {

    private String numberone;

    public String getNumberone() {
        FacesContext facesContext = FacesContext.getCurrentInstance();
        numberone = facesContext.getExternalContext().getInitParameter("number.one.in.ATP");
        return numberone;
    }
}
```
**在JSF page中访问上面定义的context parameters，可直接通过隐式对象initParam访问或由facesContext先获取到initParam再进行访问，也可以通过managed bean访问。**

index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">
<h:head>
    <title></title>
</h:head>
<h:body>
    <hr/>
    Get context parameter in JSF page (method 1):
    <h:outputText value="#{initParam['number.one.in.ATP']}"/>
    <hr/>
    Get context parameter in JSF page (method 2):
    <h:outputText value="#{facesContext.externalContext.initParameterMap['number.one.in.ATP']}"/>
    <hr/>
    Get context parameter programmatically:
    <h:outputText value="#{contextParamBean.numberone}"/>
    <hr/>
</h:body>
</html>
```
# Passing request parameters with the \<f:param\> tag
**\<f:param\>中定义的参数将被保存在隐式对象param中。**

## \<f:param\>：在\<h:commandButton\>和\<h:commandLink\>中使用，可以将parameters由一个facelet传递到另一个facelet
在第一个facelet中通过f:param定义参数

index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">
    <h:head>
        <title></title>
    </h:head>   
    <h:body>
         <h:form>
            Click to send name, 'Rafael' surname, 'Nadal', with f:param:
            <h:commandButton value="Send Rafael Nadal" action="#{playersBean.parametersAction()}">
                <f:param id="playerName" name="playerNameParam" value="Rafael"/>               
                <f:param id="playerSurname" name="playerSurnameParam" value="Nadal"/>               
            </h:commandButton>
        </h:form>
    </h:body>
</html>
```
在managed bean中通过FacesContext实例获取隐式对象param再获取参数。

**NOTE：JavaEE提供了@PostConstruct，被该注解注释的方法将在bean装配结束后，action或actionListener方法调用前执行。**

PlayersBean.java
```java
@Named
@RequestScoped
public class PlayersBean {

    private final static Logger logger = Logger.getLogger(PlayersBean.class.getName());
    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }

    // after setting the managed bean properties but before an action method is called
    @PostConstruct
    public void init() {
        FacesContext fc = FacesContext.getCurrentInstance();
        Map<String, String> params = fc.getExternalContext().getRequestParameterMap();
        logger.log(Level.INFO, "Name: {0} Surname: {1}", new Object[]{params.get("playerNameParam"), params.get("playerSurnameParam")});
    }

    public String parametersAction() {

        FacesContext fc = FacesContext.getCurrentInstance();
        Map<String, String> params = fc.getExternalContext().getRequestParameterMap();
        playerName = params.get("playerNameParam");
        playerSurname = params.get("playerSurnameParam");

        return "result";
    }
}
```
第二个facelet通过managed bean获取第一个facelet中定义的parameters，也可以通过隐式对象param获取。

result.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">
    <h:head>
        <title></title>
    </h:head>   
    <h:body>
        Name:<h:outputText value="#{playersBean.playerName}"/>
        Surname:<h:outputText value="#{playersBean.playerSurname}"/><br/>
        <h:outputFormat value="Name: {0} Surname: {1}">
            <f:param value="#{playersBean.playerName}" />
            <f:param value="#{playersBean.playerSurname}" /> 
        </h:outputFormat> 
        <br/>
        Name: #{param.playerNameParam}
        Surname: #{param.playerSurnameParam}
    </h:body>
</html>
```
## 通过@ManagedProperty将managedBean的属性与param关联
在第一个facelet中通过f:param定义参数

index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">
    <h:head>
        <title></title>
    </h:head>   
    <h:body>
         <h:form>
            Click to send name, 'Rafael' surname, 'Nadal', with f:param:
            <h:commandButton value="Send Rafael Nadal" action="result">
                <f:param id="playerName" name="playerNameParam" value="Rafael"/>               
                <f:param id="playerSurname" name="playerSurnameParam" value="Nadal"/>               
            </h:commandButton>
        </h:form>
    </h:body>
</html>
```
通过在managedBean的属性上使用注解@ManagedProperty将bean的属性与param关联起来
```java
PlayersBean.java

import javax.annotation.PostConstruct;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.ManagedProperty;
import javax.faces.bean.RequestScoped;

@ManagedBean //cannot be @Named
@RequestScoped
public class PlayersBean {

    @ManagedProperty(value = "#{param.playerNameParam}")
    private String playerName;
    @ManagedProperty(value = "#{param.playerSurnameParam}")
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }

    @PostConstruct
    public void parametersAction() {
        playerName = playerName.toUpperCase();
        playerSurname = playerSurname.toUpperCase();
    }
}
```
第二个facelet通过managed bean获取第一个facelet中定义的parameters ，也可以通过隐式对象param获取。

result.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">
    <h:head>
        <title></title>
    </h:head>   
    <h:body>
        Name:<h:outputText value="#{playersBean.playerName}"/>
        Surname:<h:outputText value="#{playersBean.playerSurname}"/><br/>
        <h:outputFormat value="Name: {0} Surname: {1}">
            <f:param value="#{playersBean.playerName}" />
            <f:param value="#{playersBean.playerSurname}" /> 
        </h:outputFormat> 
        <br/>
        Name: #{param.playerNameParam}
        Surname: #{param.playerSurnameParam}
    </h:body>
</html>
```
如前所属，@PostConstruct注释的方法将在bean装配结束后，action或actionListener方法调用前执行，因此，result.xhtml文件最终显示的结果为：
```
Name:RAFAEL Surname:NADAL
Name: RAFAEL Surname: NADAL 
Name: Rafael Surname: Nadal
```
## <f:param>可以不使用bean，直接在facelet间通信
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">
    <h:head>
        <title></title>
    </h:head>   
    <h:body>
        Click to send name, 'Rafael' surname, 'Nadal', with f:param:
        <h:link value="Send Rafael Nadal" outcome="result">
            <f:param id="playerName" name="playerNameParam" value="Rafael"/>               
            <f:param id="playerSurname" name="playerSurnameParam" value="Nadal"/>               
        </h:link> 
        <h:form >
            Click to send name, 'Rafael' surname, 'Nadal', with f:param:
            <h:commandButton value="Send Rafael Nadal" type="button" action="result">
                <f:param id="playerName" name="playerNameParam" value="Rafael"/>               
                <f:param id="playerSurname" name="playerSurnameParam" value="Nadal"/>               
            </h:commandButton>
        </h:form>                       
    </h:body>
</html>
```
在第二个facelet中，直接通过隐式对象param访问。
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        Name: #{param.playerNameParam}
        Surname: #{param.playerSurnameParam}
    </h:body>
</html>
```
# Working with view parameters——传递GET请求的参数
## \<f:metadata\>
```html
<f:metadata>
  <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>
</f:metadata>
```
**\<f:viewParam\>用于\<f:metadata\>内部，其作用和\<h:inputText\>类似，只不过它传递的是GET请求参数。其执行的时机如下：**

* 首先获取GET请求中name为playernameparam的参数
通过在\<f:viewParam\>中使用\<f:converter\>和\<f:validator\>来转换，验证参数（同\<h:inputText\>一样，该步骤不是必须的）。
* 如果通过转换和验证，将其装配给bean属性playersBean.playerName。
* 如果没有value属性，则可以在view中通过#{playernameparam}访问

先看一个简单的例子

PlayersBean.java
```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped
public class PlayersBean {

    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">
    <!-- http://localhost:8080/ch2_5/?playernameparam=Rafael&playersurnameparam=Nadal -->
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>                  
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>          
    </h:body>
</html>
```
当通过index.xhtml?playernameparam=Rafael&playersurnameparam=Nadal访问该页面时，在render之前，参数playernameparam的值Rafael，playersurnameparam的值Nadal已经被set到bean中。因此，当view render结束时，index.xhtml上显示如下所示：
```
You requested name: Rafael
You requested surname: Nadal
```
再看一个对GET参数验证的例子
```html
<f:metadata>
    <f:viewParam id="id" name="id" value="#{bean.id}" required="true">
        <f:validateLongRange minimum="10" maximum="20" />
    </f:viewParam>
</f:metadata>
<h:message for="id" />
```
## includeViewParams
**\<h:commandButton /\>和<\h:link /\> 的属性includeViewParams为true时，则可以在转发时，携带\<f:viewParam\>中定义的参数。**

index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
<f:metadata>
    <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>
    <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/>
</f:metadata>
<h:head>
    <title></title>
</h:head>
<h:body>
    <h:panelGrid columns="2">
        <h:form>
            <h:commandButton value="Submit" action="results?faces-redirect=true&amp;includeViewParams=true"/>
        </h:form>
        <h:link value="Send" outcome="results?faces-redirect=true" includeViewParams="true"/>
    </h:panelGrid>
</h:body>
</html>
```
通过index.xhtml?playernameparam=Rafael&playersurnameparam=Nadal访问index.xhtml。

以<h:commandButton value="Submit" action="results?faces-redirect=true&amp;includeViewParams=true"/>为例：

**NOTE：action携带多个参数，要使用转义字符\&amp;(&)**

第一个参数faces-redirect=true表示通过redirect的方式发送请求，includeViewParams=true表示发送GET请求并携带viewParam中定义的参数。

跳转后浏览器中的地址为results.xhtml?playernameparam=Rafael&playersurnameparam=Nadal

result.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
<f:metadata>
    <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>
    <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/>
</f:metadata>
<h:head>
    <title></title>
</h:head>
<h:body>
    You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
    You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>
</h:body>
</html>
```
includeViewParams可以用在任何URL中

index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        <h:form>
            Enter name:<h:inputText value="#{playersBean.playerName}"/>
            Enter surname:<h:inputText value="#{playersBean.playerSurname}"/>
            <h:commandButton value="Submit" action="#{playersBean.toUpperCase()}"/>
        </h:form>       
    </h:body>
</html>
```
PlayersBean.java
```java
@ManagedBean
@RequestScoped
public class PlayersBean {

    private String playerName;
    private String playerSurname;
    
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }   

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }         
    
    public String toUpperCase(){
        playerName=playerName.toUpperCase();
        playerSurname=playerSurname.toUpperCase();
  
        return "results?faces-redirect=true&includeViewParams=true";
    }
}
```
跳转后，浏览器中的地址为：results.xhtml?playernameparam=RAFAEL&playersurnameparam=NADAL

result.xhtml如下
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">    
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>       
    </h:body>
</html>
```
## required和requiredMessage
**UIViewParameter继承自UIInput，因此\<f:viewParam\>也继承了\<h:inputText>\的所有属性，因此可以使用required 和 requiredMessage来提醒用户提供GET请求的参数**

index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">    
    <f:metadata>
        <f:viewParam name="playernameparam" required="true" requiredMessage="Player name required!" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" required="true" requiredMessage="Player surname required!" value="#{playersBean.playerSurname}"/> 
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>                  
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>
    </h:body>
</html>
```
## validator和converter
**通过tag\<f:validator\>， \<f:converter\>或属性 validator ， converter 对value验证和转换。**

例1：

PlayerValidator.java
```java
import java.util.Map;
import java.util.Random;
import javax.faces.application.FacesMessage;
import javax.faces.component.UIComponent;
import javax.faces.context.FacesContext;
import javax.faces.validator.FacesValidator;
import javax.faces.validator.Validator;
import javax.faces.validator.ValidatorException;

@FacesValidator("playerValidator")
public class PlayerValidator implements Validator {

    @Override
    public void validate(FacesContext context, UIComponent component, Object value) throws ValidatorException {

        //simulate some validation here
        Random r = new Random();
        int valid = r.nextInt(20);
        if (valid < 10) {
            FacesMessage msg = new FacesMessage(" Player name/surname validation failed.", "Details about failure!");
            msg.setSeverity(FacesMessage.SEVERITY_ERROR);

            throw new ValidatorException(msg);
        }

    }
}
```
PlayersBean.java
```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped //you need @ViewScoped to perserve view parameters over any validation failures
public class PlayersBean {

    private String playerName="none";
    private String playerSurname="none";

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }   

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }         
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">    
    <f:metadata>
        <f:viewParam id="nameId" name="playernameparam"  validator="playerValidator" value="#{playersBean.playerName}"/>            
        <f:viewParam id="surnameId" name="playersurnameparam" validator="playerValidator" value="#{playersBean.playerSurname}"/>         
    </f:metadata>    
    <h:head>
        <title></title>
    </h:head>   
    <h:body>          
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>
    </h:body>
</html>
```
例2

DataBean.java
```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;
import java.text.SimpleDateFormat;
import java.util.Date;


@ManagedBean
@RequestScoped
public class DateBean {

    private Date date = new Date();

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }        

    public String sendDate() {
        String dateAsString = new SimpleDateFormat("dd-MM-yyyy").format(date);
        return "date.xhtml?faces-redirect=true&date=" + dateAsString;
    }
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">
    <h:head>
        <title></title>
    </h:head>
    <h:body>
        This page send a java.util.Date in URL of date.xhtml page
        <h:form>
            <h:commandButton value="Send Date" action="#{dateBean.sendDate()}"/>
        </h:form>
    </h:body>
</html>
```
date.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core">
    <h:head>
        <title></title>
        <f:metadata>
            <f:viewParam name="date" value="#{dateBean.date}">
                <f:convertDateTime pattern="dd-MM-yyyy" />
            </f:viewParam>
        </f:metadata>
    </h:head>
    <h:body>
        The date is: #{dateBean.date}
    </h:body>
</html>
```
## @PostConstruct和preRenderView
**当使用\<f:viewParam\>时，set value对于前面介绍的@PostConstruct是无效的。可以通过preRenderView类型的event listener来实现同样的效果。**

例

PlayersBean.java
```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped
public class PlayersBean {

    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }
   
    public void init() {        
        if (playerName != null) {
            playerName = playerName.toUpperCase();
        }
        if (playerSurname != null) {
            playerSurname = playerSurname.toUpperCase();
        }
    }
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
        <f:event type="preRenderView" listener="#{playersBean.init()}"/>
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>      
    </h:body>
</html>
```
# Calling actions on GET requests
## 4.1 \<f:viewAction\>：calling actions on GET requests
例1，通过\<f:viewAction\>实现前面的validator功能

PlayersBean.java
```java
import java.util.Random;
import javax.annotation.PostConstruct;

import javax.faces.application.FacesMessage;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;
import javax.faces.context.FacesContext;

@ManagedBean
@RequestScoped
public class PlayersBean {

    private String playerName="";
    private String playerSurname="";
    private FacesContext facesContext;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }   

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }   
    
    @PostConstruct
    public void postConstruct() {      
        facesContext = FacesContext.getCurrentInstance();
    }

    
    public String validateData() {
        
        Random r = new Random();
        int valid = r.nextInt(20);
        if (valid < 10) {
            facesContext.addMessage(null, new FacesMessage(FacesMessage.SEVERITY_FATAL, "Player name/surname validation failed.",""));                                  
        }
        
        return "index";
    }
}
```
index.xhtml 
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">    
    <f:metadata>
        <f:viewParam id="nameId" name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam id="surnameId" name="playersurnameparam" value="#{playersBean.playerSurname}"/>   
        <f:viewAction action="#{playersBean.validateData()}"/>
    </f:metadata>    
    <h:head>
        <title></title>
    </h:head>   
    <h:body>          
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>
    </h:body>
</html>
```
## \<f:viewAction\>和preRenderView的比较
\<f:viewAction\>和preRenderView有些相似但功能并不相同。

### postback request 
    
要弄清楚二者的不同之处，首先需要区分initial request和postback request。stackoverflow有对二者的讨论，截取部分评论如下：

NOTE：JSF: initial request and postback request

Initial request passes only Restore View & Render Response phases(but all the 6 phases will be executed in case of an initial GET request with view parameters),

while postback request process under all phases (Apply Request Values, Validations Phase, etc).

Initial request is created by clicking a link, pasting an URL in address bar, while a postback request is create by posting a form by clicking a submit button or any post request.

Normally you would have only one initial request, when you go to the browser and write in the URL to your app. This make an HTTP GET request to the server with your cookies e.g. JSESSIONID, but not with a javax.faces.viewid to be restored.

When you have an open page and you do hacky stuff lick: window.location = newUrl -> you will also make an initial request.

When instead you do something like jQuery("#somoeSubmitButton").click(), you will POST to the server and your old view will be restored - and if you ask faces context.isPostback() ? you will get true.
    
**preRenderView event listener只在postback request情况下执行，可以在代码中通过如下语句，控制无需在postback请求下执行的语句。**
```java
if (!FacesContext.getCurrentInstance().isPostback()) {
  // code that should not be executed in postback phase
}
```
例1： 

PlayerBean2.java
```java
import java.util.logging.Logger;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;
import javax.faces.context.FacesContext;


@ManagedBean
@RequestScoped
public class PlayersBean2 {

    private final static Logger logger = Logger.getLogger(PlayersBean.class.getName());
    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean2() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }

    public void init() {
        logger.info("The init() method was called !!!");
        if (!FacesContext.getCurrentInstance().isPostback()) {
            logger.info("not a postback !!!");
            if (playerName != null) {
                playerName = playerName.toUpperCase();
            }
            if (playerSurname != null) {
                playerSurname = playerSurname.toUpperCase();
            }
        }
    }

    public void userAction() {
        logger.info("The userAction() method was called !!!");
        System.out.println("The userAction() method was called !!!");
    }
}
```
index2.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean2.playerName}"/>
        <f:viewParam name="playersurnameparam" value="#{playersBean2.playerSurname}"/>
        <f:event type="preRenderView" listener="#{playersBean2.init()}"/>
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean2.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean2.playerSurname}"/>
        <h:form>
            <h:commandButton value="User Action" action="#{playersBean2.userAction()}"/>
        </h:form>
    </h:body>
</html> 
```
在浏览器输入框键入http://localhost:8080/preRenderView/faces/index2.xhtml?playernameparam=%22a%22&playersurnameparam=%22b%22

查看log，打印如下
```
PlayersBean2.init The init() method was called !!!
PlayersBean2.init not a postback !!!
```
点击button

查看log，打印如下
```
PlayersBean2.userAction The userAction() method was called !!!
PlayersBean2.init The init() method was called !!!
```
**使用preRenderView时，可以通过这种方式来控制那些不想要在postBack请求被重复执行的语句，另外可以看出，preRenderView在action后面执行。**

例2：

PlayerBean2.java
```java
import java.util.logging.Logger;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped
public class PlayersBean2 {

    private final static Logger logger = Logger.getLogger(PlayersBean.class.getName());
    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean2() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }

    public void init() { 
            logger.info("The init() method was called !!!");
            if (playerName != null) {
                playerName = playerName.toUpperCase();
            }
            if (playerSurname != null) {
                playerSurname = playerSurname.toUpperCase();
            }        
    }

    public void userAction() {
        logger.info("The userAction() method was called !!!");
    }
}
```
index2.xhtml 
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean2.playerName}"/>
        <f:viewParam name="playersurnameparam" value="#{playersBean2.playerSurname}"/>
        <f:viewAction action="#{playersBean2.init()}" onPostback="true"/>
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean2.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean2.playerSurname}"/>
        <h:form>
            <h:commandButton value="User Action" action="#{playersBean2.userAction()}"/>
        </h:form>
    </h:body>
</html>
```
在浏览器输入框键入http://localhost:8080/viewAcionTag/faces/index2.xhtml?playernameparam=%22a%22&playersurnameparam=%22b%22

查看log，打印如下
```
PlayersBean2.init The init() method was called !!!
```
点击button

查看log，打印如下
```
PlayersBean2.init The init() method was called !!!
PlayersBean2.userAction The userAction() method was called !!!
```
**\<f:viewAction\> tag有一属性onPostback（默认false），上面index2.xhtml中，该属性值被设为true，此时，action="#{playersBean2.init()在postback请求下仍会被执行。当设置为false时，其不会被执行。另外其在action前被执行**

### 导航
**preRenderView event listener不支持导航，想要其实现导航功能，需要借助于 ConfigurableNavigationHandler**

例1

PlayersBean.java
```java
import javax.faces.application.ConfigurableNavigationHandler;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;
import javax.faces.context.FacesContext;

@ManagedBean
@RequestScoped
public class PlayersBean {

    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }  

    //adding navigation capabilities to preRenderView
    public void init() {
        if (playerName != null) {
            playerName = playerName.toUpperCase();
        }
        if (playerSurname != null) {
            playerSurname = playerSurname.toUpperCase();
        }

        ConfigurableNavigationHandler handler = (ConfigurableNavigationHandler) FacesContext.getCurrentInstance().getApplication().getNavigationHandler();
        handler.performNavigation("start");
    }
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
        <f:event type="preRenderView" listener="#{playersBean.init()}"/>
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>          
    </h:body>
</html>
start.xhtml

<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">       
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        The navigation successfully ended!     
        Name: #{param.playernameparam}
        Surname: #{param.playersurnameparam}
    </h:body>
</html>
```

**\<f:viewAction\>支持导航(())

例2

PlayersBean.java
```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped
public class PlayersBean {
   
    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }

    public String init() {       
        if (playerName != null) {
            playerName = playerName.toUpperCase();
        }
        if (playerSurname != null) {
            playerSurname = playerSurname.toUpperCase();
        }

        return "start"; //you can use "start?includeViewParams=true" to pass the view params further
    }
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
        <f:viewAction action="#{playersBean.init()}"/>
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>          
    </h:body>
</html>
```
start.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">       
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        The navigation successfully ended!           
    </h:body>
</html>
```
例3

PlayerBean.java
```java
import javax.faces.bean.ManagedBean;
import javax.faces.bean.RequestScoped;

@ManagedBean
@RequestScoped
public class PlayersBean {
   
    private String playerName;
    private String playerSurname;

    /**
     * Creates a new instance of PlayersBean
     */
    public PlayersBean() {
    }

    public String getPlayerName() {
        return playerName;
    }

    public void setPlayerName(String playerName) {
        this.playerName = playerName;
    }

    public String getPlayerSurname() {
        return playerSurname;
    }

    public void setPlayerSurname(String playerSurname) {
        this.playerSurname = playerSurname;
    }

    public String init() {       
        if (playerName != null) {
            playerName = playerName.toUpperCase();
        }
        if (playerSurname != null) {
            playerSurname = playerSurname.toUpperCase();
        }

        return "start"; //you can use "start?includeViewParams=true" to pass the view params further
    }
}
```
index.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core">   
    <f:metadata>
        <f:viewParam name="playernameparam" value="#{playersBean.playerName}"/>            
        <f:viewParam name="playersurnameparam" value="#{playersBean.playerSurname}"/> 
        <f:viewAction action="#{playersBean.init()}"/>
    </f:metadata>
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        You requested name: <h:outputText value="#{playersBean.playerName}"/><br/>
        You requested surname: <h:outputText value="#{playersBean.playerSurname}"/>          
    </h:body>
</html>
```
start.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">       
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        The navigation successfully ended!        
    </h:body>
</html>
```
rafa.xhtml
```html
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html">       
    <h:head>
        <title></title>
    </h:head>   
    <h:body>        
        Rafael Nadal Home Page!        
    </h:body>
</html>
```
faces-config.xml
```html
<?xml version='1.0' encoding='UTF-8'?>
<faces-config version="2.2"
              xmlns="http://xmlns.jcp.org/xml/ns/javaee"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-facesconfig_2_2.xsd">

    <navigation-rule>
        <from-view-id>index.xhtml</from-view-id>
        <navigation-case>
            <from-action>#{playersBean.init()}</from-action> 
            <from-outcome>start</from-outcome>                 
            <to-view-id>rafa.xhtml</to-view-id>
            <redirect/>
        </navigation-case>
    </navigation-rule>
</faces-config>
```
### 执行阶段 
**默认情况下view action在invoke Application phase阶段执行。当设置immediate=true时(\<f:viewAction action="#{playersBean.init()}" immediate="true"/\>)，执行阶段改为Apply Request Values phase.**

**还可以指定它具体的执行时期：\<f:viewAction action="#{playersBean.init()}" phase="UPDATE_MODEL_VALUES"/\>**

支持的值包括
* APPLY_REQUEST_VALUES, 
* INVOKE_APPLICATION,
* PROCESS_VALIDATIONS, * UPDATE_MODEL_VALUES.

 

 

 

 

 

 

 

 