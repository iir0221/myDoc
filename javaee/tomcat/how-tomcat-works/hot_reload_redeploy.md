# 热部署和热加载
Tomcat支持热部署和热加载

技术支持：WebAppLoader backgroundProcess

* 热加载：服务器会监听 class 文件改变，包括web-inf/class,wen-inf/lib,web-inf/web.xml等文件，若发生更改，则局部进行加载，不清空session ，不释放内存。开发中用的多，但是要考虑内存溢出的情况。

* 热部署： 整个项目从新部署，包括你从新打上.war 文件。 会清空session ，释放内存。项目打包的时候用的多。

## 配置
可以通过如下配置设置应用热部署和热加载
context.xml(需配置)
```xml
<Context reloadable="true">

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>WEB-INF/tomcat-web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->
</Context>
```
server.xml(默认)
```xml
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
```
## 实现原理
Tomcat热部署和热加载主要依赖**backgroundProcess**和**WebAppLoader**实现

### backgroundProcess
Catalina支持定期执行自身及其子容器的后台处理过程(该机制位于容器父类ContainerBase中，
默认由Engine维护),具体处理过程在容器的backgroundProcess()方法中定义。

该机制常用于定时扫描Web应用的变更，并进行重新加载或重新部署。后台任务处理完成后，将触发PERIODIC_EVENT事件。

当Engine启动时，会调用父类ContainerBase的startInternal()方法，从而开启backgroundProcess线程，循环检查：
* StandardEngine#startInternal
    * ContainerBase#startInternal
