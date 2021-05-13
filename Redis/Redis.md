## 1. Redis是什么？**简述它的优缺点？**

﻿Redis本质上是一个Key-Value类型的内存数据库，很像memcached，整个数据库统统加载在内存当中进行操作，定期通过异步操作把数据库数据flush到硬盘上进行保存。

﻿因为是纯内存操作，Redis的性能非常出色，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。

﻿Redis的出色之处不仅仅是性能，Redis最大的魅力是支持保存多种数据结构，此外单个value的最大限制是1GB，不像 memcached只能保存1MB的数据，因此Redis可以用来实现很多有用的功能。

﻿比方说用他的List来做FIFO双向链表，实现一个轻量级的高性 能消息队列服务，用他的Set可以做高性能的tag系统等等。

﻿另外Redis也可以对存入的Key-Value设置expire时间，因此也可以被当作一 个功能加强版的memcached来用。 Redis的主要缺点是数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上。

## 2. Redis为什么这么快？

1、完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；

2、数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的；

3、采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

4、使用多路I/O复用模型，非阻塞IO；

5、使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；

## 3. **Redis相比memcached有哪些优势？**

* redis支持更丰富的数据类型（支持更复杂的应用场景）：Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，zset，hash等数据结构的存储。memcache支持简单的数据类型，String。
* Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而Memecache把数据全部存在内存之中。
* 集群模式：memcached没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 redis 目前是原生支持 cluster 模式的.
* Memcached是多线程，非阻塞IO复用的网络模型；Redis使用单线程的多路 IO 复用模型。

## 4. Redis 的数据类型？

Redis 支持五种数据类型： string（ 字符串），hash（ 哈希）， list（ 列表）， set（ 集合） 及 zsetsorted set： 有序集合)。

- string：redis 中字符串 value 最大可为512M。可以用来做一些计数功能的缓存（也是实际工作中最常见的）。 常规计数：微博数，粉丝数等。
- list：简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部（左边）或者尾部（右边），Redis list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。可以实现一个简单消息队列功能，做基于redis的分页功能等。(另外可以通过 lrange 命令，就是从某个元素开始读取多少个元素，可以基于 list 实现分页查询，这个很棒的一个功能，基于 redis 实现简单的高性能分页，可以做类似微博那种下拉不断分页的东西（一页一页的往下走），性能高。)
- set：是一个字符串类型的无序集合。可以用来进行全局去重等。(比如：在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程)
- sorted set：是一个字符串类型的有序集合，给每一个元素一个固定的分数score来保持顺序。可以用来做排行榜应用或者进行范围查找等。
- hash：键值对集合，是一个字符串类型的 Key和 Value 的映射表，也就是说其存储的Value是一个键值对（Key- Value）hash 特别适合用于存储对象，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。

我们实际项目中比较常用的是 string，hash。 如果你是 Redis 的高级用户，还需要加上下面几种数据结构 HyperLogLog、Geo、Pub/Sub。

如果你说还玩过 Redis Module，像 BloomFilter，RedisSearch，Redis-ML，面试官得眼睛就开始发亮了。

## 5. Redis的常用场景?

1、会话缓存（ Session Cache）

最常用的一种使用 Redis 的情景是会话缓存（ session cache）。用 Redis 缓存会话比其他存储（ 如 Memcached）的优势在于：Redis 提供持久化。当维护一个不是严格要求一致性的缓存时， 如果用户的购物车信息全部丢失， 大部分人都会不高兴的， 现在， 他们还会这样吗？ 幸运的是， 随着 Redis 这些年的改进， 很容易找到怎么恰当的使用 Redis 来缓存会话的文档。甚至广为人知的商业平台Magento 也提供 Redis 的插件。

2、全页缓存（ FPC）

除基本的会话 token 之外， Redis 还提供很简便的 FPC 平台。回到一致性问题， 即使重启了 Redis 实例， 因为有磁盘的持久化， 用户也不会看到页面加载速度的下降，这是一个极大改进，类似 PHP 本地 FPC。 再次以 Magento 为例，Magento 提供一个插件来使用 Redis 作为全页缓存后端。 此外， 对 WordPress 的用户来说， Pantheon 有一个非常好的插件 wp-redis， 这个插件能帮助你以最快速度加载你曾浏览过的页面。

