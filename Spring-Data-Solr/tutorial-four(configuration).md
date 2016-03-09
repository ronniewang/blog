从[前文](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-introduction-to-solr/)知道Solr提供了REST式的http接口来方便的进行更新和查询索引信息，但是开发环境的配置过于繁琐。

为此，solr提供了两种替代方法使我们可以在应用中使用solr。

* [内嵌Solr server](http://wiki.apache.org/solr/Solrj#EmbeddedSolrServer)直接连接Solr core，在开发环境是一个很好的选择，[但是不推荐在产品环境下使用](http://wiki.apache.org/solr/EmbeddedSolr)
* [HTTP Solr server](http://wiki.apache.org/solr/Solrj#HttpSolrServer)通过HTTP连接外部的Solr core，这是使用Solr的推荐方式

这篇文章介绍怎样通过Maven配置依赖，还有Spring Data Solr的配置

## 通过Maven配置依赖

步骤如下：

1. 添加Spring Milestone Maven repository到POM文件
2. 添加依赖到pom.xml文件

具体步骤如下：

### 添加Spring Milestone Maven repository到POM文件

添加如下xml片段到`pom.xml`文件：

```xml
<repositories>
    <repository>
        <id>spring-milestone</id>
        <name>Spring Milestone Maven Repository</name>
        <url>http://repo.springsource.org/libs-milestone</url>
    </repository>
</repositories>
```

### 添加依赖到pom.xml文件

步骤如下：

1. 添加Spring Data Solr依赖到POM文件
2. 添加Solr core依赖到POM文件，排除SLF4J JDK14依赖，因为Solr core是内嵌server使用的依赖，如果不使用内嵌server，可以跳过此步

代码如下：

```xml
<!-- Spring Data Solr -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-solr</artifactId>
    <version>1.0.0.RC1</version>
</dependency>
 
<!-- Required by embedded solr server -->
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-core</artifactId>
    <version>4.1.0</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-jdk14</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

## 配置Spring Data Solr

下面配置让Spring Data Solr在不同环境下使用不同的Solr server

步骤如下：

1. 新建Properties文件
2. 配置内嵌Solr server.
3. 配置HTTP Solr server.
4. 设置active bean definition profile.

具体步骤如下：

### 新建Properties文件

新建Properties文件，起名application.properties，新建下面两个属性：

* `solr.server.url`，配置HTTP Solr server的地址
* `solr.solr.home`，配置内嵌Solr server的主目录

`application.properties`内容如下：

```
solr.server.url=http://localhost:8983/solr/
solr.solr.home=
```

### 配置内嵌Solr Server

下面配置Spring Data Solr在开发环境下使用内嵌Solr server

#### Java Configuration

通过如下步骤建立配置类：

1. 新建`EmbeddedSolrContext`类，用`@Configuration`注解进行标注
2. 添加`@EnableSolrRepositories`注解，并配置package
3. 添加`@Profile`注解，属性值设置为‘dev’，这表明只有开发环境下这个配置类才会有效
4. 添加`@PropertySource`注解，属性值设置为‘classpath:application.properties’，这样就把这个文件对应的`PropertySource`加到了`Environment`中
5. 添加一个`Environment`类型的字段，并加上`@Resource`注解 annotation，让Spring注入`Environment`对象
6. 添加`solrServerFactoryBean()`方法，加上`@Bean`注解，方法实现返回一个`EmbeddedSolrServerFactoryBean`对象，solrHome的值设置为配置文件中的值
7. 添加`solrTemplate()`方法，加上`@Bean`注解，方法实现返回一个`SolrTemplate`对象，传入上面的`solrServerFactoryBean`作为构造参数

`EmbeddedSolrContext`类代码如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.core.env.Environment;
import org.springframework.data.solr.core.SolrTemplate;
import org.springframework.data.solr.repository.config.EnableSolrRepositories;
import org.springframework.data.solr.server.support.EmbeddedSolrServerFactoryBean;
 
import javax.annotation.Resource;
 
@Configuration
@EnableSolrRepositories("net.petrikainulainen.spring.datasolr.todo.repository.solr")
@Profile("dev")
@PropertySource("classpath:application.properties")
public class EmbeddedSolrContext {
 
    @Resource
    private Environment environment;
 
    @Bean
    public EmbeddedSolrServerFactoryBean solrServerFactoryBean() {
        EmbeddedSolrServerFactoryBean factory = new EmbeddedSolrServerFactoryBean();
 
        factory.setSolrHome(environment.getRequiredProperty("solr.solr.home"));
 
        return factory;
    }
 
    @Bean
    public SolrTemplate solrTemplate() throws Exception {
        return new SolrTemplate(solrServerFactoryBean().getObject());
    }
}
```

#### XML Configuration

也可以通过XML来进行配置：

1. Configure the used properties file by using the property-placeholder element of the context namespace.
2. Enable Solr repositories and configure the base package of our Solr repositories by using the repositories element of the solr namespace.
3. Create a bean configuration for the development profile.
4. Configure the embedded Solr server bean by using the embedded-solr-server element of the solr namespace. Set the value of the Solr home.
5. Configure the Solr template bean. Set the configured embedded Solr server bean as constructor argument.

`exampleApplicationContext-solr.xml`内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:solr="http://www.springframework.org/schema/data/solr"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd http://www.springframework.org/schema/data/solr http://www.springframework.org/schema/data/solr/spring-solr.xsd">
 
    <context:property-placeholder location="classpath:application.properties"/>
 
    <!-- Enable Solr repositories and configure repository base package -->
    <solr:repositories base-package="net.petrikainulainen.spring.datasolr.todo.repository.solr"/>
 
    <!-- Bean definitions for the dev profile -->
    <beans profile="dev">
        <!-- Configures embedded Solr server -->
        <solr:embedded-solr-server id="solrServer" solrHome="${solr.solr.home}"/>
 
        <!-- Configures Solr template -->
        <bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
            <constructor-arg index="0" ref="solrServer"/>
        </bean>
    </beans>
 
    <!-- Bean definitions for the prod profile are omitted -->
</beans>
```

### 配置Http Solr Server

下面配置Spring Data Solr在生产环境下使用HTTP Solr server

#### Java Configuration

通过下面步骤使用配置类来进行配置：

1. 新建`HttpSolrContext`类，加上`@Configuration`注解
2. 添加`@EnableSolrRepositories`注解，并配置package
3. Annotate the created class with a @Profile annotation and set its value to ‘prod’. This means that this configuration class is bypassed unless the ‘prod’ profile have been activated.
4. Annotate the class with the @PropertySource annotation and set its value to ‘classpath:application.properties’. This configures the location of our property file and adds a PropertySource to Spring’s Environment.
5. Add an Environment field to the class and annotate that field with the @Resource annotation. The injected Environment is used to access the properties which we added to our properties file.
6. Create a method called solrServerFactoryBean() and annotate this method with the @Bean annotation. The implementation of this method create a new HttpSolrServerFactoryBean object, sets the value of the Solr server url and returns the created object.
7. Create a method called solrTemplate() and annotate this method with the @Bean annotation. The implementation of this method creates a new SolrTemplate object and passes the used SolrServer implementation as a constructor argument.

`HttpSolrContext`类代码如下：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.core.env.Environment;
import org.springframework.data.solr.core.SolrTemplate;
import org.springframework.data.solr.repository.config.EnableSolrRepositories;
import org.springframework.data.solr.server.support.HttpSolrServerFactoryBean;
 
import javax.annotation.Resource;
 
@Configuration
@EnableSolrRepositories("net.petrikainulainen.spring.datasolr.todo.repository.solr")
@Profile("prod")
@PropertySource("classpath:application.properties")
public class HttpSolrContext {
 
    @Resource
    private Environment environment;
 
    @Bean
    public HttpSolrServerFactoryBean solrServerFactoryBean() {
        HttpSolrServerFactoryBean factory = new HttpSolrServerFactoryBean();
 
        factory.setUrl(environment.getRequiredProperty("solr.server.url"));
 
        return factory;
    }
 
    @Bean
    public SolrTemplate solrTemplate() throws Exception {
        return new SolrTemplate(solrServerFactoryBean().getObject());
    }
}
```

#### XML Configuration

也可使用XML进行配置：

1. Configure the used properties file by using the property-placeholder element of the context namespace.
2. Enable Solr repositories and configure the base package of our Solr repositories by using the repositories element of the solr namespace.
3. Create a bean configuration for the production profile.
4. Configure the HTTP Solr server bean by using the solr-server element of the solr namespace. Set the url of the Solr server.
5. Configure the Solr template bean. Set the configured HTTP Solr server bean as a constructor argument.

`exampleApplicationContext-solr.xml`内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:solr="http://www.springframework.org/schema/data/solr"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.2.xsd http://www.springframework.org/schema/data/solr http://www.springframework.org/schema/data/solr/spring-solr.xsd">
 
    <context:property-placeholder location="classpath:application.properties"/>
 
    <!-- Enable Solr repositories and configure repository base package -->
    <solr:repositories base-package="net.petrikainulainen.spring.datasolr.todo.repository.solr"/>
 
    <!-- Bean definitions for the dev profile are omitted -->
 
    <!-- Bean definitions for the prod profile -->
    <beans profile="prod">
        <!-- Configures HTTP Solr server -->
        <solr:solr-server id="solrServer" url="${solr.server.url}"/>
 
        <!-- Configures Solr template -->
        <bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
            <constructor-arg index="0" ref="solrServer"/>
        </bean>
    </beans>
</beans>
```

### Setting the Active Bean Definition Profile

We can select the active bean definition profile by setting the value of the spring.profiles.active system variable. The allowed values of this system variable (in the context of our example application) are described in the following:

* We can configure our application to run in the development profile by setting the value of the spring.profiles.active system variable to ‘dev’.
* When we want configure our application to run in the production profile, we have to set the of the spring.profiles.active system variable to ‘prod’.

We can configure our example application to support both profiles by following these steps:

1. Add required profiles to the POM file.
2. Create the profile specific properties files for system properties.
3. Configure the Jetty Maven plugin.

具体步骤如下：

#### Adding the Required Profiles to the POM File

We can add the required profiles to our Maven build by following these steps:

1. Create a profile for development environment. Set the id of this profile to ‘dev’ and set the value of the build.profile.id property to ‘dev’.
2. Create a profile for the production environment. Set the id of this profile to ‘prod’ and set the value of the build.profile.id property to ‘prod’.

Maven profiles配置如下：

```xml
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <build.profile.id>dev</build.profile.id>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <build.profile.id>prod</build.profile.id>
        </properties>
    </profile>
</profiles>
```

#### Creating the Profile Specific Properties Files for System Properties

The profile specific properties files are located in the sub directories of the profiles directory. The name of each sub directory matches with the values of the build.profile.id properties configured in the pom.xml file.

We can create the profile specific properties files for system properties by following these steps:

1. Create a properties file called system.properties to the profiles/dev directory. This properties file contains the system properties of the development profile.
2. Create a properties file called system.properties to the profiles/prod directory. This properties file contains the system properties of the production profile.

The content of the properties file used to configure the system properties of the development profile looks as follows:

```
spring.profiles.active=dev
```

The content of the properties file used to configure the system properties of the production profile looks as follows:

```
spring.profiles.active=prod
```

#### 配置Jetty Maven Plugin

配置[Jetty Maven plugin](http://wiki.eclipse.org/Jetty/Feature/Jetty_Maven_Plugin)步骤如下：

1. 在pom中添加Jetty Maven plugin的声明
2. 配置stopKey和stopPort
3. 配置system properties文件的位置

Jetty Maven plugin配置代码如下：

```
<plugin>
     <groupId>org.mortbay.jetty</groupId>
     <artifactId>jetty-maven-plugin</artifactId>
     <version>8.1.5.v20120716</version>
     <configuration>
         <stopKey>todostop</stopKey>
         <stopPort>9999</stopPort>
         <systemPropertiesFile>${project.basedir}/profiles/${build.profile.id}/system.properties</systemPropertiesFile>
     </configuration>
 </plugin>
 ```
 
## 总结

我们介绍了如下几点：

* 配置Maven依赖
* 下面配置Spring Data Solr在开发环境下使用内嵌Solr server
* 下面配置Spring Data Solr在生产环境下使用HTTP Solr server
* 使用Spring Framework提供bean definition profiles来管理不同环境的不同配置

The [next part待修改](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-crud-almost/) of my Spring Data Solr tutorial describes how we can add new document to Solr index, update the information of an existing documents and delete documents from the Solr index.

PS. 可在[Github](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods)获取源代码

> 更多关于Spring Data Solr，请移步[Spring Data Solr tutorial](http://www.petrikainulainen.net/spring-data-solr-tutorial/).
