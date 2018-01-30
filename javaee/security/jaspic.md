[转载]http://arjan-tijms.omnifaces.org/2012/11/implementing-container-authentication.html

JAAS comes a long way in introducing a basic set of security primitives and overall establishing a very comprehensive security framework, but one thing it doesn't have knowledge about is how to integrate with a Java EE container. Practically this means that **JAAS has no way of communicating a successful authentication to the container.** A user may be logged-in to some JAAS module, but Java EE will be totally unaware of this fact. The reverse is also true; when a protected resource is accessed by the user, or when an explicit login is triggered via the Servlet 3 HttpServletRequest#login method, the container has no notion of which JAAS login module should be called. **Finally, there is a mismatch between the very general JAAS concept of a so-called Subject having a bag of Principals and the Java EE notion of a caller principal and a collection of roles.**

**JASPIC finally standardizes how an an authentication module is integrated into a Java EE container.** However, it's not without problems and has a few quirks.


#  Step1 Registering via the factory-factory-factory(AuthConfigFactory)
We first obtain a reference to the factory-factory-factory (AuthConfigFactory), which we use to register our own factory-factory (shown highlighted). We need to specify for which layer we're doing the registration, which needs to be the constant "HttpServlet" for the Servlet Container Profile. For this example we evade the problems with the appContext and provide a null, which means we're doing the registration for all applications running on the server. 

**register factory-factory-factory**
* initContainerinitializer
    * SamRegistrationInstaller.onStartup
        ```java
        ...
        Jaspic.registerServerAuthModule(new HttpBridgeServerAuthModule(cdiPerRequestInitializer), ctx);
        ...
        ```
        * Jaspic.registerServerAuthModule()
```java
/**
    * Registers a server auth module as the one and only module for the application corresponding to
    * the given servlet context.
    * 
    * <p>
    * This will override any other modules that have already been registered, either via proprietary
    * means or using the standard API.
    * 
    * @param serverAuthModule the server auth module to be registered
    * @param servletContext the context of the app for which the module is registered
    * @return A String identifier assigned by an underlying factory corresponding to an underlying factory-factory-factory registration
    */
public static String registerServerAuthModule(ServerAuthModule serverAuthModule, ServletContext servletContext) {
    
    
    // Register the factory-factory-factory for the SAM
    String registrationId = AccessController.doPrivileged(new PrivilegedAction<String>() {
        public String run() {
            // get AuthConfigFactory form JACC
            // register AuthConfigProvider
            return AuthConfigFactory.getFactory().registerConfigProvider(
                    // register ServerAuthModule
                    new DefaultAuthConfigProvider(serverAuthModule),
                    "HttpServlet", 
                    getAppContextID(servletContext), 
                    "Default single SAM authentication config provider");
        }
    });
    
    // Remember the registration ID returned by the factory, so we can unregister the JASPIC module when the web module
    // is undeployed. JASPIC being the low level API that it is won't do this automatically.
    servletContext.setAttribute(CONTEXT_REGISTRATION_ID, registrationId);
    
    return registrationId;
}
```



# Step2  Implementing the factory-factory(AuthConfigProvider)

In the next step we look at the factory-factory that we registered above, which is an implementation of AuthConfigProvider. This factory-factory has a required constructor with a required implementation. The implementation seemed trivial; we need to do a self-registration. As I wasn't sure where some of the parameters had to be obtained from, I used my good friend null again. 

The real meat of this class is in the getServerAuthConfig method (shown highlighted), which simply has to return a factory. The flexibility that this factory-factory offers is the ability to create factories with a given handler or when this is null give the factory-factory the chance to create a default handler of some sorts. There's also a refresh method that I think asks for updating all factories created by the factory-factory if needed. It's only for dynamic factory-factories though, so I left it unimplemented. 

