# 测试Spring Data JPA的repository

Spring Data JPA的repository都是接口，怎么测试呢？

这篇文章就回答这个问题，我们测试选择`TodoRepository`中的`findBySearchTerm()`方法作为例子

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
2. 配置application context，通过`@ContextConfiguration`注解来实现
3. 配置test execution listeners，通过他们来相应Spring Test framework在测试执行中发布的事件，需要配置如下几个listener：
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

1. 通过`@DatabaseSetup`注解来配置dataset
2. 写一个测试来确认`findBySearchTerm()`方法在使用“iTl”作为参数进行查询时返回一个对象
3. 写一个测试来确认`findBySearchTerm()`方法在使用“iTl”作为参数进行查询时返回一个id为1的对象

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

> `@DatabaseSetup`有一下两点需要注意：
* 如果测试类的所有测试用例使用一个dataset，就可以将`@DatabaseSetup`注解表在类上，否则，需要表在方法上
* 如果dataset文件和测试类在同一个包下，可以直接写文件的名字，如果不在一个包下，需要些全路径，例如todo-entries.xml文件在foo.bar包下，就得写“/foo/bar/todo-entries.xml”

> 扩展阅读
* [Writing Tests for Data Access Code](http://www.petrikainulainen.net/writing-tests-for-data-access-code/)讲了怎样写干净的和可维护的数据库测试代码
* [Spring From the Trenches: Using Null Values in DbUnit Datasets](http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-using-null-values-in-dbunit-datasets/)讲了在DbUnit的dataset中怎样使用null值以及为什么要使用它们
* [Spring From the Trenches: Resetting Auto Increment Columns Before Each Test Method](http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-resetting-auto-increment-columns-before-each-test-method/)讲了怎样在测试方法运行前重置自动增长的列

## 总结

总结一下：

* 通过Spring Test DbUnit将DbUnit与Spring Test framework进行了
* 通过`DbUnitTestExecutionListener`将Spring Test DbUnit和Spring Test framework进行了集成
* 学会了使用XML编写DbUnit的dataset文件
* 学会了使用`@DatabaseSetup`注解

P.S. 项目代码可在Github ([query methods](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/query-methods), [JPA Criteria API](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/criteria-api), [Querydsl](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/querydsl))获取

> 还可访问更多[Spring Data JPA tutorial](http://www.petrikainulainen.net/spring-data-jpa-tutorial/)教程
