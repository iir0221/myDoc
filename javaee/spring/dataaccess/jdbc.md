# Config
## Annotation
config
```java
package org.zxy.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcOperations;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import javax.sql.DataSource;

@Configuration
@ComponentScan(basePackages = "org.zxy")
public class DataConfig {

    @Bean
    public DataSource dataSource() {

        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost/test");
        ds.setUsername("root");
        ds.setPassword("19900221");
        return ds;
    }

    @Bean
    public JdbcOperations jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```
entity
```java
package org.zxy.entity;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */
public class Person {

    private int id;
    private String name;
    private int age;

    public Person(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Person() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
repository
```java
package org.zxy.data;

import org.zxy.entity.Person;

import java.util.List;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */
public interface PersonRepository {


    void save(Person person);
}

```
```java
package org.zxy.data;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcOperations;
import org.springframework.stereotype.Repository;
import org.zxy.entity.Person;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */

@Repository
public class JdbcPersonRepository implements PersonRepository{


    private static String INSERT_PERSON = "INSERT INTO person(name,age) VALUES (?,?)";

    private JdbcOperations jdbcOperations;

    @Autowired
    public JdbcPersonRepository(JdbcOperations jdbcOperations) {
        this.jdbcOperations = jdbcOperations;
    }

    public void save(Person person) {
        jdbcOperations.update(
                INSERT_PERSON,
                person.getName(),
                person.getAge()
        );
    }
}
```
test
```java
package org.zxy;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.zxy.config.DataConfig;
import org.zxy.data.JdbcPersonRepository;
import org.zxy.entity.Person;

/**
 * Hello world!
 *
 */
public class App {

    private static AnnotationConfigApplicationContext appContext = new AnnotationConfigApplicationContext(DataConfig.class);
    private static JdbcPersonRepository jdbcPersonRepository = (JdbcPersonRepository)appContext.getBean("jdbcPersonRepository");

    public static void main(String[] args ) {
        Person person = new Person();
        person.setAge(11);
        person.setName("tt");
        addPerson(person);
    }

    public static void addPerson(Person person) {
        jdbcPersonRepository.save(person);
    }

}
```
## xml
config
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Scans within the base package of the application for @Component classes to configure as beans -->
    <context:component-scan base-package="org.zxy" />
    <context:property-placeholder location="jdbc.properties"/>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```
jdbc.properties
```xml
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost/test
jdbc.username=root
jdbc.password=19900221
```
entity
```java
package org.zxy.entity;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */
public class Person {

    private int id;
    private String name;
    private int age;

    public Person(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public Person() {
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
repository
```java
package org.zxy.data;

import org.zxy.entity.Person;

import java.util.List;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */
public interface PersonRepository {


    void save(Person person);
}
```
```java
package org.zxy.data;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcOperations;
import org.springframework.stereotype.Repository;
import org.zxy.entity.Person;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */

@Repository
public class JdbcPersonRepository implements PersonRepository{


    private static String INSERT_PERSON = "INSERT INTO person(name,age) VALUES (?,?)";

    private JdbcOperations jdbcOperations;

    @Autowired
    public JdbcPersonRepository(JdbcOperations jdbcOperations) {
        this.jdbcOperations = jdbcOperations;
    }

    public void save(Person person) {
        jdbcOperations.update(
                INSERT_PERSON,
                person.getName(),
                person.getAge()
        );
    }
}
```
test
```java
package org.zxy;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.zxy.data.JdbcPersonRepository;
import org.zxy.entity.Person;


public class App {

    private static ApplicationContext appContext = new ClassPathXmlApplicationContext("application-context.xml");
    private static JdbcPersonRepository jdbcPersonRepository = (JdbcPersonRepository)appContext.getBean("jdbcPersonRepository");

    public static void main(String[] args ) {
        Person person = new Person();
        person.setAge(43);
        person.setName("bds");
        addPerson(person);
    }

    public static void addPerson(Person person) {
        jdbcPersonRepository.save(person);
    }

}
```
