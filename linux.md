#### 查看文件夹大小：

~~~javascript
du --max-depth=1 -h

或者

du -h
~~~

#### 删除文件不提示：

~~~javascript
rm -f
~~~

#### 解压gz

>解压1：gunzip FileName.gz
>
>解压2：gzip -d FileName.gz
>
>压缩：gzip FileName
>
>.tar.gz 和 .tgz
>
>解压：tar zxvf FileName.tar.gz
>
>压缩：tar zcvf FileName.tar.gz DirName

### 查看端口是否被监听

> lsof -i:12345

```shell
查看Centos端口命令： 
# netstat -lntp #查看监听(Listen)的端口
# netstat -antp #查看所有建立的TCP连接
```

#### SSH 免密码登录

> root用户
>
> vim ~/.ssh/authorized_keys



#### 创建用户

> root用户
>
> useradd zhaohuan2
>
> passwd zhaohuan2

#### 查看文件夹大小



#### 查看当前java的线程

~~~shell
[tomcat@iZbp1fdnemzvxcc7ho8m1aZ ~]$ jps
16755 Jps
3111 Bootstrap
3517 Bootstrap
~~~

####查看某个进程的线程数

~~~shell
cat /proc/${pid}/status 

[tomcat@iZbp1fdnemzvxcc7ho8m1aZ ~]$ cat /proc/3111/status
Name:	java
State:	S (sleeping)
Tgid:	3111
Ngid:	0
Pid:	3111
PPid:	3109
TracerPid:	0
Uid:	507	507	507	507
Gid:	507	507	507	507
FDSize:	1024
Groups:	507
VmPeak:	 6138468 kB
VmSize:	 6135860 kB
VmLck:	       0 kB
VmPin:	       0 kB
VmHWM:	 1501004 kB
VmRSS:	 1315184 kB
VmData:	 5901688 kB
VmStk:	     136 kB
VmExe:	       4 kB
VmLib:	   24744 kB
VmPTE:	    3524 kB
VmSwap:	       0 kB
Threads:	222
SigQ:	0/31219
SigPnd:	0000000000000000
ShdPnd:	0000000000000000
SigBlk:	0000000000000000
SigIgn:	0000000000000002
SigCgt:	2000000181005ccd
CapInh:	0000000000000000
CapPrm:	0000000000000000
CapEff:	0000000000000000
CapBnd:	0000001fffffffff
Seccomp:	0
Cpus_allowed:	7fff
Cpus_allowed_list:	0-14
Mems_allowed:	00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
Mems_allowed_list:	0
voluntary_ctxt_switches:	89
nonvoluntary_ctxt_switches:	7
~~~

#### 查看线程数

~~~shell
pstree -p ${pid} 
~~~



#### 查看内核

~~~shell
uname -r
~~~

#### 启动docker并设置开启启动（centos7）

~~~shell
#启动
systemctl start docker
or
systemctl start docker.service
#设置开机启动
systemctl enable docker 
or 
systemctl enable docker.service
~~~



