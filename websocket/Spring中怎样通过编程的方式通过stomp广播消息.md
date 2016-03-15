# Spring中怎样通过编程的方式通过stomp广播消息

Spring提供了一个`SimpMessagingTemplate`类，用它即可实现

```java
@Controller
@RequestMapping("/")
public class PhotoController {

    @Autowired
    private SimpMessagingTemplate template;

    @MessageMapping("/form")
    @SendTo("/topic/greetings")
    public Greeting greeting() {

        return new Greeting("Hello world !");
    }

    public void fireGreeting() {
        this.template.convertAndSend("/topic/greetings", new Greeting("Fire"));
    }
}
```