```java
 @Override
protected synchronized void startInternal() throws LifecycleException {

    // Start our subordinate components, if any
    logger = null;
    getLogger();
    Cluster cluster = getClusterInternal();
    if (cluster instanceof Lifecycle) {
        ((Lifecycle) cluster).start();
    }
    Realm realm = getRealmInternal();
    if (realm instanceof Lifecycle) {
        ((Lifecycle) realm).start();
    }

    // Start our child containers, if any
    Container children[] = findChildren();
    List<Future<Void>> results = new ArrayList<>();
    for (int i = 0; i < children.length; i++) {
        results.add(startStopExecutor.submit(new StartChild(children[i])));
    }

    boolean fail = false;
    for (Future<Void> result : results) {
        try {
            result.get();
        } catch (Exception e) {
            log.error(sm.getString("containerBase.threadedStartFailed"), e);
            fail = true;
        }

    }
    if (fail) {
        throw new LifecycleException(
                sm.getString("containerBase.threadedStartFailed"));
    }

    // Start the Valves in our pipeline (including the basic), if any
    if (pipeline instanceof Lifecycle)
        ((Lifecycle) pipeline).start();


    setState(LifecycleState.STARTING);

    // Start our thread：开启backgroundProcess线程
    threadStart();
}
```
```java
/**
    * Start the background thread that will periodically check for
    * session timeouts.
    */
protected void threadStart() {

    if (thread != null)
        return;
    if (backgroundProcessorDelay <= 0)
        return;

    threadDone = false;
    String threadName = "ContainerBackgroundProcessor[" + toString() + "]";
    thread = new Thread(new ContainerBackgroundProcessor(), threadName);
    thread.setDaemon(true);
    thread.start();

}
```
```java
// -------------------------------------- ContainerExecuteDelay Inner Class

/**
    * Private thread class to invoke the backgroundProcess method
    * of this container and its children after a fixed delay.
    */
protected class ContainerBackgroundProcessor implements Runnable {

    @Override
    public void run() {
        Throwable t = null;
        String unexpectedDeathMessage = sm.getString(
                "containerBase.backgroundProcess.unexpectedThreadDeath",
                Thread.currentThread().getName());
        try {
            while (!threadDone) {
                try {
                    Thread.sleep(backgroundProcessorDelay * 1000L);
                } catch (InterruptedException e) {
                    // Ignore
                }
                if (!threadDone) {
                    processChildren(ContainerBase.this);
                }
            }
        } catch (RuntimeException|Error e) {
            t = e;
            throw e;
        } finally {
            if (!threadDone) {
                log.error(unexpectedDeathMessage, t);
            }
        }
    }

    protected void processChildren(Container container) {
        ClassLoader originalClassLoader = null;

        try {
            if (container instanceof Context) {
                Loader loader = ((Context) container).getLoader();
                // Loader will be null for FailedContext instances
                if (loader == null) {
                    return;
                }

                // Ensure background processing for Contexts and Wrappers
                // is performed under the web app's class loader
                originalClassLoader = ((Context) container).bind(false, null);
            }
            // 调用backgroundProcess();方法
            container.backgroundProcess();
            Container[] children = container.findChildren();
            // 调用children的backgroundProcess();方法
            for (int i = 0; i < children.length; i++) {
                if (children[i].getBackgroundProcessorDelay() <= 0) {
                    processChildren(children[i]);
                }
            }
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            log.error("Exception invoking periodic operation: ", t);
        } finally {
            if (container instanceof Context) {
                ((Context) container).unbind(false, originalClassLoader);
            }
        }
    }
}
```
对于Engine，Host来说，没有重写backgroundProcess()方法，则将调用父类ContainerBase的backgroundProcess()方法，并触发事件Lifecycle.PERIODIC_EVENT
```java
/**
    * Execute a periodic task, such as reloading, etc. This method will be
    * invoked inside the classloading context of this container. Unexpected
    * throwables will be caught and logged.
    */
@Override
public void backgroundProcess() {

    if (!getState().isAvailable())
        return;

    Cluster cluster = getClusterInternal();
    if (cluster != null) {
        try {
            cluster.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString("containerBase.backgroundProcess.cluster",
                    cluster), e);
        }
    }
    Realm realm = getRealmInternal();
    if (realm != null) {
        try {
            realm.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString("containerBase.backgroundProcess.realm", realm), e);
        }
    }
    Valve current = pipeline.getFirst();
    while (current != null) {
        try {
            current.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString("containerBase.backgroundProcess.valve", current), e);
        }
        current = current.getNext();
    }
    // 触发Lifecycle.PERIODIC_EVENT事件
    fireLifecycleEvent(Lifecycle.PERIODIC_EVENT, null);
}
```
对于Context来说，重写了backgroundProcess方法
```java
@Override
public void backgroundProcess() {

    if (!getState().isAvailable())
        return;

    Loader loader = getLoader();
    if (loader != null) {
        try {
            loader.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.loader", loader), e);
        }
    }
    Manager manager = getManager();
    if (manager != null) {
        try {
            manager.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.manager", manager),
                    e);
        }
    }
    WebResourceRoot resources = getResources();
    if (resources != null) {
        try {
            resources.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.resources",
                    resources), e);
        }
    }
    InstanceManager instanceManager = getInstanceManager();
    if (instanceManager != null) {
        try {
            instanceManager.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.instanceManager",
                    resources), e);
        }
    }
    super.backgroundProcess();
}
```
对于Wrapper来说，重写了backgroundProcess方法
```java
/**
    * Execute a periodic task, such as reloading, etc. This method will be
    * invoked inside the classloading context of this container. Unexpected
    * throwables will be caught and logged.
    */
@Override
public void backgroundProcess() {
    super.backgroundProcess();

    if (!getState().isAvailable())
        return;

    if (getServlet() instanceof PeriodicEventListener) {
        ((PeriodicEventListener) getServlet()).periodicEvent();
    }
}
```
# 触发热部署和热加载的情况
## 修改context.xml,则将触发热部署(如果是修改Server.xml中context的配置，则必须要重启Tomcat才能生效,因此推荐独立配置context.xml)
## 修改web.xml,则将触发热加载

接上文，backgroundProcess定时扫描web app应用变更。期间会触发PERIODIC_EVENT事件。

