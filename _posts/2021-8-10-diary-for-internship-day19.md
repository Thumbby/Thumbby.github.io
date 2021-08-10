---
layout: post
title:  "实习日记19(Redis集群)"
date:   2021-08-10
categories: jekyll update
---

## Day19

- **Java学习，基于https://github.com/doocs/advanced-java**

- Redis哨兵集群实现高可用

  - 哨兵是Redis集群架构中非常重要的一个组件，主要提供如下功能：

    - 集群监控：负责监控 Redis master 和 slave 进程是否正常工作。
    - 消息通知：如果某个 Redis 实例有故障，那么哨兵负责发送消息作为报警通知给管理员。
    - 故障转移：如果 master node 挂掉了，会自动转移到 slave node 上。
    - 配置中心：如果故障转移发生了，通知 client 客户端新的 master 地址。

    哨兵用于实现Redis集群的高可用，本身也是分布式的，作为一个哨兵集群去运行、相互协同工作。

    - 故障转移时，判断一个 master node 是否宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题。
    - 即使部分哨兵节点挂掉了，哨兵集群还是能正常工作的，因为如果一个作为高可用机制重要组成部分的故障转移系统本身不能是单点的。

  - 哨兵的核心知识

    - 哨兵至少需要 3 个实例，来保证自己的健壮性。
    - 哨兵 + Redis 主从的部署架构，是**不保证数据零丢失**的，只能保证 Redis 集群的高可用性。
    - 对于哨兵 + Redis 主从这种复杂的部署架构，尽量在测试环境和生产环境，都进行充足的测试和演练。

    哨兵集群必须部署2个以上节点，如果哨兵集群仅仅部署了2个哨兵实例，即配置`quorum=1`

    ```
    +----+         +----+
    | M1 |---------| R1 |
    | S1 |         | S2 |
    +----+         +----+
    ```

    如果master节点宕机， s1 和 s2 中只要有 1 个哨兵认为 master 宕机了，就可以进行切换，同时 s1 和 s2 会选举出一个哨兵来执行故障转移。但是同时这个时候，需要 majority，也就是大多数哨兵都是运行的，如下：

    ```
    2 个哨兵，majority=2
    3 个哨兵，majority=2
    4 个哨兵，majority=2
    5 个哨兵，majority=3
    ...
    ```

    如果此时仅仅是 M1 进程宕机了，哨兵 s1 正常运行，那么故障转移是 OK 的。但是如果是整个 M1 和 S1 运行的机器宕机了，那么哨兵只有 1 个，此时就没有 majority 来允许执行故障转移，虽然另外一台机器上还有一个 R1，但是故障转移不会执行。

    配置 `quorum=2` ，如果 M1 所在机器宕机了，那么三个哨兵还剩下 2 个，S2 和 S3 可以一致认为 master 宕机了，然后选举出一个来执行故障转移，同时 3 个哨兵的 majority 是 2，所以还剩下的 2 个哨兵运行着，就可以允许执行故障转移。

  - 导致数据丢失的两种情况：

    主备切换的过程，可能会导致数据丢失

    - 异步复制导致的数据丢失

      因为 master->slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时这部分数据就丢失了。

      ![async-replication-data-lose-case](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/async-replication-data-lose-case.png)

    - 脑裂导致的数据丢失

      脑裂，也就是说，某个 master 所在机器突然**脱离了正常的网络**，跟其他 slave 机器不能连接，但是实际上 master 还运行着。此时哨兵可能就会**认为** master 宕机了，然后开启选举，将其他 slave 切换成了 master。这个时候，集群里就会有两个 master ，也就是所谓的**脑裂**。

      此时虽然某个 slave 被切换成了 master，但是可能 client 还没来得及切换到新的 master，还继续向旧 master 写数据。因此旧 master 再次恢复的时候，会被作为一个 slave 挂到新的 master 上去，自己的数据会清空，重新从新的 master 复制数据。而新的 master 并没有后来 client 写入的数据，因此，这部分数据也就丢失了。

      ![Redis-cluster-split-brain](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-cluster-split-brain.png)

  - 数据丢失的解决方案

    进行如下配置：

    ```
    min-slaves-to-write 1
    min-slaves-max-lag 10
    ```

    表示，要求至少有 1 个 slave，数据复制和同步的延迟不能超过 10 秒。

    如果说一旦所有的 slave，数据复制和同步的延迟都超过了 10 秒钟，那么这个时候，master 就不会再接收任何请求了。

    - 减少异步复制数据的丢失

    有了 `min-slaves-max-lag` 这个配置，就可以确保说，一旦 slave 复制数据和 ack 延时太长，就认为可能 master 宕机后损失的数据太多了，那么就拒绝写请求，这样可以把 master 宕机时由于部分数据未同步到 slave 导致的数据丢失降低的可控范围内。

    - 减少脑裂的数据丢失

    如果一个 master 出现了脑裂，跟其他 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒的数据。

  - sdown和odown的转换机制

    - sdown 是主观宕机，一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机
    - odown 是客观宕机，如果 quorum 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机

    sdown 达成的条件：如果一个哨兵 ping 一个 master，超过了 `is-master-down-after-milliseconds` 指定的毫秒数之后，就主观认为 master 宕机了；如果一个哨兵在指定时间内，收到了 quorum 数量的其它哨兵也认为那个 master 是 sdown 的，那么就认为是 odown 了。

  - 哨兵集群的自动发现机制

    哨兵互相之间的发现，是通过 Redis 的 `pub/sub` 系统实现的，每个哨兵都会往 `__sentinel__:hello` 这个 channel 里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在。

    每隔两秒钟，每个哨兵都会往自己监控的某个 master+slaves 对应的 `__sentinel__:hello` channel 里发送一个消息，内容是自己的 host、ip 和 runid 还有对这个 master 的监控配置。

    每个哨兵也会去监听自己监控的每个 master+slaves 对应的 `__sentinel__:hello` channel，然后去感知到同样在监听这个 master+slaves 的其他哨兵的存在。

    每个哨兵还会跟其他哨兵交换对 `master` 的监控配置，互相进行监控配置的同步。

  - slave配置的自动纠正

    哨兵会负责自动纠正 slave 的一些配置，比如 slave 如果要成为潜在的 master 候选人，哨兵会确保 slave 复制现有 master 的数据；如果 slave 连接到了一个错误的 master 上，比如故障转移之后，那么哨兵会确保它们连接到正确的 master 上。

  - slave->master选举算法

    如果一个 master 被认为 odown 了，而且 majority 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个 slave 来，会考虑 slave 的一些信息：

    - 跟 master 断开连接的时长
    - slave 优先级
    - 复制 offset
    - run id

    如果一个 slave 跟 master 断开连接的时间已经超过了 `down-after-milliseconds` 的 10 倍，外加 master 宕机的时长，那么 slave 就被认为不适合选举为 master。

    接下来会对 slave 进行排序：

    - 按照 slave 优先级进行排序，slave priority 越低，优先级就越高。
    - 如果 slave priority 相同，那么看 replica offset，哪个 slave 复制了越多的数据，offset 越靠后，优先级就越高。
    - 如果上面两个条件都相同，那么选择一个 run id 比较小的那个 slave。

  - quorum和majority

    每次一个哨兵要做主备切换，首先需要 quorum 数量的哨兵认为 odown，然后选举出一个哨兵来做切换，这个哨兵还需要得到 majority 哨兵的授权，才能正式执行切换。

    如果 quorum < majority，比如 5 个哨兵，majority 就是 3，quorum 设置为 2，那么就 3 个哨兵授权就可以执行切换。

    但是如果 quorum >= majority，那么必须 quorum 数量的哨兵都授权，比如 5 个哨兵，quorum 是 5，那么必须 5 个哨兵都同意授权，才能执行切换。

  - configuration epoch

    哨兵会对一套 Redis master+slaves 进行监控，有相应的监控的配置。

    执行切换的那个哨兵，会从要切换到的新 master（salve->master）那里得到一个 configuration epoch，这就是一个 version 号，每次切换的 version 号都必须是唯一的。 

    如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待 failover-timeout 时间，然后接替继续执行切换，此时会重新获取一个新的 configuration epoch，作为新的 version 号。

  - configuration传播

    哨兵完成切换之后，会在自己本地更新生成最新的 master 配置，然后同步给其他的哨兵，就是通过之前说的 `pub/sub` 消息机制。

    这里之前的 version 号就很重要了，因为各种消息都是通过一个 channel 去发布和监听的，所以一个哨兵完成一次新的切换之后，新的 master 配置是跟着新的 version 号的。其他的哨兵都是根据版本号的大小来更新自己的 master 配置的。

