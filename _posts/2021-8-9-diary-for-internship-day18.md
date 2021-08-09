---
layout: post
title:  "实习日记18"
date:   2021-08-09
categories: jekyll update
---

## Day18

- **Java学习，基于https://github.com/doocs/advanced-java**

- ES在数据量很大的情况下如何提高查询效率

  - 在数据量巨大(数十亿级别)的情况下， ES的查询效率会显著下降，并且无法通过设置参数应付所有性能慢的场景。

  - **filesystem cache**：用户向ES中写入的数据，实际上都写入到磁盘文件中，查询的时候，操作系统会将磁盘文件中的数据自动缓存到`filesystem cache`中。

    ES搜索引擎严重依赖于底层的`filesystem cache`，如果`filesystem cache`拥有足够的内存来容纳所有的idx segment file(索引数据文件)，这样用户搜索时基本走内存，性能会非常高。所以，要使ES性能好，最佳的情况下是机器内存可以至少容纳总数据量的一半。

    在这种情况下，建议往ES中存入少量的数据，即需要用来进行搜索的索引。可以使用例如es+hbase/mysql这样的架构，在es中写入用来检索的少数几个字段，把其他的字段存入hbase/mysql中。通过es拿到需要查询数据字段，再到hbase中获取完整数据返回给前端。

    hbase是一种构建在HDFS之上的分布式、面向列的存储系统。其特点是适用于海量数据的在线存储，即对hbase可以写入海量数据，但不要做复杂的操作，只是做一些根据id或范围进行查询的简单操作。

  - **数据预热**：即使按照上述方案，也是可能出现es集群中每个机器写入的数据量超过`filesystem cache`一倍，这种情况下可以采用数据预热的方法。

    数据预热，即把很多用户经常访问的数据，提前在后台实现一个系统，每隔一段时间自动搜索一下热数据，然后刷到`filesystem cache`中，之后用户来访问这个热数据时，就会直接从内存中读取，大大提高效率。

    对于热数据，最好实现一个专门的缓存预热子系统，即对热数据每隔一段时间提前访问一下，让数据进入到`filesystem cache`中。这样用户下次访问，性能会好很多。
    
  - **冷热分离**：es可以做类似mysql的水平拆分，即把大量的访问少、频率低的数据，单独写一个索引，然后将访问很频繁的热数据单独写一个索引。最好是将冷数据写入一个索引中，然后将热数据写入另一个索引中，这样可以确保热数据在被预热后，尽量都让他们留在`filesystem os`中，避免被冷数据冲刷掉。

  - **document模型设计**：在MYSQL中经常会有一些复杂的关联查询，但在es中应当尽量避免复杂的关联查询，因为一般来说性能都不太好。

    最好是在Java系统里就完成关联将关联好的数据直接写入es中，这样在搜索的时候，就不需要利用es的搜索语法来完成join之类的关联搜索。尽量在document模型设计的时候、写入的时候完成复杂的操作。

  - **分页性能优化**：

    es存在如下三种分页方式：

    - from+size浅分页：查询前n条数据，然后截断前from条数据，只返回n-from条数据。其中，from定义了目标数据的偏移值，size定义当前返回的数目。默认from为0，size为10，即所有的查询默认仅仅返回前10条数据。越往后的分页，执行的效率越低。总体上会随着from的增加，消耗时间也会增加。而且数据量越大，就越明显。
    - search after方案：利用实时有游标来帮我们解决实时滚动的问题，根据上一页的最后一条数据来确定下一页的位置，同时在分页请求的过程中，如果有索引数据的增删改查，这些变更也会实时的反映到游标上。但是需要注意，因为每一页的数据依赖于上一页最后一条数据，所以无法跳页请求。为了找到每一页最后一条数据，每个文档必须有一个全局唯一值，官方推荐使用 _uid 作为全局唯一值，其实使用业务层的 id 也可以。
    - scroll深分页：scroll 类似于sql中的cursor，使用scroll，每次只能获取一页的内容，然后会返回一个scroll_id。根据返回的这个scroll_id可以不断地获取下一页的内容，所以scroll并不适用于有跳页的情景，是针对需要一次性或者每次查询大量的文档，对实时性要求不高的数据。可参考如下链接：https://www.cnblogs.com/wangzhuxing/p/9569727.html

    针对深分页的场景，有如下解决方法：

    - 不允许深度分页：默认深度分页性能很差。

    - 使用scroll api：类似于 app 里的推荐商品不断下拉出来一页一页的。scroll 会一次性给你生成**所有数据的一个快照**，然后每次滑动向后翻页就是通过**游标** `scroll_id` 移动，获取下一页下一页这样子，性能会比上面说的那种分页性能要高很多很多，基本上都是毫秒级的。

      但是，唯一的一点就是，这个适合于下拉翻页的，不能随意跳到任何一页的场景。也就是说，你不能先进入第 10 页，然后去第 120 页，然后又回到第 58 页，不能随意乱跳页。所以现在很多产品，都是不允许你随意翻页的，app，也有一些网站，做的就是你只能往下拉，一页一页的翻。

      初始化时必须指定 `scroll` 参数，告诉 es 要保存此次搜索的上下文多长时间。需要确保用户不会持续不断翻页翻几个小时，否则可能因为超时而失败。

      除了用 `scroll api` ，你也可以用 `search_after` 来做， `search_after` 的思想是使用前一页的结果来帮助检索下一页的数据，显然，这种方式也不允许你随意翻页，你只能一页页往后翻。初始化时，需要使用一个唯一值的字段作为 sort 字段。

    **PS**：游标（cursor）是系统为用户开设的一个数据缓冲区，存放SQL语句的执行结果。每个游标区都有一个名字,用户可以用SQL语句逐一从游标中获取记录，并赋给主变量，交由主语言进一步处理。就本质而言，游标实际上是一种能从包括多条数据记录的结果集中每次提取一条记录的机制。