监听器HostConfig监听在PERIODIC_EVENT事件上
```java
/**
    * Process the START event for an associated Host.
    *
    * @param event The lifecycle event that has occurred
    */
@Override
public void lifecycleEvent(LifecycleEvent event) {

    // Identify the host we are associated with
    try {
        host = (Host) event.getLifecycle();
        if (host instanceof StandardHost) {
            setCopyXML(((StandardHost) host).isCopyXML());
            setDeployXML(((StandardHost) host).isDeployXML());
            setUnpackWARs(((StandardHost) host).isUnpackWARs());
            setContextClass(((StandardHost) host).getContextClass());
        }
    } catch (ClassCastException e) {
        log.error(sm.getString("hostConfig.cce", event.getLifecycle()), e);
        return;
    }

    // Process the event that has occurred
    if (event.getType().equals(Lifecycle.PERIODIC_EVENT)) {
        check();
    } else if (event.getType().equals(Lifecycle.BEFORE_START_EVENT)) {
        beforeStart();
    } else if (event.getType().equals(Lifecycle.START_EVENT)) {
        start();
    } else if (event.getType().equals(Lifecycle.STOP_EVENT)) {
        stop();
    }
}
```
当该事件被触发时，将调用check()方法,该方法主要判断context.xml，web.xml是否发生变化需要，如果发生变化，则需要进行热加载或者热部署
```java
/**
    * Check status of all webapps.
    */
protected void check() {

    if (host.getAutoDeploy()) {
        // Check for resources modification to trigger redeployment
        DeployedApplication[] apps =
            deployed.values().toArray(new DeployedApplication[0]);
        for (int i = 0; i < apps.length; i++) {
            if (!isServiced(apps[i].name))
                checkResources(apps[i], false);
        }

        // Check for old versions of applications that can now be undeployed
        if (host.getUndeployOldVersions()) {
            checkUndeploy();
        }

        // Hotdeploy applications
        deployApps();
    }
}
```
```java
protected synchronized void checkResources(DeployedApplication app,
        boolean skipFileModificationResolutionCheck) {
    String[] resources =
        app.redeployResources.keySet().toArray(new String[0]);
    // Offset the current time by the resolution of File.lastModified()
    long currentTimeWithResolutionOffset =
            System.currentTimeMillis() - FILE_MODIFICATION_RESOLUTION_MS;
    for (int i = 0; i < resources.length; i++) {
        File resource = new File(resources[i]);
        if (log.isDebugEnabled())
            log.debug("Checking context[" + app.name +
                    "] redeploy resource " + resource);
        long lastModified =
                app.redeployResources.get(resources[i]).longValue();
        if (resource.exists() || lastModified == 0) {
            // File.lastModified() has a resolution of 1s (1000ms). The last
            // modified time has to be more than 1000ms ago to ensure that
            // modifications that take place in the same second are not
            // missed. See Bug 57765.
            if (resource.lastModified() != lastModified && (!host.getAutoDeploy() ||
                    resource.lastModified() < currentTimeWithResolutionOffset ||
                    skipFileModificationResolutionCheck)) {
                if (resource.isDirectory()) {
                    // No action required for modified directory
                    app.redeployResources.put(resources[i],
                            Long.valueOf(resource.lastModified()));
                } else if (app.hasDescriptor &&
                        resource.getName().toLowerCase(
                                Locale.ENGLISH).endsWith(".war")) {
                    // Modified WAR triggers a reload if there is an XML
                    // file present
                    // The only resource that should be deleted is the
                    // expanded WAR (if any)
                    Context context = (Context) host.findChild(app.name);
                    String docBase = context.getDocBase();
                    if (!docBase.toLowerCase(Locale.ENGLISH).endsWith(".war")) {
                        // This is an expanded directory
                        File docBaseFile = new File(docBase);
                        if (!docBaseFile.isAbsolute()) {
                            docBaseFile = new File(host.getAppBaseFile(),
                                    docBase);
                        }
                        reload(app, docBaseFile, resource.getAbsolutePath());
                    } else {
                        reload(app, null, null);
                    }
                    // Update times
                    app.redeployResources.put(resources[i],
                            Long.valueOf(resource.lastModified()));
                    app.timestamp = System.currentTimeMillis();
                    boolean unpackWAR = unpackWARs;
                    if (unpackWAR && context instanceof StandardContext) {
                        unpackWAR = ((StandardContext) context).getUnpackWAR();
                    }
                    if (unpackWAR) {
                        addWatchedResources(app, context.getDocBase(), context);
                    } else {
                        addWatchedResources(app, null, context);
                    }
                    return;
                } else {
                    // Everything else triggers a redeploy
                    // (just need to undeploy here, deploy will follow)
                    undeploy(app);
                    deleteRedeployResources(app, resources, i, false);
                    return;
                }
            }
        } else {
            // There is a chance the the resource was only missing
            // temporarily eg renamed during a text editor save
            try {
                Thread.sleep(500);
            } catch (InterruptedException e1) {
                // Ignore
            }
            // Recheck the resource to see if it was really deleted
            if (resource.exists()) {
                continue;
            }
            // Undeploy application
            undeploy(app);
            deleteRedeployResources(app, resources, i, true);
            return;
        }
    }
    resources = app.reloadResources.keySet().toArray(new String[0]);
    boolean update = false;
    for (int i = 0; i < resources.length; i++) {
        File resource = new File(resources[i]);
        if (log.isDebugEnabled()) {
            log.debug("Checking context[" + app.name + "] reload resource " + resource);
        }
        long lastModified = app.reloadResources.get(resources[i]).longValue();
        // File.lastModified() has a resolution of 1s (1000ms). The last
        // modified time has to be more than 1000ms ago to ensure that
        // modifications that take place in the same second are not
        // missed. See Bug 57765.
        if ((resource.lastModified() != lastModified &&
                (!host.getAutoDeploy() ||
                        resource.lastModified() < currentTimeWithResolutionOffset ||
                        skipFileModificationResolutionCheck)) ||
                update) {
            if (!update) {
                // Reload application
                reload(app, null, null);
                update = true;
            }
            // Update times. More than one file may have been updated. We
            // don't want to trigger a series of reloads.
            app.reloadResources.put(resources[i],
                    Long.valueOf(resource.lastModified()));
        }
        app.timestamp = System.currentTimeMillis();
    }
}
```
### 重新加载
```java
private void reload(DeployedApplication app, File fileToRemove, String newDocBase) {
    if(log.isInfoEnabled())
        log.info(sm.getString("hostConfig.reload", app.name));
    Context context = (Context) host.findChild(app.name);
    if (context.getState().isAvailable()) {
        if (fileToRemove != null && newDocBase != null) {
            context.addLifecycleListener(
                    new ExpandedDirectoryRemovalListener(fileToRemove, newDocBase));
        }
        // Reload catches and logs exceptions
        context.reload();
    } else {
        // If the context was not started (for example an error
        // in web.xml) we'll still get to try to start
        if (fileToRemove != null && newDocBase != null) {
            ExpandWar.delete(fileToRemove);
            context.setDocBase(newDocBase);
        }
        try {
            context.start();
        } catch (Exception e) {
            log.warn(sm.getString
                        ("hostConfig.context.restart", app.name), e);
        }
    }
}
```

