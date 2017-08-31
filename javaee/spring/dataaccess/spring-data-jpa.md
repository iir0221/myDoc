# Core
Spring data jpa提供如下核心接口

Repository
```java
package org.springframework.data.repository;

import java.io.Serializable;

public interface Repository<T, ID extends Serializable> {
}
```
CRUD
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.data.repository;

import java.io.Serializable;

@NoRepositoryBean
public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
    <S extends T> S save(S var1);

    <S extends T> Iterable<S> save(Iterable<S> var1);

    T findOne(ID var1);

    boolean exists(ID var1);

    Iterable<T> findAll();

    Iterable<T> findAll(Iterable<ID> var1);

    long count();

    void delete(ID var1);

    void delete(T var1);

    void delete(Iterable<? extends T> var1);

    void deleteAll();
}
```
分页
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.data.repository;

import java.io.Serializable;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort var1);

    Page<T> findAll(Pageable var1);
}
```
JpaRepository
```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.data.jpa.repository;

import java.io.Serializable;
import java.util.List;
import org.springframework.data.domain.Example;
import org.springframework.data.domain.Sort;
import org.springframework.data.repository.NoRepositoryBean;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.data.repository.query.QueryByExampleExecutor;

@NoRepositoryBean
public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();

    List<T> findAll(Sort var1);

    List<T> findAll(Iterable<ID> var1);

    <S extends T> List<S> save(Iterable<S> var1);

    void flush();

    <S extends T> S saveAndFlush(S var1);

    void deleteInBatch(Iterable<T> var1);

    void deleteAllInBatch();

    T getOne(ID var1);

    <S extends T> List<S> findAll(Example<S> var1);

    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
}
```
MongoRepository
```java
package org.springframework.data.mongodb.repository; 
 
import java.io.Serializable; 
import java.util.List; 
 
import org.springframework.data.domain.Sort; 
import org.springframework.data.repository.NoRepositoryBean; 
import org.springframework.data.repository.PagingAndSortingRepository; 
 
/**
 * Mongo specific {@link org.springframework.data.repository.Repository} interface. 
 *  
 * @author Oliver Gierke 
 */ 
@NoRepositoryBean 
public interface MongoRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> { 
 
 /*
  * (non-Javadoc) 
  * @see org.springframework.data.repository.CrudRepository#save(java.lang.Iterable) 
  */ 
 <S extends T> List<S> save(Iterable<S> entites); 
 
 /*
  * (non-Javadoc) 
  * @see org.springframework.data.repository.CrudRepository#findAll() 
  */ 
 List<T> findAll(); 
 
 /*
  * (non-Javadoc) 
  * @see org.springframework.data.repository.PagingAndSortingRepository#findAll(org.springframework.data.domain.Sort) 
  */ 
 List<T> findAll(Sort sort); 
}
```

Repository 接口是 Spring Data 的一个核心接口，它不提供任何方法，开发者需要在自己定义的接口中声明需要的方法。

如果持久层接口较多，且每一个接口都需要声明相似的增删改查方法，直接继承 Repository 就显得有些啰嗦，这时可以继承 CrudRepository，它会自动为域对象创建增删改查方法，供业务层直接使用。

但是，使用 CrudRepository 也有副作用，它可能暴露了你不希望暴露给业务层的方法。比如某些接口你只希望提供增加的操作而不希望提供删除的方法。针对这种情况，开发者只能退回到 Repository 接口，然后到 CrudRepository 中把希望保留的方法声明复制到自定义的接口中即可。

Selectively exposing CRUD methods
```java
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

  T findOne(ID id);

  T save(T entity);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

分页查询和排序是持久层常用的功能，Spring Data 为此提供了 PagingAndSortingRepository 接口，它继承自 CrudRepository 接口，在 CrudRepository 基础上新增了两个与分页有关的方法。但是，我们很少会将自定义的持久层接口直接继承自 PagingAndSortingRepository，而是在继承 Repository 或 CrudRepository 的基础上，在自己声明的方法参数列表最后增加一个 Pageable 或 Sort 类型的参数，用于指定分页或排序信息即可，这比直接使用 PagingAndSortingRepository 提供了更大的灵活性。

JpaRepository 是继承自 PagingAndSortingRepository 的针对 JPA 技术提供的接口，它在父接口的基础上，提供了其他一些方法，比如 flush()，saveAndFlush()，deleteInBatch() 等。如果有这样的需求，则可以继承该接口。

# use
* 声明一个接口，继承自Repository或其子接口，该接口使用了泛型，需要为其提供两个类型：第一个为该接口处理的domain对象类型，第二个为该domain对象的主键类型。
    ```java
    interface PersonRepository extends Repository<Person, Long> { … }
    ``` 
    * @ RepositoryDefinition等价于继承接口Repository
        ```java
        public interface UserDao extends Repository<AccountInfo, Long> { …… } 
        
        @RepositoryDefinition(domainClass = AccountInfo.class, idClass = Long.class) 
        public interface UserDao { …… }
        ```
* 声明一个方法
    ```java
    interface PersonRepository extends Repository<Person, Long> {
        List<Person> findByLastname(String lastname);
    }
    ```
* 设置Spring 创建代理
    * JavaConfig
        ```
        import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

        @EnableJpaRepositories
        class Config {

        }
        ```
    * XML configuration:
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:jpa="http://www.springframework.org/schema/data/jpa"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/data/jpa
            http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

        <jpa:repositories base-package="com.acme.repositories"/>

        </beans>
        ```

* 注入repository instance
    ```java
    Get the repository instance injected and use it.

    public class SomeClient {

        @Autowired
        private PersonRepository repository;

        public void doSomething() {
            List<Person> persons = repository.findByLastname("Matthews");
        }
    }
    ```