- 项目中使用缓存

  - 项目中使用缓存的目的：
    - **高性能**：假设一个mysql进行一个查询耗时600ms，但查询结果可能在一定时间(例如几个小时)内不会发生变化，或者发生的变化不用立即返回用户。这个时候可以将查询结果放入缓存中，一个key对应一个value，下次用户再查询可以直接从缓存中获取查询结果。就是说对于一些需要复杂操作耗时查出来的结果，且确定后面不怎么变化，但是有很多读请求，那么直接将查询出来的结果放在缓存中，后面直接读缓存就好。
    - **高并发**：缓存由于走内存，天然支持高并发。mysql单机支撑到2000QPS开始容易报警了。假设系统在高峰期接收的请求有1w条，这时候可以把数据放在缓存中。
  - 使用了缓存后可能导致的不良后果：
    - 缓存与数据库双写不一致
    - 缓存雪崩、缓存穿透、缓存击穿
    - 缓存并发竞争

- Redis和Memcached、Redis的线程模型：

  - Redis和Memcached的区别：

    - **Redis支持复杂的数据结构**：Redis 相比 Memcached 来说，拥有更多的数据结构，能支持更丰富的数据操作。如果需要缓存能够支持更复杂的结构和操作， Redis 会是不错的选择。
    - **Redis原生支持集群模式**：在 Redis3.x 版本中，便能支持 cluster 模式，而 Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据。
    - **性能对比**：由于 Redis 只使用**单核**，而 Memcached 可以使用**多核**，所以平均每一个核上 Redis 在存储小数据时比 Memcached 性能更高。而在 100k 以上的数据中，Memcached 性能要高于 Redis。虽然 Redis 最近也在存储大数据的性能上进行优化，但是比起 Memcached，还是稍有逊色。

  - Redis的线程模型：Redis内部使用文件事件处理器file event handler，这个文件事务处理器是单线程的，所以Redis是**单线程的模型**。它采用IO多路复用机制来同时监听多个socket，将产生事件的socket压入内存队列中，事件分派器根据socket上的事件类型来选择对应的事件处理器进行处理。

    该文件事件处理器包含4个部分：

    - 多个socket
    - IO多路复用程序
    - 文件事件分派器
    - 事件处理器(连接应答处理器，命令请求处理器，命令回复处理器)

    多个 socket 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 socket，会将产生事件的 socket 放入队列中排队，事件分派器每次从队列中取出一个 socket，根据 socket 的事件类型交给对应的事件处理器进行处理。

    客户端与Redis的一次通信过程(通信由socket完成)：

    - Redis服务端进程初始化，将server socket的`AE_READABLE`事件与连接应答处理器关联。
    - 客户端socket01向Redis进程的server socket请求建立连接，此时server socket会产生一个`AE_READABLE`事件，IO多路复用程序监听到server socket产生的事件后将该socket压入队列中，此时事件分派器从队列中获取到socket01产生的`AE_READABLE`事件，由于之前socket01`AE_READABLE`事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。命令请求处理器读取socket01的`key value`并在自己内存中完成`key value`的设置。操作完成后，它会将socket01的`AE_WAITABLE`事件与命令回复处理器关联。
    - 如果此时客户端准备好接收返回结果了，那么Redis中的socket01会产生一个`AE_WAITABLE`事件，同样压入队列中，事件分派器找到相关联的命令回复处理器，由命令回复处理器对socket01输入本次操作的一个结果，之后解除socket01的`AE_WAITABLE`事件与命令回复处理器的关联。

  - 为什么Redis单线程模型效率这么高

    - 纯内存操作。
    - 核心是基于非阻塞的 IO 多路复用机制。
    - C 语言实现，一般来说，C 语言实现的程序“距离”操作系统更近，执行速度相对会更快。
    - 单线程反而避免了多线程的频繁上下文切换问题，预防了多线程可能产生的竞争问题。

  - Redis6.0开始引入的多线程

    Redis 6.0 之后的版本抛弃了单线程模型这一设计，**原本使用单线程运行的 Redis 也开始选择性地使用多线程模型**。

    前面还在强调 Redis 单线程模型的高效性，现在为什么又要引入多线程？这其实说明 Redis 在有些方面，单线程已经不具有优势了。因为读写网络的 Read/Write 系统调用在 Redis 执行期间占用了大部分 CPU 时间，如果把网络读写做成多线程的方式对性能会有很大提升。

    **Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。** 之所以这么设计是不想 Redis 因为多线程而变得复杂，需要去控制 key、lua、事务、LPUSH/LPOP 等等的并发问题。

