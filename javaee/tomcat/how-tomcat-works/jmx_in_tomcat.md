# Tomcat 中的JMX

关于JMX的基础知识以及Commons Modeler的说明，见[JMX详解](../../jmx/basic.md)

Tomcat使用了Commons Modeler library来创建注册MBean。

参考[JMX详解](../../jmx/basic.md)中使用Commons Modeler library创建并注册MBean的过程：
 ## 1. 根据mbeans-descriptors.xml创建Registry实例
* BootStrap时，将会调用Catalina的[load()方法](./bootstrap.md)。该方法会根据conf/server.xml文件创建server实例。
* 在server.xml中有以下配置
```xml
<Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
```
* 因此GlobalResourcesLifecycleListener类将会被加载。
* GlobalResourcesLifecycleListener类拥有一个static的属性registry,也将被加载
```java
/**
    * The configuration information registry for our managed beans.
    */
protected static final Registry registry = MBeanUtils.createRegistry();
```
* MBeanUtils.createRegistry()方法如下
```java
    // -------------------- Static methods  --------------------
// Factories

/**
    * Factory method to create (if necessary) and return our
    * <code>Registry</code> instance.
    *
    * The current version uses a static - future versions could use
    * the thread class loader.
    *
    * @param key Support for application isolation. If null, the context class
    * loader will be used ( if setUseContextClassLoader is called ) or the
    * default registry is returned.
    * @param guard Prevent access to the registry by untrusted components
    * @return the registry
    * @since 1.1
    */
public static synchronized Registry createRegistry() {

    if (registry == null) {
        registry = Registry.getRegistry(null, null);
        ClassLoader cl = MBeanUtils.class.getClassLoader();

        registry.loadDescriptors("org.apache.catalina.mbeans",  cl);
        registry.loadDescriptors("org.apache.catalina.authenticator", cl);
        registry.loadDescriptors("org.apache.catalina.core", cl);
        registry.loadDescriptors("org.apache.catalina", cl);
        registry.loadDescriptors("org.apache.catalina.deploy", cl);
        registry.loadDescriptors("org.apache.catalina.loader", cl);
        registry.loadDescriptors("org.apache.catalina.realm", cl);
        registry.loadDescriptors("org.apache.catalina.session", cl);
        registry.loadDescriptors("org.apache.catalina.startup", cl);
        registry.loadDescriptors("org.apache.catalina.users", cl);
        registry.loadDescriptors("org.apache.catalina.ha", cl);
        registry.loadDescriptors("org.apache.catalina.connector", cl);
        registry.loadDescriptors("org.apache.catalina.valves",  cl);
        registry.loadDescriptors("org.apache.catalina.storeconfig",  cl);
        registry.loadDescriptors("org.apache.tomcat.util.descriptor.web",  cl);
    }
    return (registry);

}
```
* 由此就获得了registry实例。
## 2. 查找ManagedBean实例，创建MBean实例，并将MBean注册到MBean Server
* Bootstrap处理start命令，会通过反射调用Catalina的setAwait(),load(),start()方法
    * Catalina的load()方法将会调用Server的init()方法，启动生命周期的初始化工作。
* Server的init()方法首先被调用：Server实现了Lifecycle接口，实际调用的是父类LifecycleBase的init()方法，而该方法将调用StandardServer的模板方法initInternal()。
* initInternal()方法除了创建并注册Server对应的MBean外，还将创建并注册Service对应的MBean，以及其他Component对应的MBean。
```java
    /**
     * Invoke a pre-startup initialization. This is used to allow connectors
     * to bind to restricted ports under Unix operating environments.
     */
    @Override
    protected void initInternal() throws LifecycleException {

        /*
        调用父类LifecycleMBeanBase的initInternal()方法，创建StandardServer对应的MBean，并将其注册到MBean Server。
        */
        super.initInternal();

        // Register global String cache
        // Note although the cache is global, if there are multiple Servers
        // present in the JVM (may happen when embedding) then the same cache
        // will be registered under multiple names
        // 创建global string cache对应的MBean，并注册
        onameStringCache = register(new StringCache(), "type=StringCache");

        // Register the MBeanFactory
        // 创建MBeanFactory对应的MBean，并注册
        MBeanFactory factory = new MBeanFactory();
        factory.setContainer(this);
        onameMBeanFactory = register(factory, "type=MBeanFactory");

        // Register the naming resources
        // 创建naming resources对应的MBean，并注册
        globalNamingResources.init();

        // Populate the extension validator with JARs from common and shared
        // class loaders
        if (getCatalina() != null) {
            ClassLoader cl = getCatalina().getParentClassLoader();
            // Walk the class loader hierarchy. Stop at the system class loader.
            // This will add the shared (if present) and common class loaders
            while (cl != null && cl != ClassLoader.getSystemClassLoader()) {
                if (cl instanceof URLClassLoader) {
                    URL[] urls = ((URLClassLoader) cl).getURLs();
                    for (URL url : urls) {
                        if (url.getProtocol().equals("file")) {
                            try {
                                File f = new File (url.toURI());
                                if (f.isFile() &&
                                        f.getName().endsWith(".jar")) {
                                    ExtensionValidator.addSystemResource(f);
                                }
                            } catch (URISyntaxException e) {
                                // Ignore
                            } catch (IOException e) {
                                // Ignore
                            }
                        }
                    }
                }
                cl = cl.getParent();
            }
        }
        // Initialize our defined Services
        // 初始化所有的Services，该过程将创建services对应的MBean，并注册
        for (int i = 0; i < services.length; i++) {
            services[i].init();
        }
    }
```
### 以Server为例，其MBean创建并注册的过程如下

