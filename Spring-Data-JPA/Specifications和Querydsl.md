# Spring Data JPA进阶——Specifications和Querydsl

本篇介绍一下Spring Data JPA中能为数据访问程序的开发带来更多便利的特性，Spring Data repository的配置很简单，一个典型的repository像下面这样：

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

为了重用查询条件，我们引入了Specification接口，这是从Eric Evans’ Domain Driven Design 一书中的概念衍生出来的，它定义了作为判定条件的规格It defines a specification as a predicate over an entity which is exactly what our Specification interface represents. 这个接口只包含下面一个方法：

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

The basic repository implementation will prepare the CriteriaQuery, Root and CriteriaBuilder for you, apply the Predicate created by the given Specification and execute the query. But couldn’t we just have created simple query methods to achieve that? Correct, but remember our second initial requirement. We wanted to be able to freely combine atomic Specifications to create new ones one the fly. To do so we have a helper class Specifications that provides and(…) and or(…) methods to concatenate atomic Specifications. There’s also a where(…) that provides some syntactic sugar to make the expression more readable. The use case sample I came up with in the beginning looks like this:

customerRepository.findAll(where(customerHasBirthday()).and(isLongTermCustomer()));

This reads fluently, improving readability as well as providing additional flexibility as compared to the use of the JPA Criteria API alone. The only caveat here is that coming up with the Specification implementation requires quite some coding effort.

Querydsl

To cure that pain an open-source project called Querydsl has come up with a quite similar but also different approach. Just like the JPA Criteria API it uses a Java 6 annotation processor to generate meta-model objects but produces a much more approachable API. Another cool thing about the project is that it has not only has support for JPA but also allows querying Hibernate, JDO, Lucene, JDBC and even plain collections.

So to get that up and running you add Querydsl to your pom.xml and configure the APT plugin accordingly.

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

This will cause your build to create special query classes - QCustomer inside the very same package in our case.

QCustomer customer = QCustomer.customer;
LocalDate today = new LocalDate();
BooleanExpression customerHasBirthday = customer.birthday.eq(today);
BooleanExpression isLongTermCustomer = customer.createdAt.lt(today.minusYears(2));

This is not only almost fluent English out of the box, the BooleanExpressions are even reusable without further wrapping which lets us get rid off the additional (and a bit ugly to implement) Specification wrapper. A further plus is that you get IDE code completion at every dot on the right hand side of the assignments, so customer. + CTRL + SPACE would list all properties. customer.birthday. + CTRL + SPACE would list all available keywords and so on. To execute Querydsl predicates you simply let your repository extend QueryDslPredicateExecutor:

public interface CustomerRepository extends JpaRepository<Customer>, QueryDslPredicateExecutor {
  // Your query methods here
}

Clients can then simply do:

BooleanExpression customerHasBirthday = customer.birthday.eq(today);
BooleanExpression isLongTermCustomer = customer.createdAt.lt(today.minusYears(2));
customerRepository.findAll(customerHasBirthday.and(isLongTermCustomer));

Summary

Spring Data JPA repository abstraction allows executing predicates either via JPA Criteria API predicates wrapped into a Specification object or via Querydsl predicates. To enable this functionality you simply let your repository extend JpaSpecificationExecutor or QueryDslPredicateExecutor (you could even use both side by side if you liked). Note that you need the Querydsl JARs in the class in case you decide for the Querydsl approach.

One more thing

One more cool thing about the Querydsl approach is that it is not only available for our JPA repositories but for our MongoDB support as well. The functionality is included in the just released M2 release of Spring Data MongoDB already. Beyond that both the Mongo and JPA module of Spring Data are supported on the CloudFoundry platform. See the cloudfoundry-samples wiki for getting started with Spring Data and CloudFoundry.
