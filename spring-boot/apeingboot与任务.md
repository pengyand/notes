# 异步任务

**简单示例**

~~~java
@Service 
public class AsyncService{
    
    //使用注解，告诉Spring这是一个异步方法
    @Async
    public void hello(){
        try{
            Thread.sleep(1000L);
        }catch(Exception e){
            
        }
    }
}
~~~

主要在主配置类中加入@EnableAsync，来开启异步注解

~~~java
//开启异步注解
@EnableAsync
@SpringBootApplication
public class SpringBootTestApplication{
    
}
~~~



#定时任务

**两个注解**：

- @EnableScheduling  开启定时任务注解功能
- @Scheduled

**cron**表达式：

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-152340@2x.png)

**Scheduled**

~~~java

@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(Schedules.class)
public @interface Scheduled {
    String cron() default "";

    String zone() default "";

    long fixedDelay() default -1L;

    String fixedDelayString() default "";

    long fixedRate() default -1L;

    String fixedRateString() default "";

    long initialDelay() default -1L;

    String initialDelayString() default "";
}
~~~

```
秒 分 时 日 月 周几

"0 * * * * MON-FRI"

解读：周一到周五的每个月的每日的每时每分钟0秒启动一次

"* * * * * MON-FRI"

解读：周一到周五的每个月的每日的每时每分钟每秒启动一次

"0,1,2,3,4 * * * * MON-SAT"
解读：周一到周六的每个月的每日的每时没分钟的0秒1秒2秒3秒4秒分别启动一次
可以使用0-4来指定区间，和上面一样
0/4 从0秒启动，并每4秒启动一次。

？是解决星期和日期冲突的

0和7 都是周日
```

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-153538@2x.png)

