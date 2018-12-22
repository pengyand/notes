#### info

查看redis所有信息

http://ghoulich.xninja.org/2016/10/15/how-to-monitor-redis-status/



>
>
># Server
>
>redis_version:3.0.6 --redis服务版本
>
>redis_git_sha1:00000000
>
>redis_git_dirty:0
>
>redis_build_id:3edbb92b537d6123 
>
>redis_mode:cluster --运行模式，分为:Cluster,Sentinel,Standalone
>
>os:Linux 3.10.0-327.el7.x86_64 x86_64 --Redis所在机器的操作系统
>
>arch_bits:64 --架构(32或64位)
>
>multiplexing_api:epoll --redis所使用的事件处理机制
>
>gcc_version:4.1.2 --编译Redis时所使用的GCC版本
>
>process_id:13902 --Redis服务进程的PID
>
>run_id:4bd3b3cde098b2b8e952f5ba77ee0322d69a9eb2 --服务的标识符
>
>tcp_port:8029 --监听端口
>
>uptime_in_seconds:24321169 --自Redis服务启动以来，运行的秒数
>
>uptime_in_days:281 --自Reids服务启动以来，运行的天数
>
>hz:10 --serverCron 每秒运行次数
>
>lru_clock:7513980 --以分钟为单位进行自增的时钟，用于LRU管理
>
>config_file:/opt/app/redisCluster/8029/./redis_cluster/8029/redis-8029.conf --Redis的配置文件
>
>
>
>> info clients
>
># Clients
>
>connected_clients:1375 --当前客户端连接数
>
>client_longest_output_list:0 --当前所有缓冲区中队列对象个数的最大值
>
>client_biggest_input_buf:2 --当前所有输入缓冲区中占用的最大容量
>
>blocked_clients:0 --正在等待阻塞命令(例如BLPOP等)的客户端数量
>
>
>> info memory
>
># Memory
>
>used_memory:3349912688 --Redis分配器分配的内存总量，也就是内部存储的所有数据内存占有量
>
>used_memory_human:3.12G  --以可读的格式返回used_memory
>
>used_memory_rss:6814093312 --从操作系统的角度，Redis进程占用的物理内存总量
>
>used_memory_peak:5727005064 --内存使用的最大值，表示used_memory的峰值
>
>used_memory_peak_human:5.33G --以可读的格式返回used_memory_peak
>
>used_memory_lua:36864 --Lua引擎所消耗的内存大小
>
>mem_fragmentation_ratio:2.03 --used_memory_rss/used_memory比值，表示内存碎片率
>
>mem_allocator:jemalloc-3.6.0 --Redis所使用的内存分配器。默认为:jemalloc
>
>
>> info persistence
>
># Persistence
>
>loading:0  --是否在加载持久化文件。0否，1是
>
>rdb_changes_since_last_save:1224395935 --自上次RDB，Redis数据改动条数
>
>rdb_bgsave_in_progress:0 --标识RDB的bgsave操作是否进行中。0否，1是
>
>rdb_last_save_time:1493145644 --上次bgsave操作的时间戳
>
>rdb_last_bgsave_status:ok --上次bgsave操作状态
>
>rdb_last_bgsave_time_sec:29 --上次bgsave使用的时间
>
>rdb_current_bgsave_time_sec:-1 --如何bgsave操作正在进行，则记录当前bgsave操作使用的时间(单位是秒)
>
>aof_enabled:0 --是否开启了AOF功能。0否，1是
>
>aof_rewrite_in_progress:0  
>
>aof_rewrite_scheduled:0 --标识是否将要在RDB的bgsave操作结束后执行AOF rewrite操作 
>
>aof_last_rewrite_time_sec:-1 --上次AOF rewrite操作使用的时间(单位是秒) 
>
>aof_current_rewrite_time_sec:-1 --如果rewrite操作正在进行，则记录当前AOF rewrite所使用的时间(单位是秒)
>
>aof_last_bgrewrite_status:ok --上次AOF重写操作的状态
>
>aof_last_write_status:ok --上次AOF写磁盘的结果
>
>
>
>> info stats
>
># Stats
>
>total_connections_received:4393241 --连接过的客户端总数
>
>total_commands_processed:1756721337 --执行过的命令总数
>
>instantaneous_ops_per_sec:13 --每秒处理命令条数
>
>total_net_input_bytes:636866047148 --输入总网络流量(以字节为单位)
>
>total_net_output_bytes:1558439303474 --输出总网络流量(以字节为单位)
>
>instantaneous_input_kbps:5.42 --每秒输入字节数
>
>instantaneous_output_kbps:9.89 --每秒输出字节数
>
>rejected_connections:0 --拒绝的连接个数
>
>sync_full:2 --主从完全同步成功次数
>
>sync_partial_ok:3 --主从部分同步成功次数
>
>sync_partial_err:0 --主从部分同步失败次数
>
>expired_keys:64979568 --过期的key数量
>
>evicted_keys:0 --剔除(超过了maxmemory后)的key数量
>
>keyspace_hits:8573735 --命中次数
>
>keyspace_misses:19578062 --不命中次数
>
>pubsub_channels:1 --当前使用中的频道数量
>
>pubsub_patterns:0 --当前使用中的模式数量
>
>latest_fork_usec:63895 --最近一次fork操作消耗的时间(微秒)
>
>migrate_cached_sockets:0 --记录当前Redis正在进行migrate操作的目标Redis个数。例如RedisA 分别向Redis B和C执行migrate操作，那么这个值就是2
>
>
>
>> info replication
>
># Replication
>
>role:master
>
>connected_slaves:2
>
>slave0:ip=172.16.35.76,port=8028,state=online,offset=597361953364,lag=0
>
>slave1:ip=172.16.35.218,port=8010,state=online,offset=597361951155,lag=1
>
>master_repl_offset:597361953577 --主节点偏移量
>
>repl_backlog_active:1 --复制缓冲区状态
>
>repl_backlog_size:67108864 --复制缓冲区尺寸(单位:字节)
>
>repl_backlog_first_byte_offset:597294844714 --复制缓冲区起始偏移量，标识当前缓冲区可用范围
>
>repl_backlog_histlen:67108864 --标识复制缓冲区已存有效数据长度
>
>
>> info cpu
>
># CPU
>
>used_cpu_sys:91573.59 --Redis主进程在内核态所占用的CPU时钟总和
>
>used_cpu_user:129416.68 --Redis主进程在用户态所占用的CPU时钟总和
>
>used_cpu_sys_children:5.00 --Redis子进程在内核态所占用的CPU时钟总和
>
>used_cpu_user_children:17.58 --Redis子进程在用户态所占用的CPU时钟总和
>
>
>> info Commandstats
>
># Commandstats
>
>cmdstat_get:calls=2,usec=14,usec_per_call=7.00 --get 命令调用总次数，总耗时，平均耗时(单位是毫秒)
>
>cmdstat_set:calls=167609,usec=5659321,usec_per_call=33.77 --set 命令总次数，总耗时，平均耗时(单位是毫秒)
>
>
>> info cluster
>
># Cluster
>
>cluster_enabled:1 --节点是否为cluster模式 。1是0否
>
>
>> info keyspace
>
># Keyspace
>
>db0:keys=4868218,expires=1265463,avg_ttl=265690817 --当前数据库key总数，带有过期时间的key总数，平均存活时间
>
>---------------------
>
>