### 重新部署

```java
private void undeploy(DeployedApplication app) {
    if (log.isInfoEnabled())
        log.info(sm.getString("hostConfig.undeploy", app.name));
    Container context = host.findChild(app.name);
    try {
        host.removeChild(context);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        log.warn(sm.getString
                    ("hostConfig.context.remove", app.name), t);
    }
    deployed.remove(app.name);
}
```
## 当web app的类文件，jar包发生变化时，将进行热加载
backgroundProcess定时扫描web app应用变更，Context重写了backgroundProcess方法，通过调用loader.backgroundProcess()来重新加载Context
```java
@Override
public void backgroundProcess() {

    if (!getState().isAvailable())
        return;

    Loader loader = getLoader();
    if (loader != null) {
        try {
            loader.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.loader", loader), e);
        }
    }
    Manager manager = getManager();
    if (manager != null) {
        try {
            manager.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.manager", manager),
                    e);
        }
    }
    WebResourceRoot resources = getResources();
    if (resources != null) {
        try {
            resources.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.resources",
                    resources), e);
        }
    }
    InstanceManager instanceManager = getInstanceManager();
    if (instanceManager != null) {
        try {
            instanceManager.backgroundProcess();
        } catch (Exception e) {
            log.warn(sm.getString(
                    "standardContext.backgroundProcess.instanceManager",
                    resources), e);
        }
    }
    super.backgroundProcess();
}
```
WebAppLoader#backgroundProcess
```java
@Override
public void backgroundProcess() {
    if (reloadable && modified()) {
        try {
            Thread.currentThread().setContextClassLoader
                (WebappLoader.class.getClassLoader());
            if (context != null) {
                context.reload();
            }
        } finally {
            if (context != null && context.getLoader() != null) {
                Thread.currentThread().setContextClassLoader
                    (context.getLoader().getClassLoader());
            }
        }
    }
}
```
Context#reload

