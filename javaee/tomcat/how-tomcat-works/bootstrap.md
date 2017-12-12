# Bootstrap
* Tomcat的组件均拥有start,stop等生命周期方法，拥有管理生命周期的特性，这一特性抽象为Lifecycle接口
* Tomcat所有组件均实现了Lifecycle接口：
    * **LifecycleBase**：Lifecycle的默认实现类是LifecycleBase
        * **init(),start(),stop(),destroy()方法的实现就在这个类中。**
        * **start()方法会检查LifecycleState是否为NEW,如果是，则将先调用init()方法首先进行初始化**
        * **这四个方法又会调用模板方法initInternal(),startInternal(),stopInternal(),destroyInternal()，这些模板方法由各个组件具体实现。**
    * **LifecycleMBeanBase**：抽象类LifecycleMBeanBase继承自LifecycleBase，并实现了接口JmxEnabled（public interface JmxEnabled extends MBeanRegistration）从而将自身注册为MBean(JMX)。Tomcat可利用管理工具对其进行动态维护，参见Tomcat文档
    [Monitoring and Managing Tomcat](http://tomcat.apache.org/tomcat-9.0-doc/monitoring.html)
    * StandardServer，StandardService通过继承LifecycleMBeanBase间接实现Lifecycle接口
    * ContainerBase通过继承LifecycleMBeanBase间接实现Lifecycle接口
        * 四个Container通过继承ContainerBase间接实现Lifecycle接口

## BootStrap启动:Tomcat入口类
```java
/**
    * Main method and entry point when starting Tomcat via the provided
    * scripts.
    *
    * @param args Command line arguments to be processed
    */
public static void main(String args[]) {

    if (daemon == null) {
        // Don't set daemon until init() has completed
        Bootstrap bootstrap = new Bootstrap();
        try {
            // 初始化ClassLoader,通过ClassLoader创建Catalina实例，并将其赋给属性catalinaDaemon
            bootstrap.init();
        } catch (Throwable t) {
            handleThrowable(t);
            t.printStackTrace();
            return;
        }
        daemon = bootstrap;
    } else {
        // When running as a service the call to stop will be on a new
        // thread so make sure the correct class loader is used to prevent
        // a range of class not found exceptions.
        Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
    }

    // 处理传入的命令，如果args参数为空，默认执行start
    try {
        String command = "start";
        if (args.length > 0) {
            command = args[args.length - 1];
        }

        if (command.equals("startd")) {
            args[args.length - 1] = "start";
            daemon.load(args);
            daemon.start();
        } else if (command.equals("stopd")) {
            args[args.length - 1] = "stop";
            daemon.stop();
        } else if (command.equals("start")) {
            daemon.setAwait(true);
            daemon.load(args);
            daemon.start();
        } else if (command.equals("stop")) {
            daemon.stopServer(args);
        } else if (command.equals("configtest")) {
            daemon.load(args);
            if (null==daemon.getServer()) {
                System.exit(1);
            }
            System.exit(0);
        } else {
            log.warn("Bootstrap: command \"" + command + "\" does not exist.");
        }
    } catch (Throwable t) {
        // Unwrap the Exception for clearer error reporting
        if (t instanceof InvocationTargetException &&
                t.getCause() != null) {
            t = t.getCause();
        }
        handleThrowable(t);
        t.printStackTrace();
        System.exit(1);
    }

}

```
start命令的处理调用了三个方法setAwait(),load(),start()。在这三个方法内部具体通过Catalina的实例catalinaDaemon属性，通过反射来调用catalinaDaemon的方法执行。以start()方法为例：
```java
/**
    * Start the Catalina daemon.
    * @throws Exception Fatal start error
    */
public void start() throws Exception {
    // 先判断catalinaDaemon有没有实例化，如果没有，则实例化
    if( catalinaDaemon==null ) 
        init();
    // 通过反射调用catalinaDaemon的start()方法
    Method method = catalinaDaemon.getClass()
                    .getMethod("start", (Class [] )null);
    method.invoke(catalinaDaemon, (Object [])null);

}
```
## Catalina启动
```
Startup/Shutdown shell program for Catalina. The following command line options are recognized:
-config {pathname} - Set the pathname of the configuration file to be processed. If a relative path is specified, it will be interpreted as relative to the directory pathname specified by the "catalina.base" system property. [conf/server.xml]
-help - Display usage information.
-nonaming - Disable naming support.
configtest - Try to test the config
start - Start an instance of Catalina.
stop - Stop the currently running instance of Catalina. Should do the same thing as Embedded, but using a server.xml file.
```
* Bootstrap会通过反射调用Catalina的setAwait(),load(),start()方法来处理shell start命令

### setAwait()方法
* 设置Tomcat启动完成后是否进入等待状态的flag：await属性，该属性会在start()方法中被用与判断是否进入等待状态
```java
public boolean isAwait() {
    return await;
}
```
### load()方法
* **根据配置文件conf/server.xml创建Server对象，并赋值给属性server**(即创建整个tomcat组件实例)
* 具体的解析操作由Apache开源项目Digester完成
* 调用server的init()方法，开始生命周期的初始化工作
```java
/**
* Start a new server instance.
*/
public void load() {

    long t1 = System.nanoTime();

    initDirs();

    // Before digester - it may be needed
    initNaming();

    // Create and execute our Digester
    Digester digester = createStartDigester();
    // 通过Digester创建Server实例,并赋值给属性server:
    InputSource inputSource = null;
    InputStream inputStream = null;
    File file = null;
    try {
        try {
            // 读取conf/server.xml文件
            file = configFile();
            inputStream = new FileInputStream(file);
            inputSource = new InputSource(file.toURI().toURL().toString());
        } catch (Exception e) {
            if (log.isDebugEnabled()) {
                log.debug(sm.getString("catalina.configFail", file), e);
            }
        }
        if (inputStream == null) {
            try {
                inputStream = getClass().getClassLoader()
                    .getResourceAsStream(getConfigFile());
                inputSource = new InputSource
                    (getClass().getClassLoader()
                    .getResource(getConfigFile()).toString());
            } catch (Exception e) {
                if (log.isDebugEnabled()) {
                    log.debug(sm.getString("catalina.configFail",
                            getConfigFile()), e);
                }
            }
        }

        // This should be included in catalina.jar
        // Alternative: don't bother with xml, just create it manually.
        if (inputStream == null) {
            try {
                inputStream = getClass().getClassLoader()
                        .getResourceAsStream("server-embed.xml");
                inputSource = new InputSource
                (getClass().getClassLoader()
                        .getResource("server-embed.xml").toString());
            } catch (Exception e) {
                if (log.isDebugEnabled()) {
                    log.debug(sm.getString("catalina.configFail",
                            "server-embed.xml"), e);
                }
            }
        }


        if (inputStream == null || inputSource == null) {
            if  (file == null) {
                log.warn(sm.getString("catalina.configFail",
                        getConfigFile() + "] or [server-embed.xml]"));
            } else {
                log.warn(sm.getString("catalina.configFail",
                        file.getAbsolutePath()));
                if (file.exists() && !file.canRead()) {
                    log.warn("Permissions incorrect, read permission is not allowed on the file.");
                }
            }
            return;
        }

        // 使用digester解析配置文件
        try {
            inputSource.setByteStream(inputStream);
            digester.push(this);
            digester.parse(inputSource);
        } catch (SAXParseException spe) {
            log.warn("Catalina.start using " + getConfigFile() + ": " +
                    spe.getMessage());
            return;
        } catch (Exception e) {
            log.warn("Catalina.start using " + getConfigFile() + ": " , e);
            return;
        }
    } finally {
        if (inputStream != null) {
            try {
                inputStream.close();
            } catch (IOException e) {
                // Ignore
            }
        }
    }

    // 
    getServer().setCatalina(this);
    getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
    getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());

    // Stream redirection
    initStreams();

    // Start the new server
    try {
        // 正式开启Tomcat生命周期,调用server的init()方法
        getServer().init();
    } catch (LifecycleException e) {
        if (Boolean.getBoolean("org.apache.catalina.startup.EXIT_ON_INIT_FAILURE")) {
            throw new java.lang.Error(e);
        } else {
            log.error("Catalina.start", e);
        }
    }

    long t2 = System.nanoTime();
    if(log.isInfoEnabled()) {
        log.info("Initialization processed in " + ((t2 - t1) / 1000000) + " ms");
    }
}
```
### start()方法
* 调用server的start()方法，开始启动工作
* 根据属性await判断是否让程序进入等待状态
```java
/**
    * Start a new server instance.
    */
public void start() {

    // 首先判断server是否实例化，如果没有，则实例化
    if (getServer() == null) {
        load();
    }

    if (getServer() == null) {
        log.fatal("Cannot start server. Server instance is not configured.");
        return;
    }

    long t1 = System.nanoTime();

    // Start the new server
    try {
        // 调用server的start()方法启动服务器
        getServer().start();
    } catch (LifecycleException e) {
        log.fatal(sm.getString("catalina.serverStartFail"), e);
        try {
            getServer().destroy();
        } catch (LifecycleException e1) {
            log.debug("destroy() failed for failed Server ", e1);
        }
        return;
    }

    long t2 = System.nanoTime();
    if(log.isInfoEnabled()) {
        log.info("Server startup in " + ((t2 - t1) / 1000000) + " ms");
    }

    // Register shutdown hook
    if (useShutdownHook) {
        if (shutdownHook == null) {
            shutdownHook = new CatalinaShutdownHook();
        }
        // 注册一个新的虚拟机关闭挂钩（即一个简单初始化但还没有启动的线程）
        // 程序退出时,启动该线程
        // 将会调用Catalina.this.stop()方法
        Runtime.getRuntime().addShutdownHook(shutdownHook);

        // If JULI is being used, disable JULI's shutdown hook since
        // shutdown hooks run in parallel and log messages may be lost
        // if JULI's hook completes before the CatalinaShutdownHook()
        LogManager logManager = LogManager.getLogManager();
        if (logManager instanceof ClassLoaderLogManager) {
            ((ClassLoaderLogManager) logManager).setUseShutdownHook(
                    false);
        }
    }
    // 进入等待状态，等待tomcat的stop命令
    if (await) {
        // 调用server的await()方法
        await();
        stop();
    }
}
```
## Server启动
* **Server的启动 主要分为init和start两个阶段。**
    * **init阶段主要负责创建并注册MBean**
    * **start阶段主要负责启动组件以及Service**
### init
* Bootstrap处理start命令，会通过反射调用Catalina的setAwait(),load(),start()方法
    * Catalina的load()方法将会调用Server的init()方法，开始生命周期的初始化工作。
* Server的init()方法首先被调用：Server实现了Lifecycle接口，实际调用的是父类LifecycleBase的init()方法，而该方法将调用StandardServer的模板方法initInternal()：
    * Server将初始化naming resources
    * Server将初始化所有的Service。
* **init的一个重要的作用是创建并注册MBean，参见[Tomcat中的JMX](./jmx_in_tomcat.md)**

```java
/**
    * Invoke a pre-startup initialization. This is used to allow connectors
    * to bind to restricted ports under Unix operating environments.
    */
@Override
protected void initInternal() throws LifecycleException {

    // 调用LifecycleMBeanBase的initInternal()方法,创建并注册对应StandardServer的MBean
    super.initInternal();

    // Register global String cache
    // Note although the cache is global, if there are multiple Servers
    // present in the JVM (may happen when embedding) then the same cache
    // will be registered under multiple names
    onameStringCache = register(new StringCache(), "type=StringCache");

    // Register the MBeanFactory
    MBeanFactory factory = new MBeanFactory();
    factory.setContainer(this);
    onameMBeanFactory = register(factory, "type=MBeanFactory");

    // Register the naming resources
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
    for (int i = 0; i < services.length; i++) {
        services[i].init();
    }
}
```

### start
* Bootstrap处理start命令，会通过反射调用Catalina的setAwait(),load(),start()方法
    * Catalina的start()方法将会调用Server的start(),await()，启动服务器并进入等待状态。当shutdown命令到来时，调用Server的stop()方法关闭服务器。

* Server的start()方法首先被调用：Server实现了Lifecycle接口，实际调用的是父类LifecycleBase的start()方法，而该方法将调用StandardServer的模板方法startInternal()：
    * Server将启动naming resources
    * Server将启动所有的Service。
    ```java
    @Override
    protected void startInternal() throws LifecycleException {

        fireLifecycleEvent(CONFIGURE_START_EVENT, null);
        setState(LifecycleState.STARTING);
        // 调用NamingResourcesImpl的startInternal()方法
        globalNamingResources.start();

        // Start our defined Services
        synchronized (servicesLock) {
            for (int i = 0; i < services.length; i++) {
                // 调用Service的startInternal()方法
                services[i].start();
            }
        }
    }
    ```
* Server的await()方法随后被调用：
    * 根据conf/server.xml中的配置
    ```xml
    <Server port="8005" shutdown="SHUTDOWN">
    ```
    * 首先判断端口号port，然后根据port的值有三种处理方法
    
    ```java
    /**
        * Wait until a proper shutdown command is received, then return.
        * This keeps the main thread alive - the thread pool listening for http
        * connections is daemon threads.
        */
    @Override
    public void await() {
        // Negative values - don't wait on port - tomcat is embedded or we just don't like ports
        // 当在配置文件中设置port为-2,不进入await状态，直接退出tomcat
        if( port == -2 ) {
            // undocumented yet - for embedding apps that are around, alive.
            return;
        }
        // 当在配置文件中设置port为-1，则进入await循环，该循环内部没有break语句
        // 即只有在外部调用了stop()方法将stopAwait置为true，才能关闭Tomcat
        if( port==-1 ) {
            try {
                awaitThread = Thread.currentThread();
                while(!stopAwait) {
                    try {
                        Thread.sleep( 10000 );
                    } catch( InterruptedException ex ) {
                        // continue and check the flag
                    }
                }
            } finally {
                awaitThread = null;
            }
            return;
        }
        // 当在配置文件中设置port为其他值，在port端口启动一个ServerSocket监听关闭命令。
        // 进入await循环，如果接收到退出命令，则退出循环，关闭tomcat。
        // Set up a server socket to wait on
        try {
            awaitSocket = new ServerSocket(port, 1,
                    InetAddress.getByName(address));
        } catch (IOException e) {
            log.error("StandardServer.await: create[" + address
                                + ":" + port
                                + "]: ", e);
            return;
        }

        try {
            awaitThread = Thread.currentThread();

            // Loop waiting for a connection and a valid command
            while (!stopAwait) {
                ServerSocket serverSocket = awaitSocket;
                if (serverSocket == null) {
                    break;
                }

                // Wait for the next connection
                Socket socket = null;
                StringBuilder command = new StringBuilder();
                try {
                    InputStream stream;
                    long acceptStartTime = System.currentTimeMillis();
                    try {
                        socket = serverSocket.accept();
                        socket.setSoTimeout(10 * 1000);  // Ten seconds
                        stream = socket.getInputStream();
                    } catch (SocketTimeoutException ste) {
                        // This should never happen but bug 56684 suggests that
                        // it does.
                        log.warn(sm.getString("standardServer.accept.timeout",
                                Long.valueOf(System.currentTimeMillis() - acceptStartTime)), ste);
                        continue;
                    } catch (AccessControlException ace) {
                        log.warn("StandardServer.accept security exception: "
                                + ace.getMessage(), ace);
                        continue;
                    } catch (IOException e) {
                        if (stopAwait) {
                            // Wait was aborted with socket.close()
                            break;
                        }
                        log.error("StandardServer.await: accept: ", e);
                        break;
                    }

                    // Read a set of characters from the socket
                    int expected = 1024; // Cut off to avoid DoS attack
                    while (expected < shutdown.length()) {
                        if (random == null)
                            random = new Random();
                        expected += (random.nextInt() % 1024);
                    }
                    while (expected > 0) {
                        int ch = -1;
                        try {
                            ch = stream.read();
                        } catch (IOException e) {
                            log.warn("StandardServer.await: read: ", e);
                            ch = -1;
                        }
                        // Control character or EOF (-1) terminates loop
                        if (ch < 32 || ch == 127) {
                            break;
                        }
                        command.append((char) ch);
                        expected--;
                    }
                } finally {
                    // Close the socket now that we are done with it
                    try {
                        if (socket != null) {
                            socket.close();
                        }
                    } catch (IOException e) {
                        // Ignore
                    }
                }

                // Match against our command string
                boolean match = command.toString().equals(shutdown);
                if (match) {
                    log.info(sm.getString("standardServer.shutdownViaPort"));
                    break;
                } else
                    log.warn("StandardServer.await: Invalid command '"
                            + command.toString() + "' received");
            }
        } finally {
            ServerSocket serverSocket = awaitSocket;
            awaitThread = null;
            awaitSocket = null;

            // Close the server socket and return
            if (serverSocket != null) {
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    // Ignore
                }
            }
        }
    }
    ```

## Service启动
### init
* Bootstrap.main()
    * Catalina.load()
        * Server.init()
        * StandardServer.initInternal()
            * Service.init()
            * StandardService.initInternal()


* 同Server一样，第一句代码super.initInternal()
    * 调用LifecycleMBeanBase的initInternal()方法,创建并注册对应StandardService的MBean。
* Initialize Engine
    * 调用LifecycleMBeanBase的initInternal()方法，创建并注册对应的MBean。
* Initialize Engine
    * 调用LifecycleMBeanBase的initInternal()方法，创建并注册对应的MBean。
* Initialize any Executors
    * 调用LifecycleMBeanBase的initInternal()方法，创建并注册对应的MBean。
* Initialize mapper listener
    * 调用LifecycleMBeanBase的initInternal()方法，创建并注册对应的MBean。
* Initialize our defined Connectors
    * 调用LifecycleMBeanBase的initInternal()方法，创建并注册对应的MBean。
```java
/**
    * Invoke a pre-startup initialization. This is used to allow connectors
    * to bind to restricted ports under Unix operating environments.
    */
@Override
protected void initInternal() throws LifecycleException {

    super.initInternal();

    // Initialize Engine
    if (engine != null) {
        engine.init();
    }

    // Initialize any Executors
    for (Executor executor : findExecutors()) {
        if (executor instanceof JmxEnabled) {
            ((JmxEnabled) executor).setDomain(getDomain());
        }
        executor.init();
    }

    // Initialize mapper listener
    mapperListener.init();

    // Initialize our defined Connectors
    synchronized (connectorsLock) {
        for (Connector connector : connectors) {
            connector.init();
        }
    }
}
```


### start
* Bootstrap.main(
    * Catalina.start()
        * Server.start()
        * StandardServer.startInternal()
            * Service.start()
            * StandardService.startInternal()

* 启动Engine
* 启动Executor
* 启动MapperListener
* 启动Connector
```java
/**
    * Start nested components ({@link Executor}s, {@link Connector}s and
    * {@link Container}s) and implement the requirements of
    * {@link org.apache.catalina.util.LifecycleBase#startInternal()}.
    *
    * @exception LifecycleException if this component detects a fatal error
    *  that prevents this component from being used
    */
@Override
protected void startInternal() throws LifecycleException {

    if(log.isInfoEnabled())
        log.info(sm.getString("standardService.start.name", this.name));
    setState(LifecycleState.STARTING);

    // Start our defined Container first
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }

    synchronized (executors) {
        for (Executor executor: executors) {
            executor.start();
        }
    }

    mapperListener.start();

    // Start our defined Connectors second
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            // If it has already failed, don't try and start it
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```
![](image/timing_diagram.jpg)