#### mysql创建用户并授权

~~~shell
create user nbcircle identified by 'nbshixian';

grant all privileges on *.* to 'nbcircle'@'%' identified by 'nbshixian' with grant option; 

flush privileges; 
~~~

~~~shell
#查看binlog日志  https://blog.csdn.net/leshami/article/details/41962243
/home/test/mysql/bin/mysqlbinlog --base64-output=decode-rows -v  --database fabulous --start-datetime='2018-10-16 16:42:00'  mysql-bin.000025  --result-file=extra02.sql
~~~

