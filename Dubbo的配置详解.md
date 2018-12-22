# Dubbo的配置详解

####  Dubbo常用配置

```xml
dubbo:service/ 服务配置，用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心。
<dubbo:service ref="demoService" interface="com.unj.dubbotest.provider.DemoService" />

dubbo:reference/ 引用服务配置，用于创建一个远程服务代理，一个引用可以指向多个注册中心。
<dubbo:reference id="demoService" interface="com.unj.dubbotest.provider.DemoService" />

dubbo:protocol/ 协议配置，用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受。
<dubbo:protocol name="dubbo" port="20880" />

dubbo:application/ 应用配置，用于配置当前应用信息，不管该应用是提供者还是消费者。
<dubbo:application name="xixi_provider" />
    <dubbo:application name="hehe_consumer" />

dubbo:module/ 模块配置，用于配置当前模块信息，可选。
dubbo:registry/ 注册中心配置，用于配置连接注册中心相关信息。
<dubbo:registry address="zookeeper://192.168.2.249:2181" />

dubbo:monitor/ 监控中心配置，用于配置连接监控中心相关信息，可选。
dubbo:provider/ 提供方的缺省值，当ProtocolConfig和ServiceConfig某属性没有配置时，采用此缺省值，可选。
dubbo:consumer/ 消费方缺省配置，当ReferenceConfig某属性没有配置时，采用此缺省值，可选。
dubbo:method/ 方法配置，用于ServiceConfig和ReferenceConfig指定方法级的配置信息。
dubbo:argument/ 用于指定方法参数配置。

```



#### 配置优先级

#### 配置调用超时配置

![270324-20160910122039394-1078905366](/Users/yanpeng/Pictures/270324-20160910122039394-1078905366.png)

> 上图中以timeout为例，显示了配置的查找顺序，其它retries, loadbalance, actives也类似。
> 方法级优先，接口级次之，全局配置再次之。（reference优先 consumer优先）
> 如果级别一样，则消费方优先，提供方次之。
>
> 其中，服务提供方配置，通过URL经由注册中心传递给消费方。
> 建议由服务提供方设置超时，因为一个方法需要执行多长时间，服务提供方更清楚，如果一个消费方同时引用多个服务，就不需要关心每个服务的超时设置。
> 理论上ReferenceConfig的非服务标识配置，在ConsumerConfig，ServiceConfig, ProviderConfig均可以缺省配置。

#### Dubbo服务启动检查

> Dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止Spring初始化完成，以便上线时，能及早发现问题，默认check=true。
>
> 如果你的Spring容器是懒加载的，或者通过API编程延迟引用服务，请关闭check，否则服务临时不可用时，会抛出异常，拿到null引用，如果check=false，总是会返回引用，当服务恢复时，能自动连上。
>
> 可以通过check="false"关闭检查，比如，测试时，有些服务不关心，或者出现了循环依赖，必须有一方先启动。

```xml
1、关闭某个服务的启动时检查：(没有提供者时报错)
<dubbo:reference interface="com.foo.BarService" check="false" />

2、关闭所有服务的启动时检查：(没有提供者时报错)  写在定义服务消费者一方
<dubbo:consumer check="false" />

3、关闭注册中心启动时检查：(注册订阅失败时报错)
<dubbo:registry check="false" />
```

> 引用缺省是延迟初始化的，只有引用被注入到其它Bean，或被getBean()获取，才会初始化。
> 如果需要饥饿加载，即没有人引用也立即生成动态代理，可以配置：

``` xml
<dubbo:reference interface="com.foo.BarService" init="true" />
```

#### 订阅

> 1. 问题
>
>    为方便开发测试，经常会在线下共用一个所有服务可用的注册中心，这时，如果一个正在开发中的服务提供者注册，可能会影响消费者不能正常运行。
>
> 2. 解决方案
>
>    可以让服务提供者开发方，只订阅服务(开发的服务可能依赖其它服务)，而不注册正在开发的服务，通过直连测试正在开发的服务。

~~~ xml
禁用注册配置：
<dubbo:registry address="10.20.153.10:9090" register="false" />
或者：
<dubbo:registry address="10.20.153.10:9090?register=false" />
~~~

#### 回声测试

> 回声测试用于检测服务是否可用，回声测试按照正常请求流程执行，能够测试整个调用是否通畅，可用于监控。
> 所有服务自动实现EchoService接口，只需将任意服务引用强制转型为EchoService，即可使用。

~~~ java
<dubbo:reference id="memberService" interface="com.xxx.MemberService" />

