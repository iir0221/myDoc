# Security

## Classload

### Class.forName的类加载器
* Class.forName(String className)使用的类加载器是调用者的类加载器，不一定是System类加载器
* Class.forName(String name,boolean initialize,ClassLoader loader)可以指定类加载器，推荐通过这种方式来获取类


### 当前线程上下文类加载器
* 主线程的类加载器是系统的类加载器
* 当新线程创建时，它的上下文类加载器会被设置为创建该线程的上下文类加载器
* 因此，如果没有任何特殊操作，所有线程的上下文类加载器都是系统类加载器
* 当然也可以设置某个线程的类加载器
```java
Thread t = Thread.currentThread();
t.setContextClassLoader(loader);
```

## Classloader作为命名空间
一个类是有它的限定名+Classloader来决定的，因此在一个虚拟机中，可以由两个类拥有同样的限定名

## security manager
```
public class SecurityManager
extends Object
The security manager is a class that allows applications to implement a security policy. It allows an application to determine, before performing a possibly unsafe or sensitive operation, what the operation is and whether it is being attempted in a security context that allows the operation to be performed. The application can allow or disallow the operation.
The SecurityManager class contains many methods with names that begin with the word check. These methods are called by various methods in the Java libraries before those methods perform certain potentially sensitive operations. The invocation of such a check method typically looks like this:

     SecurityManager security = System.getSecurityManager();
     if (security != null) {
         security.checkXXX(argument,  . . . );
     }
 
The security manager is thereby given an opportunity to prevent completion of the operation by throwing an exception. A security manager routine simply returns if the operation is permitted, but throws a SecurityException if the operation is not permitted. The only exception to this convention is checkTopLevelWindow, which returns a boolean value.

The current security manager is set by the setSecurityManager method in class System. The current security manager is obtained by the getSecurityManager method.

The special method checkPermission(java.security.Permission) determines whether an access request indicated by a specified permission should be granted or denied. The default implementation calls

   AccessController.checkPermission(perm);
 
If a requested access is allowed, checkPermission returns quietly. If denied, a SecurityException is thrown.

As of Java 2 SDK v1.2, the default implementation of each of the other check methods in SecurityManager is to call the SecurityManager checkPermission method to determine if the calling thread has permission to perform the requested operation.

Note that the checkPermission method with just a single permission argument always performs security checks within the context of the currently executing thread. Sometimes a security check that should be made within a given context will actually need to be done from within a different context (for example, from within a worker thread). The getSecurityContext method and the checkPermission method that includes a context argument are provided for this situation. The getSecurityContext method returns a "snapshot" of the current calling context. (The default implementation returns an AccessControlContext object.) A sample call is the following:

   Object context = null;
   SecurityManager sm = System.getSecurityManager();
   if (sm != null) context = sm.getSecurityContext();
 
The checkPermission method that takes a context object in addition to a permission makes access decisions based on that context, rather than on that of the current execution thread. Code within a different context can thus call that method, passing the permission and the previously-saved context object. A sample call, using the SecurityManager sm obtained as in the previous example, is the following:

   if (sm != null) sm.checkPermission(permission, context);
 
Permissions fall into these categories: File, Socket, Net, Security, Runtime, Property, AWT, Reflect, and Serializable. The classes managing these various permission categories are java.io.FilePermission, java.net.SocketPermission, java.net.NetPermission, java.security.SecurityPermission, java.lang.RuntimePermission, java.util.PropertyPermission, java.awt.AWTPermission, java.lang.reflect.ReflectPermission, and java.io.SerializablePermission.

All but the first two (FilePermission and SocketPermission) are subclasses of java.security.BasicPermission, which itself is an abstract subclass of the top-level class for permissions, which is java.security.Permission. BasicPermission defines the functionality needed for all permissions that contain a name that follows the hierarchical property naming convention (for example, "exitVM", "setFactory", "queuePrintJob", etc). An asterisk may appear at the end of the name, following a ".", or by itself, to signify a wildcard match. For example: "a.*" or "*" is valid, "*a" or "a*b" is not valid.

FilePermission and SocketPermission are subclasses of the top-level class for permissions (java.security.Permission). Classes like these that have a more complicated name syntax than that used by BasicPermission subclass directly from Permission rather than from BasicPermission. For example, for a java.io.FilePermission object, the permission name is the path name of a file (or directory).

Some of the permission classes have an "actions" list that tells the actions that are permitted for the object. For example, for a java.io.FilePermission object, the actions list (such as "read, write") specifies which actions are granted for the specified file (or for files in the specified directory).

Other permission classes are for "named" permissions - ones that contain a name but no actions list; you either have the named permission or you don't.

Note: There is also a java.security.AllPermission permission that implies all permissions. It exists to simplify the work of system administrators who might need to perform multiple tasks that require all (or numerous) permissions.

See Permissions in the JDK for permission-related information. This document includes, for example, a table listing the various SecurityManager check methods and the permission(s) the default implementation of each such method requires. It also contains a table of all the version 1.2 methods that require permissions, and for each such method tells which permission it requires.

For more information about SecurityManager changes made in the JDK and advice regarding porting of 1.1-style security managers, see the security documentation.
```