# JMX(Java Management Extensions)基础
一个可以使用JMX管理器来管理的Java对象称为JMX管理资源(JMX manageable resource)。

事实上,一个JMX管理资源也可以是一个应用程序、一个实现或者一个服务、设备、用户等等。

JMX管理资源用Java写或者提供一个Java包装。

要想让一个Java对象称为JMX管理资源,必须创建一个名为 Managed Bean 或者 MBean 的对象。

MBean将Java 对象的属性和方法暴露给管理应用程序(management application)。管理应用程序本身不能直接访问 Java 对象。因此可以选择任意的属性和方法让管理应用程序访问。

一旦你有一个MBean类,你需要初始化它的一个对象并将其注册到一个MBean服务器的对象(MBean server)。

MBean服务器是应用程序中所有的MBean的中心登记处(central registry)。管理应用程序通过MBean服务器访问MBeans。 将JMX和Servlet应用程序相比较,管理应用程序相当于一个web浏览器。MBean服务器相当于一个 Servlet容器,它为客户端提供管理资源的访问。而 MBeans相当于Servlet或者JSP页面。就像是web浏览器从来不直接接触Servlet/JSP页面,而是通过容器访问。管理应用程序也不会直接访问MBeans,而是通过 MBean 服务器来进行。

* 一共有四种 MBean:
    * 标准 standard
    * 动态 dynamic
    * 打开 open
    * 和模型 model

# 标准MBeans
* 最简单的一种，缺点是不灵活，耦合性高
* 步骤：
    * 创建一个接口,名为你的类名加上后缀MBean。例如,如果要管理的Java类是 Car,接口名就为CarMBean。
    * 修改Java类,让它实现你创建的接口。
    * 创建一个代理,该代理必须包括一个MBean服务器。
    * 为你的MBean创建一个ObjectName。
    * 初始化MBean服务器。
    * 向MBean服务器注册MBean。

```java
package lifecycle.jmx;

// 1. 实现CarMBean接口
package lifecycle.jmx2.standardMBean;

/**
 * Created by xinyuan.zhang on 11/12/17.
 */
public class Car implements CarMBean {
    private String color = "red";
    public String getColor() {
        return color;
    }
    public void setColor(String color) {
        this.color = color;
    }
    public void drive() {
        System.out.println("Baby you can drive my car.");
    }
}
```
```java
package lifecycle.jmx;

// 2. 接口中声明所有需要暴露的方法
package lifecycle.jmx2.standardMBean;

/**
 * Created by xinyuan.zhang on 11/12/17.
 */
public interface CarMBean {

    public String getColor();
    public void setColor(String color);
    public void drive();
}
```
```java
package lifecycle.jmx2.standardMBean;

/**
 * Created by xinyuan.zhang on 11/12/17.
 */

import javax.management.Attribute;
import javax.management.MBeanServer;
import javax.management.MBeanServerFactory;
import javax.management.ObjectName;
import java.util.HashMap;
import java.util.Map;

/*

 */
public class StandardAgent {
    private MBeanServer standardMBeanServer = null;
    private static String managedResourceClassName =
            "lifecycle.jmx2.standardMBean.Car";

    public StandardAgent() {
        standardMBeanServer =MBeanServerFactory.createMBeanServer();

    }

    // create MBeanServer
    public MBeanServer getMBeanServer() {

        return standardMBeanServer;
    }

    // create Object Name
    public ObjectName createObjectName(String name) {
        ObjectName objectName = null;
        try {
            objectName = new ObjectName(name);
        } catch (Exception e) {
        }
        return objectName;
    }

    // create MBean
    public void createMBean(ObjectName objectName,
                                    String managedResourceClassName) {
        try {
            standardMBeanServer.createMBean(managedResourceClassName, objectName);
        } catch (Exception e) {
        }
    }


    public static void main(String[] args) {
        // create Agent
        StandardAgent agent = new StandardAgent();
        // create MBeanServer
        MBeanServer mBeanServer = agent.getMBeanServer();

        // create objectName
        String domain = mBeanServer.getDefaultDomain();
        ObjectName objectName = agent.createObjectName(domain + ":type=" + managedResourceClassName);

        // create MBean
        agent.createMBean(objectName, managedResourceClassName);

        // manage MBean
        try {
            Attribute colorAttribute = new Attribute("Color", "blue");
            mBeanServer.setAttribute(objectName, colorAttribute);
            System.out.println(mBeanServer.getAttribute(objectName,
                    "Color"));
            mBeanServer.invoke(objectName, "drive", null, null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
```
blue
Baby you can drive my car.
```


# 动态MBeans
对于很多已有的实现，其Coding Convention并不符合标准MBean的要求。重构所有这些 SubAgent 以符合标准 MBean 标准既费力也不实际。JMX 中给出了动态（Dynamic） MBean 的概念，MBServer 不再依据 Coding Convention 而是直接查询动态 MBean 给出的元数据（meta data）以获得 MBean 的对外接口。

```java
package lifecycle.jmx2.dynamicMBean;