```java
/**
    * Reload this web application, if reloading is supported.
    * <p>
    * <b>IMPLEMENTATION NOTE</b>:  This method is designed to deal with
    * reloads required by changes to classes in the underlying repositories
    * of our class loader and changes to the web.xml file. It does not handle
    * changes to any context.xml file. If the context.xml has changed, you
    * should stop this Context and create (and start) a new Context instance
    * instead. Note that there is additional code in
    * <code>CoyoteAdapter#postParseRequest()</code> to handle mapping requests
    * to paused Contexts.
    *
    * @exception IllegalStateException if the <code>reloadable</code>
    *  property is set to <code>false</code>.
    */
@Override
public synchronized void reload() {

    // Validate our current component state
    if (!getState().isAvailable())
        throw new IllegalStateException
            (sm.getString("standardContext.notStarted", getName()));

    if(log.isInfoEnabled())
        log.info(sm.getString("standardContext.reloadingStarted",
                getName()));

    // Stop accepting requests temporarily.
    setPaused(true);

    try {
        stop();
    } catch (LifecycleException e) {
        log.error(
            sm.getString("standardContext.stoppingContext", getName()), e);
    }

    try {
        start();
    } catch (LifecycleException e) {
        log.error(
            sm.getString("standardContext.startingContext", getName()), e);
    }

    setPaused(false);

    if(log.isInfoEnabled())
        log.info(sm.getString("standardContext.reloadingCompleted",
                getName()));

}
```
## JSP热加载
当改变JSP内容时，再次访问该JSP页面，将会触发

访问JSP，调用Servlet类

