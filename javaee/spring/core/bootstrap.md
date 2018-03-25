# Spring启动过程
以一个SpringMvc Web App为例，讲解该过程

Tomcat Context启动时，会发布CONFIGURE_START_EVENT事件(启动子节点前)。

```java
protected synchronized void startInternal() throws LifecycleException {

...
// Notify our interested LifecycleListeners

// 发布Lifecycle.CONFIGURE_START_EVENT事件
// ContextConfig监听该事件以完成web容器初始化
// web应用部署符可来源于WEB-INF/web.xml,
// META-INF/web-fragment.xml,
// ServletContainerInitializer
// annotation
fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);

// Start our child containers, if not already started
// 启动子节点Wrapper
for (Container child : findChildren()) {
    if (!child.getState().isAvailable()) {
        child.start();
    }
}
...
}
```

ContextConfig会在该事件中扫描所有ServletContainerInitializer的实现类，并将其加到ServletContext中

```java
protected void webConfig() {
    ...
    // Step 3. Look for ServletContainerInitializer implementations
    if (ok) {
        processServletContainerInitializers();
    }
    ...
    // Step 11. Apply the ServletContainerInitializer config to the
    // context
    if (ok) {
        for (Map.Entry<ServletContainerInitializer,
                Set<Class<?>>> entry :
                    initializerClassMap.entrySet()) {
            if (entry.getValue().isEmpty()) {
                context.addServletContainerInitializer(
                        entry.getKey(), null);
            } else {
                context.addServletContainerInitializer(
                        entry.getKey(), entry.getValue());
            }
        }
    }
}
```
回到Tomcat Context启动中，接下来会调用ServletContainerInitializer的onStartup()方法
```java
protected synchronized void startInternal() throws LifecycleException {

...
// Notify our interested LifecycleListeners

// 发布Lifecycle.CONFIGURE_START_EVENT事件
// ContextConfig监听该事件以完成web容器初始化
// web应用部署符可来源于WEB-INF/web.xml,
// META-INF/web-fragment.xml,
// ServletContainerInitializer
// annotation
fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);

// Start our child containers, if not already started
// 启动子节点Wrapper
for (Container child : findChildren()) {
    if (!child.getState().isAvailable()) {
        child.start();
    }
}

...

// Call ServletContainerInitializers
// 主要用于可编程的方式添加Web应用的配置，如servlet,filter
for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :
    initializers.entrySet()) {
    try {
        entry.getKey().onStartup(entry.getValue(),
                getServletContext());
    } catch (ServletException e) {
        log.error(sm.getString("standardContext.sciFail"), e);
        ok = false;
        break;
    }
}
...
```
Spring提供了ServletContainerInitializer的实现org.springframework.web.SpringServletContainerInitializer
```
spring-framework-5.0.2.RELEASE/spring-web/src/main/resources/META-INF/services/javax.servlet.ServletContainerInitializer
```
SpringServletContainerInitializer将实际的配置任务交给实现了WebApplicationInitializer接口的类来完成。开发者使用springmvc时通过实现WebApplicationInitializer接口或者继承Spring提供的一个WebApplicationInitializer接口的基础实现类AbstractAnnotationConfigDispatcherServletInit来完成配置
```java
@Override
public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
        throws ServletException {

    List<WebApplicationInitializer> initializers = new LinkedList<>();

    if (webAppInitializerClasses != null) {
        for (Class<?> waiClass : webAppInitializerClasses) {
            // Be defensive: Some servlet containers provide us with invalid classes,
            // no matter what @HandlesTypes says...
            if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
                    WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                try {
                    initializers.add((WebApplicationInitializer)
                            ReflectionUtils.accessibleConstructor(waiClass).newInstance());
                }
                catch (Throwable ex) {
                    throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
                }
            }
        }
    }

    if (initializers.isEmpty()) {
        servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
        return;
    }

    servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
    AnnotationAwareOrderComparator.sort(initializers);
    for (WebApplicationInitializer initializer : initializers) {
        initializer.onStartup(servletContext);
    }
}
```
假设某个Web App使用SpringMvc提供如下配置
```java
public class SpittrWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[]{RootConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[]{WebConfig.class};
    }

    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

}
```
则Spring启动过程如下

在Tomcat Context启动过程中，调用SpringServletContainerInitializer的onStartup方法。

该方法将找到用户的配置类，即本例的SpittrWebAppInitializer。

