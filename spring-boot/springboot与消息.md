# 概念

1、大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力。

2、消息服务中两个重要该概念：

- 消息代理-message broker

- 目的地-destination

  当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

3、消息队列主要有两种形式的目的地

- 队列（queue）：点对点消息通信（point-to-point）
- 主题（topic）：发布（publish）/订阅（subscribe）消息通信

4、点对点式：

- 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移除队列
- 消息只有唯一一个发送者和接受者，并并不是说只能有一个接收者。（接受者是真正处理消息的对象）

5、发布订阅式：

- 发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息。

6、JMS（java Message Service）Java消息服务：

- 基于JVM消息代理的规范。ActiveMq、HornetMQ是JMS实现。

7、AMQP（Advanced Message Queuing Protocol）

- 高级消息队列，也是一个消息代理的规范，兼容JMS
- RabbitMQ式AMQP的实现。

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-103234@2x.png)

# Spring支持

- spring-jms提供了JMS的支持
- spring-rabbit提供了对AMQP的支持
- 需要ConnectionFactory的实现来连接消息代理
- 提供JmsTemplate、RabbitTemplate来发送消息
- @JsmListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
- @EnableJms、@EnableRabbit开启支持。

# Spring boot自动配置

- JmsAutoConfiguration
- RabbitAutoConfiguration



# 使用MQ

https://docs.spring.io/spring-boot/docs/1.5.18.RELEASE/reference/htmlsingle/#boot-features-connecting-to-redis



# RabbitMQ简介

RabbitMQ是一个由Erlang开发的AMQP（Advanced Message Queue Protocol）的开源实现。

**Message**

>消息 由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键），priority（相对于其他消息的优先权）、delivery-mode（指出该消息肯呢个需要持久化存储）等。

**Publish**

> 消息的生产者，也是一个向交换机发布消息的客户端应用程序

**Exchange**

> 交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
>
> Exchange有4种类型：direct（默认,点对点）、fanout、topic和headers，不同类型的Exchange转发消息的策略有所区别。

**Queue**

> 消息队列 用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者链接到这个队列将其取走。

**Binding**

> 绑定，用于消息队列和交换器直接的关联。一个绑定就是基于路由键将交换器和消息队列链接起来的路由规则，所以可以将交换器理解成一个由绑定结构的路由表。
>
> Exchange和Queue的绑定可以是多对多的关系。

**Connection**

> 网络连接，比如一个tcp链接

**Channel**

> 信道，多路复用连接中的一个独立的双向数据流通道。信道是建立在真实的TCP链接内的虚拟连接，AMQP命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成的。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。

**Consumer**

> 消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

**Virtual Host**

> 虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证的加密环境的独立服务器域。每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。vhost是AMQP概念的基础，必须在连接时指定，RabbitMQ默认的vhost是 / 。

**Broker**

> 表示消息队列服务器实体。

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-105901@2x.png)

# RabbitMQ运行机制

**AMQP中的消息路由**

> AMQP中消息的路由过程和Java开发者属性的JMS存在一些差别，AMQP中增加了Exchange和Binding的角色。生产者把消息发布到Exchange上，消息最终达到队列并被消费者接收，而Binding决定交换器的消息应该发送到哪个队列上。
>
> ![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-110445@2x.png)

**Exchange类型**

> Exchange分发消息时根据类型的不同分发策略所有区别，目前共有四种类型：direct、fanout、topic、headers。headers匹配AMQP消息的header而不是路由键，headers交换器和direct交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种 类型

> **direct Exchange**
>
> 消息中的路由键（routing key）如果和Binding中的binding key一致，交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为"dog"，则只转发routing key标记为“dog”的消息，不会转发“dog.puppy“，也不会转发“dog.guard”登登。它是完全匹配、单播的模式。
>
> ![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-111453@2x.png)

> **Fanout Exchange**
>
> 每个发到fanout类型交换器的消息都会被分到所有绑定队列上去。fanout交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout类型转发消息是最快的。
>
> ![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-111851@2x.png)

> **Topic Exchange**
>
> topic交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行拼配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单次，这个单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#拼配0个或多个单词，\* 匹配一个单词。
>
> ![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-111859@2x.png)

# RabbitMQ整合

1、引入spring-boot-starter-amqp

2、application.yml配置

3、测试RabbitMQ

- AmqpAdmin：管理组件
- RabbitTemplate：消息发送处理组件

## 自动配置

