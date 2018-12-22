# 监控管理

通过引入Spring-boot-starter-actuator，可以使用Springboot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过http、jms、ssh协议来进行操作，自动的到审计健康及指标信息等。

需要设置

~~~properties
management.security.enable=false
~~~

可以查看的监控项：

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181203-210825@2x.png)

~~~properties
#开启shutdown端点
endpoints.shutdown.enable=true

使用post请求发送 可以远程关闭应用
http://localhost:8080/shudown
~~~

