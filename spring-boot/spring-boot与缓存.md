# JSR107

1、Java Caching定义了5个核心接口，分别是CachingProvider、CacheManager、Cache、Entry和Expiry

- CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager，一个应用 可以在运行期间访问多个CachingProvider。
- CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅能被一个CachingProvider所拥有
- Cache是一个类似于Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
- Entry是一个存储在Cache中的key-value对
- Expiry每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态，一旦过期，条目将不可以访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

![如图所示](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181201-192230.png)

要使用jsr107 需要导入

~~~xml
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
</dependency>
~~~



# Spring的缓存抽象

Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术，并坚持使用JCache（JSR107）注解简化我们开发：

- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合。

- Cache接口下Spring提供了各种XXCache的实现；如RedisCache、EhCacheCache，ConcurrentMapCache等

- 每次调用需要缓存功能的方法时，Spring会检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法，并缓存结果后返回给用户，下次调用直接从缓存中获取。

- 使用Spring缓存抽象时我们需要关注以下两点：
  1、确定方法需要被缓存以及他们的缓存策略。

  2、从缓存中读取之前缓存存储的数据。

## 几个重要的概念&缓存注解

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache |
| -------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件                        |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存     |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，并希望结果被缓存                             |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |

###Cacheable

~~~java

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {
    @AliasFor("cacheNames")
    String[] value() default {};
	
    //指定缓存组件的名字
    @AliasFor("value")
    String[] cacheNames() default {};
	//缓存数据时使用的key，如果不指定。默认使用方法参数的值。
    //也可以使用SpEL表达式
    //比如：@Cacheable(cacheNames="emp" ,key ="#id")
    String key() default "";

    //key的生成器；可以自己指定key的生成器的组件id key和keyGenerator只能选择一个
    String keyGenerator() default "";

    //指定缓存管理器：从哪个缓存管理器拿到缓存
    String cacheManager() default "";
	//指定缓存解析器：和缓存管理器二选一，作用一样。
    String cacheResolver() default "";
	
    //指定符合条件下，才缓存。也可以使用SpEL表达式
    String condition() default "";

    //否定缓存，当unless的条件为true，方法的返回值就不会被缓存。可以获取到结果进行判断
    //比如：@Cacheable(cacheNames="emp",unless="#result == null") #result获取方法执行的结果，当结果为null时，就不缓存
    String unless() default "";
	//缓存是否使用异步模式  异步模式下 unless不支持
    boolean sync() default false;
}
~~~

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/1543666620016.jpg)

可以使用的SpEL的表示式有：

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181201-202039.png)

####缓存的工作原理

1、自动配置类；

~~~java
@Configuration
@ConditionalOnClass(CacheManager.class)
@ConditionalOnBean(CacheAspectSupport.class)
@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")
@EnableConfigurationProperties(CacheProperties.class)
@AutoConfigureBefore(HibernateJpaAutoConfiguration.class)
@AutoConfigureAfter({ CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class,
		RedisAutoConfiguration.class })
//1、导入CacheConfigurationImportSelector
@Import(CacheConfigurationImportSelector.class)
public class CacheAutoConfiguration {
 
    //缓存管理器的自定义器
    @Bean
	@ConditionalOnMissingBean
	public CacheManagerCustomizers cacheManagerCustomizers(
			ObjectProvider<List<CacheManagerCustomizer<?>>> customizers) {
		return new CacheManagerCustomizers(customizers.getIfAvailable());
	}
    
    
    /** 
    	2、缓存配置导入选择器
    	imports 中一共有如下的配置类：
    	org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
    	//什么都不配置的话，这个生效
    	org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration
    	org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
    	
    	哪个配置类会生效？？
    		由下面的Condition来决定
    		@Configuration
            @ConditionalOnBean(Cache.class)
            @ConditionalOnMissingBean(CacheManager.class)
            @Conditional(CacheCondition.class)
            class GenericCacheConfiguration {

    	
	 * {@link ImportSelector} to add {@link CacheType} configuration classes.
	 */
	static class CacheConfigurationImportSelector implements ImportSelector {

		@Override
		public String[] selectImports(AnnotationMetadata importingClassMetadata) {
			CacheType[] types = CacheType.values();
			String[] imports = new String[types.length];
			for (int i = 0; i < types.length; i++) {
				imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
			}
			return imports;
		}

	}	
~~~

**SimpleCacheConfiguration配置类**:

~~~java
@Configuration
@ConditionalOnMissingBean(CacheManager.class)
@Conditional(CacheCondition.class)
class SimpleCacheConfiguration {

	private final CacheProperties cacheProperties;

	private final CacheManagerCustomizers customizerInvoker;

	SimpleCacheConfiguration(CacheProperties cacheProperties,
			CacheManagerCustomizers customizerInvoker) {
		this.cacheProperties = cacheProperties;
		this.customizerInvoker = customizerInvoker;
	}