```java
package org.glassfish.soteria.mechanisms.jaspic;

import java.util.Map;

import javax.security.auth.callback.CallbackHandler;
import javax.security.auth.message.AuthException;
import javax.security.auth.message.config.AuthConfigFactory;
import javax.security.auth.message.config.AuthConfigProvider;
import javax.security.auth.message.config.ClientAuthConfig;
import javax.security.auth.message.config.ServerAuthConfig;
import javax.security.auth.message.config.ServerAuthContext;
import javax.security.auth.message.module.ServerAuthModule;

/**
 * This class functions as a kind of factory-factory for {@link ServerAuthConfig} instances, which are by themselves factories
 * for {@link ServerAuthContext} instances, which are delegates for the actual {@link ServerAuthModule} (SAM) that we're after.
 * 
 * @author Arjan Tijms
 */
public class DefaultAuthConfigProvider implements AuthConfigProvider {

    private static final String CALLBACK_HANDLER_PROPERTY_NAME = "authconfigprovider.client.callbackhandler";

    private Map<String, String> providerProperties;
    private ServerAuthModule serverAuthModule;

    public DefaultAuthConfigProvider(ServerAuthModule serverAuthModule) {
        this.serverAuthModule = serverAuthModule;
    }

    /**
     * Constructor with signature and implementation that's required by API.
     * 
     * @param properties provider properties
     * @param factory the auth config factory
     */
    public DefaultAuthConfigProvider(Map<String, String> properties, AuthConfigFactory factory) {
        this.providerProperties = properties;

        // API requires self registration if factory is provided. Not clear
        // where the "layer" (2nd parameter)
        // and especially "appContext" (3rd parameter) values have to come from
        // at this place.
        if (factory != null) {
            // If this method ever gets called, it may throw a SecurityException.
            // Don't bother with a PrivilegedAction as we don't expect to ever be
            // constructed this way.
            factory.registerConfigProvider(this, null, null, "Auto registration");
        }
    }

    /**
     * The actual factory method that creates the factory used to eventually obtain the delegate for a SAM.
     */
    @Override
    public ServerAuthConfig getServerAuthConfig(String layer, String appContext, CallbackHandler handler) throws AuthException,
        SecurityException {
        return new DefaultServerAuthConfig(layer, appContext, handler == null ? createDefaultCallbackHandler() : handler,
            providerProperties, serverAuthModule);
    }

    @Override
    public ClientAuthConfig getClientAuthConfig(String layer, String appContext, CallbackHandler handler) throws AuthException,
        SecurityException {
        return null;
    }

    @Override
    public void refresh() {
    }

    /**
     * Creates a default callback handler via the system property "authconfigprovider.client.callbackhandler", as seemingly
     * required by the API (API uses wording "may" create default handler). TODO: Isn't
     * "authconfigprovider.client.callbackhandler" JBoss specific?
     * 
     * @return
     * @throws AuthException
     */
    private CallbackHandler createDefaultCallbackHandler() throws AuthException {
        String callBackClassName = System.getProperty(CALLBACK_HANDLER_PROPERTY_NAME);

        if (callBackClassName == null) {
            throw new AuthException("No default handler set via system property: " + CALLBACK_HANDLER_PROPERTY_NAME);
        }

        try {
            return (CallbackHandler) Thread.currentThread().getContextClassLoader().loadClass(callBackClassName).newInstance();
        } catch (Exception e) {
            throw new AuthException(e.getMessage());
        }
    }

}
```
# Step 3 - Implementing the factory(ServerAuthConfig)
In the next step we look at the factory-factory that we registered above, which is an implementation of AuthConfigProvider. This factory-factory has a required constructor with a required implementation. The implementation seemed trivial; we need to do a self-registration. As I wasn't sure where some of the parameters had to be obtained from, I used my good friend null again. 

The real meat of this class is in the getServerAuthConfig method (shown highlighted), which simply has to return a factory. The flexibility that this factory-factory offers is the ability to create factories with a given handler or when this is null give the factory-factory the chance to create a default handler of some sorts. There's also a refresh method that I think asks for updating all factories created by the factory-factory if needed. It's only for dynamic factory-factories though, so I left it unimplemented. 