3、队列

Reids 在内存存储引擎领域的一大优点是提供 list 和 set 操作， 这使得 Redis 能作为一个很好的消息队列平台来使用。Redis 作为队列使用的操作，就类似于本地程序语言（ 如 Python）对 list 的 push/pop 操作。 如果你快速的在 Google 中搜索“ Redis queues”， 你马上就能找到大量的开源项目， 这些项目的目的就是利用 Redis 创建非常好的后端工具， 以满足各种队列需求。例如， Celery 有一个后台就是使用 Redis 作为 broker， 你可以从这里去查看。

4， 排行榜/计数器

Redis 在内存中对数字进行递增或递减的操作实现的非常好。集合（ Set） 和有序集合（ Sorted Set）也使得我们在执行这些操作的时候变的非常简单，Redis 只是正好提供了这两种数据结构。所以， 我们要从排序集合中获取到排名最靠前的 10 个用户– 我们称之为“ user_scores”， 我们只需要像下面一样执行即可：  当然，这是假定你是根据你用户的分数做递增的排序。如果你想返回用户及用户的分数，  你需要这样执行： ZRANGE user_scores 0 10 WITHSCORES Agora Games 就是一个很好的例子， 用 Ruby 实现的， 它的排行榜就是使用 Redis 来存储数据的， 你可以在这里看到。

5、发布/订阅

发布/订阅的使用场景确实非常多。我已看见人们在社交网络连接中使用， 还可作为基于发布/订阅的脚本触发器， 甚至用 Redis 的发布/订阅功能来建立聊天系统！

## 6. redis 过期键的删除策略？

1、定时删除:在设置键的过期时间的同时，创建一个定时器 timer). 让定时器在键的过期时间来临时， 立即执行对键的删除操作。

2、惰性删除:放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是  否过期， 如果过期的话， 就删除该键;如果没有过期， 就返回该键。

3、定期删除:每隔一段时间程序就对数据库进行一次检查，删除里面的过期键。至 于要删除多少过期键， 以及要检查多少个数据库， 则由算法决定。

## 7. redis 内存淘汰机制？

redis v4.0前提供 6种数据淘汰策略：

- volatile-lru：利用LRU算法移除设置过过期时间的key (LRU:最近使用 Least Recently Used )
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）
- volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
- volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
- no-eviction：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！ 

**redis v4.0后增加以下两种**：

- volatile-lfu：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰(LFU(Least Frequently Used)算法，也就是最频繁被访问的数据将来最有可能被访问到)
- allkeys-lfu：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的key。

## 8. redis 持久化机制

为了能够重用`Redis`数据，或者防止系统故障，我们需要将`Redis`中的数据写入到磁盘空间中，即持久化。`Redis`提供了两种不同的持久化方法可以将数据存储在磁盘中，一种叫快照`RDB`，另一种叫只追加文件`AOF`

### RDB

在指定的时间间隔内将内存中的数据集快照写入磁盘(`Snapshot`)，它恢复时是将快照文件直接读到内存里。

 `Redis`会单独创建（`fork`）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那`RDB`方式要比`AOF`方式更加的高效。`RDB`的缺点是最后一次持久化后的数据可能丢失。

`Fork`的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等）数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程。**`Rdb` 保存的是`dump.rdb`文件。**

**优势**

适合大规模的数据恢复
对数据完整性和一致性要求不高

**劣势**

在一定间隔时间做一次备份，所以如果`redis`意外`down`掉的话，就会丢失最后一次快照后的所有修改。

### AOF(Append Only File)

以日志的形式来记录每个写操作，将`Redis`执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，`redis`启动之初会读取该文件重新构建数据，换言之，`redis`重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。**AOF保存的是appendonly.aof文件**

#### AOF重写

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时， Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof。

AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据， 每条记录有一条的set语句。重写aof文件的操作，并没有读取旧的aof文件， 而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

**优势**