MemberService memberService = ctx.getBean("memberService"); // 远程服务引用
EchoService echoService = (EchoService) memberService; // 强制转型为EchoService
String status = echoService.$echo("OK"); // 回声测试可用性
assert(status.equals("OK"))
~~~



#### 延迟连接

> 延迟连接，用于减少长连接数，当有调用发起时，再创建长连接。
> 只对使用长连接的dubbo协议生效。

~~~ xml
<dubbo:protocol name="dubbo" lazy="true" />
~~~



#### 令牌验证

> 防止消费者绕过注册中心访问提供者，在注册中心控制权限，以决定要不要下发令牌给消费者，注册中心可灵活改变授权方式，而不需修改或升级提供者

~~~xml
1、全局设置开启令牌验证：
<!--随机token令牌，使用UUID生成-->
<dubbo:provider interface="com.foo.BarService" token="true" />

<!--固定token令牌，相当于密码-->
<dubbo:provider interface="com.foo.BarService" token="123456" />

2、服务级别设置开启令牌验证：
<!--随机token令牌，使用UUID生成-->
<dubbo:service interface="com.foo.BarService" token="true" />

<!--固定token令牌，相当于密码-->
<dubbo:service interface="com.foo.BarService" token="123456" />

3、协议级别设置开启令牌验证：
<!--随机token令牌，使用UUID生成-->
<dubbo:protocol name="dubbo" token="true" />

<!--固定token令牌，相当于密码-->
<dubbo:protocol name="dubbo" token="123456" />
~~~



### 日志适配

> 缺省自动查找：log4j、slf4j、jcl、jdk
>
> 可以通过以下方式配置日志输出策略：dubbo:application logger="log4j"/>
>
> 访问日志：
> 如果你想记录每一次请求信息，可开启访问日志，类似于apache的访问日志。此日志量比较大，请注意磁盘容量。
>
> 将访问日志输出到当前应用的log4j日志：

~~~xml
<dubbo:protocol accesslog="true" />
~~~

~~~xml
将访问日志输出到指定文件
<dubbo:protocol accesslog="http://10.20.160.198/wiki/display/dubbo/foo/bar.log" />
~~~

#### 配置Dubbo缓存文件

> 配置方法如下：

~~~xml
<dubbo:registryfile=”${user.home}/output/dubbo.cache” />
~~~

> 注意：
> 文件的路径，应用可以根据需要调整，保证这个文件不会在发布过程中被清除。如果有多个应用进程注意不要使用同一个文件，避免内容被覆盖。
>
> 这个文件会缓存：
> 注册中心的列表
> 服务提供者列表
>
> 有了这项配置后，当应用重启过程中，Dubbo注册中心不可用时则应用会从这个缓存文件读取服务提供者列表的信息，进一步保证应用可靠性。



#### Dubbo各种协议：

> Dubbo默认使用的是netty协议实现的长链接。（大并发时）
>
> 有时，我们需要使用短链接，可以使用hessian协议。（需要指定jetty）



#### 多版本支持

> version字段可以指定版本
>
> 服务发布的时候需要指定版本

~~~xml
<!--服务发布的服务配置，需要暴露的服务接口 -->
    <dubbo:service interface="com.pengyand.dubbo.IOrderService" ref="orderService" version="1.0"/>
~~~

> 服务引用的时候 也要指定版本

~~~xml
 <!--生成一个远程服务的调用代理 check在循环依赖的时候需要设置为false，默认是true-->
    <dubbo:reference id="orderService" interface="com.pengyand.dubbo.IOrderService"
                     check="false" protocol="hessian" version="1.0" />
~~~



#### 异步调用

~~~java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("order-consumer.xml");

        IOrderService orderService = (IOrderService) context.getBean("orderService");

        DoOrderRequest request = new DoOrderRequest();
        request.setName("pengyand11");
        //DoOrderResponse response = orderService.doOrder(request);

        orderService.doOrder(request);
		//获取异步调用的结果返回
        Future<DoOrderResponse> future = RpcContext.getContext().getFuture();
        DoOrderResponse response = future.get();


        System.out.println(response);

~~~

~~~xml
    <!--生成一个远程服务的调用代理 check在循环依赖的时候需要设置为false，默认是true-->
    <dubbo:reference id="orderService" interface="com.pengyand.dubbo.IOrderService"
                     check="false" protocol="dubbo" version="1.0" >
        <!--指定异步调用的方法-->
        <dubbo:method name="doOrder" async="true"></dubbo:method>
    </dubbo:reference>
~~~

> 异步调用必须使用dubbo协议。

![QQ20180928-110637](/Users/yanpeng/Pictures/QQ20180928-110637.png)

#### 只订阅 不注册

