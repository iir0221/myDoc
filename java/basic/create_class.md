# 获取一个类的几种方式

## Class.forName
动态加载类，即在运行时才加载类
```
forName
public static Class<?> forName(String name,
                               boolean initialize,
                               ClassLoader loader)
                        throws ClassNotFoundException
使用给定的类加载器，返回与带有给定字符串名的类或接口相关联的 Class 对象。
（以 getName 所返回的格式）给定一个类或接口的完全限定名，
此方法会试图定位、加载和链接该类或接口。
指定的类加载器用于加载该类或接口。
如果参数 loader 为 null，则该类通过引导类加载器加载。
只有 initialize 参数为 true 且以前未被初始化时，才初始化该类。

如果 name 表示一个基本类型或 void，则会尝试在未命名的包中定位用户定义的名为 name 的类。
因此，该方法不能用于获得表示基本类型或 void 的任何 Class 对象。

如果 name 表示一个数组类，则会加载但不初始化该数组类的组件类型。

例如，在一个实例方法中，表达式：

  Class.forName("Foo")
 
等效于：
  Class.forName("Foo", true, this.getClass().getClassLoader())
 
注意，此方法会抛出与加载、链接或初始化相关的错误， 
Java Language Specification 的第 12.2、12.3 和 12.4 节对此进行了详细说明。 
注意，此方法不检查调用者是否可访问其请求的类。

如果 loader 为 null，也存在安全管理器，并且调用者的类加载器不为 null，
则此方法通过 RuntimePermission("getClassLoader") 权限调用安全管理器的 checkPermission 方法，
以确保可以访问引导类加载器。

参数：
name - 所需类的完全限定名
initialize - 是否必须初始化类
loader - 用于加载类的类加载器
返回：
表示所需类的类对象
抛出：
LinkageError - 如果链接失败
ExceptionInInitializerError - 如果该方法激发的初始化失败
ClassNotFoundException - 如果指定的类加载器无法定位该类
从以下版本开始：
1.2
另请参见：
forName(String), ClassLoader
```
### 只有 initialize 参数为 true 且以前未被初始化时，才初始化该类。
```java
For example, in an instance method the expression:

Class.forName("Foo")
is equivalent to:
Class.forName("Foo", true, this.getClass().getClassLoader())
```
* Class.forName("Foo")，默认为true(即执行static块)
* Class.forName("Foo",false, this.getClass().getClassLoader()),设置为false，则在newInstance()时才初始化
```java
/**
 * Created by xinyuan.zhang on 1/16/18.
 */
public class TestClassForName {

    public static void main(String[] args)  {

        TestClassForName testClassForName = new TestClassForName();
        testClassForName.test();
        //testClassForName.test2();
    }

    public void test() {
        try {
            Class car = Class.forName("Car");
            Class bus = Class.forName("Bus",false,this.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    public void test2() {
        try {
            Class car = Class.forName("Car");
            Class bus = Class.forName("Bus",false,this.getClass().getClassLoader());
            Bus bus1 = (Bus)bus.newInstance();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```
```java
/**
 * Created by xinyuan.zhang on 1/16/18.
 */
public class Car {

    static {
        System.out.println("car initialization");
    }
}
```
```java
/**
 * Created by xinyuan.zhang on 1/16/18.
 */
public class Bus {

    static {
        System.out.println("bus initialization");
    }
}
```
test
```
car initialization
```
test2
```
car initialization
bus initialization
```
#### JDBC中使用Class.forName("xxx")的意义
JDBC规范中明确要求Driver(数据库驱动)类必须向DriverManager注册自己。

以Mysql为例
```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {  
    // ~ Static fields/initializers  
    // ---------------------------------------------  
  
    //  
    // Register ourselves with the DriverManager  
    //  
    static {  
        try {  
            java.sql.DriverManager.registerDriver(new Driver());  
        } catch (SQLException E) {  
            throw new RuntimeException("Can't register driver!");  
        }  
    }  
  
    // ~ Constructors  
    // -----------------------------------------------------------  
  
    /** 
     * Construct a new driver and register it with DriverManager 
     *  
     * @throws SQLException 
     *             if a database error occurs. 
     */  
    public Driver() throws SQLException {  
        // Required for Class.forName().newInstance()  
    }  
}  
```
在使用JDBC连接MySQL数据库时，使用Class.forName("com.mysql.jdbc.Driver")就是为了向DriverManager注册自己；当然使用Class.forName("com.mysql.jdbc.Driver").newInstance()当然也没错，只是没有必要，因为后者还会生成Driver类的实例，而这个是我们没有用的，没有必要创建它。如果在Driver类中那个static块里面的部分写在了构造方法中，那么就必须使用Class.forName("com.mysql.jdbc.Driver").newInstance()来向DriverManager注册了。

### 如果参数 loader 为 null，则该类通过引导类加载器加载。
```java
For example, in an instance method the expression:

Class.forName("Foo")
is equivalent to:
Class.forName("Foo", true, this.getClass().getClassLoader())
```
* Class.forName(String className)使用的类加载器是调用者的类加载器，不一定是System类加载器
* Class.forName(String name,boolean initialize,ClassLoader loader)可以指定类加载器，推荐通过这种方式来获取类
```java
public class Test {
    public static void main(String[] args) {
        Class<?> clazz = null;
        try {
            clazz = Test.class.getClassLoader().loadClass("Test");
            Method method = clazz.getMethod("currentClassLoader");
            method.invoke(clazz.newInstance());
        } catch (Exception e) {
            e.printStackTrace();
        }

        ClassLoader classLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                InputStream in = getClass().getResourceAsStream(fileName);
                if (in == null) {
                    return super.loadClass(name);
                }
                byte[] b = null;
                try {
                    b = new byte[in.available()];
                    in.read(b);
                    in.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                return defineClass(name, b, 0, b.length);
            }
        };

        Class<?> clazz2 = null;
        try {
            clazz2 = classLoader.loadClass("Test");
            Method method = clazz2.getMethod("currentClassLoader");
            method.invoke(clazz2.newInstance());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void currentClassLoader() {
        try {
            System.out.println(Class.forName("Color").getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
```
sun.misc.Launcher$AppClassLoader@18b4aac2
Test$1@77459877
```

## 通过类的class属性 
不会初始化
```java
public void test3() {
    Class c = Car.class;
}

public void test4() {

    try {
        Class c = Car.class;
        Car car = (Car)c.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
}
```
test3
```
```
test4
```
car initialization
```
## 通过对象的getClass()方法