```java
package org.glassfish.soteria.mechanisms.jaspic;

import java.util.Map;

import javax.security.auth.Subject;
import javax.security.auth.callback.CallbackHandler;
import javax.security.auth.message.AuthException;
import javax.security.auth.message.MessageInfo;
import javax.security.auth.message.config.ServerAuthConfig;
import javax.security.auth.message.config.ServerAuthContext;
import javax.security.auth.message.module.ServerAuthModule;

/**
 * This class functions as a kind of factory for {@link ServerAuthContext} instances, which are delegates for the actual
 * {@link ServerAuthModule} (SAM) that we're after.
 * 
 * @author Arjan Tijms
 */
public class DefaultServerAuthConfig implements ServerAuthConfig {

    private String layer;
    private String appContext;
    private CallbackHandler handler;
    private Map<String, String> providerProperties;
    private ServerAuthModule serverAuthModule;

    public DefaultServerAuthConfig(String layer, String appContext, CallbackHandler handler,
        Map<String, String> providerProperties, ServerAuthModule serverAuthModule) {
        this.layer = layer;
        this.appContext = appContext;
        this.handler = handler;
        this.providerProperties = providerProperties;
        this.serverAuthModule = serverAuthModule;
    }

    @Override
    public ServerAuthContext getAuthContext(String authContextID, Subject serviceSubject,
        @SuppressWarnings("rawtypes") Map properties) throws AuthException {
        return new DefaultServerAuthContext(handler, serverAuthModule);
    }

    // ### The methods below mostly just return what has been passed into the
    // constructor.
    // ### In practice they don't seem to be called

    @Override
    public String getMessageLayer() {
        return layer;
    }

    /**
     * It's not entirely clear what the difference is between the "application context identifier" (appContext) and the
     * "authentication context identifier" (authContext). In early iterations of the specification, authContext was called
     * "operation" and instead of the MessageInfo it was obtained by something called an "authParam".
     */
    @Override
    public String getAuthContextID(MessageInfo messageInfo) {
        return appContext;
    }

    @Override
    public String getAppContext() {
        return appContext;
    }

    @Override
    public void refresh() {
    }

    @Override
    public boolean isProtected() {
        return false;
    }

    public Map<String, String> getProviderProperties() {
        return providerProperties;
    }

}
```
# Step 4 - Implementing the delegator(ServerAuthContext)
In the delegator (an implementation of ServerAuthContext) that was returned from the factory above we finally get a chance to create our authentication module (shown highlighted). 

The rest of the delegator class can be pretty simple. As we don't have any selection between modules to do, we just delegate directly to the one and only module that we encapsulate. 
```java
package org.glassfish.soteria.mechanisms.jaspic;

import java.util.Collections;

import javax.security.auth.Subject;
import javax.security.auth.callback.CallbackHandler;
import javax.security.auth.message.AuthException;
import javax.security.auth.message.AuthStatus;
import javax.security.auth.message.MessageInfo;
import javax.security.auth.message.ServerAuth;
import javax.security.auth.message.config.ServerAuthContext;
import javax.security.auth.message.module.ServerAuthModule;

/**
 * The Server Authentication Context is an extra (required) indirection between the Application Server and the actual Server
 * Authentication Module (SAM). This can be used to encapsulate any number of SAMs and either select one at run-time, invoke
 * them all in order, etc.
 * <p>
 * Since this simple example only has a single SAM, we delegate directly to that one. Note that this {@link ServerAuthContext}
 * and the {@link ServerAuthModule} (SAM) share a common base interface: {@link ServerAuth}.
 * 
 * @author Arjan Tijms
 */
public class DefaultServerAuthContext implements ServerAuthContext {

    private final ServerAuthModule serverAuthModule;

    public DefaultServerAuthContext(CallbackHandler handler, ServerAuthModule serverAuthModule) throws AuthException {
        this.serverAuthModule = serverAuthModule;
        serverAuthModule.initialize(null, null, handler, Collections.<String, String> emptyMap());
    }

    @Override
    public AuthStatus validateRequest(MessageInfo messageInfo, Subject clientSubject, Subject serviceSubject)
        throws AuthException {
        return serverAuthModule.validateRequest(messageInfo, clientSubject, serviceSubject);
    }

    @Override
    public AuthStatus secureResponse(MessageInfo messageInfo, Subject serviceSubject) throws AuthException {
        return serverAuthModule.secureResponse(messageInfo, serviceSubject);
    }

    @Override
    public void cleanSubject(MessageInfo messageInfo, Subject subject) throws AuthException {
        serverAuthModule.cleanSubject(messageInfo, subject);
    }

}
```
# Step 5 - Implementing the authentication module(ServerAuthModule)
At long last, we finally get to implement our authentication module, which is an instance of ServerAuthModule. With respect to the API, it's interesting to note that this time around there's an initialize method present instead of a mandatory constructor. 