- Redis数据类型：https://www.runoob.com/redis/redis-tutorial.html

  - String：最简单的类型，用于普通的get和set，做简单的KV缓存
  - Hashes：类似map的一种结构，一般可以将结构化的数据。比如一个对象（前提是这个对象没嵌套其他的对象）给缓存在 Redis 里，然后每次读写缓存的时候，可以就操作 hash 里的某个字段。
  - Lists：有序列表，可以存储一些列表型数据结构、通过lrange命令，读取某个闭区间内的元素，基于list实现分页查询、实现一个简单的消息队列等。
  - Sets：无序集合，自动去重。直接基于 set 将系统里需要去重的数据扔进去，自动就给去重了，如果你需要对一些数据进行快速的全局去重，你当然也可以基于 jvm 内存里的 HashSet 进行去重，但是如果系统部署在多台机器上，得基于 Redis 进行全局的 set 去重。可以基于 set 实现交集、并集、差集的操作
  - Sorted Sets：排序的 set，去重但可以排序，写进去的时候给一个分数，自动根据分数排序。

- Redis的过期策略和淘汰机制

  - 常见的两种问题：

    - 往Redis中写入的数据消失了
    - 数据明明过期了，怎么还占用着内存

  - Redis的过期策略：定期删除+惰性删除

    - 定期删除：Redis默认每隔100ms就随机抽取了一些设置了过期时间的key，检查其是否过期，如果过期就删除，但定期删除可能会导致很多过期 key 到了时间并没有被删除掉
    - 惰性删除：在用户获取某个key的时候，Redis会检查这个key如果设置了过期时间是否已经过期，如果过期了此时就会删除，不会返回任何东西。即获取 key 的时候，如果此时 key 已经过期，就删除，不会返回任何东西。但如果定期删除漏掉了很多过期 key，用户也没及时去查，也就没走惰性删除会导致大量过期 key 堆积在内存里，导致 Redis 内存块耗尽了，需要使用内存淘汰机制。

  - 内存淘汰机制：

    - noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用吧，实在是太恶心了。
    - allkeys-lru：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。
    - allkeys-random：当内存不足以容纳新写入数据时，在**键空间**中，随机移除某个 key（一般不用）
    - volatile-lru：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。
    - volatile-random：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key。
    - volatile-ttl：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。

  - Redis如何保证高并发和高可用

    用 Redis 来加多台机器，保证 Redis 是高并发的，还有就是如何让 Redis 保证自己不是挂掉以后就直接死掉了，即 redis 高可用。

    主要包括以下两个内容：

    - Redis主从架构
    - Redis基于哨兵实现高可用

    redis 实现**高并发**主要依靠**主从架构**，一主多从，一般来说，很多项目其实就足够了，单主用来写入数据，单机几万 QPS，多从用来查询数据，多个从实例可以提供每秒 10w 的 QPS。

    如果想要在实现高并发的同时，容纳大量的数据，那么就需要 redis 集群，使用 redis 集群之后，可以提供每秒几十万的读写并发。

    redis 高可用，如果是做主从架构部署，那么加上哨兵就可以了，就可以实现，任何一个实例宕机，可以进行主备切换。

  - Redis主从架构

    单机的 Redis，能够承载的 QPS 大概就在上万到几万不等。对于缓存来说，一般都是用来支撑**读高并发**的。因此架构做成主从(master-slave)架构，一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读。所有的**读请求全部走从节点**。这样也可以很轻松实现水平扩容，**支撑读高并发**。

    Redis replication -> 主从架构 -> 读写分离 -> 水平扩容支撑读高并发

    ![Redis-master-slave](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-master-slave.png)

    **Redis replication的核心机制**：

    - Redis 采用**异步方式**复制数据到 slave 节点，不过 Redis2.8 开始，slave node 会周期性地确认自己每次复制的数据量；
    - 一个 master node 是可以配置多个 slave node 的；
    - slave node 也可以连接其他的 slave node；
    - slave node 做复制的时候，不会 block master node 的正常工作；
    - slave node 在做复制的时候，也不会 block 对自己的查询操作，它会用旧的数据集来提供服务；但是复制完成的时候，需要删除旧数据集，加载新数据集，这个时候就会暂停对外服务了；
    - slave node 主要用来进行横向扩容，做读写分离，扩容的 slave node 可以提高读的吞吐量。

    注意，如果采用了主从架构，那么建议必须**开启** master node 的[持久化](https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/redis-persistence.md)，不建议用 slave node 作为 master node 的数据热备，因为那样的话，如果你关掉 master 的持久化，可能在 master 宕机重启的时候数据是空的，然后可能一经过复制， slave node 的数据也丢了。

    另外，master 的各种备份方案，也需要做。万一本地的所有文件丢失了，从备份中挑选一份 rdb 去恢复 master，这样才能**确保启动的时候，是有数据的**，即使采用了高可用机制](https://github.com/doocs/advanced-java/blob/main/docs/high-concurrency/redis-sentinel.md)，slave node 可以自动接管 master node，但也可能 sentinel 还没检测到 master failure，master node 就自动重启了，还是可能导致上面所有的 slave node 数据被清空。

    **Redis主从复制的核心原理**：

    当启动一个 slave node 的时候，它会发送一个 `PSYNC` 命令给 master node。

    如果这是 slave node 初次连接到 master node，那么会触发一次 `full resynchronization` 全量复制。此时 master 会启动一个后台线程，开始生成一份 `RDB` 快照文件，同时还会将从客户端 client 新收到的所有写命令缓存在内存中。 `RDB` 文件生成完毕后， master 会将这个 `RDB` 发送给 slave，slave 会先写入本地磁盘，然后再从本地磁盘加载到内存中，接着 master 会将内存中缓存的写命令发送到 slave，slave 也会同步这些数据。slave node 如果跟 master node 有网络故障，断开了连接，会自动重连，连接之后 master node 仅会复制给 slave 部分缺少的数据。

    ![Redis-master-slave-replication](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-master-slave-replication.png)

    **主从复制的断点续传**：

    从 Redis2.8 开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。

    master node 会在内存中维护一个 backlog，master 和 slave 都会保存一个 replica offset 还有一个 master run id，offset 就是保存在 backlog 中的。如果 master 和 slave 网络连接断掉了，slave 会让 master 从上次 replica offset 开始继续复制，如果没有找到对应的 offset，那么就会执行一次 `resynchronization` 。

    > 如果根据 host+ip 定位 master node，是不靠谱的，如果 master node 重启或者数据出现了变化，那么 slave node 应该根据不同的 run id 区分。

    **无磁盘化复制**：

    master 在内存中直接创建 `RDB` ，然后发送给 slave，不会在自己本地落地磁盘了。只需要在配置文件中开启 `repl-diskless-sync yes` 即可。

    ```
    repl-diskless-sync yes
    
    # 等待 5s 后再开始复制，因为要等更多 slave 重新连接过来
    repl-diskless-sync-delay 5
    ```

    **过期key处理**：

    slave 不会过期 key，只会等待 master 过期 key。如果 master 过期了一个 key，或者通过 LRU 淘汰了一个 key，那么会模拟一条 del 命令发送给 slave。

    **复制的完整流程**：

    slave node 启动时，会在自己本地保存 master node 的信息，包括 master node 的 `host` 和 `ip` ，但是复制流程没开始。

    slave node 内部有个定时任务，每秒检查是否有新的 master node 要连接和复制，如果发现，就跟 master node 建立 socket 网络连接。然后 slave node 发送 `ping` 命令给 master node。如果 master 设置了 requirepass，那么 slave node 必须发送 masterauth 的口令过去进行认证。master node **第一次执行全量复制**，将所有数据发给 slave node。而在后续，master node 持续将写命令，异步复制给 slave node。

    ![Redis-master-slave-replication-detail](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-master-slave-replication-detail.png)

    **全量复制**：

    - master 执行 bgsave ，在本地生成一份 rdb 快照文件。
    - master node 将 rdb 快照文件发送给 slave node，如果 rdb 复制时间超过 60 秒（repl-timeout），那么 slave node 就会认为复制失败，可以适当调大这个参数(对于千兆网卡的机器，一般每秒传输 100MB，6G 文件，很可能超过 60s)
    - master node 在生成 rdb 时，会将所有新的写命令缓存在内存中，在 slave node 保存了 rdb 之后，再将新的写命令复制给 slave node。
    - 如果在复制期间，内存缓冲区持续消耗超过 64MB，或者一次性超过 256MB，那么停止复制，复制失败。

    - slave node 接收到 rdb 之后，清空自己的旧数据，然后重新加载 rdb 到自己的内存中，同时基于旧的数据版本对外提供服务。
    - 如果 slave node 开启了 AOF，那么会立即执行 BGREWRITEAOF，重写 AOF。

    **增量复制**：

    - 如果全量复制过程中，master-slave 网络连接断掉，那么 slave 重新连接 master 时，会触发增量复制。
    - master 直接从自己的 backlog 中获取部分丢失的数据，发送给 slave node，默认 backlog 就是 1MB。
    - master 就是根据 slave 发送的 psync 中的 offset 来从 backlog 中获取数据的。

    **heartbeat**：

    主从节点互相都会发送 heartbeat 信息。

    master 默认每隔 10 秒发送一次 heartbeat，slave node 每隔 1 秒发送一个 heartbeat。

    **异步复制**：

    master 每次接收到写命令之后，先在内部写入数据，然后异步发送给 slave node。