~~~xml
  <!--配置注册中心 resgister为false 只订阅不注册，主要用来开发测试-->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"  register="false" />
~~~



#### 只注册 不订阅

> 多注册中心的时候 使用，只注册到某个注册中心，但是不订阅

~~~xml
<!--配置注册中心 resgister为false 只订阅不注册，主要用来开发测试-->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"  subscribe="false" />
~~~



#### 负载均衡

> 默认时 Random（随机）
>
> roundrobin 轮询：按照公约后的权重设置轮询比率。
>
> LeastActive LoadBalance 最少活跃调用数，对于相应时间比较短的服务会优先。
>
> Consistent LoadBalance，相同hash
>
> 设置方法：在服务发布的时候 设置loadbalance

~~~xml
 <dubbo:service interface="com.pengyand.dubbo.IOrderService" ref="orderService" version="1.0" timeout="2000"
                   loadbalance="roundrobin"
                   cluster="failfast"/>
~~~



#### 超时处理

> 服务处理的超时时间
>
> 超时处理：timeout = xxxms
>
> 生成环境最好配置

~~~xml
 <!--服务发布的服务配置，需要暴露的服务接口 -->
    <dubbo:service interface="com.pengyand.dubbo.IOrderService" ref="orderService" version="1.0" timeout="2000"/>
~~~



#### 集群容错

> `Failover cluster` 失败的时候自动切换并重试其他服务器。(默认)
>
> 通过retries=2，来设置重试次数。（一共3次）

> `Failfast cluster` 快速失败，只发起一次调用。一般在写操作。非幂等操作。

> `failsafe cluster` 失败安全，失败的时候不抛异常，出现异常时，直接忽略异常。一般在日志记录时使用

> `failback cluster` 失败自动恢复。后台记录失败请求，定时重发。

> `forking cluster` 并行调用多个服务器，只要一个成功就返回。只能应用在读请求。缺点，浪费服务器的资源。

> `broadcast cluster` 广播调用所有提供者，逐个调用。其中一台保存就会返回异常。

> 配置方式：
>
> ~~~xml
> <dubbo:service interface="com.pengyand.dubbo.IOrderService" ref="orderService" version="1.0" timeout="2000" cluster="failfast"/>
> ~~~
>
>



 #### 指定服务发布主机的host

~~~xml
<!--通过host可以指定 只能设置一个-->
<dubbo:protocol name="dubbo" port="20880" host="192.168.1.126"/>
~~~

> 对应dubbo的源码：
>
> `dubbo-config` >`dubbo-config-api` > `org.apache.dubbo.config` > `ServiceConfig`
>
> ```java
> /**
>  * Register & bind IP address for service provider, can be configured separately.
>  * Configuration priority: environment variables -> java system properties -> host property in config file ->
>  * /etc/hosts -> default network address -> first available network address
>  *
>  * @param protocolConfig
>  * @param registryURLs
>  * @param map
>  * @return
>  */
> private String findConfigedHosts(ProtocolConfig protocolConfig, List<URL> registryURLs, Map<String, String> map) {
>     boolean anyhost = false;
> 
>     String hostToBind = getValueFromConfig(protocolConfig, Constants.DUBBO_IP_TO_BIND);
>     if (hostToBind != null && hostToBind.length() > 0 && isInvalidLocalHost(hostToBind)) {
>         throw new IllegalArgumentException("Specified invalid bind ip from property:" + Constants.DUBBO_IP_TO_BIND + ", value:" + hostToBind);
>     }
> 
>     // if bind ip is not found in environment, keep looking up
>     if (hostToBind == null || hostToBind.length() == 0) {
>         hostToBind = protocolConfig.getHost();
>         if (provider != null && (hostToBind == null || hostToBind.length() == 0)) {
>             hostToBind = provider.getHost();
>         }
>         if (isInvalidLocalHost(hostToBind)) {
>             anyhost = true;
>             try {
>                 hostToBind = InetAddress.getLocalHost().getHostAddress();
>             } catch (UnknownHostException e) {
>                 logger.warn(e.getMessage(), e);
>             }
>             if (isInvalidLocalHost(hostToBind)) {
>                 if (registryURLs != null && !registryURLs.isEmpty()) {
>                     for (URL registryURL : registryURLs) {
>                         if (Constants.MULTICAST.equalsIgnoreCase(registryURL.getParameter("registry"))) {
>                             // skip multicast registry since we cannot connect to it via Socket
>                             continue;
>                         }
>                         try {
>                             Socket socket = new Socket();
>                             try {
>                                 SocketAddress addr = new InetSocketAddress(registryURL.getHost(), registryURL.getPort());
>                                 socket.connect(addr, 1000);
>                                 hostToBind = socket.getLocalAddress().getHostAddress();
>                                 break;
>                             } finally {
>                                 try {
>                                     socket.close();
>                                 } catch (Throwable e) {
>                                 }
>                             }
>                         } catch (Exception e) {
>                             logger.warn(e.getMessage(), e);
>                         }
>                     }
>                 }
>                 if (isInvalidLocalHost(hostToBind)) {
>                     hostToBind = getLocalHost();
>                 }
>             }
>         }
>     }
> 
>     map.put(Constants.BIND_IP_KEY, hostToBind);
> 
>     // registry ip is not used for bind ip by default
>     String hostToRegistry = getValueFromConfig(protocolConfig, Constants.DUBBO_IP_TO_REGISTRY);
>     if (hostToRegistry != null && hostToRegistry.length() > 0 && isInvalidLocalHost(hostToRegistry)) {
>         throw new IllegalArgumentException("Specified invalid registry ip from property:" + Constants.DUBBO_IP_TO_REGISTRY + ", value:" + hostToRegistry);
>     } else if (hostToRegistry == null || hostToRegistry.length() == 0) {
>         // bind ip is used as registry ip by default
>         hostToRegistry = hostToBind;
>     }
> 
>     map.put(Constants.ANYHOST_KEY, String.valueOf(anyhost));
> 
>     return hostToRegistry;
> }
> ```
>
>  

