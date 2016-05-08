# Spring Data JPA进阶——Specifications和Querydsl

本篇介绍一下Spring Data JPA中能为数据访问程序的开发带来更多便利的特性，我们知道，Spring Data repository的配置很简单，一个典型的repository像下面这样：

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {

  Customer findByEmailAddress(String emailAddress);

  List<Customer> findByLastname(String lastname, Sort sort);

  Page<Customer> findByFirstname(String firstname, Pageable pageable);
}
```

第一个方法表示根据email查询一个Customer，第二个方法表示根据lastName和排序条件查询一个Customer的集合，第三个方法表示根据fristName和分页的信息查询一页Customer

这样的方式非常简单，甚至不用编写方法的实现就可以实现查询的功能，但是这仍然有个弊端，如果查询条件增长，方法会越来越多，如果能动态的组装查询条件就好了

那么，可以吗？答案当然是yes

我们都知道JPA提供了Criteria API，下面我们就用一个例子，展示一下Criteria的使用，想象这样一个场景，我们想针对长期客户，在生日那天给他发一段祝福，我们怎么做呢？

## 使用Criteria API

我们有两个条件，生日和长期客户，我们假设两年前注册的就是长期客户吧，怎么用JPA 2.0的Criteria API实现呢：

```java
LocalDate today = new LocalDate();

CriteriaBuilder builder = em.getCriteriaBuilder();
CriteriaQuery<Customer> query = builder.createQuery(Customer.class);
Root<Customer> root = query.from(Customer.class);

Predicate hasBirthday = builder.equal(root.get(Customer_.birthday), today);
Predicate isLongTermCustomer = builder.lessThan(root.get(Customer_.createdAt), today.minusYears(2);
query.where(builder.and(hasBirthday, isLongTermCustomer));
em.createQuery(query.select(root)).getResultList();
```

我们先创建了一个LocalDate对象，然后是三行样板代码啊，后面两行是建立查询条件，然后通过where子句连在一起，然后执行查询

上面查询有两个问题

- 第一，由于每次要先建立CriteriaBuilder，CriteriaQuery，Root，所以导致查询条件的重用和扩展性不是很好
- 第二，上面程序可读性一般，并不能一目了然知道程序在干嘛

## 使用Specifications

为了重用查询条件，我们引入了Specification接口，这是从Eric Evans’ Domain Driven Design 一书中的概念衍生出来的，它为对一个实体查询的谓词定义了一个规范，实体类型由Specification接口的泛型参数来决定，这个接口只包含下面一个方法：

```java
public interface Specification<T> {
  Predicate toPredicate(Root<T> root, CriteriaQuery query, CriteriaBuilder cb);
}
```

我们现在可以通过一个工具类很容易的使用它：

```java
public CustomerSpecifications {

  public static Specification<Customer> customerHasBirthday() {
    return new Specification<Customer> {
      public Predicate toPredicate(Root<T> root, CriteriaQuery query, CriteriaBuilder cb) {
        return cb.equal(root.get(Customer_.birthday), today);
      }
    };
  }

  public static Specification<Customer> isLongTermCustomer() {
    return new Specification<Customer> {
      public Predicate toPredicate(Root<T> root, CriteriaQuery query, CriteriaBuilder cb) {
        return cb.lessThan(root.get(Customer_.createdAt), new LocalDate.minusYears(2));
      }
    };
  }
}
```

诚然，这并不是最优雅的代码，但是至少很好地解决了我们重用判定条件的需求，如何执行呢，很简单，我们只要让repository继承JpaSpecificationExecutor接口即可：

```java
public interface CustomerRepository extends JpaRepository<Customer>, JpaSpecificationExecutor {
  // Your query methods here
}
```

然后可以像下面这样调用：

```java
customerRepository.findAll(hasBirthday());
customerRepository.findAll(isLongTermCustomer());
```

默认实现会为你提供CriteriaQuery，Root，CriteriaBuilder等对象，通过给定的Specification应用判定条件，然后执行查询，这样的好处就是我们可以随意组合查询条件，而不用写很多个方法，Specifications工具类提供了一写遍历方法来组合条件，例如and(…)、or(…)等连接方法，还有where(…)提供了更易读的表达形式，下面我们看一下效果：

```java
customerRepository.findAll(where(customerHasBirthday()).and(isLongTermCustomer()));
```

相比JPA Criteria API的原生接口，我们的实现更加具有扩展性和可读性，当时实现Specification的时候需要一点小波折，但这是值得的

## 使用Querydsl

为了解决上述的痛苦，一个叫Querydsl的开源项目也提供了类似的解决方案，但是实现有所不同，提供了更有好的API，而且不仅支持JPA，还支持Hibernate，JDO，Lucene，JDBC甚至是原始集合的查询

为了使用Querydsl，需要在pom.xml中引入依赖并且配置一个额外的APT插件

```xml
<plugin>
  <groupId>com.mysema.maven</groupId>
  <artifactId>maven-apt-plugin</artifactId>
  <version>1.0</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>process</goal>
      </goals>
      <configuration>
        <outputDirectory>target/generated-sources</outputDirectory>
        <processor>com.mysema.query.apt.jpa.JPAAnnotationProcessor</processor>
      </configuration>
    </execution>
  </executions>
</plugin>
```

下面就可以通过QCustomer来实现我们上述的功能了

```java
QCustomer customer = QCustomer.customer;
LocalDate today = new LocalDate();
BooleanExpression customerHasBirthday = customer.birthday.eq(today);
BooleanExpression isLongTermCustomer = customer.createdAt.lt(today.minusYears(2));
```

上面的写法不仅读来很顺畅，BooleanExpressions还可以直接重用，免去使用更多包装方法的写法，更酷的是还可以得到IDE代码自动完成的支持，要执行查询，跟Specification类似，让repository继承QueryDslPredicateExecutor接口即可：

```java
public interface CustomerRepository extends JpaRepository<Customer>, QueryDslPredicateExecutor {
  // Your query methods here
}
```

可以通过下面的方式调用

```java
BooleanExpression customerHasBirthday = customer.birthday.eq(today);
BooleanExpression isLongTermCustomer = customer.createdAt.lt(today.minusYears(2));
customerRepository.findAll(customerHasBirthday.and(isLongTermCustomer));
```

## 总结

Spring Data JPA repository抽象允许通过把JPA Criteria API包装到Specification中来简化开发，还可以使用Querydsl，实现方法也很简单，分别集成JpaSpecificationExecutor或者QueryDslPredicateExecutor即可，当然，如果需要的话，一起使用也没问题

原文<https://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl/>
