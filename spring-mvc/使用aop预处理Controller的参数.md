
实际编程中，可能会有这样一种情况，前台传过来的参数，我们需要一定的处理才能使用，比如有这样一个`Controller`

```java
@Controller
public class MatchOddsController {

    @Autowired
    private MatchOddsServcie matchOddsService;

    @RequestMapping(value = "/listOdds", method = RequestMethod.GET, produces = {MediaType.APPLICATION_JSON_VALUE})
    @ResponseBody
    public List<OddsModel> listOdds(@RequestParam Date startDate, @RequestParam Date endDate) {

        return matchOddsService.listOdds(startDate, endDate);
    }
}
```

前台传过来的`startDate`和`endDate`是两个日期，实际使用中我们需要将之转换为两个日期对应的当天11点，如果只有这么一个类的话，我们是可以直接在方法最前面处理就可以了

但是，还有下面两个类具有同样的业务逻辑

```java
@Controller
public class MatchProductController {

    @Autowired
    private MatchProductService matchProductService;

    @RequestMapping(value = "/listProduct", method = RequestMethod.GET, produces = { MediaType.APPLICATION_JSON_VALUE })
    @ResponseBody
    public List<ProductModel> listProduct(@RequestParam Date startDate, @RequestParam Date endDate) {

        return matchProductService.listMatchProduct(startDate, endDate);
    }
}

```

```java
@Controller
public class MatchController {

    @Autowired
    private MatchService matchService;
    
    @RequestMapping(value = "/listMatch", method = RequestMethod.GET, produces = {MediaType.APPLICATION_JSON_VALUE})
    @ResponseBody
    public List<MatchModel> listMatch(@RequestParam Date startDate, @RequestParam Date endDate) {

        return matchService.listMatch(startDate, endDate);
    }
}
```

当然也可以写两个util方法，分别处理`startDate`和`endDate`，但是为了让`Controller`看起来更干净一些，我们还是用AOP来实现吧，顺便为AOP更复杂的应用做做铺垫

本应用中使用Configuration Class来进行配置，主配置类如下：

```java
@SpringBootApplication
@EnableAspectJAutoProxy(proxyTargetClass = true) //开启AspectJ代理，并将proxyTargetClass置为true，表示启用cglib对Class也进行代理
public class Application extends SpringBootServletInitializer {
    ...
}
```

下面新建一个Aspect类，代码如下

```java
@Aspect //1
@Configuration //2
public class SearchDateAspect {

    @Pointcut("execution(* com.ronnie.controller.*.list*(java.util.Date,java.util.Date)) && args(startDate,endDate)") //3
    private void searchDatePointcut(Date startDate, Date endDate) { //4

    }

    @Around(value = "searchDatePointcut(startDate,endDate)", argNames = "startDate,endDate") //5
    public Object dealSearchDate(ProceedingJoinPoint joinpoint, Date startDate, Date endDate) throws Throwable { //6

        Object[] args = joinpoint.getArgs(); //7
        if (args[0] == null) {
            args[0] = Calendars.getTodayEleven();
            args[1] = DateUtils.add(new Date(), 7, TimeUnit.DAYS);//默认显示今天及以后的所有赔率
        } else {
            args[0] = DateUtils.addHours(startDate, 11);
            args[1] = DateUtils.addHours(endDate, 11);
        }
        return joinpoint.proceed(args); //8
    }
}

```

分别解释一下上面各个地方的意思，标号与语句之后的注释一致

1. 表示这是一个切面类
2. 表示这个类是一个配置类，在`ApplicationContext`启动时会加载配置，将这个类扫描到
3. 定义一个切点，`execution(* com.ronnie.controller.*.list*(java.util.Date,java.util.Date)) `表示任意返回值，在`com.ronnie.controller`包下任意类的以list开头的方法，方法带有两个Date类型的参数，`args(startDate,endDate)`表示需要Spring传入这两个参数
4. 定义切点的名称
5. 配置环绕通知
6. `ProceedingJoinPoint`会自动传入，用于处理真实的调用
7. 获取参数，下面代码是修改参数
8. 使用修改过的参数调用目标类

更多可参考
* <http://docs.spring.io/spring/docs/current/spring-framework-reference/html/aop.html>
* <http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/>
