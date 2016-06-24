提到审计，首先从脑海中蹦出的就是审计日志，记录了实体版本的修改信息。但是实现审计日志是个既耗时又复杂的任务，幸运的是，大部分时候我们都不需要

然而，还是有些经常碰到的问题：
* 实体什么时候被创建和修改？
* 谁创建和修改了这个实体？

Spring Data JPA的审计功能可以帮助我们解决这两个问题，下面我们介绍一下怎样通过Spring Data JPA提供的审计功能记录实体的创建和修改时间

一开始，我们需要先写一个返回当前日期和时间的服务，定义一个`DateTimeService`接口

为什么使用接口而不是类实现这个服务呢，原因如下：

* 我们想创建两个不同的实现类
 * 第一个实现返回当前的日期和时间
 * 第二个实现用于测试，每次返回相同的日期和时间
* 如果是个产品应用，其他组件也会通过接口进行调用

`DateTimeService`接口的声明只有一个方法：

* `getCurrentDateAndTime()`，该方法返回一个`ZonedDateTime`对象

`DateTimeService`接口代码如下：
```java
import java.time.ZonedDateTime;

public interface DateTimeService {

    ZonedDateTime getCurrentDateAndTime();
}
```

`CurrentTimeDateTimeService`实现了`DateTimeService`接口，`getCurrentDateAndTime()`的实现只是简单的返回了当前时间

`CurrentTimeDateTimeService`实现如下：
```java
import java.time.ZonedDateTime;

public class CurrentTimeDateTimeService implements DateTimeService {

    @Override
    public ZonedDateTime getCurrentDateAndTime() {
        return ZonedDateTime.now();
    }
}
```

下面我们看看怎样将我们写的service与Spring Data JPA进行集成

Spring Data JPA使用`DateTimeProvider`接口获取日期和时间，所以我们只要实现这个接口就可以将我们的service集成到Spring中

步骤如下：

1. 创建一个`AuditingDateTimeProvider`类，实现`DateTimeProvider`接口
2. 添加一个`DateTimeService`类型的字段，通过构造函数注入
3. 实现`getNow()`方法，通过`DateTimeService`对象获取当前日期和时间，返回一个`GregorianCalendar`对象

`AuditingDateTimeProvider`实现如下：
```java
import org.springframework.data.auditing.DateTimeProvider;
import java.util.Calendar;
import java.util.GregorianCalendar;

public class AuditingDateTimeProvider implements DateTimeProvider {

    private final DateTimeService dateTimeService;

    public AuditingDateTimeProvider(DateTimeService dateTimeService) {
        this.dateTimeService = dateTimeService;
    }

    @Override
    public Calendar getNow() {
        return GregorianCalendar.from(dateTimeService.getCurrentDateAndTime());
    }
}
```
下一步是对`ApplicationContext`进行配置，先创建一个`DateTimeService`类型的bean，在configuration class(XML configuration file)中声明。通过如下步骤配置bean：

1. 创建`currentTimeDateTimeService()`方法返回一个`CurrentTimeDateTimeService`对象
2. 加上`@Bean`注解
3. 加上`@Profile`注解，值设置为`Profiles.APPLICATION`，确保应用启动时bean被创建

相应的配置类如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.Profile;

@Configuration
@ComponentScan("net.petrikainulainen.springdata.jpa")
@Import({WebMvcContext.class, PersistenceContext.class})
public class ExampleApplicationContext {

    @Profile(Profiles.APPLICATION)
    @Bean
    DateTimeService currentTimeDateTimeService() {
        return new CurrentTimeDateTimeService();
    }
}
```
然后，将我们的`DateTimeProvider` bean配置到Spring Data JPA，通过更改事例程序的持久化层配置来实现：

1. 创建`dateTimeProvider()`方法返回一个`DateTimeProvider`对象，构造函数传入我们的`DateTimeService`对象
2. 返回一个`AuditingAwareDateTimeProvider`对象
3. 加上`@Bean`注解
4. 加上`@EnableJpaAuditing`注解，dataTimeProviderRef的值设置为dateTimeProvider

相应的PersistenceContext配置如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.auditing.DateTimeProvider;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableJpaAuditing(dateTimeProviderRef = "dateTimeProvider")
@EnableJpaRepositories(basePackages = {
"net.petrikainulainen.springdata.jpa.todo"
})
@EnableTransactionManagement
class PersistenceContext {

    @Bean
    DateTimeProvider dateTimeProvider(DateTimeService dateTimeService) {
        return new AuditingDateTimeProvider(dateTimeService);
    }
}
```

下面修改实体类，要实现如下需求：

1. 确保creationTime在第一次存储时被设置
2. 确保modificationTime在第一次存储和每次更新时被设置

通过下面步骤就能实现上述目标：

1. 为creationTime字段加上`@CreatedDate`注解
2. 为modificationTime字段加上`@LastModifiedDate`注解
3. 为实体类加上`@EntityListeners`注解，将值设置为AuditingEntityListener.class

相应的Todo实体类修改如下：

```java
import org.hibernate.annotations.Type;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EntityListeners;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Version;
import java.time.ZonedDateTime;

@Entity
@EntityListeners(AuditingEntityListener.class)
@Table(name = "todos")
final class Todo {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(name = "creation_time", nullable = false)
    @Type(type = "org.jadira.usertype.dateandtime.threeten.PersistentZonedDateTime")
    @CreatedDate
    private ZonedDateTime creationTime;

    @Column(name = "description", length = 500)
    private String description;

    @Column(name = "modification_time")
    @Type(type = "org.jadira.usertype.dateandtime.threeten.PersistentZonedDateTime")
    @LastModifiedDate
    private ZonedDateTime modificationTime;

    @Column(name = "title", nullable = false, length = 100)
    private String title;

    @Version
    private long version;
}
```
OK，搞定！

[下一章教程](https://github.com/ronniewang/blog/blob/master/Spring%20Data%20JPA%E6%95%99%E7%A8%8B%EF%BC%9A%E5%AE%A1%E8%AE%A1%EF%BC%88%E4%BA%8C%EF%BC%89.md)将回答最开始提出的第二个问题：谁创建和修改了实体？

项目代码在作者的github上：<https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/query-methods>

原文连接：<http://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-auditing-part-one>
