How can we write integration tests for our Spring Data JPA repositories because they are just interfaces?

This blog post answers to that question. During this blog post we will write integration tests for a Spring Data JPA repository that manages the information of todo entries (Todo objects). To be more specific, we will write integration tests for the findBySearchTerm() method of TodoRepository interface. That method ignores case and returns todo entries whose title or description contains the given search term.

## Getting the Required Dependencies With Maven

We can get the required dependencies with Maven by declaring the following dependencies in our pom.xml file:

* [JUnit](http://junit.org/) (version 4.11).
* [AssertJ Core](http://joel-costigliola.github.io/assertj/assertj-core.html) (version 3.2.0). We use AssertJ for ensuring that the tested method returns the correct information.
* [Spring Test](http://docs.spring.io/spring/docs/4.1.x/spring-framework-reference/html/testing.html) (version 4.1.6.RELEASE).
* [DbUnit](http://dbunit.sourceforge.net/) (version 2.5.1). Remember to exclude the JUnit dependency. We use DbUnit for initializing our database into a known state before each test case is invoked.
* [Spring Test DbUnit](http://springtestdbunit.github.io/spring-test-dbunit/) (version 1.2.1) integrates DbUnit with the Spring Test framework.

The relevant part of our pom.xml file looks as follows:

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

After we have configured the required dependencies in our pom.xml file, we can configure our integration tests.

## Configuring Our Integration Tests

We can configure our integration tests by following these steps:

1. Run integration tests by using the SpringJUnit4ClassRunner class. It is a custom JUnit runner that integrates the Spring Test framework with JUnit. We can configure the used JUnit runner by annotating our test class with the @RunWith annotation.
2. Configure the application context configuration class (or XML configuration file) that configures the application context used by our integration tests. We can configure the used application context configuration class (or XML configuration file) by annotating our test class with the @ContextConfiguration annotation.
3. Configure the test execution listeners which react to the test execution events that are published by the Spring Test framework. We have to configure the following test execution listeners:
* The DependencyInjectionTestExecutionListener provides dependency injection for the test object.
* The TransactionalTestExecutionListener adds transaction support (with default rollback semantics) into our integration tests.
* The DbUnitTestExecutionListener adds support for the features provided by the Spring Test DbUnit library.

After we have added this configuration into our integration test class, its source code looks as follows:

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

Additional Reading:
Spring Framework Reference Documentation: 11.3 Integration Testing
Spring Framework Reference Documentation: 14.5.2 TestExecutionListener configuration
The Javadoc of the @RunWith annotation
The Javadoc of the SpringJUnit4ClassRunner class
The Javadoc of the @ContextConfiguration annotation
The Javadoc of the @TestExecutionListeners annotation
The Javadoc of the TestExecutionListener interface
The Javadoc of the DependencyInjectionTestExecutionListener class
The Javadoc of the TransactionalTestExecutionListener class

After we have configured our integration test class, we can start writing integration tests for our Spring Data JPA repository.

## Writing Integration Tests for Our Repository

We can write integration tests for our repository by following these steps:

First, we have to inject the tested repository into our test class. Because we are writing integration tests for the TodoRepository interface, we have to inject it into our test class. The source code of our test class looks as follows:

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

Second, we have to create the DbUnit dataset that initializes our database into a known state before our test cases are invoked. We will use the flat XML dataset format because it is less verbose than the original DbUnit dataset format. This means that we can create our dataset by following these rules:

* Each XML element contains the information of a single table row.
* The name of the XML element identifies the name of the database table in which its information is inserted.
* The attributes of the XML element specify the values that are inserted into the columns of the database table.

The tested repository (TodoRepository) queries information from the todos table that has the following columns: id, created_by_user, creation_time, description, modified_by_user, modification_time, title, and version.

Because we are writing integration tests for a method that returns a list of Todo objects, we want to insert two rows to the todos table. We can do this by creating a DbUnit dataset file (todo-entries.xml) that looks as follows:

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

> Additional reading:
DbUnit documentation provides more information about the different [DbUnit dataset formats](http://dbunit.sourceforge.net/components.html#dataset).

Third, we can write integration tests for the findBySearchTerm() method of the TodoRepository interface. Let’s write integration tests which ensure that the findBySearchTerm() method is working correctly when the title of one todo entry contains the given search term. We can write these integration tests by following these steps:

1. Configure the used dataset file by annotating the integration test class with the @DatabaseSetup annotation.
2. Write an integration test which ensures that the findBySearchTerm() method returns one todo entry when the search term “iTl” is passed as a method parameter.
3. Write an integration test which ensures that the findBySearchTerm() method returns the “first” todo entry when the search term “iTl” is passed as a method parameter.

The source code of the ITFindBySearchTerm class looks as follows:

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
Additional Reading:

* [Writing Tests for Data Access Code](http://www.petrikainulainen.net/writing-tests-for-data-access-code/) is a five-part tutorial that describes how you can write tests for your data access code, and ensure that your tests are clean and easy to maintain.
* [Spring From the Trenches: Using Null Values in DbUnit Datasets](http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-using-null-values-in-dbunit-datasets/) describes why you should use null values in your DbUnit datasets and explains how you can use them.
* [Spring From the Trenches: Resetting Auto Increment Columns Before Each Test Method](http://www.petrikainulainen.net/programming/spring-framework/spring-from-the-trenches-resetting-auto-increment-columns-before-each-test-method/) describes why you should reset the auto increment columns before each test method and explains how you can do it.

Let’s move on and summarize what we learned from this blog post.

## Summary

This blog post has taught us four things:

* We can integrate DbUnit with the Spring Test framework by using Spring Test DbUnit.
* We can integrate Spring Test DbUnit with the Spring Test framework by using the DbUnitTestExecutionListener class.
* We should use the flat XML databaset format because it is less verbose than the original DbUnit dataset format.
* We can use the @DatabaseSetup annotation on the class level or on the method level.

P.S. You can get the example applications of this blog post from Github ([query methods](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/query-methods), [JPA Criteria API](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/criteria-api), [Querydsl](https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/querydsl)).

> If you want to learn how to use Spring Data JPA, you should read my [Spring Data JPA tutorial](http://www.petrikainulainen.net/spring-data-jpa-tutorial/).
