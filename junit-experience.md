# JUnit使用经验

## 对于一些测试工具类，里面没有包含工具方法，批量执行的时候，执行到此会报“no runnable methods”的错

在工具类上添加@Ignore注解即可