- Redis持久化

  防止Redis宕机重启后的数据丢失，在数据写入内存的同时，异步地将数据写入磁盘文件中，再进行持久化。这样Redis在宕机重启后可以自动从磁盘加载之前持久化的一些数据，也许会丢失少许数据，但是至少不会将所有数据都弄丢。

  - Redis持久化的两种方式：

    - RDB：RDB 持久化机制，是对 Redis 中的数据执行周期性的持久化。
    - AOF：AOF 机制对每条写入命令作为日志，以 `append-only` 的模式写入一个日志文件中，在 Redis 重启的时候，可以通过回放 AOF 日志中的写入指令来重新构建整个数据集。

    通过 RDB 或 AOF，都可以将 Redis 内存中的数据给持久化到磁盘上面来，然后可以将这些数据备份到别的地方去。

    如果 Redis 挂了，服务器上的内存和磁盘上的数据都丢了，可以从云服务上拷贝回来之前的数据，放到指定的目录中，然后重新启动 Redis，Redis 就会自动根据持久化数据文件中的数据，去恢复内存中的数据，继续对外提供服务。

    如果同时使用 RDB 和 AOF 两种持久化机制，那么在 Redis 重启的时候，会使用 **AOF** 来重新构建数据，因为 AOF 中的数据更加完整。

  - RDB优缺点：

    - RDB 会生成多个数据文件，每个数据文件都代表了某一个时刻中 Redis 的数据，这种多个数据文件的方式，非常适合做冷备(两个服务器，一台运行，一台不运行做为备份)，可以将这种完整的数据文件发送到一些远程的安全存储上去。
    - RDB 对 Redis 对外提供的读写服务，影响非常小，可以让 Redis 保持高性能，因为 Redis 主进程只需要 fork 一个子进程，让子进程执行磁盘 IO 操作来进行 RDB 持久化即可。
    - 相对于 AOF 持久化机制来说，直接基于 RDB 数据文件来重启和恢复 Redis 进程，更加快速。
    - 如果想要在 Redis 故障时，尽可能少的丢失数据，那么 RDB 没有 AOF 好。一般来说，RDB 数据快照文件，都是每隔 5 分钟，或者更长时间生成一次，这个时候就得接受一旦 Redis 进程宕机，那么会丢失最近 5 分钟（甚至更长时间）的数据。
    - RDB 每次在 fork 子进程来执行 RDB 快照数据文件生成的时候，如果数据文件特别大，可能会导致对客户端提供的服务暂停数毫秒，或者甚至数秒。

  - AOF优缺点：

    - AOF 可以更好的保护数据不丢失，一般 AOF 会每隔 1 秒，通过一个后台线程执行一次 `fsync` 操作，最多丢失 1 秒钟的数据。
    - AOF 日志文件以 `append-only` 模式写入，所以没有任何磁盘寻址的开销，写入性能非常高，而且文件不容易破损，即使文件尾部破损，也很容易修复。
    - AOF 日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。因为在 `rewrite` log 的时候，会对其中的指令进行压缩，创建出一份需要恢复数据的最小日志出来。在创建新日志文件的时候，老的日志文件还是照常写入。当新的 merge 后的日志文件 ready 的时候，再交换新老日志文件即可。
    - AOF 日志文件的命令通过可读较强的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。比如某人不小心用 `flushall` 命令清空了所有数据，只要这个时候后台 `rewrite` 还没有发生，那么就可以立即拷贝 AOF 文件，将最后一条 `flushall` 命令给删了，然后再将该 `AOF` 文件放回去，就可以通过恢复机制，自动恢复所有数据。
    - 对于同一份数据来说，AOF 日志文件通常比 RDB 数据快照文件更大。
    - AOF 开启后，支持的写 QPS 会比 RDB 支持的写 QPS 低，因为 AOF 一般会配置成每秒 `fsync` 一次日志文件，当然，每秒一次 `fsync` ，性能也还是很高的。（如果实时写入，那么 QPS 会大降，Redis 性能会大大降低）
    - 以前 AOF 发生过 bug，就是通过 AOF 记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。所以说，类似 AOF 这种较为复杂的基于命令日志 `merge` 回放的方式，比基于 RDB 每次持久化一份完整的数据快照文件的方式，更加脆弱一些，容易有 bug。不过 AOF 就是为了避免 rewrite 过程导致的 bug，因此每次 rewrite 并不是基于旧的指令日志进行 merge 的，而是基于当时内存中的数据进行指令的重新构建，这样健壮性会好很多。

  - RDB和AOF如何选择：

    - 不要仅仅使用 RDB，因为那样会导致你丢失很多数据；
    - 也不要仅仅使用 AOF，因为那样有两个问题：第一，你通过 AOF 做冷备，没有 RDB 做冷备来的恢复速度更快；第二，RDB 每次简单粗暴生成数据快照，更加健壮，可以避免 AOF 这种复杂的备份和恢复机制的 bug；
    - Redis 支持同时开启开启两种持久化方式，我们可以综合使用 AOF 和 RDB 两种持久化机制，用 AOF 来保证数据不丢失，作为数据恢复的第一选择；用 RDB 来做不同程度的冷备，在 AOF 文件都丢失或损坏不可用的时候，还可以使用 RDB 来进行快速的数据恢复。