As mentioned before, we don't do an actual authentication but just "install" the caller principal and a role into the JAAS Subject. For this example, getSupportedMessageTypes actually doesn't need to be implemented since it's only called by the delegator that encapsulates it. Since we own that delegator, we know it's not going to call this method. For completeness though I implemented it anyway to be compliant with the Servlet Container Profile.

Interesting to note is that secureResponse was treated differently by most servers. Only WebLogic and Geronimo call this method, but where WebLogic insists on seeing SEND_SUCCESS returned, Geronimo just ignores the return value. In its class org.apache.geronimo.tomcat.security.SecurityValve, it contains the following code fragment: 
// This returns a success code but I'm not sure what to do with it.
authenticator.secureResponse(request, response, authResult);
Another difference for this same secureResponse method, is that WebLogic calls it before a protected resource (e.g. Servlet) is called, while Geronimo does so after. 

```java
package org.glassfish.soteria.mechanisms.jaspic;

import static javax.security.enterprise.AuthenticationStatus.NOT_DONE;
import static javax.security.enterprise.AuthenticationStatus.SEND_FAILURE;
import static org.glassfish.soteria.mechanisms.jaspic.Jaspic.fromAuthenticationStatus;
import static org.glassfish.soteria.mechanisms.jaspic.Jaspic.setLastAuthenticationStatus;

import java.util.Map;

import javax.enterprise.inject.spi.CDI;
import javax.security.auth.Subject;
import javax.security.auth.callback.CallbackHandler;
import javax.security.auth.message.AuthException;
import javax.security.auth.message.AuthStatus;
import javax.security.auth.message.MessageInfo;
import javax.security.auth.message.MessagePolicy;
import javax.security.auth.message.config.ServerAuthContext;
import javax.security.auth.message.module.ServerAuthModule;
import javax.security.enterprise.AuthenticationException;
import javax.security.enterprise.AuthenticationStatus;
import javax.security.enterprise.authentication.mechanism.http.HttpAuthenticationMechanism;
import javax.security.enterprise.authentication.mechanism.http.HttpMessageContext;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.glassfish.soteria.cdi.spi.CDIPerRequestInitializer;
import org.glassfish.soteria.mechanisms.HttpMessageContextImpl;

/**
 *
 * @author Arjan Tijms
 *
 */
public class HttpBridgeServerAuthModule implements ServerAuthModule {

        private CallbackHandler handler;
        private final Class<?>[] supportedMessageTypes = new Class[] { HttpServletRequest.class, HttpServletResponse.class };
        private final CDIPerRequestInitializer cdiPerRequestInitializer;
        
        public HttpBridgeServerAuthModule(CDIPerRequestInitializer cdiPerRequestInitializer) {
            this.cdiPerRequestInitializer = cdiPerRequestInitializer;
        }
        
        @Override
        public void initialize(MessagePolicy requestPolicy, MessagePolicy responsePolicy, CallbackHandler handler, @SuppressWarnings("rawtypes") Map options) throws AuthException {
            this.handler = handler;
            // options not supported.
        }

        /**
         * A Servlet Container Profile compliant implementation should return HttpServletRequest and HttpServletResponse, so
         * the delegation class {@link ServerAuthContext} can choose the right SAM to delegate to.
         */
        @Override
        public Class<?>[] getSupportedMessageTypes() {
            return supportedMessageTypes;
        }

        @Override
        public AuthStatus validateRequest(MessageInfo messageInfo, Subject clientSubject, Subject serviceSubject) throws AuthException {
            
            HttpMessageContext msgContext = new HttpMessageContextImpl(handler, messageInfo, clientSubject);
            
            if (cdiPerRequestInitializer != null) {
                cdiPerRequestInitializer.init(msgContext.getRequest());
            }
            
            AuthenticationStatus status = NOT_DONE;
            setLastAuthenticationStatus(msgContext.getRequest(), status);
                
            try {
                status = CDI.current()
                            .select(HttpAuthenticationMechanism.class).get()
                            .validateRequest(
                                msgContext.getRequest(), 
                                msgContext.getResponse(), 
                                msgContext);
            } catch (AuthenticationException e) {
                // In case of an explicit AuthException, status will
                // be set to SEND_FAILURE, for any other (non checked) exception
                // the status will be the default NOT_DONE
                setLastAuthenticationStatus(msgContext.getRequest(), SEND_FAILURE);
                throw (AuthException) new AuthException("Authentication failure in HttpAuthenticationMechanism").initCause(e);
            }
            
            setLastAuthenticationStatus(msgContext.getRequest(), status);
            
            return fromAuthenticationStatus(status);
        }

        @Override
        public AuthStatus secureResponse(MessageInfo messageInfo, Subject serviceSubject) throws AuthException {
            HttpMessageContext msgContext = new HttpMessageContextImpl(handler, messageInfo, null);

            try {
                AuthenticationStatus status = CDI.current()
                                                 .select(HttpAuthenticationMechanism.class).get()
                                                 .secureResponse(
                                                     msgContext.getRequest(), 
                                                     msgContext.getResponse(), 
                                                     msgContext);
                AuthStatus authStatus = fromAuthenticationStatus(status);
                if (authStatus == AuthStatus.SUCCESS) {
                    return AuthStatus.SEND_SUCCESS;
                }
                return authStatus;
            } catch (AuthenticationException e) {
                throw (AuthException) new AuthException("Secure response failure in HttpAuthenticationMechanism").initCause(e);
            } finally {
                if (cdiPerRequestInitializer != null) {
                    cdiPerRequestInitializer.destroy(msgContext.getRequest());
                }
            }

        }

        /**
         * Called in response to a {@link HttpServletRequest#logout()} call.
         *
         */
        @Override
        public void cleanSubject(MessageInfo messageInfo, Subject subject) throws AuthException {
            HttpMessageContext msgContext = new HttpMessageContextImpl(handler, messageInfo, subject);
            
            CDI.current()
               .select(HttpAuthenticationMechanism.class).get()
               .cleanSubject(msgContext.getRequest(), msgContext.getResponse(), msgContext);
        }

}
```
# integrate into Weblogic
* registerSecurityConfigurations
    * registerLoginConfig
        * createDelegateModule
            * createModule
                * getServerAuthConfig
                    

