#### 命令

~~~shell
docker run centos:7 sleep 1000
~~~

启动centos 7版本的镜像，睡眠1秒 sleep的目的主要是演示使用，可以不加。



~~~shell
docker history pengyand/centos:1.0
~~~

查看docker镜像的历史



运行docker

~~~shell
docker run -e "SPRING_PROFILES_ACTIVE=test" -p 7931:8808 --name imReceiver7931 -v /mnt/im_chat_log/im_recevier/:/data/logs registry.cn-hangzhou.aliyuncs.com/pengyand/im-receiver:0.0.1-SNAPSHOT

docker run -e "SPRING_PROFILES_ACTIVE=pro" -p 3870:8808 --name imReceiver3870 -v /mnt/im_chat_log/im_recevier/:/data/logs registry.cn-hangzhou.aliyuncs.com/pengyand/im-receiver:0.0.1-SNAPSHOT


docker run -d -e "SPRING_PROFILES_ACTIVE=test" --name imProcessor4ShareDB -v /mnt/im_chat_log/im_processor/:/data/logs registry.cn-hangzhou.aliyuncs.com/pengyand/im-processor:0.0.1-SNAPSHOT

docker run -d -e "SPRING_PROFILES_ACTIVE=pro" --name imProcessor1 -v /mnt/im_chat_log/im_processor/:/data/logs registry.cn-hangzhou.aliyuncs.com/pengyand/im-processor:0.0.1-SNAPSHOT
~~~



# 如何获取 docker 容器(container)的 ip 地址

https://blog.csdn.net/sannerlittle/article/details/77063800



# Docker修改默认的网段

http://blog.51cto.com/wsxxsl/2060761



### Docker时区修改

https://blog.csdn.net/xl_lx/article/details/81181135

[https://docs.lvrui.io/2017/06/05/Docker容器的时区设置/](https://docs.lvrui.io/2017/06/05/Docker容器的时区设置/)

[https://brickyang.github.io/2017/03/16/Docker%20中如何设置%20container%20的时区/](https://brickyang.github.io/2017/03/16/Docker%20中如何设置%20container%20的时区/)

>容器内采用的是UTC时间，导致晚了八个小时，CST=UTC/GMT +8 小时。 
>如果此时想设置成北京的时间的话，有两种解决方案，第一个是修改容器时区
>
>CentOS
>
>~~~shell
>RUN echo "Asia/shanghai" > /etc/timezone;
>~~~
>
>
>
>Ubuntu
>
>~~~shell
>RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
>~~~
>
>
>
>第二种方案，更简单，直接把宿主机的时间设置挂载到容器里面，通过docker run -v的方式直接挂载
>
>~~~shell
>-v /etc/localtime:/etc/localtime:ro 
>~~~
>
>这种方案需要注意/etc/timezone内容，因为代码中会用到这个设置，也要改成上海时区。最后测试一下，两种方案都是可行的。

#### Docker mysql安装

https://blog.csdn.net/harris135/article/details/79712750

~~~shell
docker run -d -p 47.96.160.41:8031:3306 --privileged=true -v /mnt/docker_data/mysql/etc:/etc/mysql -v /mnt/docker_data/mysql/data:/var/lib/mysql -v /mnt/docker_data/mysql/logs:/var/log/mysql -v /etc/localtime:/etc/localtime:ro -e MYSQL_ROOT_PASSWORD=shixian.321 --name mysql8031 mysql:5.7
~~~



# [docker容器中挂载的目录没有权限的问题](https://www.cnblogs.com/codedoge/p/10037826.html)



使用docker-compose，启动的容器默认是用的root权限，但是docker中的root只是相当于普通用户

所以需要给挂载的目录或者文件开启权限，代码如下：

```
#开启目录权限
chmod a+rwx /home/user/ 

#开启docker挂载权限
chmod a+rw /var/run/docker.sock 
```



 [【docker】查看docker容器或镜像的详细信息命令，查看docker中正在运行的容器的挂载位置](https://www.cnblogs.com/sxdcgaq8080/p/9198309.html)

命令：

```
docker inspect f257d69e0035
```

格式：

```
docker inspect 容积或镜像ID
```



#docker无法启动

https://www.cnblogs.com/51kata/p/5276407.html



# docker 运行测试服 nginx

~~~shell
docker run -p 2443:2443 -p 4443:4443  --name h5nginx -v /mnt/docker_nginx_h5/wwww:/home -v /mnt/docker_nginx_h5/config:/etc/nginx -v /mnt/docker_nginx_h5/logs:/var/log/nginx -d nginx
~~~

# docker运行测试服apache

~~~shell
docker run --name myapache -v /mnt/docker_apache/htdocs/:/usr/local/apache2/htdocs/ -v /mnt/docker_apache/conf/httpd.conf:/usr/local/apache2/conf/httpd.conf -v /mnt/docker_apache/logs/:/usr/local/apache2/logs/  -p 83:80 -d httpd
~~~

docker 运行测试服 php

~~~shell
docker run --name myphp -p 84:80  -v /mnt/docker_php/data/superH5/:/var/www/html/ -v /mnt/docker_php/logs/:/var/log/apache2/ -v /mnt/docker_php/conf/apache2.conf:/etc/apache2/apache2.conf   php:5.6-apache
~~~

~~~shell
docker run --name myphp2 -p 85:80  -v /mnt/docker_php2/data/superH5/:/var/www/html/ -v /mnt/docker_php2/logs/:/var/log/apache2/ -v /mnt/docker_php2/conf/apache2.conf:/etc/apache2/apache2.conf   php:5.6-apache
~~~

