Solr是一个基于Lucene Core的开源搜索引擎，通过HTTP协议支持多种编程语言。

虽然Solr优势很多，但是安装开发环境却是很蛋疼，这篇就先解决Solr的安装问题。

我们选择通过Maven来构建，保证配置统一。

先从Solr的配置文件开始吧。

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

复制完Solr配置文件后，我们要配置Maven的构建信息，Maven构建需求如下：

* The properties of our Maven build must be read from an external property file. The only exception to this rule is that the version numbers of the dependencies can be declared in our POM file.
* Solr启动时要保证Slor的配置文件在配置目录下
* The build process must delete the home directory of our Solr instance when we execute the mvn clean command at command prompt.
* It must be possible to start our Solr instance by using the Jetty Maven plugin.

通过下面的步骤可以满足以上需求：

1. 新建一个POM文件
2. 添加依赖
3. 新建properties文件
4. 编辑solr.xml文件
5. 从外部properties文件中读取属性值
6. 复制Solr配置文件到当前目录下
7. clean build
8. 配置Jetty插件

开始搞起。

#### 新建一个POM文件

内容如下：

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

#### 添加依赖

在`pom.xml`中声明依赖，POM文件依赖部分如下：

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

#### 创建Properties文件

Our next step is to create the properties file that is used in our Maven build, and add the required build profile configuration to our POM file.

First, we have to create the properties file which is used in our Maven build. We can do this by following these steps:

1. Create the directory profiles/dev to the root directory of our Maven project.
2. Create the properties file called config.properties to the profiles/dev directory.

properties文件有如下三个属性：

* `solr.detault.core.directory`配置了默认的核心目录，它在Solr实例的主目录下，有两个子目录：
  * conf目录，包含了Solr实例的配置
  * data目录，包含了Solr的索引
* `solr.default.core.name`配置了默认核心的名称
* `solr.solr.home property`配置了Solr的安装目录，也就是 In other words, it configures the directory in which the Solr configuration file (solr.xml) and the core specific configuration files are copied when the compile phase of the Maven default lifecycle is invoked.

config.properties文件内容如下：

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

Second, we must configure the build profiles of our Maven build and use filtering to replace replace the variables included in our resources. 步骤如下：

1. Create a single profile called dev and ensure that it is the default profile of our build.
2. Declare a property called build.profile.id and set its value to ‘dev’.
3. Create a filter that reads the profile specific configuration file and replaces the variables found from our resources with the actual property values.

在POM文件中的profiles元素下加入下面的profile配置：

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

在POM文件中的build元素下加入下面的配置：

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

#### 编辑solr.xml文件

Because we configure the name and the instance directory of the Solr default core in our profile specific configuration file, we have to make some changes to the solr.xml file. 步骤如下：

1. Set the value of the defaultCoreName attribute of the cores element. Use the value of the solr.default.core.name property that is found from our profile specific properties file.
2. Set the value of the name attribute of the core element. Use the value of the solr.default.core.name property that is found from our profile specific configuration file.
3. Set the value of the instanceDir attribute of the core element. Use the value of the solr.default.core.directory property that is found from our profile specific configuration file.

solr.xml文件如下（相关的是第四行和倒数第三行）：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<solr persistent="true">
  <cores adminPath="/admin/cores"
         defaultCoreName="${solr.default.core.name}"
         host="${host:}"
         hostPort="${jetty.port:}"
         hostContext="${hostContext:}"
         zkClientTimeout="${zkClientTimeout:15000}">
    <core name="${solr.default.core.name}" instanceDir="${solr.default.core.directory}" />
  </cores>
</solr>
```

#### 从外部Properties文件读取属性值

为了让POM中用到的属性都从外部文件读取，需要用到Maven的Properties插件，配置步骤如下：

1. 配置从外部文件读取属性值
2. 配置在初始化阶段执行read-project-properties目标
3. 配置在pre-clean阶段执行read-project-properties目标

插件配置如下：

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

#### 复制Solr配置文件到正确目录

我们通过Maven的Resources插件来实现。

首先将Resources插件加入POM文件：

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

下面配置Resources插件执行复制操作，步骤如下：

1. 复制`src/main/resources`目录下的solr.xml文件到`${solr.solr.home}`目录，这是Solr实例的主目录，Also, we need apply properties filtering to it because we want replace the placeholders found from that file with the property values found from our profile specific properties file.
2. 还需要复制`src/main/config`目录下的内容到`${solr.solr.home}/${solr.default.core.directory}/conf`目录下，这个目录包含了我们Solr实例的默认核心的配置

为了实现上面的步骤1，执行下面的操作：

1. 配置Resources插件在编译阶段执行copy-resources目标
2. Configure the execution to copy the solr.xml file found from the src/main/resources directory to the ${solr.solr.home} directory. Remember to apply properties filtering to the solr.xml file.

配置内容如下：

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

为实现步骤2，执行下面操作：

1. 配置Resources插件咋编译阶段执行copy-resources目标
2. 配置execution从`src/main/config`复制内容到`${solr.solr.home}/${solr.default.core.directory}/conf`目录

配置内容如下：

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

通过Maven的Clean插件来删除这些目录，配置步骤如下：

1. 在Clean插件的配置下添加`fileSets`元素
2. Configure the deleted directories by adding fileSet elements inside the fileSets element. We can configure the path of the deleted directory by adding the directory element inside the fileSet element. We can use both relative and absolute paths. If we use relative paths, the starting point of that path is the root directory of our project.

如果我们想删除项目根目录下的foo目录，就得如下配置：

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

1. 配置Jetty监听8983端口
2. Ensure that system properties are read from the profile specific configuration file. This property file contains a property called solr.solr.home which specifies the home directory of our Solr instance. If this property is missing, we cannot start our Solr instance because Solr cannot find its configuration files.
3. Specify that the context path of our application is /solr.

Jetty插件配置如下：

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
