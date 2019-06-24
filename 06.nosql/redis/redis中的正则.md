## 线上Redis禁止使用Keys正则匹配操作！
##3 危险的命令：
flushdb 命令用于清空当前数据库中的所有key

flushall 命令用于清空整个 redis 服务的数据(删除所有的key)

config 客户端连接后可配置服务器

### 怎么禁用这些命令：
redis.conf中，在security新增如下命令：
```shll
rename-command flushall ""
rename-command flushdb ""
rename-command config ""
rename-command keys ""
```
>另外，对于FLUSHALL命令，需要设置配置文件中appendonly no，否则服务器是无法启动。
上面的这些命令可能有遗漏，大家可以查官方文档。除了Flushdb这类和redis安全隐患有关的命令意外，但凡发现时间复杂度为O(N)的命令，都要慎重，不要在生产上随便使用。例如hgetall、lrange、smembers、zrange、sinter等命令，它们并非不能使用，但这些命令的时间复杂度都为O(N)，使用这些命令需要明确N的值，否则也会出现缓存宕机。

### SCAN命令
业内建议使用scan命令来改良keys和SMEMBERS命令：

Redis2.8版本以后有了一个新命令scan，可以用来分批次扫描redis记录，这样肯定会导致整个查询消耗的总时间变大，但不会影响redis服务卡顿，影响服务使用。

具体使用，大家详情可以自己查阅下面这份文档：

http://doc.redisfans.com/key/scan.html

