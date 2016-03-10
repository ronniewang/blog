## AOP简介

AOP全称Aspect Oriented Programming，翻译过来时面向切面编程，AOP为我们提供了另一种编程的视角，在OOP中，Class是最基本的构件，在AOP中，最基本的构件是叫做Aspect的东西，AOP的出现能解决什么问题呢，举个例子，在编写事务代码时，每次都要写

```java
  try{
  dbConnection.setAutoCommit(false);
  ...
  dbConnection.commit();
  <p/>
  <p/>
  } catch (SQLException e) {
  <p/>
  dbConnection.rollback();
  <p/>
  }
```

这样的样本代码，有多少更新数据库的语句就有多少的这种代码，有了AOP，我们可以在更新数据库的方法上标上`@Transactional`注解，AOP框架就会帮我们自动实现对事务的控制

## AOP术语

这些术语并非来自于Spring，含义也并不直观，但是如果Spring发明一套自己的术语，只会更加让人纠结，所以还是沿用前人的风格，这里尽量明白的介绍一下

* Aspect: a modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented using regular classes (the schema-based approach) or regular classes annotated with the @Aspect annotation (the @AspectJ style).
* 连接点（Join point）：程序执行过程中的某个位置，例如开始前，结束后，抛出异常后等等，Spring只支持方法的连接点
* 通知/增强（Advice）：在某个连接点采取的特定动作，一共有5中类型的通知，下面会介绍，大多数框架（包括Spring）都是使用拦截器来实现的通知功能
* 切点（Pointcut）：一个连接点的定义，通知需要通过切点的定义找到相应的连接点，然后进行织入，切点表达式是AOP的核心，Spring使用AspectJ风格的切点表达式
* Introduction: declaring additional methods or fields on behalf of a type. Spring AOP allows you to introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an IsModified interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
* 目标对象（Target object）：object being advised by one or more aspects. Also referred to as the advised object. Since Spring AOP is implemented using runtime proxies, this object will always be a proxied object.
* AOP proxy: an object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy will be a JDK dynamic proxy or a CGLIB proxy.
* Weaving: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.

通知类型：

1. Before advice: 在连接点之前执行，对连接点接下来的执行并不能产生影响（除非抛出一个异常）
2. After returning advice: 在连接点正常返回之后执行，抛出异常不算
3. After throwing advice: 在方法异常退出时执行
4. After (finally) advice: 在连接点退出时执行（无论正常退出还是异常退出）
5. Around advice: 可以将连接点向方法调用一样包裹起来，可以在方法的调用前后执行自定义的逻辑，同时也可以控制是否执行连接点的逻辑，可以执行，也可以直接返回，甚至抛出异常

Around advice is the most general kind of advice. Since Spring AOP, like AspectJ, provides a full range of advice types, we recommend that you use the least powerful advice type that can implement the required behavior. For example, if you need only to update a cache with the return value of a method, you are better off implementing an after returning advice than an around advice, although an around advice can accomplish the same thing. Using the most specific advice type provides a simpler programming model with less potential for errors. For example, you do not need to invoke the proceed() method on the JoinPoint used for around advice, and hence cannot fail to invoke it.

In Spring 2.0, all advice parameters are statically typed, so that you work with advice parameters of the appropriate type (the type of the return value from a method execution for example) rather than Object arrays.

连接点技术是AOP区别于传统的提供拦截的技术的关键点，切点可以将通知独立的应用到相应的对象上，而不与源对象的继承体系发生关系，例如，事务管理可以应用到所有的被切点定义的方法上（例如service层的数据库操作）