> Container 首先需要通过Jaspic提供的Factory ServerAuthConfig，来构造一个代理模块(in weblogic is JaspicSecurityModule)
>> (实现为DefaultServerAuthConfig由Soteria提供)
```java
  public void createDelegateModule() {
    delegateModule = SecurityModule.createModule( securityContext, this, false);
  }
```

> ServerAuthConfig则由Jaspic提供的另外两个Factory得到
```java
static SecurityModule createModule(ServletSecurityContext context, final WebAppSecurity security,
                                     boolean isRecursiveCall) {
    ServerAuthConfig serverAuthConfig = JaspicUtilities.getServerAuthConfig(context, "HttpServlet",
                                                                            getAppContextId(context),
                                                                            security.getJaspicListener());
    if ((security.isJaspicEnabled() && serverAuthConfig != null) && !isRecursiveCall) {
      return new JaspicSecurityModule(serverAuthConfig, context, security);
    } else {
      return createModule(context, security, false, getAuthMethod(security));
    }
  }
```
> 首先从Jaspic Factory-Factory-Factory AuthConfigFactory得到Factory-Factory AuthConfigProvider
>>(实现AuthConfigFactoryImpl由Jaspic提供)


> 再由Factory-Factory AuthConfigProvider得到Factory ServerAuthConfig
>>(实现为DefaultAuthConfigProvider由Soteria提供)
```java
  public static ServerAuthConfig getServerAuthConfig(ServletSecurityContext ctx, String messageLayer,
                                                     String appContextId, RegistrationListener listener) {
    try {
      AuthConfigFactory factory = AuthConfigFactory.getFactory();
      if (factory == null) return null;

      AuthConfigProvider provider = factory.getConfigProvider( messageLayer, appContextId, listener );
      if (provider == null) return null;

      return provider.getServerAuthConfig( messageLayer, appContextId,
                                           new JaspicCallbackHandler( new ContextImpl( ctx ) ) );
    } catch (AuthException e) {
      return null;
    }
  }
```
> 从而调用构造函数JaspicSecurityModule产生delegateModule实例

