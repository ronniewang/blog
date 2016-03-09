从[前文](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-introduction-to-solr/)知道Solr提供了REST式的http接口来方便的进行更新和查询索引信息，但是开发环境的配置过于繁琐。

为此，solr提供了两种替代方法使我们可以在应用中使用solr。

* [embedded Solr server](http://wiki.apache.org/solr/Solrj#EmbeddedSolrServer)直接连接Solr core，在开发环境是一个很好的选择，[但是不推荐在产品环境下使用](http://wiki.apache.org/solr/EmbeddedSolr)
* [HTTP Solr server](http://wiki.apache.org/solr/Solrj#HttpSolrServer)通过HTTP连接外部的Solr core，这是使用Solr的推荐方式

这篇文章介绍怎样通过Maven配置依赖，还有Spring Data Solr的配置

## 通过Maven配置依赖

We can get the required dependencies with Maven by following these steps:

1. Add the Spring Milestone Maven repository to the POM file.
2. Add the required dependencies to the pom.xml file.

Both of these steps are described with more details in the following.

### Adding the Spring Milestone Maven Repository to the POM File

We can add the Spring milestone Maven repository to our POM file by adding the following XML to the `pom.xml` file:

```xml
<repositories>
    <repository>
        <id>spring-milestone</id>
        <name>Spring Milestone Maven Repository</name>
        <url>http://repo.springsource.org/libs-milestone</url>
    </repository>
</repositories>
```

### Adding the Required Dependencies to the POM File

We can add the required dependencies to the POM file by following these steps:

1. Add the Spring Data Solr dependency (version 1.0.0.RC1) to the dependencies section of our POM file.
2. Add the Solr core dependency (version 4.1.0) to the dependencies section of our POM file and exclude the SLF4J JDK14 binding. Because Solr core is required by the embedded Solr server, we can skip this step if we are not using the embedded Solr server.

We can complete these steps by adding the following XML to the dependencies section of the POM file:

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

## Configuring Spring Data Solr

This section describes how we can configure Spring Data Solr to use different Solr servers in the development and production environment. We will use the embedded Solr server in the development environment and the HTTP Solr server in the production environment.

We can configure Spring Data Solr by following these steps:

1. Create a properties file.
2. Configure the embedded Solr server.
3. Configure the HTTP Solr server.
4. Set the active bean definition profile.

These steps are described with more details in the following subsections.

### Creating the Properties File

The name of our properties file is application.properties and we will use it to configure two properties which are described in the following:

* The `solr.server.url` property specifies the url of the used Solr server. The value of this property is used to configure the HTTP Solr server which is used in the production environment.
* The `solr.solr.home` configures the home directory of Solr. The value of this property is used to configure the home directory of the embedded Solr server which is used in the development environment.

The content of the application.properties file looks as follows:

```
solr.server.url=http://localhost:8983/solr/
solr.solr.home=
```

### Configuring the Embedded Solr Server

This subsection describes how we can configure Spring Data Solr to use the embedded Solr server in the development environment.

#### Java Configuration

We can create a configuration class which configures the embedded Solr server by following these steps:

1. Create a class called EmbeddedSolrContext and annotate that class with the @Configuration annotation.
2. Enable Spring Data Solr repositories by annotating that class with the @EnableSolrRepositories annotation and  configuring the root package of our Solr repositories.
3. Annotate the created class with the @Profile annotation and set its value to ‘dev’. This means that this configuration class is bypassed unless the ‘dev’ profile have been activated.
4. Annotate the class with the @PropertySource annotation and set its value to ‘classpath:application.properties’. This configures the location of our property file and adds a PropertySource to Spring’s Environment.
5. Add an Environment field to the class and annotate that field with the @Resource annotation. The injected Environment is used to access the properties which we added to our properties file.
6. Create a method called solrServerFactoryBean() and annotate this method with the @Bean annotation. The implementation of this method creates a new EmbeddedSolrServerFactoryBean object, sets the value of the Solr home and returns the created object.
7. Create a method called solrTemplate() and annotate this method with the @Bean annotation. The implementation of this method creates a new SolrTemplate object and passes the used SolrServer implementation as a constructor argument.

The source code of the EmbeddedSolrContext class looks as follows:

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

We can create an XML configuration file for the embedded Solr server by following these steps:

1. Configure the used properties file by using the property-placeholder element of the context namespace.
2. Enable Solr repositories and configure the base package of our Solr repositories by using the repositories element of the solr namespace.
3. Create a bean configuration for the development profile.
4. Configure the embedded Solr server bean by using the embedded-solr-server element of the solr namespace. Set the value of the Solr home.
5. Configure the Solr template bean. Set the configured embedded Solr server bean as constructor argument.

The contents of the exampleApplicationContext-solr.xml file looks as follows:

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

### Configuring the Http Solr Server

This subsection describes how we can configure Spring Data Solr to use the HTTP Solr server in the production environment.

#### Java Configuration

We can create a configuration class which configures the HTTP Solr server by following these steps:

1. Create a class called HttpSolrContext and annotate that class with the @Configuration annotation.
2. Enable Spring Data Solr repositories by annotating that class with the @EnableSolrRepositories annotation and configuring the root package of our Solr repositories.
3. Annotate the created class with a @Profile annotation and set its value to ‘prod’. This means that this configuration class is bypassed unless the ‘prod’ profile have been activated.
4. Annotate the class with the @PropertySource annotation and set its value to ‘classpath:application.properties’. This configures the location of our property file and adds a PropertySource to Spring’s Environment.
5. Add an Environment field to the class and annotate that field with the @Resource annotation. The injected Environment is used to access the properties which we added to our properties file.
6. Create a method called solrServerFactoryBean() and annotate this method with the @Bean annotation. The implementation of this method create a new HttpSolrServerFactoryBean object, sets the value of the Solr server url and returns the created object.
7. Create a method called solrTemplate() and annotate this method with the @Bean annotation. The implementation of this method creates a new SolrTemplate object and passes the used SolrServer implementation as a constructor argument.

The source code of the HttpSolrContext class looks as follows:

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

We can create an XML configuration file for the HTTP Solr server by following these steps:

1. Configure the used properties file by using the property-placeholder element of the context namespace.
2. Enable Solr repositories and configure the base package of our Solr repositories by using the repositories element of the solr namespace.
3. Create a bean configuration for the production profile.
4. Configure the HTTP Solr server bean by using the solr-server element of the solr namespace. Set the url of the Solr server.
5. Configure the Solr template bean. Set the configured HTTP Solr server bean as a constructor argument.

The content of the exampleApplicationContext-solr.xml file looks as follows:

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

These steps are described with more details in the following.

#### Adding the Required Profiles to the POM File

We can add the required profiles to our Maven build by following these steps:

1. Create a profile for development environment. Set the id of this profile to ‘dev’ and set the value of the build.profile.id property to ‘dev’.
2. Create a profile for the production environment. Set the id of this profile to ‘prod’ and set the value of the build.profile.id property to ‘prod’.

The configuration of our Maven profiles looks as follows:

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

#### Configuring the Jetty Maven Plugin

We can configure the [Jetty Maven plugin](http://wiki.eclipse.org/Jetty/Feature/Jetty_Maven_Plugin) by following these steps:

1. Add the plugin declaration of the Jetty Maven plugin to the plugins section of our Pom file.
2. Configure the stopKey and stopPort of the Jetty Maven plugin.
3. Configure the location of the properties file containing the used system properties.

The configuration of the Jetty Maven plugin looks as follows:

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
 
## Summary

We have now successfully obtained the required dependencies with Maven and configured Spring Data Solr. This blog entry has taught us four things:

* We learned to get the required dependencies with Maven.
* We know that we should use the embedded Solr server only in the development environment and learned how we can configure Spring Data Solr to use it.
* We learned that we should always use the HTTP Solr server in the production environment and know how we can configure Spring Data Solr to use it.
* We know how we can use the bean definition profiles of Spring Framework for creating different configurations for development and production environment.

The [next part待修改](http://www.petrikainulainen.net/programming/solr/spring-data-solr-tutorial-crud-almost/) of my Spring Data Solr tutorial describes how we can add new document to Solr index, update the information of an existing documents and delete documents from the Solr index.

PS. The example application of this blog entry is available at [Github](https://github.com/pkainulainen/spring-data-solr-examples/tree/master/query-methods).

> If you want to learn how to use Spring Data Solr, you should read my [Spring Data Solr tutorial](http://www.petrikainulainen.net/spring-data-solr-tutorial/).