#### Redis-stat监控工具启动命令：

java  -jar  redis-stat-0.4.14.jar 120.26.37.196:7000 120.26.37.196:7001 120.26.37.196:7002 121.43.195.127:7006 121.43.195.127:7007 121.43.195.127:7008 --server=8088 ----daemon

nohup java  -jar  redis-stat-0.4.14.jar 120.26.37.196:7000 120.26.37.196:7001 120.26.37.196:7002 121.43.195.127:7006 121.43.195.127:7007 121.43.195.127:7008 --server=8088  10 >/mnt/redis_data/redis-stat.txt 



nohup java  -jar  redis-stat-0.4.14.jar 10.51.6.239:7000 10.169.9.100:7000 10.168.2.116:7000 10.117.55.233:7000 10.51.28.128:7000 172.16.3.117:7000 --server=8089  10 >/mnt/redis_data/redis-stat.txt 



判断一个值是否在set中

~~~shell
sismember  a69e0502ebef4a32823e6b4f2371ae28_fans 80c8cfe24cb14e6b85e38021faa38c51
~~~



设置redis 的hz

~~~shell

10.168.2.116:7000> info stats
# Stats
total_connections_received:62987500
total_commands_processed:526420135
instantaneous_ops_per_sec:2
total_net_input_bytes:25753076841
total_net_output_bytes:22739267638
instantaneous_input_kbps:0.09
instantaneous_output_kbps:0.01
rejected_connections:0
sync_full:3
sync_partial_ok:2
sync_partial_err:0
expired_keys:645765
evicted_keys:0
keyspace_hits:143182888
keyspace_misses:46785700
pubsub_channels:2
pubsub_patterns:0
latest_fork_usec:4047
migrate_cached_sockets:0
~~~