>该过程将会实例化delegateModule的属性serverConfig为上述DefaultAuthConfigProvider
```java
  public JaspicSecurityModule( ServerAuthConfig serverAuthConfig, ServletSecurityContext ctx, WebAppSecurity was ) {
    super( ctx, was, false );
    setAuthRealmBanner( ctx.getAuthRealmName() );
    serverConfig = serverAuthConfig;
  }
```
```
delegateModule = {JaspicSecurityModule@21272} 
    samSupport = {JaspicSecurityModule$1@21275} 
    serverConfig = {DefaultServerAuthConfig@21271} //注
        layer = "HttpServlet"
        appContext = "AdminServer /app-mem-basic"
        handler = {JaspicCallbackHandler@21273} 
        providerProperties = null
        serverAuthModule = {HttpBridgeServerAuthModule@21250} // 注
    webAppSecurity = {WebAppSecurityWLS@21265} 
    authRealmBanner = "Basic realm="weblogic""
    delegateControl = false
    securityContext = {ServletSecurityContextImpl@21266} 
 ```
 > 当访问一个配置了Java Security的servlet时

 > 首先由Factory ServerAuthConfig获取代理类ServerAuthContext
 ```java
 ServerAuthContext serverContext = serverConfig.getAuthContext(authContextID, null, createOptionsMap(samSupport));
 AuthStatus status = serverContext.validateRequest( messageInfo, subject, null );
```
 > 再通过代理 ServerAuthContext调用Soteria提供的桥梁类 HttpBridgeServerAuthModule 的validateRequest方法
> 最终调用到用户实际使用的HttpAuthenticationMechanism
```java
public AuthStatus validateRequest(MessageInfo messageInfo, Subject clientSubject, Subject serviceSubject) throws AuthException {
    HttpMessageContext msgContext = new HttpMessageContextImpl(this.handler, messageInfo, clientSubject);
    if(this.cdiPerRequestInitializer != null) {
        this.cdiPerRequestInitializer.init(msgContext.getRequest());
    }

    AuthenticationStatus status = AuthenticationStatus.NOT_DONE;
    Jaspic.setLastAuthenticationStatus(msgContext.getRequest(), status);

    try {
        status = ((HttpAuthenticationMechanism)CDI.current().select(HttpAuthenticationMechanism.class, new Annotation[0]).get()).validateRequest(msgContext.getRequest(), msgContext.getResponse(), msgContext);
    } catch (AuthenticationException var7) {
        Jaspic.setLastAuthenticationStatus(msgContext.getRequest(), AuthenticationStatus.SEND_FAILURE);
        throw (AuthException)(new AuthException("Authentication failure in HttpAuthenticationMechanism")).initCause(var7);
    }

    Jaspic.setLastAuthenticationStatus(msgContext.getRequest(), status);
    return Jaspic.fromAuthenticationStatus(status);
}
```