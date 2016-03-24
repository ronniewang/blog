Spring Data JPA的repository都是接口，怎么测试呢？

这篇文章就回答这个问题，我们测试`TodoRepository`中的`findBySearchTerm()`方法

## 获取maven依赖

依赖的pom文件如下：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.assertj</groupId>
    <artifactId>assertj-core</artifactId>
    <version>3.2.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.1.6.RELEASE</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.dbunit</groupId>
    <artifactId>dbunit</artifactId>
    <version>2.5.1</version>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <artifactId>junit</artifactId>
            <groupId>junit</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>com.github.springtestdbunit</groupId>
    <artifactId>spring-test-dbunit</artifactId>
    <version>1.2.1</version>
    <scope>test</scope>
</dependency>
```

下面配置集成测试需要的环境

## 配置集成测试

步骤如下：

1. 通过`SpringJUnit4ClassRunner`来运行测试，这是一个Spring Test框架对JUnit进行定制过的运行器，通过`@RunWith`来进行配置
2. Configure the application context configuration class (or XML configuration file) that configures the application context used by our integration tests. We can configure the used application context configuration class (or XML configuration file) by annotating our test class with the @ContextConfiguration annotation.
3. Configure the test execution listeners which react to the test execution events that are published by the Spring Test framework. We have to configure the following test execution listeners:
* `DependencyInjectionTestExecutionListener`为测试对象提供依赖注入
* `TransactionalTestExecutionListener`提供事务的支持
* `DbUnitTestExecutionListener`提供了Spring Test DbUnit中包含的特性

配置后的配置类如下：

```java
import com.github.springtestdbunit.DbUnitTestExecutionListener;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.TestExecutionListeners;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.support.DependencyInjectionTestExecutionListener;
import org.springframework.test.context.transaction.TransactionalTestExecutionListener;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {PersistenceContext.class})
@TestExecutionListeners({DependencyInjectionTestExecutionListener.class,
        TransactionalTestExecutionListener.class,
        DbUnitTestExecutionListener.class})
public class ITFindBySearchTermTest {
}
```

详情可了解各个类的JavaDoc

下面可以为Spring Data JPA的repository写测试了

## 为Spring Data JPA的repository写测试

步骤如下：

首先，注入要测试的repository，代码如下：

```java
import com.github.springtestdbunit.DbUnitTestExecutionListener;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.TestExecutionListeners;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.support.DependencyInjectionTestExecutionListener;
import org.springframework.test.context.transaction.TransactionalTestExecutionListener;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {PersistenceContext.class})
@TestExecutionListeners({DependencyInjectionTestExecutionListener.class,
        TransactionalTestExecutionListener.class,
        DbUnitTestExecutionListener.class})
public class ITFindBySearchTermTest {
 
    @Autowired
    private TodoRepository repository;
}
```

然后，创建DbUnit的dataset来初始化测试数据，我们使用XML格式的dataset文件，文件具有如下特点：

* 每个XML元素代表一个table的一行
* 每个XML元素的标签名表示表的名称
* 每个XML元素的属性表示一个字段

我们的dataset文件如下：

```xml
<dataset>
    <todos id="1"
           created_by_user="createdByUser"
           creation_time="2014-12-24 11:13:28"
           description="description"
           modified_by_user="modifiedByUser"
           modification_time="2014-12-25 11:13:28"
           title="title"
           version="0"/>
    <todos id="2"
           created_by_user="createdByUser"
           creation_time="2014-12-24 11:13:28"
           description="tiscription"
           modified_by_user="modifiedByUser"
           modification_time="2014-12-25 11:13:28"
           title="Foo bar"
           version="0"/>
</dataset>
```

> DbUnit的dataset的更多格式请看[DbUnit dataset formats](http://dbunit.sourceforge.net/components.html#dataset).

Third, we can write integration tests for the findBySearchTerm() method of the TodoRepository interface. Let’s write integration tests which ensure that the findBySearchTerm() method is working correctly when the title of one todo entry contains the given search term. We can write these integration tests by following these steps:

1. Configure the used dataset file by annotating the integration test class with the @DatabaseSetup annotation.
2. Write an integration test which ensures that the findBySearchTerm() method returns one todo entry when the search term “iTl” is passed as a method parameter.
3. Write an integration test which ensures that the findBySearchTerm() method returns the “first” todo entry when the search term “iTl” is passed as a method parameter.

`ITFindBySearchTerm`代码如下：

```java
import com.github.springtestdbunit.DbUnitTestExecutionListener;
import com.github.springtestdbunit.annotation.DatabaseSetup;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.TestExecutionListeners;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.support.DependencyInjectionTestExecutionListener;
import org.springframework.test.context.transaction.TransactionalTestExecutionListener;
 
import static org.assertj.core.api.Assertions.assertThat;
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {PersistenceContext.class})
@TestExecutionListeners({DependencyInjectionTestExecutionListener.class,
        TransactionalTestExecutionListener.class,
        DbUnitTestExecutionListener.class})
@DatabaseSetup("todo-entries.xml")
public class ITFindBySearchTermTest {
 
    @Autowired
    private TodoRepository repository;
     
    @Test
    public void findBySearchTerm_TitleOfFirstTodoEntryContainsGivenSearchTerm_ShouldReturnOneTodoEntry() {
        List<Todo> searchResults = repository.findBySearchTerm("iTl");
        assertThat(searchResults).hasSize(1);
    }
     
    @Test
    public void findBySearchTerm_TitleOfFirstTodoEntryContainsGivenSearchTerm_ShouldReturnFirstTodoEntry() {
        List<Todo> searchResults = repository.findBySearchTerm("iTl");
 
        Todo found = searchResults.get(0);
        assertThat(found.getId()).isEqualTo(1L);
    }   
}
```

> When you use the @DatabaseSetup annotation, you have to follow these rules:
* If all test methods of your test class use the same dataset, you can configure it by annotating your test class with @DatabaseSetup annotation. However, if all test methods of your test class do not use the same dataset, you have to annotate your test methods with the @DatabaseSetup annotation.
* If the dataset file is in the same package than the integration test class, you can configure it by using the name of the dataset file. On the other hand, if the dataset file is not in same the package than the test class, you have to configure the full path of the dataset file. For example, if your dataset file (todo-entries.xml) is in the package foo.bar, you can configure its full path by using the string: “/foo/bar/todo-entries.xml”.

> Additional Reading:
* [Writing Tests for Data Access Code](http://www.petrikainulainen.net/writing-tests-for-data-access-code/) is a five-part tutorial that describes how you can write tests for your data access code, and ensure that your tests are clean and easy to maintain.
* [Spring From the Trenches: Using Null Values in DbUnit Datasets](http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-using-null-values-in-dbunit-datasets/) describes why you should use null values in your DbUnit datasets and explains how you can use them.
* [Spring From the Trenches: Resetting Auto Increment Columns Before Each Test Method](http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-resetting-auto-increment-columns-before-each-test-method/) describes why you should reset the auto increment columns before each test method and explains how you can do it.

## 总结

总结一下：

* 通过Spring Test DbUnit将DbUnit与Spring Test framework进行了
* 通过DbUnitTestExecutionListener将Spring Test DbUnit和Spring Test framework进行了集成
* 学会了使用XML编写DbUnit的dataset文件
* 学会了使用@DatabaseSetup注解

P.S. 项目代码可在Github ([query methods](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/query-methods), [JPA Criteria API](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/criteria-api), [Querydsl](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/querydsl))获取

> 更多[Spring Data JPA tutorial](http://www.petrikainulainen.net/spring-data-jpa-tutorial/)教程