HttpServlet#service()
```java
@Override
public void service(ServletRequest req, ServletResponse res)
    throws ServletException, IOException {

    HttpServletRequest  request;
    HttpServletResponse response;

    try {
        request = (HttpServletRequest) req;
        response = (HttpServletResponse) res;
    } catch (ClassCastException e) {
        throw new ServletException("non-HTTP request or response");
    }
    // 调用JspServlet的service方法
    service(request, response);
}
```
调用
JspServlet#service()
```java
@Override
public void service (HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    // jspFile may be configured as an init-param for this servlet instance
    String jspUri = jspFile;

    if (jspUri == null) {
        /*
            * Check to see if the requested JSP has been the target of a
            * RequestDispatcher.include()
            */
        jspUri = (String) request.getAttribute(
                RequestDispatcher.INCLUDE_SERVLET_PATH);
        if (jspUri != null) {
            /*
                * Requested JSP has been target of
                * RequestDispatcher.include(). Its path is assembled from the
                * relevant javax.servlet.include.* request attributes
                */
            String pathInfo = (String) request.getAttribute(
                    RequestDispatcher.INCLUDE_PATH_INFO);
            if (pathInfo != null) {
                jspUri += pathInfo;
            }
        } else {
            /*
                * Requested JSP has not been the target of a
                * RequestDispatcher.include(). Reconstruct its path from the
                * request's getServletPath() and getPathInfo()
                */
            jspUri = request.getServletPath();
            String pathInfo = request.getPathInfo();
            if (pathInfo != null) {
                jspUri += pathInfo;
            }
        }
    }

    if (log.isDebugEnabled()) {
        log.debug("JspEngine --> " + jspUri);
        log.debug("\t     ServletPath: " + request.getServletPath());
        log.debug("\t        PathInfo: " + request.getPathInfo());
        log.debug("\t        RealPath: " + context.getRealPath(jspUri));
        log.debug("\t      RequestURI: " + request.getRequestURI());
        log.debug("\t     QueryString: " + request.getQueryString());
    }

    try {
        boolean precompile = preCompile(request);
        
        serviceJspFile(request, response, jspUri, precompile);
    } catch (RuntimeException e) {
        throw e;
    } catch (ServletException e) {
        throw e;
    } catch (IOException e) {
        throw e;
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        throw new ServletException(e);
    }

}
```
JspServlet#serviceJspFile
```java
private void serviceJspFile(HttpServletRequest request,
                            HttpServletResponse response, String jspUri,
                            boolean precompile)
    throws ServletException, IOException {

    JspServletWrapper wrapper = rctxt.getWrapper(jspUri);
    if (wrapper == null) {
        synchronized(this) {
            wrapper = rctxt.getWrapper(jspUri);
            if (wrapper == null) {
                // Check if the requested JSP page exists, to avoid
                // creating unnecessary directories and files.
                if (null == context.getResource(jspUri)) {
                    handleMissingResource(request, response, jspUri);
                    return;
                }
                wrapper = new JspServletWrapper(config, options, jspUri,
                                                rctxt);
                rctxt.addWrapper(jspUri,wrapper);
            }
        }
    }

    try {
        wrapper.service(request, response, precompile);
    } catch (FileNotFoundException fnfe) {
        handleMissingResource(request, response, jspUri);
    }

}
```
注意看该方法注释，当处于Development模式(默认)或者第一次访问Jsp页面时，要compile Jsp.
在Development模式下，如果修改了Jsp页面的内容，将会reload该页面

JspServletWrapper#service
```java
public void service(HttpServletRequest request,
                    HttpServletResponse response,
                    boolean precompile)
        throws ServletException, IOException, FileNotFoundException {

    Servlet servlet;

    try {

        if (ctxt.isRemoved()) {
            throw new FileNotFoundException(jspUri);
        }

        if ((available > 0L) && (available < Long.MAX_VALUE)) {
            if (available > System.currentTimeMillis()) {
                response.setDateHeader("Retry-After", available);
                response.sendError
                    (HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                        Localizer.getMessage("jsp.error.unavailable"));
                return;
            }

            // Wait period has expired. Reset.
            available = 0;
        }

        /*
            * (1) Compile
            */
        if (options.getDevelopment() || firstTime ) {
            synchronized (this) {
                firstTime = false;

                // The following sets reload to true, if necessary
                ctxt.compile();
            }
        } else {
            if (compileException != null) {
                // Throw cached compilation exception
                throw compileException;
            }
        }

        /*
            * (2) (Re)load servlet class file
            */
        servlet = getServlet();

        // If a page is to be precompiled only, return.
        if (precompile) {
            return;
        }

    } catch (ServletException ex) {
        if (options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw ex;
    } catch (FileNotFoundException fnfe) {
        // File has been removed. Let caller handle this.
        throw fnfe;
    } catch (IOException ex) {
        if (options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw ex;
    } catch (IllegalStateException ex) {
        if (options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw ex;
    } catch (Exception ex) {
        if (options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw new JasperException(ex);
    }

    try {
        /*
            * (3) Handle limitation of number of loaded Jsps
            */
        if (unloadAllowed) {
            synchronized(this) {
                if (unloadByCount) {
                    if (unloadHandle == null) {
                        unloadHandle = ctxt.getRuntimeContext().push(this);
                    } else if (lastUsageTime < ctxt.getRuntimeContext().getLastJspQueueUpdate()) {
                        ctxt.getRuntimeContext().makeYoungest(unloadHandle);
                        lastUsageTime = System.currentTimeMillis();
                    }
                } else {
                    if (lastUsageTime < ctxt.getRuntimeContext().getLastJspQueueUpdate()) {
                        lastUsageTime = System.currentTimeMillis();
                    }
                }
            }
        }

        /*
            * (4) Service request
            */
        if (servlet instanceof SingleThreadModel) {
            // sync on the wrapper so that the freshness
            // of the page is determined right before servicing
            synchronized (this) {
                servlet.service(request, response);
            }
        } else {
            servlet.service(request, response);
        }
    } catch (UnavailableException ex) {
        String includeRequestUri = (String)
            request.getAttribute(RequestDispatcher.INCLUDE_REQUEST_URI);
        if (includeRequestUri != null) {
            // This file was included. Throw an exception as
            // a response.sendError() will be ignored by the
            // servlet engine.
            throw ex;
        }

        int unavailableSeconds = ex.getUnavailableSeconds();
        if (unavailableSeconds <= 0) {
            unavailableSeconds = 60;        // Arbitrary default
        }
        available = System.currentTimeMillis() +
            (unavailableSeconds * 1000L);
        response.sendError
            (HttpServletResponse.SC_SERVICE_UNAVAILABLE,
                ex.getMessage());
    } catch (ServletException ex) {
        if(options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw ex;
    } catch (IOException ex) {
        if (options.getDevelopment()) {
            throw new IOException(handleJspException(ex).getMessage(), ex);
        }
        throw ex;
    } catch (IllegalStateException ex) {
        if(options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw ex;
    } catch (Exception ex) {
        if(options.getDevelopment()) {
            throw handleJspException(ex);
        }
        throw new JasperException(ex);
    }
}
```

