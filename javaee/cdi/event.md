# Event
* listen events
    * 通过在方法参数上使用@Observes注解，来监听事件（观察者模式）
* fire events
    * javax.enterprise.event.Event#fire()方法

## @Observes通过事件对象的类型，来确定要监听的事件
* Event observers,使用CDI的@Observes注解订阅要处理的事件
```java
package org.cdisandbox.event;

import javax.enterprise.event.Observes;
import java.util.List;
import java.util.Map;

/**
 * @author Antoine Sabot-Durand
 */
public class ParameterizedObservingBean {
    
    // 监听“事件对象”类型为List<Number>的事件
    public void processNumberList(@Observes List<Number> event) {
        event.add(new Integer(1));
    }
    // 监听“事件对象”类型为List<Integer>的事件
    public void processIntegerList(@Observes List<Integer> event) {
            event.add(2);
        }
    // 监听“事件对象”类型为Map<?, ?>的事件
    public void observesMap(@Observes Map<?, ?> event) {
        System.out.println("I'm here !");
    }
    
}
```
* Event producers,使用参数化Event interface的实例。通过@Inject注入该接口的一个实例。
* 通过调用fire()方法，并传递“事件对象”来激活事件处理
```java
import junit.framework.Assert;
import org.cdisandbox.event.ParameterizedObservingBean;
import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.shrinkwrap.api.Archive;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.EmptyAsset;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.junit.Test;
import org.junit.runner.RunWith;

import javax.enterprise.event.Event;
import javax.inject.Inject;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author Antoine Sabot-Durand
 */

@RunWith(Arquillian.class)
public class ParameterizedEventTest {

    // 注入Event实例，事件对象为List<Number>类型
    @Inject
    Event<List<Number>> listNumberEvent;
    // 注入Event实例，事件对象为List<Integer>类型
    @Inject
    Event<List<Integer>> listIntegerEvent;
    // 注入Event实例，事件对象为Map<?, ?>类型
    @Inject
    Event<Map<?, ?>> paramEvtMap;

    @Deployment
    public static Archive<?> createTestArchive() throws FileNotFoundException {

        WebArchive ret = ShrinkWrap
                .create(WebArchive.class, "test.war")
                .addClasses(ParameterizedObservingBean.class)
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");

        return ret;
    }

    @Test
    public void shouldCallNumberObserver() {
        List<Number> pl = new ArrayList<>();
        listNumberEvent.fire(pl);
        Assert.assertEquals(1, pl.size());
        Assert.assertEquals(1, pl.get(0).intValue());

    }

    @Test
    public void shouldCallIntegerObserver() {
        List<Integer> pl = new ArrayList<>();
        listIntegerEvent.fire(pl);
        Assert.assertEquals(1, pl.size());
        Assert.assertEquals(2, pl.get(0).intValue());

    }

    @Test
    public void shouldWork() {
        Map<?, ?> pl = new HashMap<>();
        paramEvtMap.fire(pl);
        Assert.assertTrue(true);

    }
}
```
## event.fire(subClass)将会触发observersMethod(@Observes supClass)
* Super Event Object
```java
package org.cdisandbox.event.Objects;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
public class Role {

    public int totalNum = 0;

    public int managerNum = 0;
}
```
* Sub Event Object
```java
package org.cdisandbox.event.Objects;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
public class SubRole extends Role{
}
```
* Event Observers
```java
package org.cdisandbox.event.Observes;

import org.cdisandbox.event.Objects.Role;
import org.cdisandbox.event.Objects.SubRole;
import org.cdisandbox.event.Qualifiers.AddManager;

import javax.enterprise.event.Observes;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
public class RoleObservingBean {

    public void observesAddRole(@Observes Role event) {
        event.totalNum += 1;
        System.out.println("all role");
    }

    public void observesAddRole2(@Observes SubRole event) {
        event.totalNum += 1;
        System.out.println("all SubRole");
    }

}
```
* Event producers
```java
import junit.framework.Assert;
import org.cdisandbox.event.*;
import org.cdisandbox.event.Objects.Role;
import org.cdisandbox.event.Objects.SubRole;
import org.cdisandbox.event.Observes.RoleObservingBean;
import org.cdisandbox.event.Qualifiers.AddManager;
import org.cdisandbox.event.Qualifiers.AddManagerLiteral;
import org.jboss.arquillian.container.test.api.Deployment;
import org.jboss.arquillian.junit.Arquillian;
import org.jboss.shrinkwrap.api.Archive;
import org.jboss.shrinkwrap.api.ShrinkWrap;
import org.jboss.shrinkwrap.api.asset.EmptyAsset;
import org.jboss.shrinkwrap.api.spec.WebArchive;
import org.junit.Test;
import org.junit.runner.RunWith;

import javax.enterprise.event.Event;
import javax.enterprise.inject.Any;
import javax.enterprise.util.AnnotationLiteral;
import javax.inject.Inject;
import java.io.FileNotFoundException;

/**
 * @author Antoine Sabot-Durand
 */

@RunWith(Arquillian.class)
public class MyTest {

    @Deployment
    public static Archive<?> createTestArchive() throws FileNotFoundException {

        WebArchive ret = ShrinkWrap
                .create(WebArchive.class, "test.war")
                .addClasses(SimpleObservingBean.class,
                        Role.class,
                        AddManager.class,
                        RoleObservingBean.class
                )
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml");

        return ret;
    }

    @Inject
    @Any
    Event<Role> addRoleEvent;

    @Test
    public void UnqualifiedObserver() {
        Role role = new Role();
        addRoleEvent.fire(role);
        Assert.assertEquals(1, role.totalNum);
    }

    @Test
    public void UnqualifiedSubObserver() {
        SubRole subRole = new SubRole();
        addRoleEvent.fire(subRole);
        Assert.assertEquals(2,subRole.totalNum);
    }
}
```
```
Test UnqualifiedObserver 将会触发observesAddRole
Test UnqualifiedSubObserver将会触发observesAddRole和observesAddRole2
```