- 每修改同步：`appendfsync always` 同步持久化,每次发生数据变更会被立即记录到磁盘,性能较差但数据完整性比较好
- 每秒同步：`appendfsync everysec` 异步操作，每秒记录,如果一秒内宕机，有数据丢失
- 不同步：`appendfsync no`   从不同步

**劣势**

- 相同数据集的数据而言`aof`文件要远大于`rdb`文件，恢复速度慢于`rdb`
- `aof`运行效率要慢于`rdb`,每秒同步策略效率较好，不同步效率和`rdb`相同

### Redis 4.0 对于持久化机制的优化

redis4.0相对与3.X版本其中一个比较大的变化是4.0添加了新的混合持久化方式。

混合持久化同样也是通过bgrewriteaof完成的，不同的是当开启混合持久化时，fork出的子进程先将共享的内存副本全量的以RDB方式写入aof文件，然后在将重写缓冲区的增量命令以AOF方式写入到文件，写入完成后通知主进程更新统计信息，并将新的含有RDB格式和AOF格式的AOF文件替换旧的的AOF文件。简单的说：新的AOF文件前半段是RDB格式的全量数据后半段是AOF格式的增量数据，如下图：

![](https://images2018.cnblogs.com/blog/1075473/201807/1075473-20180726181756270-1907770368.png)

**优点**

混合持久化结合了RDB持久化 和 AOF 持久化的优点, 由于绝大部分都是RDB格式，加载速度快，同时结合AOF，增量的数据以AOF方式保存了，数据更少的丢失。

**缺点**

兼容性差，一旦开启了混合持久化，在4.0之前版本都不识别该aof文件，同时由于前部分是RDB格式，阅读性较差。

## 9. 缓存雪崩、缓存击穿和缓存穿透问题解决方案

### 缓存击穿

举例：redis中存储的是热点数据，当高并发请求访问redis中热点数据的时候，如果redis中的数据过期了，会造成缓存击穿的现象，请求都打到了数据库上。

解决办法：使用互斥锁，只让一个请求去load DB，成功之后重新写缓存，其余请求没有获取到互斥锁，可以尝试重新获取缓存中的数据。。

### 缓存穿透

缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上(举例：故意的去请求缓存中不存在的数据，导致请求都打到了数据库上，导致数据库异常。)

解决办法

1.首先做好参数校验，一些不合法的参数请求直接抛出异常信息返回给客户端。比如查询的数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等 
2.缓存无效 key :如果缓存和数据库都查不到某个 key 的数据就写一个到 redis 中去并设置过期时间,尽量将无效的 key 的过期时间设置短一点比如 1 分钟。 
3.布隆过滤器.把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，我会先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程 

![](http://blog-img.coolsen.cn/img/1584173117136_7.png)

### 缓存雪崩

缓存同一时间大面积的失效，所以，后面的请求都会落到数据库上，造成数据库短时间内承受大量请求而崩掉

- 事前：尽量保证整个 redis 集群的高可用性，发现机器宕机尽快补上。选择合适的内存淘汰策略。
- 事中：本地ehcache缓存 + hystrix限流&降级，避免MySQL崩掉
- 事后：利用 redis 持久化机制保存的数据尽快恢复缓存 

![](http://blog-img.coolsen.cn/img/1584172719093_6.png)



## 10. 为什么 Redis 不支持回滚（roll back）

- Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 [INCR key](http://redisdoc.com/string/incr.html#incr) 命令将键的值加上 `1` ， 却不小心加上了 `2` ， 又或者对错误类型的键执行了 [INCR key](http://redisdoc.com/string/incr.html#incr) ， 回滚是没有办法处理这些情况的。

鉴于没有任何机制能避免程序员自己造成的错误， 并且这类错误通常不会在生产环境中出现， 所以 Redis 选择了更简单、更快速的无回滚方式来处理事务。

## 11. Redis中跳跃表(skiplist)实现原理

跳表（skiplist）是一个特殊的链表，相比一般的链表，有更高的查找效率，其效率可比拟于二叉查找树。

### 跳跃表来源

跳跃表在 1990 年由 William Pugh 提出，而红黑树早在 1972 年由鲁道夫·贝尔发明了。红黑树在空间和时间效率上略胜跳表一筹，但跳跃表实现相对简单得到程序猿们的青睐。Redis 和 Leveldb 中都有采用跳跃表。

以下是个典型的跳跃表例子

<img src="C:\Users\mashuaisen\OneDrive\typora-user-images\Redis数据结构-跳跃表\image-20200714130333639.png" alt="image-20200714130333639" style="zoom:80%;" /> 

按照上面生成链表的方式，上面每一层链表的节点个数，是下面一层的节点个数的一半，这样查找过程就非常类似于一个二分查找，使得查找的时间复杂度可以降低到O(log n)。

但是，这种方法在插入数据的时候有很大的问题。新插入一个节点之后，就会打乱上下相邻两层链表上节点个数严格的2:1的对应关系。如果要维持这种对应关系，就必须把新插入的节点后面的所有节点（也包括新插入的节点）重新进行调整，这会让时间复杂度重新蜕化成O(n)。删除数据也有同样的问题。

下面看Redis 跳跃表的实现，如何解决的这个问题。

### Redis 跳跃表的实现

为了满足自身的功能需要， Redis 基于 William Pugh 论文中描述的跳跃表进行了以下修改：

1. 允许重复的 `score` 值：多个不同的 `member` 的 `score` 值可以相同。
2. 进行对比操作时，不仅要检查 `score` 值，还要检查 `member` ：当 `score` 值可以重复时，单靠 `score` 值无法判断一个元素的身份，所以需要连 `member` 域都一并检查才行。
3. 每个节点都带有一个高度为 1 层的后退指针，用于从表尾方向向表头方向迭代：当执行 [ZREVRANGE](http://redis.readthedocs.org/en/latest/sorted_set/zrevrange.html#zrevrange) 或 [ZREVRANGEBYSCORE](http://redis.readthedocs.org/en/latest/sorted_set/zrevrangebyscore.html#zrevrangebyscore) 这类以逆序处理有序集的命令时，就会用到这个属性。

跳跃表的结构定义：

```c
typedef struct zskiplist {

    // 头节点，尾节点
    struct zskiplistNode *header, *tail;

    // 节点数量
    unsigned long length;

    // 目前表内节点的最大层数
    int level;

} zskiplist;
```

跳跃表的节点定义：

```c
typedef struct zskiplistNode {

    // member 对象
    robj *obj;

    // 分值
    double score;

    // 后退指针
    struct zskiplistNode *backward;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 这个层跨越的节点数量
        unsigned int span;

    } level[];

} zskiplistNode;
```

<img src="C:\Users\mashuaisen\OneDrive\typora-user-images\Redis数据结构-跳跃表\image-20200714131425529.png" alt="image-20200714131425529" style="zoom:80%;" />

上图就是跳跃列表的示意图，图中只画了3层，Redis 的跳跃表共有 64 层，容纳 2^64 个元素应该不成问题。

跳表的性质：

1. 由很多层结构组成
2. 每一层都是一个有序的链表
3. 最底层(Level 1) 的链表包含所有元素
4. 如果一个元素出现在 Level i 的链表中，则它在 Level i 之下的链表也都会出现。
5. 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素。

### 随机层数的设计

Redis 使用随机层数，解决插入、删除时，时间复杂度重新蜕化成O(n)的问题

它不要求上下相邻两层链表之间的节点个数有严格的对应关系，而是为每个节点随机出一个层数(level)。比如，一个节点随机出的层数是3，那么就把它链入到第1层到第3层这三层链表中。

<img src="C:\Users\mashuaisen\OneDrive\typora-user-images\Redis数据结构-跳跃表\image-20200714140215699.png" alt="image-20200714140215699" style="zoom: 67%;" />

插入操作只需要修改插入节点前后的指针，而不需要对很多节点都进行调整。这就降低了插入操作的复杂度，这让它在插入性能上明显优于平衡树。

#### 随机层数的计算方式

执行插入操作时计算随机数的过程，是一个很关键的过程，它对skiplist的统计特性有着很重要的影响。这并不是一个普通的服从均匀分布的随机数，它的计算过程如下：

- 首先，每个节点肯定都有第1层指针（每个节点都在第1层链表里）。
- 如果一个节点有第i层(i>=1)指针（即节点已经在第1层到第i层链表中），那么它有第(i+1)层指针的概率为p。
- 节点最大的层数不允许超过一个最大值，记为MaxLevel。

这个计算随机层数的伪码如下所示：

```
randomLevel()
    level := 1
    // random()返回一个[0...1)的随机数
    while random() < p and level < MaxLevel do
        level := level + 1
    return level复制代码
```

randomLevel()的伪码中包含两个参数，一个是p，一个是MaxLevel。在Redis的skiplist实现中，这两个参数的取值为：

```
p = 1/4
MaxLevel = 32
```

#### skiplist的算法性能分析

根据前面randomLevel()的伪码，我们很容易看出，产生越高的节点层数，概率越低。

当skiplist中有n个节点的时候，它的总层数的概率均值是多少。这个问题直观上比较好理解。根据节点的层数随机算法，容易得出：

- 第1层链表固定有n个节点；
- 第2层链表平均有n*p个节点；
- 第3层链表平均有n*p2个节点；

计算很复杂，没有看懂，总结来说：平均时间复杂度为O(log n)

### rank排名计算

图中前向指针上面括号中的数字，表示对应的span的值。即当前指针跨越了多少个节点，这个计数不包括指针的起点节点，但包括指针的终点节点。

<img src="C:\Users\mashuaisen\OneDrive\typora-user-images\Redis数据结构-跳跃表\69c7c2284f89003cc7278e2ccdc468ba" alt="img" style="zoom:67%;" />

举例：

在这个skiplist中查找score=89.0的元素（即Bob的成绩数据），在查找路径中，我们会跨域图中标红的指针，这些指针上面的span值累加起来，就得到了Bob的排名(2+2+1)-1=4（减1是因为rank值以0起始）。需要注意这里算的是从小到大的排名，而如果要算从大到小的排名，只需要用skiplist长度减去查找路径上的span累加值，即6-(2+2+1)=1。

通过这种方式就能得到一条O(log n)的查找路径

## 12. Redis为什么用skiplist(跳跃表)而不用平衡树？

Redis的作者 @antirez 从内存占用、对范围查找的支持和实现难易程度做了解释。[原文](https://news.ycombinator.com/item?id=1171423)：

> There are a few reasons:
>
> 1) They are not very memory intensive. It's up to you basically. Changing parameters about the probability of a node to have a given number of levels will make then less memory intensive than btrees.
>
> 2) A sorted set is often target of many ZRANGE or ZREVRANGE operations, that is, traversing the skip list as a linked list. With this operation the cache locality of skip lists is at least as good as with other kind of balanced trees.
>
> 3) They are simpler to implement, debug, and so forth. For instance thanks to the skip list simplicity I received a patch (already in Redis master) with augmented skip lists implementing ZRANK in O(log(N)). It required little changes to the code.

总结来说：

* 不需要太多内存
* 范围查询和平衡树一样好
* 容易实现和调试

## 13. 假如 Redis 里面有 1 亿个key，其中有 10w 个key 是以某个固定的已知的前缀开头的，如果将它们全部找出来？

使用 keys 指令可以扫出指定模式的 key 列表。

追问： 如果这个 redis 正在给线上的业务提供服务， 那使用 keys 指令会有什么问题？

这个时候你要回答 redis 关键的一个特性：redis 的单线程的。keys 指令会导致线程阻塞一段时间， 线上服务会停顿， 直到指令执行完毕， 服务才能恢复。这个时候可以使用 scan 指令， scan 指令可以无阻塞的提取出指定模式的 key 列表， 但是会有一定的重复概率， 在客户端做一次去重就可以了， 但是整体所花费的时间会比直接用 keys 指令长。

## 14. **Redis6.0为什么要引入多线程呢？**

Redis将所有数据放在内存中，内存的响应时长大约为100纳秒，对于小数据包，Redis服务器可以处理80,000到100,000 QPS，这也是Redis处理的极限了，对于80%的公司来说，单线程的Redis已经足够使用了。

但随着越来越复杂的业务场景，有些公司动不动就上亿的交易量，因此需要更大的QPS。常见的解决方案是在分布式架构中对数据进行分区并采用多个服务器，但该方案有非常大的缺点，例如要管理的Redis服务器太多，维护代价大；某些适用于单个Redis服务器的命令不适用于数据分区；数据分区无法解决热点读/写问题；数据偏斜，重新分配和放大/缩小变得更加复杂等等。

从Redis自身角度来说，因为读写网络的read/write系统调用占用了Redis执行期间大部分CPU时间，瓶颈主要在于网络的 IO 消耗, 优化主要有两个方向:

• 提高网络 IO 性能，典型的实现比如使用 DPDK 来替代内核网络栈的方式 • 使用多线程充分利用多核，典型的实现比如 Memcached。

协议栈优化的这种方式跟 Redis 关系不大，支持多线程是一种最有效最便捷的操作方式。所以总结起来，redis支持多线程主要就是两个原因：

• 可以充分利用服务器 CPU 资源，目前主线程只能利用一个核 • 多线程任务可以分摊 Redis 同步 IO 读写负荷

![img](https://obs-emcsapp-public.obs.cn-north-4.myhwclouds.com/wechatSpider/modb_20200720_110226.png)

**Redis6.0采用多线程后，性能的提升效果如何？**

Redis 作者 antirez 在 RedisConf 2019分享时曾提到：Redis 6 引入的多线程 IO 特性对性能提升至少是一倍以上。国内也有大牛曾使用unstable版本在阿里云esc进行过测试，GET/SET 命令在4线程 IO时性能相比单线程是几乎是翻倍了。

### 实现机制

**流程简述如下**：

1、主线程负责接收建立连接请求，获取 socket 放入全局等待读处理队列 

2、主线程处理完读事件之后，通过 RR(Round Robin) 将这些连接分配给这些 IO 线程 

3、主线程阻塞等待 IO 线程读取 socket 完毕 

4、主线程通过单线程的方式执行请求命令，请求数据读取并解析完成，但并不执行 

5、主线程阻塞等待 IO 线程将数据回写 socket 完毕 

6、解除绑定，清空等待队列

<img src="https://user-gold-cdn.xitu.io/2020/5/5/171e4f8d42c844f3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" alt="在这里插入图片描述" style="zoom: 80%;" />

该设计有如下特点：

 1、IO 线程要么同时在读 socket，要么同时在写，不会同时读或写 

2、IO 线程只负责读写 socket 解析命令，不负责命令处理

****开启多线程后，是否会存在线程并发安全问题？**

Redis的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程顺序执行。所以我们不需要去考虑控制 key、lua、事务，LPUSH/LPOP 等等的并发及线程安全问题。

## 15. **Redis集群方案应该怎么做？都有哪些方案？**

* codis。目前用的最多的集群方案，基本和twemproxy一致的效果，但它支持在 节点数量改变情况下，旧节点数据可恢复到新hash节点。
* redis cluster3.0自带的集群，特点在于他的分布式算法不是一致性hash，而是hash槽的概念，以及自身支持节点设置从节点。具体看官方文档介绍。
* 在业务代码层实现，起几个毫无关联的redis实例，在代码层，对key 进行hash计算，然后去对应的redis实例操作数据。 这种方式对hash层代码要求比较高，考虑部分包括，节点失效后的替代算法方案，数据震荡后的自动脚本恢复，实例的监控，等等。

## 16.讲一讲 位图(Bitmap)

位图不是特殊的数据结构，它的内容其实就是普通的字符串，也就是 byte 数组。我们可以使用普通的 get/set 直接获取和设置整个位图的内容，也可以使用位图操作 getbit/setbit 等将 byte 数组看成「位数组」来处理。

![img](https://user-gold-cdn.xitu.io/2018/7/2/1645926f4520d0ce?imageslim)

当我们要统计月活的时候，因为需要去重，需要使用 set 来记录所有活跃用户的 id，这非常浪费内存。这时就可以考虑使用位图来标记用户的活跃状态。每个用户会都在这个位图的一个确定位置上，0 表示不活跃，1 表示活跃。然后到月底遍历一次位图就可以得到月度活跃用户数。不过这个方法也是有条件的，那就是 userid 是整数连续的，并且活跃占比较高，否则可能得不偿失。

## 17. 讲一讲HyperLogLog 

`HyperLogLog`，下面简称为`HLL`，它是 `LogLog` 算法的升级版，作用是能够提供不精确的去重计数。存在以下的特点：

- 代码实现较难。
- 能够使用极少的内存来统计巨量的数据，在 `Redis` 中实现的 `HyperLogLog`，只需要`12K`内存就能统计`2^64`个数据。
- 计数存在一定的误差，误差率整体较低。标准误差为 0.81% 。
- 误差可以被设置`辅助计算因子`进行降低。

### 为什么用HyperLogLog

如果要实现这么一个功能：

> 统计 APP或网页 的一个页面，每天有多少用户点击进入的次数。同一个用户的反复点击进入记为 1 次。

用 `HashMap` 这种数据结构就可以，假设 APP 中日活用户达到`百万`或`千万以上级别`的话，我们采用 `HashMap` 的做法，就会导致程序中占用大量的内存。

估算下 `HashMap` 的在应对上述问题时候的内存占用。假设定义`HashMap` 中 `Key` 为 `string` 类型，`value` 为 `bool`。`key` 对应用户的`Id`,`value`是`是否点击进入`。明显地，当百万不同用户访问的时候。此`HashMap` 的内存占用空间为：`100万 * (string + bool)`。

### HyperLogLog原理

如图，给定一系列的随机整数，我们记录下低位连续零位的最大长度 k，通过这个 k 值可以估算出随机数的数量。

<img src="C:\Users\mashuaisen\OneDrive\typora-user-images\Redis-HyperLogLog\7f698cf742b08017cfac781250378b56" alt="img" style="zoom:67%;" />

HyperLogLog与伯努利试验有关，具体可参考[HyperLogLog 算法的原理讲解](https://juejin.im/post/5c7900bf518825407c7eafd0)

## 18. `Redis`单线程如何处理那么多的并发客户端连接？
因为Redis 的线程模型：基于非阻塞的IO多路复用机制。

Redis 是基于 reactor 模式开发了网络事件处理器，这个处理器叫做文件事件处理器（file event handler）。由于这个文件事件处理器是单线程的，所以 Redis 才叫做单线程的模型。采用 IO 多路复用机制同时监听多个 Socket，根据 socket 上的事件来选择对应的事件处理器来处理这个事件。模型如下图：

[![img](http://cmsblogs.com/wp-content/resources/image.cmsblogs/sike-java/sike-redis/redis-202002171001.png)](http://cmsblogs.com/wp-content/resources/image.cmsblogs/sike-java/sike-redis/redis-202002171001.png)

从上图可知，文件事件处理器的结构包含了四个部分：

- 多个 Socket
- IO 多路复用程序
- 文件事件分派器
- 事件处理器

多个 socket 会产生不同的事件，不同的事件对应着不同的操作，IO 多路复用程序监听着这些 Socket，当这些 Socket 产生了事件，IO 多路复用程序会将这些事件放到一个队列中，通过这个队列，以有序、同步、每次一个事件的方式向文件时间分派器中传送。当事件处理器处理完一个事件后，IO 多路复用程序才会继续向文件分派器传送下一个事件。

## 参考

https://juejin.cn/post/6844903663224225806

https://juejin.cn/post/6844903716290576392

https://www.cnblogs.com/wdliu/p/9377278.html

https://researchlab.github.io/2018/10/08/redis-11-redisio/



一份后端开发的综合笔记：

> 链接：https://pan.baidu.com/s/1UNq4egIlxe3hijwjTfvW1w 
> 提取码：u9go 

![](http://blog-img.coolsen.cn/img/image-20210508113649706.png)