# 引入redis-start

[文档]: https://docs.spring.io/spring-boot/docs/1.5.18.RELEASE/reference/htmlsingle/#boot-features-caching-provider-redis



# redis自动配置类

~~~java
@Configuration
@ConditionalOnClass({ JedisConnection.class, RedisOperations.class, Jedis.class })
@EnableConfigurationProperties(RedisProperties.class)
public class RedisAutoConfiguration {
    
    	
    	//自动注入连接工程类
    	@Bean
		@ConditionalOnMissingBean(RedisConnectionFactory.class)
		public JedisConnectionFactory redisConnectionFactory()
				throws UnknownHostException {
			return applyProperties(createJedisConnectionFactory());
		}
    
    /**
	 * Standard Redis configuration.
	  spring简化redis操作的两个template
	  可以直接注入使用
	 */
	@Configuration
	protected static class RedisConfiguration {

        //操作Object的
		@Bean
		@ConditionalOnMissingBean(name = "redisTemplate")
		public RedisTemplate<Object, Object> redisTemplate(
				RedisConnectionFactory redisConnectionFactory)
				throws UnknownHostException {
			RedisTemplate<Object, Object> template = new RedisTemplate<Object, Object>();
			template.setConnectionFactory(redisConnectionFactory);
			return template;
		}

        //专门操作字符串的
		@Bean
		@ConditionalOnMissingBean(StringRedisTemplate.class)
		public StringRedisTemplate stringRedisTemplate(
				RedisConnectionFactory redisConnectionFactory)
				throws UnknownHostException {
			StringRedisTemplate template = new StringRedisTemplate();
			template.setConnectionFactory(redisConnectionFactory);
			return template;
		}

	}


~~~

示例

~~~java
@Autowired
StringRedisTemplate stringRedisTemplate

@Test
public void test(){
	//为msg键 追加 hello字符串的值
	stringRedisTemplate.opsForValue().append("msg","hello");
	//获取msg的值
	String msg = stringRedisTemplate.opsForValue.get("msg");
}

//使用redisTemplate保存对象时，如果不指定序列化器，被保存的对象一定要实现jdk的序列化接口
//可以自定义序列化器
@Test
public void test02(){
	Employee emp = employeeMapper.getEmp(1);
	redisTemplate.opsForValue.set("emp-01",emp);
}
~~~

自定义redis的序列化器

~~~java
//RedisTemplate<Object,Object>的源码：

public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {
...
    public void afterPropertiesSet() {

		super.afterPropertiesSet();

		boolean defaultUsed = false;

		if (defaultSerializer == null) {
			//默认使用的是jdk的序列化器
			defaultSerializer = new JdkSerializationRedisSerializer(
					classLoader != null ? classLoader : this.getClass().getClassLoader());
		}

~~~

~~~java
@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, Employee> redisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        //Jackson2JsonRedisSerializer 是 RedisSerializer<T> 其中的一个实现
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
}

~~~



##redis配置

~~~properties
# REDIS (RedisProperties)
spring.redis.cluster.max-redirects= # Maximum number of redirects to follow when executing commands across the cluster.
spring.redis.cluster.nodes= # Comma-separated list of "host:port" pairs to bootstrap from.
spring.redis.database=0 # Database index used by the connection factory.
spring.redis.url= # Connection URL, will override host, port and password (user will be ignored), e.g. redis://user:password@example.com:6379
spring.redis.host=localhost # Redis server host.
spring.redis.password= # Login password of the redis server.
spring.redis.ssl=false # Enable SSL support.
spring.redis.pool.max-active=8 # Max number of connections that can be allocated by the pool at a given time. Use a negative value for no limit.
spring.redis.pool.max-idle=8 # Max number of "idle" connections in the pool. Use a negative value to indicate an unlimited number of idle connections.
spring.redis.pool.max-wait=-1 # Maximum amount of time (in milliseconds) a connection allocation should block before throwing an exception when the pool is exhausted. Use a negative value to block indefinitely.
spring.redis.pool.min-idle=0 # Target for the minimum number of idle connections to maintain in the pool. This setting only has an effect if it is positive.
spring.redis.port=6379 # Redis server port.
spring.redis.sentinel.master= # Name of Redis server.
spring.redis.sentinel.nodes= # Comma-separated list of host:port pairs.
spring.redis.timeout=0 # Connection timeout in milliseconds.
~~~

# CacheManager使用redis

只要引入redis-starter就会自动使用RedisCacheManager

~~~java
@Configuration
@AutoConfigureAfter(RedisAutoConfiguration.class)
//条件成立
@ConditionalOnBean(RedisTemplate.class)
//条件成立
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class RedisCacheConfiguration {

	private final CacheProperties cacheProperties;

	private final CacheManagerCustomizers customizerInvoker;

	RedisCacheConfiguration(CacheProperties cacheProperties,
			CacheManagerCustomizers customizerInvoker) {
		this.cacheProperties = cacheProperties;
		this.customizerInvoker = customizerInvoker;
	}

    //自动创建一个CacheManager
	@Bean
	public RedisCacheManager cacheManager(RedisTemplate<Object, Object> redisTemplate) {
		RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
		cacheManager.setUsePrefix(true);
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}

}

~~~

​	默认保存数据k-v，利用序列化进行保存。

如何修改为自己想要的序列化？

1、默认创建的RedisCacheManager 使用的redisTemplate 默认使用的是jdk的序列化机制

2、所以需要我们自定义cacheManager

~~~java
	@Bean
    public RedisTemplate<Object, Employee> employeeRedisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Employee> template = new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser = new Jackson2JsonRedisSerializer<Employee>();
        template.setDefaultSerializer(ser);
        return template;
    }
	//自定义cacheManager使用了自己的employeeRedisTemplate
    @Bean
    public RedisCacheManager cacheManager(RedisTemplate<Object, Employee> employeeRedisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        //key会有一个前缀
        //设置使用前缀，会将CacheName作为key的前缀 比如键 emp:1  (emp就是前缀)
        cacheManager.setUsePrefix(true);
        return cacheManager;
    }
~~~