## Event qualifiers
* @Qualifer
    * @Qualifer注释Event与注释Bean的用法相同，用来区别于其他相同类型的事件Event(Bean)，通过@Qualifer（选择器）组合可以缩小事件通知
* Event Object
```java
package org.cdisandbox.event.Objects;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
public class Role {

    public int totalNum = 0;

    public int managerNum = 0;
}
```
* Qualifier
```java
package org.cdisandbox.event.Qualifiers;

import javax.inject.Qualifier;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
@Qualifier
@Target({TYPE, METHOD, PARAMETER, FIELD})
@Retention(RUNTIME)
@Documented
public @interface AddManager {

    String value() default "";

}
```
* Event Observers
```java
package org.cdisandbox.event.Observes;

import org.cdisandbox.event.Objects.Role;
import org.cdisandbox.event.Qualifiers.AddManager;

import javax.enterprise.event.Observes;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
public class RoleObservingBean {


    // 将会监听所有Role类型Event
    public void observesAddRole(@Observes Role event) {
        event.totalNum += 1;
        System.out.println("all role");
    }

    // 仅仅监听Qualified为AddManager的Role类型Event
    public void observesAddManager(@Observes @AddManager Role event) {
        event.managerNum += 1;
        System.out.println("only manager");
    }

}
```
* Event producers
```java
@Inject
Event<Role> addRoleEvent;

@Inject
@AddManager
Event<Role> addManagerEvent;

@Test
public void UnqualifiedObserver() {
    Role role = new Role();
    addRoleEvent.fire(role);
    Assert.assertEquals(1, role.totalNum);
}
@Test
public void QualifiedAndUnqualifiedObserver() {
    Role role = new Role();
    addManagerEvent.fire(role);
    Assert.assertEquals(1,role.managerNum);
    Assert.assertEquals(1,role.totalNum);
}
```
```
Test UnqualifiedObserver将会触发observesAddRole
Test QualifiedAndUnqualifiedObserver将会触发observesAddRole以及observesAddManager
```

### 使用方式
Qualifiers 在事件中应用方式有两种:
#### 注解到Event上
* 缺点：不能动态的指定event qualifiers
* 如上例所示
```java
@Inject
Event<Role> addRoleEvent;

@Inject
@AddManager
Event<Role> addManagerEvent;
```

