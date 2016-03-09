In the previous part of my Spring Data Solr tutorial, we learned that Solr provides a REST-like HTTP API which can be used to add information to Solr index and execute queries against indexed data. The problem is that running a separate Solr instance in a development environment is a bit cumbersome.

However, not all hope is lost because Solr provides two alternative server implementations which we can use in our applications. These implementations are described in the following:

The embedded Solr server connects directly to Solr core. We can use this server for development purposes but we must also remember that using it in production environment is not recommended. However, using the embedded Solr server is still a viable option in the development environment.
The HTTP Solr server connects to an external Solr server by using HTTP. This is the recommended way of using the Solr search server and that is why we should always use it in the production environment.
This blog entry describes how we can get the required dependencies with Maven. We also learn to configure the Spring Data Solr to use the embedded Solr server in the development environment and the HTTP Solr server in the production environment.

These blog entries provides additional information which helps us to understand the concepts described in this blog entry:
Running Solr with Maven
Spring Data Solr Tutorial: Introduction to Solr
Let’s get started.

Getting the Required Dependencies with Maven

We can get the required dependencies with Maven by following these steps:

Add the Spring Milestone Maven repository to the POM file.
Add the required dependencies to the pom.xml file.
Both of these steps are described with more details in the following.

Adding the Spring Milestone Maven Repository to the POM File
We can add the Spring milestone Maven repository to our POM file by adding the following XML to the pom.xml file:

1
2
3
4
5
6
7
<repositories>
    <repository>
        <id>spring-milestone</id>
        <name>Spring Milestone Maven Repository</name>
        <url>http://repo.springsource.org/libs-milestone</url>
    </repository>
</repositories>
Adding the Required Dependencies to the POM File
We can add the required dependencies to the POM file by following these steps:

Add the Spring Data Solr dependency (version 1.0.0.RC1) to the dependencies section of our POM file.
Add the Solr core dependency (version 4.1.0) to the dependencies section of our POM file and exclude the SLF4J JDK14 binding. Because Solr core is required by the embedded Solr server, we can skip this step if we are not using the embedded Solr server.
We can complete these steps by adding the following XML to the dependencies section of the POM file:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
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
Configuring Spring Data Solr

This section describes how we can configure Spring Data Solr to use different Solr servers in the development and production environment. We will use the embedded Solr server in the development environment and the HTTP Solr server in the production environment.

We can configure Spring Data Solr by following these steps:

Create a properties file.
Configure the embedded Solr server.
Configure the HTTP Solr server.
Set the active bean definition profile.
These steps are described with more details in the following subsections.

Creating the Properties File
The name of our properties file is application.properties and we will use it to configure two properties which are described in the following:

The solr.server.url property specifies the url of the used Solr server. The value of this property is used to configure the HTTP Solr server which is used in the production environment.
The solr.solr.home configures the home directory of Solr. The value of this property is used to configure the home directory of the embedded Solr server which is used in the development environment.
The content of the application.properties file looks as follows:

1
2
solr.server.url=http://localhost:8983/solr/
solr.solr.home=
Configuring the Embedded Solr Server
This subsection describes how we can configure Spring Data Solr to use the embedded Solr server in the development environment.

Java Configuration
We can create a configuration class which configures the embedded Solr server by following these steps:

Create a class called EmbeddedSolrContext and annotate that class with the @Configuration annotation.
Enable Spring Data Solr repositories by annotating that class with the @EnableSolrRepositories annotation and configuring the root package of our Solr repositories.
Annotate the created class with the @Profile annotation and set its value to ‘dev’. This means that this configuration class is bypassed unless the ‘dev’ profile have been activated.
Annotate the class with the @PropertySource annotation and set its value to ‘classpath:application.properties’. This configures the location of our property file and adds a PropertySource to Spring’s Environment.
Add an Environment field to the class and annotate that field with the @Resource annotation. The injected Environment is used to access the properties which we added to our properties file.
Create a method called solrServerFactoryBean() and annotate this method with the @Bean annotation. The implementation of this method creates a new EmbeddedSolrServerFactoryBean object, sets the value of the Solr home and returns the created object.
Create a method called solrTemplate() and annotate this method with the @Bean annotation. The implementation of this method creates a new SolrTemplate object and passes the used SolrServer implementation as a constructor argument.
The source code of the EmbeddedSolrContext class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
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
XML Configuration
We can create an XML configuration file for the embedded Solr server by following these steps:

Configure the used properties file by using the property-placeholder element of the context namespace.
Enable Solr repositories and configure the base package of our Solr repositories by using the repositories element of the solr namespace.
Create a bean configuration for the development profile.
Configure the embedded Solr server bean by using the embedded-solr-server element of the solr namespace. Set the value of the Solr home.
Configure the Solr template bean. Set the configured embedded Solr server bean as constructor argument.
The contents of the exampleApplicationContext-solr.xml file looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
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
Configuring the Http Solr Server
This subsection describes how we can configure Spring Data Solr to use the HTTP Solr server in the production environment.

