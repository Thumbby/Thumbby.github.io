---
layout: post
title:  "实习日记21(Redis rehash+分库分表)"
date:   2021-08-12
categories: jekyll update
---

## Day21

- **Java学习，基于https://github.com/doocs/advanced-java**

- Redis rehash

  与Redis的扩容有关，Redis主要存储键值对，键值对通过字典进行存储，而Redis字典底层由哈希表来实现。通过哈希表中的节点保存字典中的键值对，将key通过哈希函数映射到哈希表节点位置。

  - Redis中的字典数据结构

    ```c
    // 字典对应的数据结构，有关hash表的结构可以参考redis源码，再次就不进行描述
    typedef struct dict {
        dictType *type;  // 字典类型
        void *privdata;  // 私有数据
        dictht ht[2];    // 2个哈希表，这也是进行rehash的重要数据结构，从这也看出字典的底层通过哈希表进行实现。
        long rehashidx;   // rehash过程的重要标志，值为-1表示rehash未进行
        int iterators;   //  当前正在迭代的迭代器数
    } dict;
    ```

  - 在对哈希表进行扩展或者收缩操作时，程序需要将现有哈希表包含的所有键值对rehash到新哈希表中，具体过程如下：

    - 对字典的备用哈希表分配空间

      如果执行的是扩展操作，那么备用哈希表的大小为第一个大于等于需要扩容的哈希表的键值对数量*2 的 2"(2 的 n 次方幂);【`5*2=10,`所以备用哈希表的容量为第一个大于 10 的 2"，即 16】

      如果执行的是收缩操作,那么备用哈希表的大小为第一个大于等于需要扩容的哈希表的键值对数量（`ht[0] .used`）的 2"。

    - 渐进式rehash

      rehash在数据量非常大的情况下(几千万、几亿条数据)不是一次性完成的，而是渐进地完成。渐进rehash好处在于避免对服务器造成影响。

      渐进式rehash的本质：

      - 借助 rehashidx，将 rehash 键值对所需的计算工作均摊到对字典的每个添加、删除、查找和更新操作上，从而避免了集中式 rehash 而带来的庞大计算量。
      - 在 rehash 进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序除了执行指定的操作以外，还会顺带将原哈希表在 rehashidx 索引上的所有键值对 rehash 到备用哈希表，当 rehash 工作完成之后，程序将 rehashidx 属性的值加 1。

