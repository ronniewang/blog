Solr is an open source search server that is built by using the indexing and search capabilities of Lucene Core, and it can be used for implementing scalable search engines with almost any programming language.

Even though Solr has many advantages, setting up a development environment is not one of them. This blog post fixes that problem.

We will learn how we can run Solr by using Maven and ensure that each developer uses the same configuration, schema, and Solr version.

从Solr的配置文件开始吧。

### 搞定Solr配置文件

首先要复制Solr的配置文件到我们的项目下：

1. [下载Solr 4.3.0二进制版本](https://archive.apache.org/dist/lucene/solr/4.3.0/)
2. 解压到任意目录
3. 到根目录下
4. 将下列文件从`example/solr/collection1/conf`目录复制到`src/main/config`目录：admin-extra.html, admin-extra-menu.menu-bottom.html, admin-extra.menu-top.html, currency.xml, elevate.xml, mapping-FoldToASCII.txt, mapping-ISOLatin1Accent.txt, protwords.xml, schema.xml, scripts.conf, solrconfig.xml, spellings.txt, stopwords.txt, synonyms.txt and update-script.js.
5. 复制`example/solr/collection1/conf/lang`目录下的文件到`src/main/config/lang`目录下
6. 复制`example/solr/collection1/conf/velocity`目录下的所有文件到`src/main/config/velocity`目录下
7. 复制`example/solr/collection1/conf/xslt`目录下的所有文件到`src/main/config/xslt`目录下
8. 复制`exaple/solr`目录下的solr.xml文件到`src/main/resources`目录下
9. 创建`src/main/webapp/WEB-INF`目录，否则无法启动Solr

下面开始Maven的构建配置。

### Maven的构建配置

After we have copied the Solr configuration files to our project, we have to configure our Maven build. The requirements of our Maven build are:

* The properties of our Maven build must be read from an external property file. The only exception to this rule is that the version numbers of the dependencies can be declared in our POM file.
* The build process must copy the Solr configuration files to the configured directory when our Solr instance is started.
* The build process must delete the home directory of our Solr instance when we execute the mvn clean command at command prompt.
* It must be possible to start our Solr instance by using the Jetty Maven plugin.

We can create a Maven build that fulfils these requirements by following these steps:

1. Create a POM file.
2. Get the required dependencies.
3. Create the properties file that contains the properties of our Maven build.
4. Edit the solr.xml file.
5. Read the property values from an external properties file.
6. Copy the Solr configuration files to the correct directories.
7. Clean the build.
8. Configure the Jetty Maven plugin.

Let’s get started.

#### Creating the POM File

We have to create a basic POM file for a web application project. This POM file looks as follows:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>net.petrikainulainen.maven</groupId>
    <artifactId>running-solr-with-maven</artifactId>
    <packaging>war</packaging>
    <version>0.1</version>
      
    <profiles>
        <!-- Add profile configuration here -->
    </profiles>
    <dependencies>
        <!-- Add dependencies here -->
    </dependencies>
    <build>
        <finalName>solr</finalName>
        <!-- Add filter configuration here -->
        <!-- Add resources configuration here -->
        <plugins>
            <!-- Add plugin configuration here -->
        </plugins>
    </build>
</project>
```

#### Getting the Required Dependencies

We can get the required dependencies by declaring the following dependencies in our pom.xml file:

* SLF4J
* SLF4J interceptors for the java.util.logging (JUL) and the java.commons.logging (JCL) logging frameworks.
* SLF4J Log4J 1.2.X binding.
* Log4J
* Solr 4.3.0 (war)

> We need to declare the logging dependencies in our POM file because [the logging configuration of Solr](http://wiki.apache.org/solr/SolrLogging) was changed when Solr 4.3.0 was released.
The dependencies section of our POM file looks as follows:

```xml
<!-- SLF4J -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.7</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.7</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jul-to-slf4j</artifactId>
    <version>1.7.7</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.7</version>
</dependency>
<!-- Log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<!-- Solr 4.3.0 -->
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr</artifactId>
    <version>4.3.0</version>
    <type>war</type>
</dependency>
```

#### Creating the Properties File

Our next step is to create the properties file that is used in our Maven build, and add the required build profile configuration to our POM file.

First, we have to create the properties file which is used in our Maven build. We can do this by following these steps:

1. Create the directory profiles/dev to the root directory of our Maven project.
2. Create the properties file called config.properties to the profiles/dev directory.

Our properties file has three properties which are described in the following:

* The solr.detault.core.directory property configures the value of the default core directory. This is a directory that is created under the home directory of our Solr instance. This directory has two subdirectories:
  * The conf directory contains the configuration of our Solr instance.
  * The data directory contains the Solr index.
* The solr.default.core.name property configures the name of the default core.
* The solr.solr.home property configures the home directory of our Solr installation. In other words, it configures the directory in which the Solr configuration file (solr.xml) and the core specific configuration files are copied when the compile phase of the Maven default lifecycle is invoked.

Our config.properties file looks as follows:

```
#SOLR PROPERTIES
#Configures the directory used to store the data and configuration of the Solr default core
solr.default.core.directory=todo
#Configures the name of the Solr default core.
solr.default.core.name=todo
  
#SYSTEM PROPERTIES
#Configures the home directory of Solr. Set the preferred directory path here.
solr.solr.home=
```

Second, we must configure the build profiles of our Maven build and use filtering to replace replace the variables included in our resources. We can do this by following these steps:

1. Create a single profile called dev and ensure that it is the default profile of our build.
2. Declare a property called build.profile.id and set its value to ‘dev’.
3. Create a filter that reads the profile specific configuration file and replaces the variables found from our resources with the actual property values.

We can finish the steps one and two by adding the following profile configuration to the profiles section of our POM file:

```xml
<profile>
    <id>dev</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <properties>
        <build.profile.id>dev</build.profile.id>
    </properties>
</profile>
```

We can finish the step three by adding the following XML to the build section of our POM file:

```xml
<filters>
    <filter>${project.basedir}/profiles/${build.profile.id}/config.properties</filter>
</filters>
<resources>
    <resource>
        <filtering>true</filtering>
        <directory>src/main/resources</directory>
    </resource>
</resources>
```

#### Editing the solr.xml File

Because we configure the name and the instance directory of the Solr default core in our profile specific configuration file, we have to make some changes to the solr.xml file. We can make these changes by following these steps:

1. Set the value of the defaultCoreName attribute of the cores element. Use the value of the solr.default.core.name property that is found from our profile specific properties file.
2. Set the value of the name attribute of the core element. Use the value of the solr.default.core.name property that is found from our profile specific configuration file.
3. Set the value of the instanceDir attribute of the core element. Use the value of the solr.default.core.directory property that is found from our profile specific configuration file.

The solr.xml file looks as follows (the relevant parts are highlighted):

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<solr persistent="true">
  <cores adminPath="/admin/cores"
>         defaultCoreName="${solr.default.core.name}"
         host="${host:}"
         hostPort="${jetty.port:}"
         hostContext="${hostContext:}"
         zkClientTimeout="${zkClientTimeout:15000}">
 >   <core name="${solr.default.core.name}" instanceDir="${solr.default.core.directory}" />
  </cores>
</solr>
```

#### Reading the Property Values From an External Properties File

Because we want that all property values used in our POM file are read from an external properties file, we have to use a plugin called the Properties Maven plugin. We can configure this plugin by following these steps:

1. Ensure that the properties are read from the profile specific configuration file.
2. Create an execution that runs the read-project-properties goal of the Properties Maven plugin in the initialize phase of the Maven default lifecycle.
3. Create an execution that runs the read-project-properties goal of the Properties Maven plugin in the pre-clean phase of the Maven clean lifecycle.

The configuration of the Properties Maven plugin looks as follows:

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>properties-maven-plugin</artifactId>
    <version>1.0-alpha-2</version>
    <configuration>
        <files>
            <!-- Properties are read from profile specific property file -->
            <file>${project.basedir}/profiles/${build.profile.id}/config.properties</file>
        </files>
    </configuration>
    <executions>
        <!-- Load properties for the default lifecycle -->
        <execution>
            <id>default-lifecycle-properties</id>
            <phase>initialize</phase>
            <goals>
                <goal>read-project-properties</goal>
            </goals>
        </execution>
        <!-- Load properties for the clean lifecycle -->
        <execution>
            <id>clean-lifecycle-properties</id>
            <phase>pre-clean</phase>
            <goals>
                <goal>read-project-properties</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

#### Copying the Solr Configuration Files to the Correct Directories

Our next step is to copy the Solr configuration files to the home directory of our Solr instance. We will use the Maven Resources plugin for this purpose.

The first thing that we have to do is to add the Maven Resources plugin to the plugins section of our pom.xml file. We can do this by adding the following snippet to the plugins section of our POM file:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <executions>
        <!-- Add executions here -->
    </executions>
</plugin>
```

Now we need to configure the executions that copies the Solr configuration files to the correct directories. We can do this by following these steps:

1. We need to copy the solr.xml file found from the src/main/resources directory to the ${solr.solr.home} directory. This directory is the home directory of our Solr instance. Also, we need apply properties filtering to it because we want replace the placeholders found from that file with the property values found from our profile specific properties file.
2. We need to copy the contents of the src/main/config directory to the ${solr.solr.home}/${solr.default.core.directory}/conf directory. This directory contains the configuration of our Solr instance’s default core.

First, we need to copy the solr.xml file to the home directory of our Solr instance. We can do this by following these steps:

1. Create an execution that invokes the copy-resources goal of the Resources Maven plugin in the compile phase of the default lifecycle.
2. Configure the execution to copy the solr.xml file found from the src/main/resources directory to the ${solr.solr.home} directory. Remember to apply properties filtering to the solr.xml file.

The configuration of our execution looks as follows:

```xml
<execution>
    <id>copy-solr-xml</id>
    <phase>compile</phase>
    <goals>
        <goal>copy-resources</goal>
    </goals>
    <configuration>
        <!-- Configure the directory in which the file is copied. -->
        <outputDirectory>${solr.solr.home}</outputDirectory>
        <resources>
            <!--
                Configure the copied file and apply properties filtering to it.
            -->
            <resource>
                <directory>${project.basedir}/src/main/resources</directory>
                <filtering>true</filtering>
                <includes>
                    <include>solr.xml</include>
                </includes>
            </resource>
        </resources>
    </configuration>
</execution>
```

Second, we need to copy the contents of the src/main/config directory to the ${solr.solr.home}/${solr.default.core.directory}/conf directory. We can do this by following these steps:

1. Create an execution that invokes the copy-resources goal of the Resources Maven plugin in the compile phase of the default lifecycle.
2. Configure the execution to copy contents of the src/main/config directory to the to the ${solr.solr.home}/${solr.default.core.directory}/conf directory.

The configuration of our execution looks as follows:

```xml
<execution>
    <id>copy-solr-config</id>
    <phase>compile</phase>
    <goals>
        <goal>copy-resources</goal>
    </goals>
    <configuration>
        <!-- Configure the target directory in which the files are copied. -->
        <outputDirectory>${solr.solr.home}/${solr.default.core.directory}/conf</outputDirectory>
        <!--
            Configure the directory which contents are copied to the target directory.
            Disable properties filtering.
        -->
        <resources>
            <resource>
                <directory>${project.basedir}/src/main/config</directory>
                <filtering>false</filtering>
            </resource>
        </resources>
    </configuration>
</execution>
```

#### Cleaning the Build

When we clean our build, we have to delete two directories that are described in the following:

* We need to delete the home directory of our Solr instance.
* We need to delete the overlays directory that is created to the root directory of our project when we start our Solr instance by using the Jetty Maven plugin.

We can configure the Maven Clean plugin to delete these directories. If we want to delete additional directories when the command mvn clean is invoked, we have to follow these steps:

1. Add the fileSets element to the configuration of the Maven Clean plugin.
2. Configure the deleted directories by adding fileSet elements inside the fileSets element. We can configure the path of the deleted directory by adding the directory element inside the fileSet element. We can use both relative and absolute paths. If we use relative paths, the starting point of that path is the root directory of our project.

If we want to delete the foo directory that is found from the root directory of our project, we have to use the following configuration:

```xml
<configuration>
    <filesets>
        <!-- Configure the deleted directory. -->
        <fileset>
            <directory>foo</directory>
        </fileset>
    </filesets>
</configuration>
```

The configuration of the Maven Clean plugin that deletes the target directory, the overlays directory, and the home directory of our Solr instance looks as follows:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-clean-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <filesets>
            <!-- Delete the overlays directory from the project root directory -->
            <fileset>
                <directory>overlays</directory>
            </fileset>
            <!-- Delete the Solr home directory -->
            <fileset>
                <directory>${solr.solr.home}</directory>
            </fileset>
        </filesets>
    </configuration>
</plugin>
```

#### 配置Jetty插件

我们通过Jetty插件运行Solr，配置如下：

1. Configure Jetty to listen the port 8983.
2. Ensure that system properties are read from the profile specific configuration file. This property file contains a property called solr.solr.home which specifies the home directory of our Solr instance. If this property is missing, we cannot start our Solr instance because Solr cannot find its configuration files.
3. Specify that the context path of our application is /solr.

The configuration of the Jetty Maven plugin looks as follows:

```xml
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.2.3.v20140905</version>
    <configuration>
        <stopPort>9966</stopPort>
        <stopKey>stop</stopKey>
        <!-- Listen to port 8983 -->
        <httpConnector>
            <port>8983</port>
            <idleTimeout>60000</idleTimeout>
        </httpConnector>
        <!-- Read system properties from profile specific configuration file -->
        <systemPropertiesFile>${project.basedir}/profiles/${build.profile.id}/config.properties
</systemPropertiesFile>
        <!-- Set the context path -->
        <webApp>
            <contextPath>/solr</contextPath>
        </webApp>
    </configuration>
</plugin>
```

### 运行Solr

现在Solr的环境就搭好了，启动它有两个方式：

* 执行`mvn jetty:run`命令
* 执行`mvn jetty:run-war`命令

启动后，访问<http://localhost:8983/solr>就能看到运行成功了

If you want play around with the example application, [you can get it from Github](https://github.com/pkainulainen/maven-examples/tree/master/running-solr-with-maven). This example uses a custom schema because I plan to use it in my Spring Data Solr tutorial. The original example schema is found from the etc directory.

> 了解更多关于Spring Data Solr，请移步[Spring Data Solr tutorial.](http://www.petrikainulainen.net/spring-data-solr-tutorial/)