- Redis集群模式工作原理

  现在Redis使用的是Redis cluster，即Redis原生支持的Redis集群模式，主要针对海量数据+高并发+高可用的场景。Redis cluster 支撑 N 个 Redis master node，每个 master node 都可以挂载多个 slave node。这样整个 Redis 就可以横向扩容了。如果要支撑更大数据量的缓存，那就横向扩容更多的 master 节点，每个 master 节点就能存放更多的数据了。

  - Redis cluster介绍

    - 自动将数据进行分片，每个 master 上放一部分数据
    - 提供内置的高可用支持，部分 master 不可用时，还是可以继续工作的

    在 Redis cluster 架构下，每个 Redis 要放开两个端口号，比如一个是 6379，另外一个就是 加 1w 的端口号，比如 16379。

    16379 端口号是用来进行节点间通信的，也就是 cluster bus 的东西，cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议， `gossip` 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

  - 节点内部通信机制

    - 基本通信原理

      集群元数据的维护有两种方式：集中式、Gossip 协议。Redis cluster 节点间采用 gossip 协议进行通信。

      **集中式**

      集中式是将集群元数据（节点信息、故障等等）集中存储在某个节点上。集中式元数据集中存储的一个典型代表，就是大数据领域的 `storm` 。它是分布式的大数据实时计算引擎，是集中式的元数据存储的结构，底层基于 zookeeper（分布式协调的中间件）对所有元数据进行存储维护。

      ![zookeeper-centralized-storage](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/zookeeper-centralized-storage.png)

      集中式优点：元数据的读取和更新，时效性非常好，一旦元数据出现了变更，就立即更新到集中式的存储中，其它节点读取的时候就可以感知到

      集中式缺点：不好在于，所有的元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力。

      **gossip协议**

      所有节点都持有一份元数据，不同的节点如果出现了元数据的变更，就不断将元数据发送给其它的节点，让其它节点也进行元数据的变更。

      ![Redis-gossip](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/redis-gossip.png)

      - 0000 端口：每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如 7001，那么用于节点间通信的就是 17001 端口。每个节点每隔一段时间都会往另外几个节点发送 `ping` 消息，同时其它几个节点接收到 `ping` 之后返回 `pong` 。
      - 交换的信息：信息包括故障信息，节点的增加和删除，hash slot 信息等等。

      gossip 协议包含多种消息，包含 `ping` , `pong` , `meet` , `fail` 等等。

      - meet：某个节点发送 meet 给新加入的节点，让新节点加入集群中，然后新节点就会开始与其它节点进行通信。

      ```
      Redis-trib.rb add-node
      ```

      其实内部就是发送了一个 gossip meet 消息给新加入的节点，通知那个节点去加入我们的集群。

      - ping：每个节点都会频繁给其它节点发送 ping，其中包含自己的状态还有自己维护的集群元数据，互相通过 ping 交换元数据。
      - pong：返回 ping 和 meeet，包含自己的状态和其它信息，也用于信息广播和更新。
      - fail：某个节点判断另一个节点 fail 之后，就发送 fail 给其它节点，通知其它节点说，某个节点宕机啦。

      ping消息深入

      ping 时要携带一些元数据，如果很频繁，可能会加重网络负担。

      每个节点每秒会执行 10 次 ping，每次会选择 5 个最久没有通信的其它节点。当然如果发现某个节点通信延时达到了 `cluster_node_timeout / 2` ，那么立即发送 ping，避免数据交换延时过长，落后的时间太长了。比如说，两个节点之间都 10 分钟没有交换数据了，那么整个集群处于严重的元数据不一致的情况，就会有问题。所以 `cluster_node_timeout` 可以调节，如果调得比较大，那么会降低 ping 的频率。

      每次 ping，会带上自己节点的信息，还有就是带上 1/10 其它节点的信息，发送出去，进行交换。至少包含 `3` 个其它节点的信息，最多包含 `总节点数减 2` 个其它节点的信息。

      gossip优点：元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续打到所有节点上去更新，降低了压力

      gossip缺点：元数据的更新有延时，可能导致集群中的一些操作会有一些滞后。
    
  - 分布式寻址算法
  
    主要有以下三类算法：
  
    - hash 算法（大量缓存重建）
  
      来了一个 key，首先计算 hash 值，然后对节点数取模。然后打在不同的 master 节点上。一旦某一个 master 节点宕机，所有请求过来，都会基于最新的剩余 master 节点数去取模，尝试去取数据。这会导致大部分的请求过来，全部无法拿到有效的缓存，导致大量的流量涌入数据库。
  
      ![hash](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/hash.png)
  
    - 一致性 hash 算法（自动缓存迁移）+ 虚拟节点（自动负载均衡）
  
      一致性 hash 算法将整个 hash 值空间组织成一个虚拟的圆环，整个空间按顺时针方向组织，下一步将各个 master 节点（使用服务器的 ip 或主机名）进行 hash。这样就能确定每个节点在其哈希环上的位置。
  
      来了一个 key，首先计算 hash 值，并确定此数据在环上的位置，从此位置沿环**顺时针“行走”**，遇到的第一个 master 节点就是 key 所在位置。
  
      在一致性哈希算法中，如果一个节点挂了，受影响的数据仅仅是此节点到环空间前一个节点（沿着逆时针方向行走遇到的第一个节点）之间的数据，其它不受影响。增加一个节点也同理。
  
      但是一致性哈希算法在节点太少时，容易因为节点分布不均匀而造成**缓存热点**的问题。为了解决这种热点问题，一致性 hash 算法引入了虚拟节点机制，即对每一个节点计算多个 hash，每个计算结果位置都放置一个虚拟节点。这样就实现了数据的均匀分布，负载均衡。
  
      ![consistent-hashing-algorithm](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/consistent-hashing-algorithm.png)
  
    - Redis cluster 的 hash slot 算法
  
      Redis cluster 有固定的 `16384` 个 hash slot，对每个 `key` 计算 `CRC16` 值，然后对 `16384` 取模，可以获取 key 对应的 hash slot。
  
      Redis cluster 中每个 master 都会持有部分 slot，比如有 3 个 master，那么可能每个 master 持有 5000 多个 hash slot。hash slot 让 node 的增加和移除很简单，增加一个 master，就将其他 master 的 hash slot 移动部分过去，减少一个 master，就将它的 hash slot 移动到其他 master 上去。移动 hash slot 的成本是非常低的。客户端的 api，可以对指定的数据，让他们走同一个 hash slot，通过 `hash tag` 来实现。
  
      任何一台机器宕机，另外两个节点不受影响的。因为 key 找的是 hash slot，不是机器。
  
      ![hash-slot](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/hash-slot.png)
  
  - Redis cluster的高可用与主备切换原理
  
    - 判断节点宕机
  
      **主管宕机**：如果一个节点认为另外一个节点宕机，那么就是 `pfail` 。
  
      **客观宕机**：如果多个节点都认为另外一个节点宕机了，那么就是 `fail`。
  
      在 `cluster-node-timeout` 内，某个节点一直没有返回 `pong` ，那么就被认为 `pfail` 。
  
      如果一个节点认为某个节点 `pfail` 了，那么会在 `gossip ping` 消息中， `ping` 给其他节点，如果**超过半数**的节点都认为 `pfail` 了，那么就会变成 `fail` 。
  
    - 从节点过滤
  
      对宕机的 master node，从其所有的 slave node 中，选择一个切换成 master node。
  
      检查每个 slave node 与 master node 断开连接的时间，如果超过了 `cluster-node-timeout * cluster-slave-validity-factor` ，那么就没有资格切换成 `master` 。
  
    - 从节点选举
  
      每个从节点，都根据自己对 master 复制数据的 offset，来设置一个选举时间，offset 越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举。
  
      所有的 master node 开始 slave 选举投票，给要进行选举的 slave 进行投票，如果大部分 master node `（N/2 + 1）` 都投票给了某个从节点，那么选举通过，那个从节点可以切换成 master。
  
      从节点执行主备切换，从节点切换为主节点。
  
    - 与哨兵比较
  
      整个流程跟哨兵相比，非常类似，所以说，Redis cluster 功能强大，直接集成了 replication 和 sentinel 的功能。