#### AnnotationLiteral动态注入事件
* 优点：可以在程序中判断后进行分支处理
* AddManagerLiteral extends AnnotationLiteral\<AddManager\> implements AddManager
```java
package org.cdisandbox.event.Qualifiers;

import javax.enterprise.util.AnnotationLiteral;

/**
 * Created by xinyuan.zhang on 11/30/17.
 */
public class AddManagerLiteral extends AnnotationLiteral<AddManager> implements AddManager {

    private String value="";

    public AddManagerLiteral(String value) {
        this.value = value;
    }

    public AddManagerLiteral() {
        this("");
    }

    @Override
    public String value() {
        return value;
    }
}
```
* Event producers
```java
@Inject
// @Any qualifier is totally useless on events or observers
@Any
Event<Role> addRoleEvent;

@Inject
@AddManager
Event<Role> addManagerEvent;

//@Test
public void QualifiedAndUnqualifiedObserver() {
    Role role = new Role();
    addManagerEvent.fire(role);
    Assert.assertEquals(1, role.managerNum);
}

@Test
public void QualifiedAndUnqualifiedObserver2() {
    Role role = new Role();
    // Event is a tool to build an event from an object and qualifiers. 
    // So keep in mind that when you use Event.select(Annotation... qualifiers)
    // you are adding qualifier to the event you’ll be firing
    addRoleEvent.select(new AddManagerLiteral() {})
            .fire(role);
    Assert.assertEquals(1,role.managerNum);
    Assert.assertEquals(1,role.totalNum);
}
```
```
Test QualifiedAndUnqualifiedObserver QualifiedAndUnqualifiedObserver2 结果一样
```
#### 组合使用
Event qualifiers can comprise a combination of annotations at the Event injection point and qualifier instances passed to select().

### Members of event qualifiers
>@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。可以通过default来声明参数的默认值。

event qualifiers的 members即可以用来区分qualifier，也可以用来提供额外的信息
## other parameters in Observer method
* Note that the observer method may have other beans as parameters. They will be injected by the container before calling the method
```java
public void listenToPayload(@Observes Payload event, PayloadService service) {
  ...
}
```
* 见下例
## Events Metadata
*  It allows an observer to get all the metadata about an event. You get the EventMetadata by adding it to the observer parameters
* The EventMetadata contains the following methods:
    * getQualifiers() returns the set of qualifiers with which the event was fired.
    * getInjectionPoint() returns the InjectionPoint from which this event payload was fired, or null if it was fired from BeanManager.fireEvent(…).
    * getType() returns the type representing runtime class of the event object with type variables resolved.
* This bring a solution to add more fine-grained filtering on observer execution depending on actual metadata of the triggered event

* Event Observes
```java
public void observesAddManagerWithParamRole(@Observes @AddManager("special") Role event, EventMetadata meta) {
    AddManager addManager;
    for (Annotation a : meta.getQualifiers()) {
        if (a.annotationType().equals(AddManager.class)) {
            addManager = (AddManager) a;
            System.out.println("***** Memeber is: " + addManager.value());
        }
    }
    event.managerNum+=1;
}
```
* Event producers
```java
@Test
public void QualifiedWithParamAndUnqualifiedObserver() {
    Role role = new Role();
    addRoleEvent.select(new AddManagerLiteral("special")).fire(role);

    Assert.assertEquals(1,role.managerNum);
    Assert.assertEquals(1,role.totalNum);
}
```
## Pattern and tips with events
### The plugin Pattern
We saw that CDI event data is totally free. You can choose any object (again avoid no dependent bean) to fire an event and this object will be received as a playlod by each observer matching the event type and qualifier. An other interesting fact is that this payload is mutable and can be modified by its observers. Following this idea, observers can become a way to enrich a given object with new data. We can use this approach to seamlessly enhance content by adding a CDI archive to an existing application.

### The catch them all pattern
Need to observe all fired event and have their info (for logging purpose for instance), you only have to observe Object.

public void processPayload(@Observes Object event, EventMetadata meta) {}
EventMetadata will even help you to know in which bean the event was fired. A nice way to build a bridge with a messaging service (did I say JMS? ;) )