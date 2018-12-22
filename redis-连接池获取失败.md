https://neway6655.github.io/redis/2015/12/19/善待Redis里的数据.html



~~~shell
redis-cli> monitor
1503511180.523051 [0 10.99.122.148:54946] "EXISTS" "someCacheName~lock"
1503511180.523440 [0 10.99.122.149:53998] "EXISTS" "someCacheName~lock"
1503511180.525419 [0 10.99.122.147:48400] "EXISTS" "someCacheName~lock"
1503511180.527089 [0 10.99.122.148:55252] "EXISTS" "someCacheName~lock"
1503511180.527229 [0 10.99.122.148:55376] "EXISTS" "someCacheName~lock"
1503511180.527389 [0 10.99.122.148:54236] "EXISTS" "someCacheName~lock"
1503511180.527463 [0 10.99.122.149:55332] "EXISTS" "someCacheName~lock"
1503511180.540654 [0 10.99.122.149:55168] "EXISTS" "someCacheName~lock"
1503511180.546141 [0 10.99.122.149:53064] "EXISTS" "someCacheName~lock"
1503511180.546407 [0 10.99.122.149:53172] "EXISTS" "someCacheName~lock"
1503511180.547578 [0 10.99.122.146:38110] "EXISTS" "someCacheName~lock"
1503511180.547670 [0 10.99.122.146:37834] "EXISTS" "someCacheName~lock"
1503511180.547820 [0 10.99.122.146:38398] "EXISTS" "someCacheName~lock"
1503511180.553981 [0 10.99.122.149:54374] "EXISTS" "someCacheName~lock"
~~~

# redis-config-hz

https://idning.github.io/redis-config-hz.html



https://blog.csdn.net/liuxiao723846/article/details/78089577