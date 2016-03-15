# 使用stomp结合websocket实现向浏览器广播消息

## 添加依赖

```xml

<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-messaging</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-websocket</artifactId>
	<version>${spring.version}</version>
</dependency>

```
## 添加配置

### Configuration Class

```java
@Configuration
@EnableWebSocketMessageBroker //1
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {

        config.enableSimpleBroker("/topic"); //2
        config.setApplicationDestinationPrefixes("/SBC"); //3
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {

        registry.addEndpoint("/pushLogs"); //4
                //.withSockJS(); //5
    }

}
```
### XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
    http://www.springframework.org/schema/websocket
    http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<websocket:message-broker application-destination-prefix="/SBC">
		<websocket:stomp-endpoint path="/pushLogs">
			<!--<websocket:sockjs/>-->
		</websocket:stomp-endpoint>
		<websocket:simple-broker prefix="/topic"/>
	</websocket:message-broker>

</beans>
```
## 解释一下配置
 
以Configuration Class为例，xml请自行参照

1. `@EnableWebSocketMessageBroker` 表示启用WebSocket的MessageBroker
2. `config.enableSimpleBroker("/topic");` 表示broker的地址为topic
3. `config.setApplicationDestinationPrefixes("/SBC")` 可以理解为应用的contextPath
4. `registry.addEndpoint("/pushLogs");` 表示对pushLogs的请求由stomp处理
5. `.withSockJS();` 表示启用js，这里我们不使用

## 添加Controller

```java

@Controller
public class PushLogsController {

    @MessageMapping("pushLogs")
    @SendTo("/topic/logs")
    public String pushLogs(String message) {

        return message;
    }

    @RequestMapping("/test")
    public String test() {

        return "test";
    }
}

```

这里test是一个jsp页面，供我们后续测试使用

`@MessageMapping("pushLogs")`与endpoint对应

`@SendTo("/topic/logs")`表示消息被转发给/topic/logs，任何订阅/topic/logs的客户端都可以收到推送，如果不太清楚，看看下面的测试代码就理解了

## test.jsp

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<head>
    <title>Hello WebSocket</title>
    <script src="/SBC/js/stomp.js"></script>
    <script type="text/javascript">
        var stompClient = null;

        function setConnected(connected) {
            document.getElementById('connect').disabled = connected;
            document.getElementById('disconnect').disabled = !connected;
            document.getElementById('conversationDiv').style.visibility = connected ? 'visible' : 'hidden';
            document.getElementById('response').innerHTML = '';
        }

        function connect() {
            var socket = new WebSocket('ws:localhost:8080/SBC/pushLogs');
            stompClient = Stomp.over(socket);
            stompClient.connect({}, function (frame) {
                setConnected(true);
                console.log('Connected: ' + frame);
                stompClient.subscribe('/topic/logs', function (greeting) {
                    showGreeting(JSON.parse(greeting.body).message);
                });
            });
        }

        function disconnect() {
            if (stompClient != null) {
                stompClient.disconnect();
            }
            setConnected(false);
            console.log("Disconnected");
        }

        function sendName() {
            var name = document.getElementById('name').value;
            stompClient.send("/SBC/pushLogs", {}, JSON.stringify({'message': name}));
        }

        function showGreeting(message) {
            var response = document.getElementById('response');
            var p = document.createElement('p');
            p.style.wordWrap = 'break-word';
            p.appendChild(document.createTextNode(message));
            response.appendChild(p);
        }
    </script>
</head>
<body onload="disconnect()">
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>
<div>
    <div>
        <button id="connect" onclick="connect();">Connect</button>
        <button id="disconnect" disabled="disabled" onclick="disconnect();">Disconnect</button>
    </div>
    <div id="conversationDiv">
        <label>What is your name?</label><input type="text" id="name"/>
        <button id="sendName" onclick="sendName();">Send</button>
        <p id="response"></p>
    </div>
</div>
</body>
</html>
```
需要引入stomp.js文件，去网上搜一下就有了

## 运行效果

访问test页面，点击connect，在输入框中随意输入一些，点击Send，会收到服务器返回的数据，效果图如下

![websocket-test](https://github.com/ronniewang/blog/blob/master/image/stomp-websocket.png)

打开Chrome的Console也可以看到输出的日志