(1)compile

如果jsp页面过期，则设置reload为true

重新compile

JspCompilationContext#compile()
```java
// ==================== Compile and reload ====================

public void compile() throws JasperException, FileNotFoundException {
    createCompiler();
    if (jspCompiler.isOutDated()) {
        if (isRemoved()) {
            throw new FileNotFoundException(jspUri);
        }
        try {
            jspCompiler.removeGeneratedFiles();
            jspLoader = null;
            // compile
            jspCompiler.compile();
            jsw.setReload(true);
            jsw.setCompilationException(null);
        } catch (JasperException ex) {
            // Cache compilation exception
            jsw.setCompilationException(ex);
            if (options.getDevelopment() && options.getRecompileOnFail()) {
                // Force a recompilation attempt on next access
                jsw.setLastModificationTest(-1);
            }
            throw ex;
        } catch (FileNotFoundException fnfe) {
            // Re-throw to let caller handle this - will result in a 404
            throw fnfe;
        } catch (Exception ex) {
            JasperException je = new JasperException(
                    Localizer.getMessage("jsp.error.unable.compile"),
                    ex);
            // Cache compilation exception
            jsw.setCompilationException(je);
            throw je;
        }
    }
}
```
(2) (Re)load servlet class file

创建servlet实例

init方法在这里被调用

JspServletWrapper#getServlet
```java
public Servlet getServlet() throws ServletException {
    // DCL on 'reload' requires that 'reload' be volatile
    // (this also forces a read memory barrier, ensuring the
    // new servlet object is read consistently)
    if (reload) {
        synchronized (this) {
            // Synchronizing on jsw enables simultaneous loading
            // of different pages, but not the same page.
            if (reload) {
                // This is to maintain the original protocol.
                destroy();

                final Servlet servlet;

                try {
                    InstanceManager instanceManager = InstanceManagerFactory.getInstanceManager(config);
                    servlet = (Servlet) instanceManager.newInstance(ctxt.getFQCN(), ctxt.getJspLoader());
                } catch (Exception e) {
                    Throwable t = ExceptionUtils
                            .unwrapInvocationTargetException(e);
                    ExceptionUtils.handleThrowable(t);
                    throw new JasperException(t);
                }

                servlet.init(config);

                if (!firstTime) {
                    ctxt.getRuntimeContext().incrementJspReloadCount();
                }

                theServlet = servlet;
                reload = false;
                // Volatile 'reload' forces in order write of 'theServlet' and new servlet object
            }
        }
    }
    return theServlet;
}
```
