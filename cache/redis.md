1.为什么用缓存，用过哪些缓存，redis和memcache的区别
答：为什么要使用缓存：为了提高响应速度，直接把热点内容放到内存缓存，提高响应速度。
      用过哪些缓存：用过redis，memcached是初步使用和了解过。
      redis和memcached的区别：
      相同点：
      都是将数据放在内存中，都是内存数据库。都是key-value的形式存放数据。
      不同点：
      1）、redis不仅仅支持简单的字符串键值对，redis还提供了一系列的数据结构类型，如列表（list）、集合（set）、有序集合（zset）、哈希（hash）。
      2）、从存储安全来说，memcached宕机后，数据就丢失了，不能恢复，而redis是可以根据aof或者rdb的方式持久化，数据可以恢复。redis支持持久化，rdb和aof持久化。
      3）、redis支持数据备份，主从（master-slave）模式备份数据。
      4）、redis支持事务等
      5）、redis支持发布订阅

2.redis的数据结构
答：redis的value的数据结构有：string、列表List、集合Set、有序集合ZSet、哈希。
redis命令参考：
一、键（key）：
1）、del key [key...]    删除给定的一个或多个key
2）、exists key    判断key是否存在
3）、expire key seconds    为给定key设置生存时间
4）、keys pattern    查找所有给定模式的key
5）、move key db    将当前数据库的key移动到给定数据库
6）、persist key    移除给定key的生存时间
7）、pexpire key milliseconds    毫秒为单位设置生存时间
8）、pttl key    以毫秒为单位返回 key 的剩余生存时间
9）、randomkey    数据库随机返回一个key
10）、rename key newkey    key改名
11）、ttl key    以秒为单位返回 key 的剩余生存时间
12）、type key    返回key所存储的值的类型

二、字符串（string）：
1）、decr key    将key中存储的数字减一
2）、decrby key    decrement    将key中存储的数字减去decrement
3）、get key    返回key所关联的字符串
4）、getrange key start end    返回key中所存储的子字符串
5）、getset key value    将给定key的值设为value，并返回key的旧值
6）、incr key    将key中存储的数字加一
7）、incrby key decrement    将key中存储的数字加上decrement
8）、mget key [key...]    返回所有（一个或多个）给定key的值
9）、mset key [key value...]    同时设置一个或多个key-value对
10）、strlen key    返回key所储存的字符串值的长度

总结：Redis的字符串表示为sds（simple dynamic string），而不是C字符串（以\0结尾的char*）

对比C字符串，sds有以下特性：

-可以高效地执行长度计算（strlen）

-可以高效地执行追加操作（append）

sds会为追加操作进行优化：加快追加操作的速度，并降低内存分配的次数，代价是多占用了一些内存，而且这些内存不会主动释放

三、列表（list）：有序，可重复，双端链表实现，当列表不大时，查询两端元素较快，如果列表大查询中间元素时较慢
1）、rpush key value [value...]    将一个或多个值插入到列表key的表尾
2）、rpushx key    value    将值value插入到列表key的表尾，当且仅当key存在并且是一个列表
3）、lpop key    移除并返回列表key的头元素
4）、lrange key start stop    返回列表key中指定区间内的元素
5）、llen key    返回列表key的长度
6）、lpush key value [value...]    将一个或多个值插入到列表key的表头
7）、lpushx key value    将值value插入到列表key的表头，当且仅当key存在并且是一个列表
8）、rpop key    移除并返回列表key的尾元素

四、集合（set）：无序，不可重复
1）、sadd key member [member...]    将一个或多个member加入到集合key中，已经存在于集合的member元素将被忽略
2）、scard key    返回集合中元素的数量
3）、sdiff key1 [key2...]    返回给定所有集合的差集
4）、sdiff destination key1 [key2...]    返回给定所有集合的差集，并且把差集保留到destination集合
5）、sinter key1 [key2...]    返回给定所有集合的交集
6）、sinterstore destinatin key [key2...]    返回给定所有集合的交集，并且把交集保留到destination集合
7）、sismember key member    判断member元素是否是集合key的成员
8）、smembers key    返回集合key中的所有成员
9）、spop key    移除并返回集合中的一个随机元素
10）、srem key member [member...]    移除集合key中的一个或多个member元素，不存在的member元素会被忽略
11）、sunion key1 [key2...]    返回给定所有集合的并集
12）、sunionstore destination key1 [key2...]    返回给定所有集合的并集，并且把并集保留到destination集合

五、有序集合（zset）：
1）、zadd key score member [[score member]...]    将一个或多个member元素及其score值加入到有序集key当中
2）、zcard key    返回有序集合中元素的数量
3）、zcount key min max    返回有序集key中，score值在min和max之间元素的数量
4）、zrange key star stop
六、哈希表（hash）：
1）、hset key field value    将哈希表key中的域field的值设为value
2）、hmset key field value [[field value]...]    同时将多个field-value设置到哈希表key中
3）、hlen key    返回哈希表key中域的数量
4）、hget key field    返回哈希表key中给定域的值
5）、hmget key field [field...]    返回哈希表key中，一个或多个给定域的值
6）、hgetall key    返回哈希表key中，所有的域和值
7）、hdel key field [field...]    删除哈希表key中的一个或多个指定域
8）、hexists key field    查看哈希表key中，给定域field是否存在
9）、hkeys key    返回哈希表key中的所有域

七、事务：
1）multi：multi命令用于开启一个事务，multi执行后，客户端可以继续向服务器发送任意多条命令， 这些命令不会立即执行，而是被放到一个队列中。
2）exec：exec命令负责触发并执行事务中的所有命令。
3）discard：放弃事务
4）watch
redis是不支持事务回滚的。

3.redis的持久化方式，以及项目中用的哪种，为什么？
答：redis持久化是在redis宕机或重启后仍然可用，redis分别提供了rdb和aof两种持久化模式。
两种持久化方式比较：
rdb：
     在redis运行时，rdb程序在指定的时间间隔将当前内存中的数据库生成快照保存到磁盘中， 在redis重新启动时，rdb程序可用通过载入rdb文件来还原数据库的状态。rdb功能最核心的是rdbSave和rdbLoad两个函数，前者用于生成RDB文件到磁盘，而后者则用于 将RDB文件中的数据重新载入到内存中。
aof（append only file）：
     记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行 这些命令来还原数据集。aof文件中的命令全部以redis协议的格式来保存，新命令会被追加到 文件的末尾。
rdb的优点：
     rdb文件是一个非常紧凑的文件，它保存了redis在某个时间点上的数据集。这种文件非常适合用于 备份：比如每一天或每一月的某一天备份一个rdb文件。
     rdb可以最大化redis的性能，父进程在保存rdb文件唯一要做的就是fork出一个子进程，然后这个子 进程负责处理接下来的保存操作。
     rdb在恢复大数据集时的速度比aof的恢复速度要快。
rdb的缺点：
     如果你需要尽量避免在服务器故障时丢失数据，那么rdb不适合你。因为你设置rdb持久化时，可以设置 多久的间隔生成快照，这意味着，你可能在这间隔内丢失数据。
     因为我们的项目对耐久性不是要求特别高，我们选用的是rdb方式持久化，设置每隔30分钟备份一次rdb文件。

4.redis集群的理解，怎么动态增加或者删除一个节点，而保证数据不丢失。（一致性哈希问题）
答：一致性哈希算法主要使用在分布式数据存储系统中，按照一定的策略将数据尽可能均匀分到所有的存储节点
上去，使得系统具有良好的负载均衡性能和扩展性。