import javax.management.*;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.Iterator;

/**
 * Created by xinyuan.zhang on 11/19/17.
 */
public class CarDynamic implements DynamicMBean {


    public Object getAttribute(String attribute)
            throws AttributeNotFoundException, MBeanException,
            ReflectionException {

        try {
            Field attr = this.getClass().getDeclaredField(attribute);
            // this make the private attribute accessible
            attr.setAccessible(true);
            return attr.get(this);
        } catch (Exception e) {
            // eatting all
        }
        return null;
    }

    public AttributeList getAttributes(String[] attributes) {
        AttributeList attrList = new AttributeList();
        try {
            Attribute attribute = null;
            for (String attr : attributes) {
                attribute = new Attribute(attr, getAttribute(attr));
                attrList.add(attribute);
            }
        } catch (Exception e) {
            // eatting all
        }
        return attrList;
    }

    public Object invoke(String actionName, Object[] params, String[] signature)
            throws MBeanException, ReflectionException {
        try {
            // first, constructing the param types
            Class[] paramTypes = new Class[signature.length];
            int i = 0;
            for (String sig : signature) {
                paramTypes[i++] = this.getClass().getClassLoader().loadClass(
                        sig);
            }

            // find the method and invoke it
            Method md = this.getClass().getMethod(actionName, paramTypes);
            return md.invoke(this, params);
        } catch (Exception e) {
            // eatting all
        }
        return null;
    }

