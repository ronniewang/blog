书接[上文](https://github.com/ronniewang/blog/blob/master/Spring%20Data%20JPA%E6%95%99%E7%A8%8B%EF%BC%9A%E5%AE%A1%E8%AE%A1%EF%BC%88%E4%B8%80%EF%BC%89.md)

本文解决前面两个问题中的第二个问题，我们将为实体加上创建者和修改者的信息

首先创建一个返回授权用户信息的组件

### 获取授权用户信息 

Spring Data JPA使用`AuditorAware<T>`接口获取用户信息，`AuditorAware`接口的泛型参数T描述了实体类中审计人的类型

现在开始创建一个返回用户信息的类：

1. 创建`UsernameAuditorAware`类实现`AuditorAware`接口，我们想存储`String`类型的用户名，所以参数T设置为`String`
2. 实现`getCurrentAuditor()`方法：
	1. 从`SecurityContext`获取`Authentication`对象
	2. 如果得到的授权对象为`null`或者未经认证，返回`null`
	3. 返回username

`UsernameAuditorAware`类源码如下：

```java
import org.springframework.data.domain.AuditorAware;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.User;

public class UsernameAuditorAware implements AuditorAware<String> {

    @Override
    public String getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication == null || !authentication.isAuthenticated()) {
            return null;
        }
        return ((User) authentication.getPrincipal()).getUsername();
    }
}
```
### 配置Application Context

下面我们将`UsernameAuditorAware `配置成一个bean，修改`PersistenceContext`类，步骤如下：

1. 创建`auditorProvider()`方法返回一个`AuditorAware<String>`对象
2. 方法实现里new一个`UsernameAuditorAware`对象返回
3. 给方法加上`@Bean`注解
4. 加上`@EnableJpaAuditing`注解

`PersistenceContext`类代码如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.auditing.DateTimeProvider;
import org.springframework.data.domain.AuditorAware;
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
    AuditorAware<String> auditorProvider() {

        return new UsernameAuditorAware();
    }

    @Bean
    DateTimeProvider dateTimeProvider(DateTimeService dateTimeService) {

        return new AuditingDateTimeProvider(dateTimeService);
    }
}
```

因为我们只有一个`AuditorAware`bean，所以Spring需要的时候会自动找到它，如果有多个bean，可以通过配置`@EnableJpaAuditing`注解的auditorAwareRef属性值来设置。

如果使用XML配置，参照<https://github.com/pkainulainen/spring-data-jpa-examples/blob/master/query-methods/src/main/resources/applicationContext-persistence.xml>

下面修改实体类

### 修改实体类

有如下两个需求：

1. 确保`createdByUser`字段在实体第一次存储时被设置
2. 确保`modifiedByUser`字段在实体以一次存储和以后被修改时被设置

具体步骤如下：

1. 创建一个`String`类型的`createdByUser`字段：
	1. 加上`@Column`注解，配置字段名为created_by_user，非空
	2. 加上`@CreatedBy`注解，说明这个字段表示的是谁创建了这个实体

2. 创建一个`String`类型的`modifiedByUser`字段：
	1. 加上`@Column`注解，配置字段名为modified_by_user，非空
	2. 加上`@LastModified`注解，说明这个字段表示的是谁最近修改了这个实体

修改后的`Todo`实体类如下：

```java
import org.hibernate.annotations.Type;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
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

    @Column(name = "created_by_user", nullable = false)
    @CreatedBy
    private String createdByUser;

    @Column(name = "creation_time", nullable = false)
    @Type(type = "org.jadira.usertype.dateandtime.threeten.PersistentZonedDateTime")
    @CreatedDate
    private ZonedDateTime creationTime;

    @Column(name = "description", length = 500)
    private String description;

    @Column(name = "modified_by_user", nullable = false)
    @LastModifiedBy
    private String modifiedByUser;

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
如果有多个实体类需要审计，也可以将审计字段移到一个抽象基类中，就像下面：

```java
import org.hibernate.annotations.Type;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import javax.persistence.Column;
import javax.persistence.MappedSuperClass

@EntityListeners(AuditingEntityListener.class)
@MappedSuperClass
public abstract class BaseEntity {

    @Column(name = "created_by_user", nullable = false)
    @CreatedBy
    private String createdByUser;

    @Column(name = "creation_time", nullable = false)
    @Type(type = "org.jadira.usertype.dateandtime.threeten.PersistentZonedDateTime")
    @CreatedDate
    private ZonedDateTime creationTime;

    @Column(name = "modified_by_user", nullable = false)
    @LastModifiedBy
    private String modifiedByUser;

    @Column(name = "modification_time")
    @Type(type = "org.jadira.usertype.dateandtime.threeten.PersistentZonedDateTime")
    @LastModifiedDate
    private ZonedDateTime modificationTime;
}
```

如果不想使用注解，可以实现`Auditable`接口或者继承`AbstractAuditable`类。`Auditable`接口声明了所有的审计字段的getter和setter方法，`AbstractAuditable`类提供了这些方法的实现，但有个劣势是你的实体类和Spring Data就产生了耦合

让我们看看为什么使用Spring Data JPA提供的审计支持而不是使用Java Persistence API提供的回调方法

### 为什么使用Spring Data JPA提供的审计支持？

我们还可以通过使用JPA的生命周期注解来标注一些方法，这些方法会在存储和更新时执行回调，例如：

```java
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.User;
import javax.persistence.Column;
import javax.persistence.MappedSuperClass
import javax.persistence.PrePersist;
import javax.persistence.PreUpdate;

@MappedSuperClass
public abstract class BaseEntity {

    @Column(name = "created_by_user", nullable = false)
    @CreatedBy
    private String createdByUser;

    @Column(name = "modified_by_user", nullable = false)
    @LastModifiedBy
    private String modifiedByUser;

    @PrePersist
    public void prePersist() {

        String createdByUser = getUsernameOfAuthenticatedUser();
        this.createdByUser = createdByUser;
        this.modifiedByUser = createdByUser;
    }

    @PreUpdate
    public void preUpdate() {

        String modifiedByUser = getUsernameOfAuthenticatedUser();
        this.modifiedByUser = modifiedByUser;
    }

    private String getUsernameOfAuthenticatedUser() {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication == null || !authentication.isAuthenticated()) {
            return null;
        }
        return ((User) authentication.getPrincipal()).getUsername();
    }
}
```

即使这个方式比使用Spring Data JPA提供的基础架构更简单和直接一些，仍然有两个原因支持我们使用更复杂的解决方案：

* 使用回调方法会和Spring Security产生耦合
* 如果我想用Spring Data JPA记录创建时间和修改时间的信息，而设置修改人还是用上面那种方式，用两种方式设置审计信息是没有什么意义的

### 总结一下

本篇说了下面三个事：

* Spring Data JPA使用`AuditorAware<T>`接口获取用户信息
* 我们可以通过注解，或者实现`Auditable`接口，或者继承`AbstractAuditable`类来实现审计功能
* 通过回调lifecycle的方法比较简单，但劣势是我们的抽象基类和Spring Security产生了耦合

项目代码在作者的github上：<https://github.com/pkainulainen/spring-data-jpa-examples/tree/master/query-methods>

原文连接：<http://www.petrikainulainen.net/programming/spring-framework/spring-data-jpa-tutorial-auditing-part-two>