LifecycleMBeanBase的initInternal()方法如下：
```java
/* Cache components of the MBean registration. */
private String domain = null;
private ObjectName oname = null;
protected MBeanServer mserver = null;

/**
* Sub-classes wishing to perform additional initialization should override
* this method, ensuring that super.initInternal() is the first call in the
* overriding method.
*/
@Override
protected void initInternal() throws LifecycleException {

// If oname is not null then registration has already happened via
// preRegister().
if (oname == null) {
    // 获取MBean 服务器
    mserver = Registry.getRegistry(null, null).getMBeanServer();
    
    oname = register(this, getObjectNameKeyProperties());
}
}
```
register()方法如下
```java
/**
    * Utility method to enable sub-classes to easily register additional
    * components that don't implement {@link JmxEnabled} with an MBean server.
    * <br>
    * Note: This method should only be used once {@link #initInternal()} has
    * been called and before {@link #destroyInternal()} has been called.
    *
    * @param obj                       The object the register
    * @param objectNameKeyProperties   The key properties component of the
    *                                  object name to use to register the
    *                                  object
    *
    * @return  The name used to register the object
    */
protected final ObjectName register(Object obj,
        String objectNameKeyProperties) {

    // Construct an object name with the right domain
    StringBuilder name = new StringBuilder(getDomain());
    name.append(':');
    name.append(objectNameKeyProperties);

    ObjectName on = null;

    try {
        // 获取ObjectName实例
        on = new ObjectName(name.toString());
        
        Registry.getRegistry(null, null).registerComponent(obj, on, null);
    } catch (MalformedObjectNameException e) {
        log.warn(sm.getString("lifecycleMBeanBase.registerFail", obj, name),
                e);
    } catch (Exception e) {
        log.warn(sm.getString("lifecycleMBeanBase.registerFail", obj, name),
                e);
    }

    return on;
}
```
* registerComponent方法中查找到ManagedBean
* 查找到ManagedBean后，将由它创建MBean实例
* 创建完MBean，将其注册到Server中
```java
/**
    * Register a component
    *
    * @param bean The bean
    * @param oname The object name
    * @param type The registry type
    * @throws Exception Error registering component
    */
public void registerComponent(Object bean, ObjectName oname, String type)
        throws Exception
{
    if( log.isDebugEnabled() ) {
        log.debug( "Managed= "+ oname);
    }

    if( bean ==null ) {
        log.error("Null component " + oname );
        return;
    }

    try {
        if( type==null ) {
            type=bean.getClass().getName();
        }

        // 查找到ManagedBean
        ManagedBean managed = findManagedBean(null, bean.getClass(), type);

        // 创建MBean实例
        // The real mbean is created and registered
        DynamicMBean mbean = managed.createMBean(bean);

        if(  getMBeanServer().isRegistered( oname )) {
            if( log.isDebugEnabled()) {
                log.debug("Unregistering existing component " + oname );
            }
            getMBeanServer().unregisterMBean( oname );
        }
        // 注册MBean
        getMBeanServer().registerMBean( mbean, oname);
    } catch( Exception ex) {
        log.error("Error registering " + oname, ex );
        throw ex;
    }
}
```