	@Bean
	public ConcurrentMapCacheManager cacheManager() {
		ConcurrentMapCacheManager cacheManager = new ConcurrentMapCacheManager();
		List<String> cacheNames = this.cacheProperties.getCacheNames();
		if (!cacheNames.isEmpty()) {
			cacheManager.setCacheNames(cacheNames);
		}
		return this.customizerInvoker.customize(cacheManager);
	}

}

~~~

**ConcurrentMapCacheManager**

~~~java
public class ConcurrentMapCacheManager implements CacheManager, BeanClassLoaderAware {
    ...
        
        //缓存的名字和缓存保存在map中
        //获取一个Cache
	@Override
	public Cache getCache(String name) {
		Cache cache = this.cacheMap.get(name);
		if (cache == null && this.dynamic) {
			synchronized (this.cacheMap) {
				cache = this.cacheMap.get(name);
				if (cache == null) {
					cache = createConcurrentMapCache(name);
					this.cacheMap.put(name, cache);
				}
			}
		}
		return cache;
	}
    ...
        	/**
        	创建一个Cache
	 * Create a new ConcurrentMapCache instance for the specified cache name.
	 * @param name the name of the cache
	 * @return the ConcurrentMapCache (or a decorator thereof)
	 */
	protected Cache createConcurrentMapCache(String name) {
		SerializationDelegate actualSerialization = (isStoreByValue() ? this.serialization : null);
		return new ConcurrentMapCache(name, new ConcurrentHashMap<Object, Object>(256),
				isAllowNullValues(), actualSerialization);

	}
~~~

####运行流程

1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；第一次获取Cache，如果没有会自动创建一个。（CacheManager先获取Cache）

2、去Cache中使用key查找缓存。（key如果没有指定则是方法的参数：是按照某种策略生成）

~~~java
public abstract class CacheAspectSupport extends AbstractCacheInvoker
		implements BeanFactoryAware, InitializingBean, SmartInitializingSingleton {
		
		...
		//默认使用SimpleKeyGenerator
	private KeyGenerator keyGenerator = new SimpleKeyGenerator();
	
//	SimpleKeyGenerator 生成key的策略：
如果没有参数： key = new　SimpleKey();
如果有一次参数：key=参数的值
如果有多个参数：key = new SimpleKey(params);
	
~~~

3、如果在Cache中没有找到缓存，则调用目标方法

4、将目标方法返回的结果放入到缓存中。

自定义IdGenerator

~~~java
@Cacheable(keyGenerator="keyGenerator")


@Configuration
public class KeyGeneratorCustomer {
    
    @Bean 
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+"["+ Arrays.asList(params).toString()+"]";
            }
        };
    }
}
~~~

### CachePut

运行时机：

1、先调用目标方法

2、将目标方法的返回值进行缓存

~~~java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CachePut {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";

    String unless() default "";
}

~~~

示例

~~~java
//key可以是#employee.id 也可以是#result.id(返回后的对象) 
@CachePut(value = "emp",key="#employee.id")
public Employee updateEmp(Employee employee){
    employeeMapper.updateEmp(employee);
    return employee;
}
~~~

### CacheEvict

~~~java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheEvict {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";
	//删除缓存中的所有的key的缓存 
    boolean allEntries() default false;

    //是否在方法之前执行 
    boolean beforeInvocation() default false;
}
~~~

示例

~~~java

@CacheEvict(value ="emp" ,key = "#id")//value可以使用cacheNames替换
public void deleteEmp(Integer id){
    employee.deleteById(id);
}
~~~

### Caching

是一个组合注解

~~~java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Caching {
    Cacheable[] cacheable() default {};

    CachePut[] put() default {};

    CacheEvict[] evict() default {};
}

~~~

示例

~~~java

@Caching(
    cacheable={
        @Cacheable(value="emp",key="#lastName")
    },
    put = {
        //如果按照lastName查询，该方法还是会执行，因为有CachePut注解。
        @CachePut(value="emp",key="#result.id"),
        @CachePut(value="emp",key="#result.email")
    }
    
)
public Employee getEmpByLastName(String lastName){
    return employeeMapper.getEmpByLastName(lastName);
}
~~~



### CacheConfig

在class上指定公共属性

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CacheConfig {
    String[] cacheNames() default {};

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";
}

~~~

示例

~~~java

@CacheConfig(cacheNames="emp",chacheManager="empCacheManager")
public Class EmployeeService{
    
}
~~~



### 使用编码的方式使用缓存

~~~java
@Autowred
DeptCacheManager deptCacheManager;

pubic Department getDeptById(Integer id){
    Department department = departmentMapper.getDeptById(Id);
    Cache deptCache = depetCacheManager.getCache("dept");
    deptCache.put("dept-01",department);
    return department;
}
~~~