~~~java
@Configuration
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
//封装了rabbitmq的所有配置
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {

    ...
       //帮我们配置了连接工厂
        @Bean
		public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties config)
				throws Exception {
        
        
        ...
            //给rabbitmq发送消息
        @Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnMissingBean(RabbitTemplate.class)
		public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
            
            
            ...
                //RabbitMQ的系统管理功能组件 可以用来创建 Exchange、queue、bingding等
        @Bean
		@ConditionalOnSingleCandidate(ConnectionFactory.class)
		@ConditionalOnProperty(prefix = "spring.rabbitmq", name = "dynamic", matchIfMissing = true)
		@ConditionalOnMissingBean(AmqpAdmin.class)
		public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {
			return new RabbitAdmin(connectionFactory);
		}
            ...

	}
~~~

RabbitTemplate发送消息Message：

~~~java
package org.springframework.amqp.core;

public class Message implements Serializable {
    private static final long serialVersionUID = -7177590352110605597L;
    private static final String ENCODING = Charset.defaultCharset().name();
    private static final SerializerMessageConverter SERIALIZER_MESSAGE_CONVERTER = new SerializerMessageConverter();
	//消息头信息
    private final MessageProperties messageProperties;
    //消息体
    private final byte[] body;
~~~

两种发送消息的方法：

~~~java
//Message需要自己构造一个，定义消息体内容和消息头
rabbitTemplate.send(exchange,routeKey,message);

//常用
//object对象默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq
//默认使用java的序列化方式
rabbitTemplate.convertAndSend(exchange,routeKey,object)
~~~

接收消息的方法

~~~java
//返回Message对象
rabbitTemplate.receive(String queueName);

//返回消息体对象
rabbitTemplate.receiveAndConvert(String queueName);
~~~



如何将数据自动转成json

在RabbitTemplate的类中有一个消息转换器：

默认使用的是SimpleMessageConverter

~~~java
private volatile MessageConverter messageConverter = new SimpleMessageConverter();
~~~

可以换一个类型

~~~java
@Configuration
public class MyAMqpConfig {
	//换一个消息转换器
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}

~~~

## 监听

~~~java
//需要在主配置类加入@EnableRabbit的注解
@EnableRabbit //开启基于注解的Rabbitmq模式
@SpringBootApplication
public class SpringBootTestApplication{
    
}


@Service
public class BookService{
    
    @RabbitListener(queue={"queue1","queue2"})
    //参数可以是Object(直接接收消息体) 也可以是Message(Message 可以得到消息头)
    public void receiver(Book book){
        
    }
    
}
~~~

## 使用AmqpAdmin

使用AmqpAdmin 自动创建和删除Queue，Exchange，Binding

~~~java
//直接使用

@Autowired
AmqpAdmin amqpAdmin;
~~~

~~~java
public interface AmqpAdmin {
    //declare声明、创建 declare都是创建
    void declareExchange(Exchange var1);

    boolean deleteExchange(String var1);

    Queue declareQueue();

    String declareQueue(Queue var1);

    boolean deleteQueue(String var1);

    void deleteQueue(String var1, boolean var2, boolean var3);

    // purge清除，删除。purge都是删除
    void purgeQueue(String var1, boolean var2);

    int purgeQueue(String var1);

    void declareBinding(Binding var1);

    void removeBinding(Binding var1);

    Properties getQueueProperties(String var1);

    default void initialize() {
    }
}

~~~

~~~java
//创建一个Exchange
amqpAdmin.declareExchange(new DirectExchange("exchange.myexchange"));

//创建一个需要持久化的queue(持久化是指mq重启时，不消失)
amqpAdmin.delareQueue(new Queue("queue.myqueue",true));

//创建一个绑定规则
amqpAdmin.declareBinding(new Binding("queue.myqueue",Bingding.DestinationType.QUEUE,exchange.myexchange);

~~~

Binding对象

~~~java
public class Binding extends AbstractDeclarable {
    //目的地，可以是queue也可以是Exchange
    private final String destination;
    //转发器
    private final String exchange;
    //
    private final String routingKey;
    private final Map<String, Object> arguments;
    private final Binding.DestinationType destinationType;
    
      public Binding(String destination, Binding.DestinationType destinationType, String exchange, String routingKey, Map<String, Object> arguments) {
        this.destination = destination;
        this.destinationType = destinationType;
        this.exchange = exchange;
        this.routingKey = routingKey;
        this.arguments = arguments;
    }
~~~