- 分库分表

  分库分表是为了支撑高并发、数据量大两个问题。分库和分表是两回事，可能光分库不分表，也可能光分表不分库。

  - 分表

    单表数据量太大，会严重影响到sql的执行性能，一般情况下单表到几百万的时候，性能就会相对差一些，这个时候需要分表了。

    分表，即把一个表的数据放到多个表中，用户每次查询只查询一个表，这样可以控制每张表的数据量在可控的范围内。

  - 分库

    一般情况下，单个库最多支撑到并发2000的级别，否则就需要扩容，而且一个健康的单库并发值最好保证在每秒1000左右，那么可以将一个库的数据拆分到多个库中，访问的时候只访问一个库。

  - 分库分表前后对比

    |              | 分库分表前                  | 分库分表后                                   |
    | ------------ | --------------------------- | -------------------------------------------- |
    | 并发支撑情况 | MYSQL单机部署，扛不住高并发 | MYSQL从单机到多机，能承受的并发增加了多倍    |
    | 磁盘使用情况 | MYSQL单机磁盘容量几乎撑满   | 拆分为多个库，数据库服务器磁盘使用率大大降低 |
    | SQL执行性能  | 单表数据量太大，SQL越跑越慢 | 单表数据量减少，SQL执行效率显著提升          |

  - 常见的分库分表中间件

    - Cobar

      阿里 b2b 团队开发和开源的，属于 proxy 层方案，就是介于应用服务器和数据库服务器之间。应用程序通过 JDBC 驱动访问 Cobar 集群，Cobar 根据 SQL 和分库规则对 SQL 做分解，然后分发到 MySQL 集群不同的数据库实例上执行，不支持读写分离、存储过程、跨库 join 和分页等操作。(现在没什么人用)

    - TDDL

      淘宝团队开发的，属于 client 层方案。支持基本的 crud 语法和读写分离，但不支持 join、多表查询等语法。目前使用的也不多，因为还依赖淘宝的 diamond 配置管理系统。

    - Atlas

      360 开源的，属于 proxy 层方案(社区长期不维护，现在用的人也很少)

    - Sharding-jdbc

      当当开源的，属于 client 层方案，是[ `ShardingSphere` ](https://shardingsphere.apache.org/)的 client 层方案，[ `ShardingSphere` ](https://shardingsphere.apache.org/)还提供 proxy 层的方案 Sharding-Proxy。确实之前用的还比较多一些，因为 SQL 语法支持也比较多，没有太多限制。截至 2019.4，已经推出到了 `4.0.0-RC1` 版本，支持分库分表、读写分离、分布式 id 生成、柔性事务（最大努力送达型事务、TCC 事务）。

    - Mycat

      基于 Cobar 改造的，属于 proxy 层方案，支持的功能非常完善，属于新兴中间件。

  - 水平拆分与垂直拆分

    - 水平拆分：把一个表的数据给弄到多个库的多个表里去，但是每个库的表结构都一样，只不过每个库表放的数据是不同的，所有库表的数据加起来就是全部数据。水平拆分的意义，就是将数据均匀放更多的库里，然后用多个库来扛更高的并发，还有就是用多个库的存储容量来进行扩容。

    - 垂直拆分：把一个有很多字段的表给拆分成多个表**，**或者是多个库上去。每个库表的结构都不一样，每个库表都包含部分字段。一般来说，会将较少的访问频率很高的字段放到一个表里去，然后将较多的访问频率很低的字段放到另外一个表里去。因为数据库是有缓存的，你访问频率高的行字段越少，就可以在缓存里缓存更多的行，性能就越好。这个一般在表层面做的较多一些。

      无论分库还是分表，上面说的那些数据库中间件都是可以支持的。就是基本上那些中间件可以做到你分库分表之后，中间件可以根据你指定的某个字段值，自动路由到对应的库上去，然后再自动路由到对应的表里去。

  - 另外两种分库分表方式

    - 一种是按照 range 来分，就是每个库一段连续的数据，这个一般是按比如时间范围来的，但是这种一般较少用，因为很容易产生热点问题，大量的流量都打在最新的数据上了。好处在于说，扩容的时候很简单，因为你只要预备好，给每个月都准备一个库就可以了，到了一个新的月份的时候，自然而然，就会写新的库了；缺点，但是大部分的请求，都是访问最新的数据。实际生产用 range，要看场景。
    - 或者是按照某个字段 hash 一下均匀分散，这个较为常用。好处在于说，可以平均分配每个库的数据量和请求压力；坏处在于说扩容起来比较麻烦，会有一个数据迁移的过程，之前的数据需要重新计算 hash 值重新分配到不同的库或表。

- 分库分表方案设计方案

  - 停机迁移方案

    即把老的单库单表数据库停了，通过预先写好的导数一次性工具进行分库分表。

  - 双写迁移方案

    即在线上系统里面，之前所有写库的地方，增删改操作，除了对老库增删改，都加上对新库的增删改，这就是所谓的**双写**，同时写俩库，老库和新库。

    然后**系统部署**之后，新库数据差太远，用之前说的导数工具，跑起来读老库数据写新库，写的时候要根据 gmt_modified 这类字段判断这条数据最后修改的时间，除非是读出来的数据在新库里没有，或者是比新库的数据新才会写。简单来说，就是不允许用老数据覆盖新数据。

    导完一轮之后，有可能数据还是存在不一致，那么就程序自动做一轮校验，比对新老库每个表的每条数据，接着如果有不一样的，就针对那些不一样的，从老库读数据再次写。反复循环，直到两个库每个表的数据都完全一致为止。

    接着当数据完全一致了，就 ok 了，基于仅仅使用分库分表的最新代码，重新部署一次，就仅仅基于分库分表操作了。

- 可动态扩容缩容的分库分表方案

  尽量保证一次性分表就分够，目前国内可以使用分32个库，每个库32张表，总共1024张表，保证够用并且并发支撑和数据量支撑没有问题。步骤总结如下：

  - 设定好几台数据库服务器，每台服务器上几个库，每个库多少个表，推荐是 32 库 * 32 表，对于大部分公司来说，可能几年都够了。
  - 路由的规则，orderId 模 32 = 库，orderId / 32 模 32 = 表
  - 扩容的时候，申请增加更多的数据库服务器，装好 MySQL，呈倍数扩容，4 台服务器，扩到 8 台服务器，再到 16 台服务器。
  - 由 DBA 负责将原先数据库服务器的库，迁移到新的数据库服务器上去，库迁移是有一些便捷的工具的。
  - 我们这边就是修改一下配置，调整迁移的库所在数据库服务器的地址。
  - 重新发布系统，上线，原先的路由规则变都不用变，直接可以基于 n 倍的数据库服务器的资源，继续进行线上系统的提供服务。

- 分库分表后的主键id处理

  分成多个表后，每张表的id不能从1开始累加，而是需要一个全局id的支持。

  - 数据库自增id

    系统每次得到一个id，都是往一个库中的一个表插入一条没有实际业务含义的数据，然后获取一个数据库的自增id，拿到这个id之后再往对应的分库分表里写入。

    **优点**：方便简单，易于使用

    **缺点**：单库生成自增id要是高并发的话会产生瓶颈，如果一定需要改进的话，就需要专门开一个服务，这个服务每次拿到当前id最大值，然后自己递增几个id，一次性返回一批的id，然后再把当前最大id值修改成递增几个id之后的一个值，但是无论如何这种方法都是基于单个数据库。

    **适合场景**：并发不高，但是数据量太大导致的分库分表扩容。
    
  - 设置数据库 sequence 或者表自增字段步长
    
    可以设置数据库sequence或者表自增字段步长来进行水平伸缩。比如说，现在有 8 个服务节点，每个服务节点使用一个 sequence 功能来产生 ID，每个 sequence 的起始 ID 不同，并且依次递增，步长都是 8。
    
    **适合场景**：用户防止产生ID重复的场景下这种方法比较简单，也能达到性能要求。但是服务节点固定，步长也固定，将来如果还要增加服务节点，就会很麻烦。
    
  - UUID
  
    即唯一辨识码，关于UUID的具体细节可以参考该连接：https://www.jianshu.com/p/015f49fe1ff9
  
    **优点**：本地生成，无需基于数据库
  
    **缺点**：UUID太长，占用空间太大，作为主键性能很差；更重要的是，UUID不具有有序性，会导致B+数索引在写时有过多的随机写操作(如果是连续的ID可以产生部分顺序写)；以及由于在写的时候不能产生有顺序的append操作，而需要进行insert操作，将会读取整个B+树节点到内存，在插入这条记录后会将整个节点写回磁盘，这种操作在记录占用空间比较大的情况下性能会明显下降。
  
    **适合的场景**：如果用户需要随机生成个文件名、编号的时候，但UUID不能作为主键
  
  - 获取系统当前时间
  
    即获取当前时间作为id，但是如果在并发度很高的情况下，会有重复的情况，所以不能作为主键。
  
    **适合的场景**：将当前时间和其他业务字段拼接起来作为一个id，组成一个全局唯一的编号。
  
  - snowflake算法
  
    snowflake算法是一种twitter开源的分布式id生成算法，采用Scala语言实现，是把一个64位的long型id，1个bit是不用的用其中41bits作为毫秒数，用10bits作为工作机器id，12bits作为序列号。
  
    - 1 bit：不用，因为二进制中如果第一个bit是1，那么代表负数，由于生成的id都需要是正数，所以第一个bit统一是0.
    - 41 bits：表示的是时间戳，单位是毫秒。41 bits 可以表示的数字多达 `2^41 - 1` ，也就是可以标识 `2^41 - 1` 个毫秒值，换算成年就是表示 69 年的时间。
    - 10 bits：记录工作机器 id，代表的是这个服务最多可以部署在 2^10 台机器上，也就是 1024 台机器。但是 10 bits 里 5 个 bits 代表机房 id，5 个 bits 代表机器 id。意思就是最多代表 `2^5` 个机房（32 个机房），每个机房里可以代表 `2^5` 个机器（32 台机器）。
    - 12 bits：记录同一个毫秒内产生的不同 id，12 bits 可以代表的最大正整数是 `2^12 - 1 = 4096` ，也就是说可以用这个 12 bits 代表的数字来区分**同一个毫秒内**的 4096 个不同的 id。
  
    代码实现：
  
    ```java
    public class IdWorker {
    
        private long workerId;
        private long datacenterId;
        private long sequence;
    
        public IdWorker(long workerId, long datacenterId, long sequence) {
            // sanity check for workerId
            // 这儿不就检查了一下，要求就是你传递进来的机房id和机器id不能超过32，不能小于0
            if (workerId > maxWorkerId || workerId < 0) {
                throw new IllegalArgumentException(
                        String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
            }
            if (datacenterId > maxDatacenterId || datacenterId < 0) {
                throw new IllegalArgumentException(
                        String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
            }
            System.out.printf(
                    "worker starting. timestamp left shift %d, datacenter id bits %d, worker id bits %d, sequence bits %d, workerid %d",
                    timestampLeftShift, datacenterIdBits, workerIdBits, sequenceBits, workerId);
    
            this.workerId = workerId;
            this.datacenterId = datacenterId;
            this.sequence = sequence;
        }
    
        private long twepoch = 1288834974657L;
    
        private long workerIdBits = 5L;
        private long datacenterIdBits = 5L;
    
        // 这个是二进制运算，就是 5 bit最多只能有31个数字，也就是说机器id最多只能是32以内
        private long maxWorkerId = -1L ^ (-1L << workerIdBits);
    
        // 这个是一个意思，就是 5 bit最多只能有31个数字，机房id最多只能是32以内
        private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
        private long sequenceBits = 12L;
    
        private long workerIdShift = sequenceBits;
        private long datacenterIdShift = sequenceBits + workerIdBits;
        private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
        private long sequenceMask = -1L ^ (-1L << sequenceBits);
    
        private long lastTimestamp = -1L;
    
        public long getWorkerId() {
            return workerId;
        }
    
        public long getDatacenterId() {
            return datacenterId;
        }
    
        public long getTimestamp() {
            return System.currentTimeMillis();
        }
    
        public synchronized long nextId() {
            // 这儿就是获取当前时间戳，单位是毫秒
            long timestamp = timeGen();
    
            if (timestamp < lastTimestamp) {
                System.err.printf("clock is moving backwards.  Rejecting requests until %d.", lastTimestamp);
                throw new RuntimeException(String.format(
                        "Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
            }
    
            if (lastTimestamp == timestamp) {
                // 这个意思是说一个毫秒内最多只能有4096个数字
                // 无论你传递多少进来，这个位运算保证始终就是在4096这个范围内，避免你自己传递个sequence超过了4096这个范围
                sequence = (sequence + 1) & sequenceMask;
                if (sequence == 0) {
                    timestamp = tilNextMillis(lastTimestamp);
                }
            } else {
                sequence = 0;
            }
    
            // 这儿记录一下最近一次生成id的时间戳，单位是毫秒
            lastTimestamp = timestamp;
    
            // 这儿就是将时间戳左移，放到 41 bit那儿；
            // 将机房 id左移放到 5 bit那儿；
            // 将机器id左移放到5 bit那儿；将序号放最后12 bit；
            // 最后拼接起来成一个 64 bit的二进制数字，转换成 10 进制就是个 long 型
            return ((timestamp - twepoch) << timestampLeftShift) | (datacenterId << datacenterIdShift)
                    | (workerId << workerIdShift) | sequence;
        }
    
        private long tilNextMillis(long lastTimestamp) {
            long timestamp = timeGen();
            while (timestamp <= lastTimestamp) {
                timestamp = timeGen();
            }
            return timestamp;
        }
    
        private long timeGen() {
            return System.currentTimeMillis();
        }
    
        // ---------------测试---------------
        public static void main(String[] args) {
            IdWorker worker = new IdWorker(1, 1, 1);
            for (int i = 0; i < 30; i++) {
                System.out.println(worker.nextId());
            }
        }
    
    }
    ```
  
    就是说 41 bit 是当前毫秒单位的一个时间戳，就这意思；然后 5 bit 是你传递进来的一个**机房** id（但是最大只能是 32 以内），另外 5 bit 是你传递进来的**机器** id（但是最大只能是 32 以内），剩下的那个 12 bit 序列号，就是如果跟你上次生成 id 的时间还在一个毫秒内，那么会把顺序给你累加，最多在 4096 个序号以内。
  
    所以你自己利用这个工具类，自己搞一个服务，然后对每个机房的每个机器都初始化这么一个东西，刚开始这个机房的这个机器的序号就是 0。然后每次接收到一个请求，说这个机房的这个机器要生成一个 id，你就找到对应的 Worker 生成。
  
    利用这个 snowflake 算法，你可以开发自己公司的服务，甚至对于机房 id 和机器 id预留了 5 bit + 5 bit，你换成别的有业务含义的东西也可以的。
  
- 晚上有字节的面试，今天之后的时间复习一下。