> 首先通过配置去查找。
>
> 然后通过InetAddress.getLocalHost().getHostAddress()去找。
>
> ...



#### Dubbo服务的最佳实践

1. 分包

> 服务接口、请求服务模型，异常信息 都要放到api模块中。（对外暴露）符合重用发布等价原则，共同重用原则。

2. api模块中放入Spring的引用配置。也可以放在模块的包目录下。比如com.pengyan.xxx.xx/xxxx-xx.xml

3. 粒度

   >近可能把接口设置成粗粒度。每个服务方法代表一个独立的功能，而不是某个功能的步骤，否则就会涉及到分布式事务。

   > 服务接口建议以业务场景为单位划分。并且相近业务做抽象，防止接口暴增。

   > 不建议使用过于抽象的通用接口。比如T T<泛型>，避免出现接口没有明确的语义，带来后期的维护困难

4. 版本

   > 每个接口都应该定义版本，为后续的兼容性提供前瞻性的考虑 version
   >
   > 建议使用两位版本号，因为第三方版本号表示的是兼容性升级。只有不兼容时才需要变更服务版本。
   >
   > 当接口做到不兼容升级的时候，先升级一半或者一台提供微新版本，再将部分消费者全部升级新版本，然后再将剩下的一半提供者升级新版本。

5. 推荐配置用法

   > 尽可能在provider端配置consumer端的属性：
   >
   > 比如 timeout、 retires、线程池大小 、loadbalance
   >
   > 配置管理员信息：
   >
   > application上面配置的owner，owner建议配置2个人以上。因为owner可以在监控中心看到。

6. 缓存文件

   > 在registry中配置file，可以缓存信息。

   > 主要将注册中心的关键信息缓存到本地，防止注册中心挂掉，提高可用性，在生产环境时必须配置的。

   > 缓存的信息重要包括：注册中心的列表 服务提供者列表。

   ~~~xml
   <!--配置注册中心-->
       <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"  register="true" file="/Users/yanpeng/dubbo.cache" />
   ~~~



#### Dubbo源码分析：

> dubbo是基于Spring的扩展做的，
>
> 基于Spring配置文件的扩展需要实现两个接口：
>
> `NamespaceHandler` ：作用，注册BeanDefinitionParser，利用其解析。
>
> `BeanDefinitionParser` : 解析配置文件的元素



> 扩展Spring时，spring会默认加载jar包下的/META-INF/spring.handlers文件，找到对应的NamespaceHandler

~~~properties
http\://dubbo.apache.org/schema/dubbo=org.apache.dubbo.config.spring.schema.DubboNamespaceHandler
http\://code.alibabatech.com/schema/dubbo=org.apache.dubbo.config.spring.schema.DubboNamespaceHandler
~~~

> `ServiceBean实现了` `InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener<ContextRefreshedEvent>, BeanNameAware` 几个接口

`InitializingBean` : 当Spring容器初始化Bean之后，自动调用afterPropertiesSet()方法

`DisposableBean`:Bean被销毁时调用 destroy()方法

`ApplicationContextAware` ：自动注入`applicationContext`.

`ApplicationListener<ContextRefreshedEvent>`: Spring容器启动后，事件通知。