    public void setAttribute(Attribute attribute)
            throws AttributeNotFoundException, InvalidAttributeValueException,
            MBeanException, ReflectionException {
        try {
            Field attr = this.getClass().getDeclaredField(attribute.getName());
            // this make the private attribute accessible
            attr.setAccessible(true);
            attr.set(this, attribute.getValue());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public AttributeList setAttributes(AttributeList attributes) {
        try {
            Attribute attribute = null;
            for (int i = 0; i < attributes.size(); i++) {
                attribute = (Attribute) attributes.get(i);
                setAttribute(attribute);
            }
        } catch (Exception e) {
            // eatting all
        }
        return attributes;
    }

    public MBeanInfo getMBeanInfo() {
        // exposes the attribute name
        MBeanAttributeInfo attributeInfo = new MBeanAttributeInfo("color",
                "java.lang.String", "color exposed", true, true, false);

        // exposes the printName operation
        MBeanOperationInfo operationInfo = new MBeanOperationInfo("drive",
                "drive car info ", new MBeanParameterInfo[0], "void", 2);

        return new MBeanInfo(this.getClass().getName(),
                "The manageable resource is DynamicImpl",
                new MBeanAttributeInfo[] { attributeInfo },
                new MBeanConstructorInfo[0],
                new MBeanOperationInfo[] { operationInfo },
                new MBeanNotificationInfo[0]);
    }

    private String color = "red";
    public String getColor() {
        return color;
    }
    public void setColor(String color) {
        this.color = color;
    }
    public void drive() {
        System.out.println("Baby you can drive my "+color+" car.");
    }

}
```
```java
package lifecycle.jmx2.dynamicMBean;

import com.sun.jdmk.comm.HtmlAdaptorServer;
import lifecycle.jmx.dynamic.HelloDynamic;

import javax.management.*;
import java.lang.management.ManagementFactory;

/**
 * Created by xinyuan.zhang on 11/19/17.
 */
public class DynamicAgent {

    public static void main(String[] args) throws MalformedObjectNameException, NullPointerException, InstanceAlreadyExistsException, MBeanRegistrationException, NotCompliantMBeanException {

        //create mbean server
        MBeanServer server = ManagementFactory.getPlatformMBeanServer();

        //create object name
        ObjectName objectName = new ObjectName("jmx:name=dynamicBeanCar");

        //create mbean and register mbean
        server.registerMBean(new CarDynamic(), objectName);

        //create adaptor, adaptor is just a form as show mbean. It has no relation to specific mbean.
        HtmlAdaptorServer adaptor  = new HtmlAdaptorServer();
        //create adaptor name
        ObjectName adaptorName = new ObjectName("jmxAdaptor:name=adaptor,port=5050");
        //register adaptor and adaptor name
        server.registerMBean(adaptor, adaptorName);

        adaptor.setPort(9999);
        adaptor.start();
        System.out.println("....................jmx server start....................");
    }
}
```

# 模型MBeans
## ModelMBean接口和RequiredModelMBean类
跟标准MBean不同,使用模型MBean的时候,不需要编写任何接口。而是使用javax.management.modelmbean.ModelMBean接口来表示模型MBean。
只需要实现这个接口即可,而javax.management.modelmbean.RequiredModelMBean 类提供了该接口的默认实现。
## ModelMBeanInfo
要编写模型MBean的最大挑战是告诉你的模型MBean对象哪些属性和方法要暴露给代理。

可以通过创建javax.management.modelmbean.ModelMBeanInfo对象实现这一点。

ModelMBeanInfo对象用于描述暴露给代理的构造器、属性、操作以 及事件监听器。创建一个ModelMBeanInfo对象是一个复杂的工作,但是一旦你做好了一个,你只需要将其跟ModelMBean对象关联即可。

* javax.management.modelmbean.ModelMBeanConstructorInfo 表示属性使用 
* javax.management.modelmbean.ModelMBeanAttributeInfo 表示操作用 
* javax.management.modelmbean.ModelMBeanOperationInfo 表示监听器用 
* javax.management.modelmbean.ModelMBeanNotificationInfo 表示通知使用

### ModelMBeanInfoSupport
JMX 提供了 ModelMBeanInfo 的默认实现: javax.management.modelmbean.ModelMBeanInfoSupport 类
```java
/**
* Creates a ModelMBeanInfoSupport with the provided information,
* but the descriptor is a default.
* The default descriptor is: name=className, descriptorType="mbean",
* displayName=className, persistPolicy="never", log="F", visibility="1"
*
* @param className classname of the MBean
* @param description human readable description of the
* ModelMBean
* @param attributes array of ModelMBeanAttributeInfo objects
* which have descriptors
* @param constructors array of ModelMBeanConstructorInfo
* objects which have descriptors
* @param operations array of ModelMBeanOperationInfo objects
* which have descriptors
* @param notifications array of ModelMBeanNotificationInfo
* objects which have descriptors
*/
public ModelMBeanInfoSupport(String className,
    String description,
    ModelMBeanAttributeInfo[] attributes,
    ModelMBeanConstructorInfo[] constructors,
    ModelMBeanOperationInfo[] operations,
    ModelMBeanNotificationInfo[] notifications)
```
* 第一个参数className表示MBean的类名
* 第二个参数description表示供人阅读的MBean的描述
* 第三个参数attributes表示ModelMBeanAttributeInfo数组，
    ```java
    /**
    * Constructs a ModelMBeanAttributeInfo object.
    *
    * @param name The name of the attribute
    * @param type The type or class name of the attribute
    * @param description A human readable description of the attribute.
    * @param isReadable True if the attribute has a getter method, false otherwise.
    * @param isWritable True if the attribute has a setter method, false otherwise.
    * @param isIs True if the attribute has an "is" getter, false otherwise.
    * @param descriptor An instance of Descriptor containing the
    * appropriate metadata for this instance of the Attribute. If
    * it is null then a default descriptor will be created.  If
    * the descriptor does not contain the field "displayName" this field
    * is added in the descriptor with its default value.
    * @exception RuntimeOperationsException Wraps an
    * IllegalArgumentException. The descriptor is invalid, or descriptor
    * field "name" is not equal to name parameter, or descriptor field
    * "descriptorType" is not equal to "attribute".
    *
    */
    public ModelMBeanAttributeInfo(String name,
                                    String type,
                                    String description,
                                    boolean isReadable,
                                    boolean isWritable,
                                    boolean isIs,
                                    Descriptor descriptor)
    ```

    * name:属性名
    * type:属性类型或类名
    * description:供人阅读的属性描述
    * isReadable:该属性是否有一个getter方法
    * isWritable:该属性是否有一个setter方法
    * isIs:该属性是否有一个is getter方法
    * descriptor:Descriptor实例，包含属性的元数据，如果为null，则创建一个默认的descriptor
* 第四个参数constructors表示ModelMBeanConstructorInfo数组
* 第五个参数operations表示ModelMBeanOperationInfo数组，
    ```java
    /**
    * Constructs a ModelMBeanOperationInfo object.
    *
    * @param name The name of the method.
    * @param description A human readable description of the operation.
    * @param signature MBeanParameterInfo objects describing the
    * parameters(arguments) of the method.
    * @param type The type of the method's return value.
    * @param impact The impact of the method, one of INFO, ACTION,
    * ACTION_INFO, UNKNOWN.
    * @param descriptor An instance of Descriptor containing the
    * appropriate metadata for this instance of the
    * MBeanOperationInfo. If it is null then a default descriptor
    * will be created.  If the descriptor does not contain
    * fields "displayName" or "role",
    * the missing ones are added with their default values.
    *
    * @exception RuntimeOperationsException Wraps an
    * IllegalArgumentException. The descriptor is invalid; or
    * descriptor field "name" is not equal to
    * operation name; or descriptor field "DescriptorType" is
    * not equal to "operation"; or descriptor optional
    * field "role" is present but not equal to "operation", "getter", or
    * "setter".
    */

    public ModelMBeanOperationInfo(String name,
                                    String description,
                                    MBeanParameterInfo[] signature,
                                    String type,
                                    int impact,
                                    Descriptor descriptor)
    ```
    * name:方法名
    * description:供人阅读的操作描述
    * signature：MBeanParameterInfo数组，描述方法参数
    * type:返回值类型
    * impact:方法的作用，有如下取值
        * INFO
        * ACTION
        * ACTION_INFO
        * UNKNOWN
    * descriptor:包含该 MBeanOperationInfo 对象的元数据的 Deccriptor
* 第五个参数表示ModelMBeanNotificationInfo数组
## 关联ModelMBean到ModelMBeanInfo
使用RequiredModelMBean作为你的ModelMBean的实现,有两种方法可以将你的 ModelMBean关联到ModelMBeanInfo:
* 传递一个 ModelMBeanInfo 对象给 RequiredModelMBean 的构造函数
* 传递一个 ModelMBeanInfo 对象给 RequiredModelMBean 对象的setModelMBeanInfo方法。

然后需要创建一个ObjectName对象并经模型MBean注册到MBean服务器。

例
```java
package lifecycle.jmx2.modelMBean;

/**
 * Created by xinyuan.zhang on 11/19/17.
 */
public class Car {
    private String color = "red";
    public String getColor() {
        return color;
    }
    public void setColor(String color) {
        this.color = color;
    }
    public void drive() {
        System.out.println("Baby you can drive my car.");
    }
}
```
```java
package lifecycle.jmx2.modelMBean;


import javax.management.*;
import javax.management.modelmbean.*;


/**
 * Created by xinyuan.zhang on 11/19/17.
 */
public class ModelAgent {

    private MBeanServer ModelMBeanServer = null;
    public ModelAgent() {
        // ModelMBeanServer = ManagementFactory.getPlatformMBeanServer();
        ModelMBeanServer =MBeanServerFactory.createMBeanServer();

    }

    public MBeanServer getMBeanServer() {
        return ModelMBeanServer;
    }

    private ObjectName createObjectName(String name) {
        ObjectName objectName = null;
        try {
            objectName = new ObjectName(name);
        }
        catch (MalformedObjectNameException e) {
            e.printStackTrace();
        }
        return objectName;
    }

    private ModelMBean createMBean(String managedResourceClassName) {
        ModelMBeanInfo mBeanInfo = createMBeanInfo(managedResourceClassName);
        RequiredModelMBean modelMBean = null;
        try {
            modelMBean = new RequiredModelMBean(mBeanInfo);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        return modelMBean;
    }

    private ModelMBeanInfo createMBeanInfo(String managedResourceClassName) {
        ModelMBeanInfo mBeanInfo = null;
        ModelMBeanAttributeInfo[] attributes = new
                ModelMBeanAttributeInfo[1];
        ModelMBeanOperationInfo[] operations = new
                ModelMBeanOperationInfo[3];
        try {
            attributes[0] = new ModelMBeanAttributeInfo("Color",
                    "java.lang.String",
                    "the color.", true, true, false, null);
            operations[0] = new ModelMBeanOperationInfo("drive",
                    "the drive method",
                    null, "void", MBeanOperationInfo.ACTION, null);
            operations[1] = new ModelMBeanOperationInfo("getColor",
                    "get color attribute",
                    null, "java.lang.String", MBeanOperationInfo.ACTION, null);
            Descriptor setColorDesc = new DescriptorSupport(new String[] {
                    "name=setColor", "descriptorType=operation",
                    "class=" + managedResourceClassName, "role=operation"});
            MBeanParameterInfo[] setColorParams = new MBeanParameterInfo[] { (new MBeanParameterInfo("new color", "java.lang.String",
                    "new Color value") )} ;
            operations[2] = new ModelMBeanOperationInfo("setColor",
                    "set Color attribute", setColorParams, "void",
                    MBeanOperationInfo.ACTION, setColorDesc);
            mBeanInfo  = new ModelMBeanInfoSupport(managedResourceClassName,
                    null, attributes, null, operations, null);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        return mBeanInfo;
    }

    public static void main(String[] args) {

        try {
            // create Agent
            ModelAgent agent = new ModelAgent();
            // create MBeanServer
            MBeanServer mBeanServer = agent.getMBeanServer();
            Car car = new Car();

            // create objectName
            String domain = mBeanServer.getDefaultDomain();
            ObjectName objectName = agent.createObjectName(domain +":name=Car");

            // create MBean
            String managedResourceClassName = "lifecycle.jmx2.modelMBean.Car";
            ModelMBean modelMBean = agent.createMBean(managedResourceClassName);
            modelMBean.setManagedResource(car, "ObjectReference");
            mBeanServer.registerMBean(modelMBean, objectName);

            // // manage the bean
            Attribute attribute = new Attribute("Color", "green");
            mBeanServer.setAttribute(objectName, attribute);
            String color = (String) mBeanServer.getAttribute(objectName,
                    "Color");
            System.out.println("Color:" + color);
            attribute = new Attribute("Color", "blue");
            mBeanServer.setAttribute(objectName, attribute);
            color = (String) mBeanServer.getAttribute(objectName, "Color");
            System.out.println("Color:" + color);
            mBeanServer.invoke(objectName, "drive", null, null);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}

```
# Commons Modeler library
创建ModelMBeanInfo对象十分繁琐，apache 的Commons Modeler library简化了这一流程。

使用Commons Modeler library,不需要再使用createModelMBeanInfo对象。对模型MBean的描述被封住到org.apache.catalina.modeler.ManagedBean对象中。

不需要再编写代码来暴露属性和操作，只需要编写一个XMl文档列出你想创建的MBean、对于每个MBean,需要写出MBean类和管理资源的完全限定名,以及暴露的方法和属性。然后使用 org.apache.commons.modeler.Registry对象读取 XML文档,就可以创建一个包括所有XML文档中描述的所有 ManagedBean实例的MBeanServer实例。

然后可以调用ManagedBean实例的 createMBean方法来创建一个模型 MBean。其它的工作就是平常需要做的了。需要创建 ObjectName实例并将MBean注册到MBean服务器。

**Commons Modeler**独立项目为
```xml
<dependency>
    <groupId>commons-modeler</groupId>
    <artifactId>commons-modeler</artifactId>
    <version>2.0.1</version>
</dependency>
```
**在Tomcat 9中，位于package org.apache.tomcat.util.modeler包中**

## MBean Descriptor
一个MBean描述符是用于描述MBean服务器管理的模型MBean的XML文档。

例如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<mbeans-descriptors>

    <mbean name="NamingResourcesImpl"
           className="org.apache.catalina.mbeans.NamingResourcesMBean"
           description="Holds and manages the naming resources defined in the
                       J2EE Enterprise Naming Context and their associated
                       JNDI context"
           domain="Catalina"
           group="Resources"
           type="org.apache.catalina.deploy.NamingResourcesImpl">

        <attribute name="container"
                   description="The container with which the naming resources are associated."
                   type="java.lang.Object"
                   writeable="false"/>

        <attribute name="environments"
                   description="MBean Names of the set of defined environment entries
                       for this web application"
                   type="[Ljava.lang.String;"
                   writeable="false"/>

        <attribute name="resources"
                   description="MBean Names of all the defined resource references
                       for this application."
                   type="[Ljava.lang.String;"
                   writeable="false"/>

        <attribute name="resourceLinks"
                   description="MBean Names of all the defined resource link references
                       for this application."
                   type="[Ljava.lang.String;"
                   writeable="false"/>

        <operation name="addEnvironment"
                   description="Add an environment entry for this web application"
                   impact="ACTION"
                   returnType="void">
            <parameter name="envName"
                       description="New environment entry name"
                       type="java.lang.String"/>
            <parameter name="type"
                       description="New environment entry type"
                       type="java.lang.String"/>
            <parameter name="value"
                       description="New environment entry value"
                       type="java.lang.String"/>
        </operation>

        <operation name="addResource"
                   description="Add a resource reference for this web application"
                   impact="ACTION"
                   returnType="void">
            <parameter name="resourceName"
                       description="New resource reference name"
                       type="java.lang.String"/>
            <parameter name="type"
                       description="New resource reference type"
                       type="java.lang.String"/>
        </operation>

        <operation name="addResourceLink"
                   description="Add a resource link reference for this web application"
                   impact="ACTION"
                   returnType="void">
            <parameter name="resourceLinkName"
                       description="New resource reference name"
                       type="java.lang.String"/>
            <parameter name="type"
                       description="New resource reference type"
                       type="java.lang.String"/>
        </operation>

        <operation name="removeEnvironment"
                   description="Remove any environment entry with the specified name"
                   impact="ACTION"
                   returnType="void">
            <parameter name="envName"
                       description="Name of the environment entry to remove"
                       type="java.lang.String"/>
        </operation>

        <operation name="removeResource"
                   description="Remove any resource reference with the specified name"
                   impact="ACTION"
                   returnType="void">
            <parameter name="resourceName"
                       description="Name of the resource reference to remove"
                       type="java.lang.String"/>
        </operation>

        <operation name="removeResourceLink"
                   description="Remove any resource link reference with the specified name"
                   impact="ACTION"
                   returnType="void">
            <parameter name="resourceLinkName"
                       description="Name of the resource reference to remove"
                       type="java.lang.String"/>
        </operation>

    </mbean>


</mbeans-descriptors>
```
它的根元素是 mbeans-descriptors
```
<mbeans-descriptors>
...
</mbeans-descriptors>
```
在 mbeans-descriptors 标签内的元素师mbean元素,每一个表示一个模型MBean。 Mbean元素包括表示属性、操作、构造器、监听器的元素。

### mbean
Mbean元素描述模型Mbean,它包括构造相应的ModelMBeanInfo对象的信息, mbean元素如下定义
```xml
<!ELEMENT mbean (descriptor?, attribute*, constructor*, notification*, operation*)>
```
* Mbean可以有选择性的descriptor元素
    * 0 或多个 attribute 元素,
    * 0 或多个 constructor 元素,
    * 0 或多个 notification 元素,
    * 0 或多个 opertion 元素。
#### property
* className
    * Fully qualified Java class name of the ModelMBean implementation. If this attribute is not present, the org.apache.commons.modeler.BaseModelMBean will be used. 
* description
    * A description of this model MBean.
* domain
    * The MBean server's domain in which the ModelMBean created by this managed bean should be registered, when creating its ObjectName
* group
    * Optional name of a "grouping classification" that can be used to select groups of similar MBean implementation classes. 
* name
    * A name that uniquely identifies this model MBean. Normally, the base class name of the corresponding server component is used.
* type
    * Fully qualified Java class name of the managed resource implementation class.
#### attribute
使用attribute元素描述MBean的JavaBean属性。Attribute元素可以有选择性的decriptor元素和如下属性:
* description
    * A description of this attribute.
* displayName
    * The display name of this attribute.
* getMethod
    * The getter method of the property represented by the attribute element.
* is
    * A boolean value indicating whether or not this attribute is a boolean with an is getter method. By default, the value of the is attribute is false.
* name
    * The name of this JavaBeans property.
* readable
    * A boolean value indicating whether or not this attribute is readable by management applications. By default, the value of readable is true.
* setMethod
    * The setter method of the property represented by this attribute element.
* type
    * The fully qualified Java class name of this attribute.
* writeable
    * A boolean value indicating whether or not this attribute can be written by management applications. By default, this is set to true.
#### operation
Operation 元素描述暴露给管理程序的 public 方法,它可以有 0 或多个参数子 元素,有如下属性:
* description
    * The description of this operation
* impact
    * This attribute indicates the impact of this method. The possible values are ACTION (write like), ACTION-INFO (write+read like), INFO (read like), or UNKNOWN.
* name
    * The name of this public method.
* returnType
    * The fully qualified Java class name of the return type of this method.
##### parameter
Parameter 元素用于描述传递给构造函数或操作的参数,它可以有如下属性:
* description
    * The description of this parameter
* name
    * The name of this parameter
* type
    * The fully qualified Java class name of this parameter
## 例子
```java
package lifecycle.jmx.modeler;

/**
 * Created by xinyuan.zhang on 11/15/17.
 */
public class Car {
    public Car() {
        System.out.println("Car constructor");
    }
    private String color = "red";
    public String getColor() {
        return color;
    }
    public void setColor(String color) {
        this.color = color;
    }
    public void drive() {
        System.out.println("Baby you can drive my car.");
    }
}
```
```java
package lifecycle.jmx.modeler;

/**
 * Created by xinyuan.zhang on 11/15/17.
 */

import java.io.InputStream;
import java.net.URL;
import javax.management.Attribute;
import javax.management.MalformedObjectNameException;
import javax.management.MBeanServer;
import javax.management.ObjectName;

import javax.management.modelmbean.ModelMBean;

import org.apache.commons.modeler.ManagedBean;
import org.apache.commons.modeler.Registry;

public class ModelAgent {
    private Registry registry;
    private MBeanServer mBeanServer;

    public ModelAgent() {
        registry = createRegistry();
        try {
            mBeanServer = Registry.getServer();
        } catch (Throwable t) {
            t.printStackTrace(System.out);
            System.exit(1);
        }
    }

    public MBeanServer getMBeanServer() {
        return mBeanServer;
    }

    // 1.根据descriptor.xml文件创建Registry实例
    public Registry createRegistry() {
        Registry registry = null;
        try {
            URL url = ModelAgent.class.getResource
                    ("/car-mbean-descriptor.xml");
            InputStream stream = url.openStream();
            Registry.loadRegistry(stream);
            stream.close();
            registry = Registry.getRegistry();
        } catch (Throwable t) {
            System.out.println(t.toString());
        }
        return (registry);
    }

    // 2.查找ManagedBean实例
    // 3.并创建MBean实例
    public ModelMBean createModelMBean(String mBeanName)
            throws Exception {
        System.out.println(mBeanName);
        ManagedBean managed = registry.findManagedBean(mBeanName);
        if (managed == null) {
            System.out.println("ManagedBean null");
            return null;
        }
        ModelMBean mbean = managed.createMBean();
        //ObjectName objectName = createObjectName();
        return mbean;
    }

    private ObjectName createObjectName() {
        ObjectName objectName = null;
        String domain = mBeanServer.getDefaultDomain();
        try {
            objectName = new ObjectName(domain + ":type=MyCar");
        } catch (MalformedObjectNameException e) {
            e.printStackTrace();
        }
        return objectName;
    }

    public static void main(String[] args) {
        ModelAgent agent = new ModelAgent();
        MBeanServer mBeanServer = agent.getMBeanServer();
        Car car = new Car();
        System.out.println("Creating ObjectName");
        ObjectName objectName = agent.createObjectName();
        try {
            ModelMBean modelMBean = agent.createModelMBean("myMBean");
            // 4.注册MBean到Server
            modelMBean.setManagedResource(car, "ObjectReference");
            mBeanServer.registerMBean(modelMBean, objectName);
        } catch (Exception e) {
            System.out.println(e.toString());
        }
        // manage the bean
        try {
            Attribute attribute = new Attribute("Color", "green");
            mBeanServer.setAttribute(objectName, attribute);
            String color = (String) mBeanServer.getAttribute(objectName,
                    "Color");
            System.out.println("Color:" + color);
            attribute = new Attribute("Color", "blue");
            mBeanServer.setAttribute(objectName, attribute);
            color = (String) mBeanServer.getAttribute(objectName, "Color");
            System.out.println("Color:" + color);
            mBeanServer.invoke(objectName, "drive", null, null);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
car-mbean-descriptor.xml
```xml
<?xml version="1.0"?>
<!DOCTYPE mbeans-descriptors PUBLIC
        "-//Apache Software Foundation//DTD Model MBeans Configuration File"
        "http://jakarta.apache.org/commons/dtds/mbeans-descriptors.dtd">
<mbeans-descriptors>
    <mbean name="myMBean"
           className="javax.management.modelmbean.RequiredModelMBean"
           description="The ModelMBean that manages our Car object"
           type="lifecycle.jmx.modeler.Car">
        <attribute name="Color"
                   description="The car color"
                   type="java.lang.String"
                   getMethod="getColor"
                   setMethod="setColor"/>
        <operation name="drive"
                   description="drive method"
                   impact="ACTION"
                   returnType="void">
        </operation>
        <operation name="getColor"
                   description="get method"
                   impact="ACTION"
                   returnType="void">

        </operation>
        <operation name="setColor"
                   description="set method"
                   impact="ACTION"
                   returnType="java.lang.String">
            <parameter name="color" description="the setColor parameter"
                       type="java.lang.String"/>
        </operation>
    </mbean>
</mbeans-descriptors>
```

