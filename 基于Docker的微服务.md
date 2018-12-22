# Docker容器的网络基础-docker0

docker0 其实就是linux的虚拟网桥

![](/Users/yanpeng/Documents/pengyand/spring-boot/images/QQ20181204-081931.png)

Linux虚拟网桥的特点：

- 可以设置IP地址
- 相当于拥有一个隐藏的虚拟网卡

## 自定义docker0

修改docker0地址：

~~~shell
#为docker0定义新的网络
$ sudo ifconfig docker0 192.168.200.1 netmask 255.255.255.0
~~~

## 自定义虚拟网桥

- 添加虚拟网桥

  ~~~shell
  $sudo brctl addbr br0
  $sudo ifconfig br0 192.168.100.1 netmask 255.255.255.0
  ~~~

- 修改docker守护进程的启动配置

  ~~~shell
  //ubantu系统中
  /etc/default/docker 中添加DOKCER_OPS值
  -b=br0
  ~~~
