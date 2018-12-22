# 小提示

elasticsearch使用的是java写的，默认占用2GB的堆内存空间，在用docker启动的时候，嗯可以限制其内存使用空间：

~~~shell
###设置初始的内存为256M 最大使用256M
docker  run -e ES_JAVA_OPTS="-Xms256 -Xmx256"
~~~

# ES快速入门

https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/

一个Elasticsearch集群可以包含多个索引，每个索引可以包含多个类型，不同的类型存储着多个文档，每个文档又有多个属性。

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181202-141203@2x.png)

# Springboot整合Elasticsearch

- 引入spring-boot-starter-data-elasticsearch
- 安装SpringData对应版本的Elastisearch
- application.xml配置
- Springboot自动配置的：ElasticsearchRepository、ElasticsearchTemplate、Client



Springboot默认支持两种技术和ES进行交互:

- Jest :  http的方式（默认不生效，导如jest的工具包 io.searchbox.client.jestClient就会自动生效）
- SpringData Elasticsearch 的方式 （默认生效）
  - 自动配置了：Client

## 自动配置-springdata

~~~java
@Configuration
@ConditionalOnClass({ Client.class, TransportClientFactoryBean.class,
		NodeClientFactoryBean.class })
@EnableConfigurationProperties(ElasticsearchProperties.class)
public class ElasticsearchAutoConfiguration implements DisposableBean {
~~~

## 自动配置-Jest

~~~java
@Configuration
@ConditionalOnClass(JestClient.class)
@EnableConfigurationProperties(JestProperties.class)
@AutoConfigureAfter(GsonAutoConfiguration.class)
public class JestAutoConfiguration {

~~~





**ElasticsearchAutoConfiguration**

~~~java
@Configuration
@ConditionalOnClass({ Client.class, TransportClientFactoryBean.class,
		NodeClientFactoryBean.class })
@EnableConfigurationProperties(ElasticsearchProperties.class)
public class ElasticsearchAutoConfiguration implements DisposableBean {
    
    ...	
    //自动配置了Client    
    @Bean
	@ConditionalOnMissingBean
	public Client elasticsearchClient() {
		try {
			return createClient();
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
	}
~~~

**ElasticsearchDataAutoConfiguration**

使用template操作es

~~~java
@Configuration
@ConditionalOnClass({ Client.class, ElasticsearchTemplate.class })
@AutoConfigureAfter(ElasticsearchAutoConfiguration.class)
public class ElasticsearchDataAutoConfiguration {

    //自动配置了ElasticsearchTemplate
    //template用来操作es
	@Bean
	@ConditionalOnMissingBean
	@ConditionalOnBean(Client.class)
	public ElasticsearchTemplate elasticsearchTemplate(Client client,
			ElasticsearchConverter converter) {
		try {
			return new ElasticsearchTemplate(client, converter);
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
	}
~~~

**ElasticsearchRepositoriesAutoConfiguration**

使用继承ElasticsearchRepository的子类来操作es

~~~java
//启用es的repository
//启用的话，就类似于jpa的操作方式，继承ElasticsearchRepository就可以操作数据库
@Configuration
@ConditionalOnClass({ Client.class, ElasticsearchRepository.class })
@ConditionalOnProperty(prefix = "spring.data.elasticsearch.repositories", name = "enabled", havingValue = "true", matchIfMissing = true)
@ConditionalOnMissingBean(ElasticsearchRepositoryFactoryBean.class)
@Import(ElasticsearchRepositoriesRegistrar.class)
public class ElasticsearchRepositoriesAutoConfiguration {

}

~~~

##Jest的简单使用

https://github.com/searchbox-io/Jest

引入maven

~~~xml
<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>5.3.3</version>
    <type>pom</type>
</dependency>

~~~

加入配置

~~~properties
##端口是9200
spring.elasticsearch.jest.uris=http://127.0.0.1:9200
~~~

使用示例

~~~java
@Autowire
JestClient jestClient;

public void insert() throw IOException{
    //1、给ES中索引(保存)一个文档
    Article a = new Article();
    a.setId("1");
    a.setConetnt("test content");
	//构建一个索引
    Index index = new Index.Builder(a).index("testIndex").type("testType").build();
    //执行
    jesetClient.execute(index);
}

//查询
public  void search() throw IOException{
    //伪代码
    String json = "{
                        "query" : {
                            "constant_score" : { 
                                "filter" : {
                                    "term" : { 
                                        "price" : 20
                                    }
                                }
                            }
                        }
                    }";
    new 
//构建搜索功能    Search.Builder(josn).addIndex("testIndex").addType("testType").build();
//执行
   SearchResult result =  jesetClient.execute(index);
}
~~~

~~~java
@Data
public class Article{
	//标明这个是主键
    @JestId
    String id;
    String content;
}
~~~

## ElasticsearchTemplate的使用

配置

~~~properties
spring.data.elasticsearch.cluster-name=fabulous
##这里是9300
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
~~~

SpringData ，ES版本可能不合适，需要调整

版本对应关系：https://github.com/spring-projects/spring-data-elasticsearch



**简单使用**

~~~java
//和jpa的使用方法 一样
public interface MyEsRepository extends ElasticsearchRepository<Book，String>{
    
}
~~~

**Book**

~~~java
//注解一定要，指定文档，指定文档的索引名称 指定type名称
@Document(indexName="atguigu",type="book")
public class Book{
    ...
}
~~~

**ElasticsearchCrudRepository**

~~~java

@NoRepositoryBean
public interface ElasticsearchRepository<T, ID extends Serializable> extends ElasticsearchCrudRepository<T, ID> {
    //索引一个文档
    <S extends T> S index(S var1);

    Iterable<T> search(QueryBuilder var1);

    Page<T> search(QueryBuilder var1, Pageable var2);

    Page<T> search(SearchQuery var1);

    Page<T> searchSimilar(T var1, String[] var2, Pageable var3);

    void refresh();

    Class<T> getEntityClass();
}
~~~



更多操作：https://docs.spring.io/spring-data/elasticsearch/docs/3.0.12.RELEASE/reference/html/