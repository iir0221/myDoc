# jpa
## Annotation
config
```java
package org.zxy.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import javax.sql.DataSource;

/**
 * Created by xinyuan.zhang on 8/26/17.
 */

@Configuration
@ComponentScan(basePackages = "org.zxy")
public class DataConfig {


    // datasource
    @Bean
    public DataSource dataSource() {

        // HikariDataSource

        HikariConfig hikariConfig = new HikariConfig();
        hikariConfig.setDriverClassName("com.mysql.jdbc.Driver");
        hikariConfig.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        hikariConfig.setUsername("root");
        hikariConfig.setPassword("19900221");

        hikariConfig.setMaximumPoolSize(5);
        hikariConfig.setConnectionTestQuery("SELECT 1");
        hikariConfig.setPoolName("springHikariCP");

        hikariConfig.addDataSourceProperty("dataSource.cachePrepStmts", "true");
        hikariConfig.addDataSourceProperty("dataSource.prepStmtCacheSize", "250");
        hikariConfig.addDataSourceProperty("dataSource.prepStmtCacheSqlLimit", "2048");
        hikariConfig.addDataSourceProperty("dataSource.useServerPrepStmts", "true");

        HikariDataSource dataSource = new HikariDataSource(hikariConfig);

        return dataSource;
    }

    // Hiberanate提供的JPA实现
    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
        HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
        adapter.setDatabase(Database.MYSQL);
        adapter.setShowSql(true);
        adapter.setGenerateDdl(false);
        adapter.setDatabasePlatform("org.hibernate.dialect.MySQL5Dialect");
        return adapter;
    }

    // EntityManagerFactory
    @Bean
    public LocalContainerEntityManagerFactoryBean emf(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter) {
        LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
        emf.setDataSource(dataSource);
        //emf.setPersistenceUnitName("person");
        emf.setJpaVendorAdapter(jpaVendorAdapter);
        emf.setPackagesToScan("org.zxy");
        return emf;
    }

}

```

```java
package org.zxy.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.transaction.annotation.TransactionManagementConfigurer;

import javax.persistence.EntityManagerFactory;

/**
 * Created by xinyuan.zhang on 8/29/17.
 */

@Configuration
@ComponentScan(basePackages = "org.zxy")
@EnableTransactionManagement
public class TranscationConfig implements TransactionManagementConfigurer {

    @Autowired
    private EntityManagerFactory emf;

    @Bean
    public PlatformTransactionManager annotationDrivenTransactionManager() {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(emf);
        return transactionManager;
    }
}
```
entity
```java
package org.zxy.entity;

import javax.persistence.*;

/**
 * Created by xinyuan.zhang on 8/27/17.
 */
@Entity
public class Person {

    @Id
    private int id;
    @Column(name="name")
    private String name;
    @Column(name="age")
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
repostory
```java
package org.zxy.data;

import org.zxy.entity.Person;


/**
 * Created by xinyuan.zhang on 8/27/17.
 */
public interface PersonRepository {

    void save(Person person);
}
```
```java
package org.zxy.data;

import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
import org.zxy.entity.Person;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;


/**
 * Created by xinyuan.zhang on 8/29/17.
 */

@Repository
public class JpaPersonRepository implements PersonRepository{

    @PersistenceContext
    private EntityManager entityManager;

    public void save(Person person) {
        entityManager.persist(person);
    }
}
```
```java
package org.zxy.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.test.annotation.Rollback;
import org.springframework.transaction.annotation.Transactional;
import org.zxy.data.JpaPersonRepository;
import org.zxy.entity.Person;

/**
 * Created by xinyuan.zhang on 8/29/17.
 */

@Service
public class PersonService {

    @Autowired
    private JpaPersonRepository jpaPersonRepository;

    @Transactional
    //@Rollback()
    public void addPerson(Person person) {
        jpaPersonRepository.save(person);

    }


}
```


https://github.com/wicketstuff/core/wiki/How-to-use-@PersistenceUnit-annotation