Java Configuration
We can create a configuration class which configures the HTTP Solr server by following these steps:

Create a class called HttpSolrContext and annotate that class with the @Configuration annotation.
Enable Spring Data Solr repositories by annotating that class with the @EnableSolrRepositories annotation and configuring the root package of our Solr repositories.
Annotate the created class with a @Profile annotation and set its value to ‘prod’. This means that this configuration class is bypassed unless the ‘prod’ profile have been activated.
Annotate the class with the @PropertySource annotation and set its value to ‘classpath:application.properties’. This configures the location of our property file and adds a PropertySource to Spring’s Environment.
Add an Environment field to the class and annotate that field with the @Resource annotation. The injected Environment is used to access the properties which we added to our properties file.
Create a method called solrServerFactoryBean() and annotate this method with the @Bean annotation. The implementation of this method create a new HttpSolrServerFactoryBean object, sets the value of the Solr server url and returns the created object.
Create a method called solrTemplate() and annotate this method with the @Bean annotation. The implementation of this method creates a new SolrTemplate object and passes the used SolrServer implementation as a constructor argument.
The source code of the HttpSolrContext class looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
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
XML Configuration
We can create an XML configuration file for the HTTP Solr server by following these steps:

Configure the used properties file by using the property-placeholder element of the context namespace.
Enable Solr repositories and configure the base package of our Solr repositories by using the repositories element of the solr namespace.
Create a bean configuration for the production profile.
Configure the HTTP Solr server bean by using the solr-server element of the solr namespace. Set the url of the Solr server.
Configure the Solr template bean. Set the configured HTTP Solr server bean as a constructor argument.
The content of the exampleApplicationContext-solr.xml file looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
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
Setting the Active Bean Definition Profile
We can select the active bean definition profile by setting the value of the spring.profiles.active system variable. The allowed values of this system variable (in the context of our example application) are described in the following:

We can configure our application to run in the development profile by setting the value of the spring.profiles.active system variable to ‘dev’.
When we want configure our application to run in the production profile, we have to set the of the spring.profiles.active system variable to ‘prod’.
We can configure our example application to support both profiles by following these steps:

Add required profiles to the POM file.
Create the profile specific properties files for system properties.
Configure the Jetty Maven plugin.
These steps are described with more details in the following.

Adding the Required Profiles to the POM File
We can add the required profiles to our Maven build by following these steps:

Create a profile for development environment. Set the id of this profile to ‘dev’ and set the value of the build.profile.id property to ‘dev’.
Create a profile for the production environment. Set the id of this profile to ‘prod’ and set the value of the build.profile.id property to ‘prod’.
The configuration of our Maven profiles looks as follows:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
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
Creating the Profile Specific Properties Files for System Properties
The profile specific properties files are located in the sub directories of the profiles directory. The name of each sub directory matches with the values of the build.profile.id properties configured in the pom.xml file.

We can create the profile specific properties files for system properties by following these steps:

Create a properties file called system.properties to the profiles/dev directory. This properties file contains the system properties of the development profile.
Create a properties file called system.properties to the profiles/prod directory. This properties file contains the system properties of the production profile.
The content of the properties file used to configure the system properties of the development profile looks as follows:

1
spring.profiles.active=dev
The content of the properties file used to configure the system properties of the production profile looks as follows:

1
spring.profiles.active=prod
Configuring the Jetty Maven Plugin
We can configure the Jetty Maven plugin by following these steps:

Add the plugin declaration of the Jetty Maven plugin to the plugins section of our Pom file.
Configure the stopKey and stopPort of the Jetty Maven plugin.
Configure the location of the properties file containing the used system properties.
The configuration of the Jetty Maven plugin looks as follows:

1
2
3
4
5
6
7
8
9
10
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
Summary

We have now successfully obtained the required dependencies with Maven and configured Spring Data Solr. This blog entry has taught us four things:

We learned to get the required dependencies with Maven.
We know that we should use the embedded Solr server only in the development environment and learned how we can configure Spring Data Solr to use it.
We learned that we should always use the HTTP Solr server in the production environment and know how we can configure Spring Data Solr to use it.
We know how we can use the bean definition profiles of Spring Framework for creating different configurations for development and production environment.
The next part of my Spring Data Solr tutorial describes how we can add new document to Solr index, update the information of an existing documents and delete documents from the Solr index.

PS. The example application of this blog entry is available at Github.

If you want to learn how to use Spring Data Solr, you should read my Spring Data Solr tutorial.
