使用jvisualvm通过JMX的方式远程监控JVM的运行情况，步骤如下

# 远程服务器的配置

## 在启动java程序时加上如下几个参数
 * -Dcom.sun.management.jmxremote 
 * -Dcom.sun.management.jmxremote.ssl=false 
 * -Dcom.sun.management.jmxremote.authenticate=false 
 * -Dcom.sun.management.jmxremote.port=22222
 
例如：
> java -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.port=22222 -jar spider-robot.jar 

这样，程序就在22222端口上打开了jmx，客户端可以通过jvisualvm来进行连接，下面来说一个客户端的配置。

# 客户端的配置

## 在文件菜单下打开“添加JMX连接”
![png01](https://github.com/ronniewang/blog/blob/master/image/jvisualvm01.png)

## 在弹出的窗口中添加连接信息
![png02](https://github.com/ronniewang/blog/blob/master/image/jvisualvm02.png)

1. 在连接一栏中填入主机和端口信息
 * 这里的主机是要程序运行的机器，这里我们要监控192.168.1.4上的程序
 * 端口是上面启动时-Dcom.sun.management.jmxremote.port参数指定的端口
2. 由于我们将-Dcom.sun.management.jmxremote.authenticate设置为了false，所以无需用户名和密码
3. 点击确定

## 

更多内容可参考<http://docs.oracle.com/javase/6/docs/technotes/guides/visualvm/jmx_connections.html>