由于该类继承自AbstractAnnotationConfigDispatcherServletInitializer，则根据SpringServletContainerInitializer的onStartup方法的最后一句initializer.onStartup(servletContext)，
将调用SpittrWebAppInitializer的onStartup()方法，

而SpittrWebAppInitializer并没有重写该方法，因此实际调用的是其父类AbstractAnnotationConfigDispatcherServletInitializer的onStartup方法。

这个方法中的两句话非常重要
```java
@Override
public void onStartup(ServletContext servletContext) throws ServletException {
    // 注册一个ServletContextListener监听器：ContextLoaderListener到servletContext中
    super.onStartup(servletContext);
    registerDispatcherServlet(servletContext);
}
```

继而调用其父类AbstractContextLoaderInitializer的onStartup方法
```java
@Override
public void onStartup(ServletContext servletContext) throws ServletException {
    registerContextLoaderListener(servletContext);
}
```
该方法将注册一个ServletContextListener监听器：ContextLoaderListener到servletContext中
```java
/**
    * Register a {@link ContextLoaderListener} against the given servlet context. The
    * {@code ContextLoaderListener} is initialized with the application context returned
    * from the {@link #createRootApplicationContext()} template method.
    * @param servletContext the servlet context to register the listener against
    */
protected void registerContextLoaderListener(ServletContext servletContext) {
    WebApplicationContext rootAppContext = createRootApplicationContext();
    if (rootAppContext != null) {
        ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);
        listener.setContextInitializers(getRootApplicationContextInitializers());
        servletContext.addListener(listener);
    }
    else {
        logger.debug("No ContextLoaderListener registered, as " +
                "createRootApplicationContext() did not return an application context");
    }
}
```
上述过程调用栈如下
```
at org.springframework.web.context.AbstractContextLoaderInitializer.onStartup(AbstractContextLoaderInitializer.java:50)
at org.springframework.web.servlet.support.AbstractDispatcherServletInitializer.onStartup(AbstractDispatcherServletInitializer.java:63)
at org.springframework.web.SpringServletContainerInitializer.onStartup(SpringServletContainerInitializer.java:172)
at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5094)
```
回到Tomcat Context的启动过程中，接下来会实例化各种监听器，并触发ServletContextListener.contextInitialized
```java
protected synchronized void startInternal() throws LifecycleException {

...
// Notify our interested LifecycleListeners

// 发布Lifecycle.CONFIGURE_START_EVENT事件
// ContextConfig监听该事件以完成web容器初始化
// web应用部署符可来源于WEB-INF/web.xml,
// META-INF/web-fragment.xml,
// ServletContainerInitializer
// annotation
fireLifecycleEvent(Lifecycle.CONFIGURE_START_EVENT, null);

// Start our child containers, if not already started
// 启动子节点Wrapper
for (Container child : findChildren()) {
    if (!child.getState().isAvailable()) {
        child.start();
    }
}

...

// Call ServletContainerInitializers
// 主要用于可编程的方式添加Web应用的配置，如servlet,filter
for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :
    initializers.entrySet()) {
    try {
        entry.getKey().onStartup(entry.getValue(),
                getServletContext());
    } catch (ServletException e) {
        log.error(sm.getString("standardContext.sciFail"), e);
        ok = false;
        break;
    }
}
...
// Configure and call application event listeners

// 实例化应用监听器（ApplicationListener）,分为
// 事件监听器(ServletContextAttributeListener，
// ServletRequestAttributeListener,
// ServletRequestListener,
// HttpSessionIdListener,
// HttpSessionAttributeListener)
// 生命周期监听器(HttpSessionListener,
// ServletContextListener)
// 这些监听器可以通过context配置文件，
// 可编程方式(ServletContainerInitializer),
// web.xml添加

// 并触发ServletContextListener.contextInitialized
if (ok) {
    if (!listenerStart()) {
        log.error(sm.getString("standardContext.listenerFail"));
        ok = false;
    }
}
```
```java
public boolean listenerStart() {
    ...
    for (int i = 0; i < instances.length; i++) {
        if (!(instances[i] instanceof ServletContextListener))
            continue;
        ServletContextListener listener =
            (ServletContextListener) instances[i];
        try {
            fireContainerEvent("beforeContextInitialized", listener);
            if (noPluggabilityListeners.contains(listener)) {
                listener.contextInitialized(tldEvent);
            } else {
                listener.contextInitialized(event);
            }
            fireContainerEvent("afterContextInitialized", listener);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            fireContainerEvent("afterContextInitialized", listener);
            getLogger().error
                (sm.getString("standardContext.listenerStart",
                                instances[i].getClass().getName()), t);
            ok = false;
        }
    }
    return ok;

}
```
这将调用到Spring注册到servletContext中的ServletContextListener监听器：ContextLoaderListener的contextInitialized方法
```java
@Override
public void contextInitialized(ServletContextEvent event) {
    initWebApplicationContext(event.getServletContext());
}
```
调用initWebApplicationContext来初始化Spring's web app context，从而调用refresh()方法正式启动IOC容器
```java
/**
    * Initialize Spring's web application context for the given servlet context,
    * using the application context provided at construction time, or creating a new one
    * according to the "{@link #CONTEXT_CLASS_PARAM contextClass}" and
    * "{@link #CONFIG_LOCATION_PARAM contextConfigLocation}" context-params.
    * @param servletContext current servlet context
    * @return the new WebApplicationContext
    * @see #ContextLoader(WebApplicationContext)
    * @see #CONTEXT_CLASS_PARAM
    * @see #CONFIG_LOCATION_PARAM
    */
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
    if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
        throw new IllegalStateException(
                "Cannot initialize context because there is already a root application context present - " +
                "check whether you have multiple ContextLoader* definitions in your web.xml!");
    }

    Log logger = LogFactory.getLog(ContextLoader.class);
    servletContext.log("Initializing Spring root WebApplicationContext");
    if (logger.isInfoEnabled()) {
        logger.info("Root WebApplicationContext: initialization started");
    }
    long startTime = System.currentTimeMillis();

    try {
        // Store context in local instance variable, to guarantee that
        // it is available on ServletContext shutdown.
        if (this.context == null) {
            this.context = createWebApplicationContext(servletContext);
        }
        if (this.context instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent ->
                    // determine parent for root web application context, if any.
                    ApplicationContext parent = loadParentContext(servletContext);
                    cwac.setParent(parent);
                }
                configureAndRefreshWebApplicationContext(cwac, servletContext);
            }
        }
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

        ClassLoader ccl = Thread.currentThread().getContextClassLoader();
        if (ccl == ContextLoader.class.getClassLoader()) {
            currentContext = this.context;
        }
        else if (ccl != null) {
            currentContextPerThread.put(ccl, this.context);
        }

        if (logger.isDebugEnabled()) {
            logger.debug("Published root WebApplicationContext as ServletContext attribute with name [" +
                    WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE + "]");
        }
        if (logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            logger.info("Root WebApplicationContext: initialization completed in " + elapsedTime + " ms");
        }

        return this.context;
    }
    catch (RuntimeException ex) {
        logger.error("Context initialization failed", ex);
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
        throw ex;
    }
    catch (Error err) {
        logger.error("Context initialization failed", err);
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, err);
        throw err;
    }
}
```
```java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
    if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
        // The application context id is still set to its original default value
        // -> assign a more useful id based on available information
        String idParam = sc.getInitParameter(CONTEXT_ID_PARAM);
        if (idParam != null) {
            wac.setId(idParam);
        }
        else {
            // Generate default id...
            wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                    ObjectUtils.getDisplayString(sc.getContextPath()));
        }
    }

    wac.setServletContext(sc);
    String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
    if (configLocationParam != null) {
        wac.setConfigLocation(configLocationParam);
    }

    // The wac environment's #initPropertySources will be called in any case when the context
    // is refreshed; do it eagerly here to ensure servlet property sources are in place for
    // use in any post-processing or initialization that occurs below prior to #refresh
    ConfigurableEnvironment env = wac.getEnvironment();
    if (env instanceof ConfigurableWebEnvironment) {
        ((ConfigurableWebEnvironment) env).initPropertySources(sc, null);
    }

    customizeContext(sc, wac);
    wac.refresh();
}
```
这个方法标志着IOC容器正式启动！
```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            initMessageSource();

            // Initialize event multicaster for this context.
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            onRefresh();

            // Check for listener beans and register them.
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            finishRefresh();
        }

        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                        "cancelling refresh attempt: " + ex);
            }

            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```
IoC容器的启动包括三个基本过程
* BeanDefinition的Resource定位
* BeanDefinition的Resource载入
* BeanDefinition的Resource注册

# BeanDefinition的Resource定位