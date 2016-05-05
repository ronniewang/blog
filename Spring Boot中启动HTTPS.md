# Spring Boot中启动HTTPS

如果你使用Spring Boot，并且想在内嵌tomcat中添加HTTPS，需要如下步骤

- 要有一个证书，买的或者自己生成的
- 在Spring Boot中启动HTTPS
- 将HTTP重定向到HTTPS（可选）

## 获取SSL证书

有两种方式

	* 自己通过keytool生成
	* 通过证书授权机构购买


这里作为演示，采用keytool生成

输入下面的命令，根据提示输入信息

```
keytool -genkey -alias tomcat  -storetype PKCS12 -keyalg RSA -keysize 2048  -keystore keystore.p12 -validity 3650

Enter keystore password:
Re-enter new password:
What is your first and last name?
[Unknown]:
What is the name of your organizational unit?
[Unknown]:
What is the name of your organization?
[Unknown]:
What is the name of your City or Locality?
[Unknown]:
What is the name of your State or Province?
[Unknown]:
What is the two-letter country code for this unit?
[Unknown]:
Is CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
[no]: yes
```

会生成一个PKCS12格式的叫做keystore.p12的证书，之后启动Spring Boot时会引用这个证书

## Spring Boot 中开启HTTPS

默认情况下Spring Boot内嵌的Tomcat服务器会在8080端口启动HTTP服务，Spring Boot允许在application.properties中配置HTTP或HTTPS，但是不可同时配置，如果两个都启动，至少有一个要以编程的方式配置，Spring Boot官方文档建议在application.properties中配置HTTPS，因为HTTPS比HTTP更复杂一些，可以参考spring-boot-sample-tomcat-multi-connectors的实例

在application.properties中配置HTTPS

```
server.port: 8443
server.ssl.key-store: classpath:keystore.p12
server.ssl.key-store-password: mypassword
server.ssl.keyStoreType: PKCS12
server.ssl.keyAlias: tomcat
```

这就够了

## 将HTTP请求重定向到HTTPS（可选）

让我们的应用支持HTTP是个好想法，但是需要重定向到HTTPS，上面说了不能同时在application.properties中同时配置两个connector，所以要以编程的方式配置HTTP connector，然后重定向到HTTPS connector

这需要在配置类中配置一个TomcatEmbeddedServletContainerFactory bean，代码如下

```java
  @Bean
  public EmbeddedServletContainerFactory servletContainer() {

    TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory() {

        @Override
        protected void postProcessContext(Context context) {

          SecurityConstraint securityConstraint = new SecurityConstraint();
          securityConstraint.setUserConstraint("CONFIDENTIAL");
          SecurityCollection collection = new SecurityCollection();
          collection.addPattern("/*");
          securityConstraint.addCollection(collection);
          context.addConstraint(securityConstraint);
        }
    };
    tomcat.addAdditionalTomcatConnectors(initiateHttpConnector());
    return tomcat;
  }

  private Connector initiateHttpConnector() {

    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    connector.setScheme("http");
    connector.setPort(8080);
    connector.setSecure(false);
    connector.setRedirectPort(8443);
    return connector;
  }
```

搞定！
