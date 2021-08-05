---
layout: post
title:  "实习日记16(消息队列)"
date:   2021-08-05
categories: jekyll update
---

## Day16

- ##### Java学习，基于https://github.com/doocs/advanced-java


- 消息队列(MQ)

  - 常用场景：
    - 解耦：

      例子：系统A需要调用其他系统的接口或者其他系统需要系统A发送数据，那么A系统需要考虑到其他系统是否会中断之类的情况。

    ![mq-1](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-1.png)

    ​	在这种情况下可以使用消息队列，如果A产生一条数据，发送到消息队列中，其它系统只需在消息队列中取得数据，这样一来系统A无需考虑向哪个系统发送数据、是否调用成功等情况，从而可以与其他系统解耦。注意，A系统需要与其他系统保持非直接同步调用(即A不用等待其他接口返回结果后才能继续工作)

    ![mq-2](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-2.png)

    - 异步：

      例子：A系统需要在本地写库并在B,C,D三个系统写库，如果在不使用消息队列的情况下，总共花费的时长为A写库时间+B写库时间+C写库时间+D写库时间，随着需要被写库的系统增多花费的时长会越来越多。

      ![mq-3](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-3.png)

      如果使用消息队列的话，A只需要把B,C,D库需要的消息发送到3个MQ中，由B,C,D三个库按自己需要取数据，那么总共花费的时长仅为A本地写库时间+A向MQ中写数据时间，时间开销会大大减少。

      ![mq-4](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-4.png)

    - 削峰：

      例子：假如系统基于MySQL实现，并且系统每天存在一个高峰期，在该高峰期内请求频率和数量大幅增加超过了MySQL的负担，这很可能导致系统崩溃，但是高峰期一过，请求数量和频率大幅降低，对系统不会产生什么负担。

    ![mq-5](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-5.png)

    ​	如果使用消息队列的话，假设每秒写入5k个请求到MQ，A系统每秒能够处理的请求数量为MySQL的上线2k个，只要不断的从MQ中取出不超过自己负担个请求进行处理即可；对于MQ，在高峰期中会有大量的请求堆积在MQ中，但这只是短期的积压，因为高峰期过后每秒流入MQ的请求数量大幅减少，这样A系统能够快速将MQ中堆积的信息处理掉。

- 消息队列的缺点

  - 系统可用性降低：如果MQ出现问题会导致整套系统的崩溃，为此需要保证消息队列的高可用性。
  - 系统复杂度提高：需要保证消息没有重复消费，需要处理信息丢失的情况，需要保证消息传递的顺序性等，会产生许多问题，导致系统整体的复杂度提高。
  - 一致性问题：因为使用了消息队列的情况下，A系统只要成功写入MQ则会返回成功，加入B,C,D三个库中出现了写入不成功的情况，则会导致数据不一致。

- 常见的MQ及其优缺点：

  | 特性                    | ActiveMQ                            | RabbitMQ                                         | RocketMQ                                                     | Kafka                                                        |
  | ----------------------- | ----------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 单机吞吐量              | 万级，比RocketMQ和Kafka低一个数量级 | 同ActiveMQ                                       | 10万级，支持高吞吐                                           | 10万级别，高吞吐，一般配合大数据类的系统来实现实时数据计算、日志采集等场景 |
  | topic数量对吞吐量的情况 |                                     |                                                  | topic可以达到几百/几千的级别，吞吐量会有小幅度的下降，这是RocketMQ的一大优势，在同等机器下，可以支撑大量的topic | topic 从几十到几百个时候，吞吐量会大幅度下降，在同等机器下，Kafka 尽量保证 topic 数量不要过多，如果要支撑大规模的 topic，需要增加更多的机器资源 |
  | 时效性                  | ms级                                | 微秒级，这是RabbitMQ一大特点，延迟最低           | ms级                                                         | 延迟在ms级以内                                               |
  | 可用性                  | 高，基于主从架构实现高可用          | 同ActiveMQ                                       | 非常高，分布式架构                                           | 非常高，分布式，一个数据多个副本，少数机器宕机不会丢失数据，不会导致不可用 |
  | 消息可靠性              | 有较低的概率丢失数据                | 基本不丢                                         | 经过参数优化配置，可以做到0丢失                              | 同RokcetMQ                                                   |
  | 功能支持                | MQ领域的功能及其完备                | 基于erlang开发，并发能力很强，性能极好，延时很低 | MQ功能较为完善，还是分布式，扩展性好                         | 功能较为简单，主要支持简单的 MQ 功能，在大数据领域的实时计算以及日志采集被大规模使用 |

- 保证消息队列的高可用

  - RabbitMQ的高可用性，其基于主从(非分布式)做高可用性，有三种模式：

    - 单机模式：demo级别，不投入生产

    - 普通集群模式(无高可用性)：在多台机器上启动多个RabbitMQ实例，每个机器启动一个。用户创建的队列只会放在一个RabbitMQ实例上，但是每个实例都会同步queue的元数据(元数据可以认为是queue的一些配置信息，通过元数据，可以找到queue的所在实例)，在用户消费的时候，实际上如果连接的是另一个实例，该实例会从队列所在实例上拉取数据过来。

      ![mq-7](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-7.png)

      这种方式确实很麻烦，也不怎么好，没做到所谓的分布式，就是个普通集群。因为这导致你要么消费者每次随机连接一个实例然后拉取数据，要么固定连接那个 queue 所在实例消费数据，前者有数据拉取的开销，后者导致单实例性能瓶颈。

      而且如果那个放 queue 的实例宕机了，会导致接下来其他实例就无法从那个实例拉取，如果你开启了消息持久化，让 RabbitMQ 落地存储消息的话，消息不一定会丢，得等这个实例恢复了，然后才可以继续从这个 queue 拉取数据。

      这种方案没有什么所谓的高可用性**，**这方案主要是提高吞吐量的，就是说让集群中多个节点来服务某个 queue 的读写操作。

    - 镜像集群模式(高可用性)

      这种模式才是RabbitMQ的高可用模式，在镜像集群模式下，用户创建的queue，无论元数据还是queue里的消息都会存在于多个实力上，即每个RabbitMQ节点都有一个这个queue的完整镜像，包含queue的全部数据。每次用户写消息到queue的时候，都会自动把消息同步到多个实例的queue上。

      ![mq-8](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-8.png)

      要开启这个集群模式，RabbitMQ有相应的管理控制台，就是在后台增加一个镜像集群模式的策略，指定的时候可以要求数据同步到所有节点，也可以要求同步到指定数量的节点，再次创建queue的时候，使用这个策略能够自动将数据同步到其他节点上。

      这样做的好处在于某一节点宕机不影响其他节点(因为其他机器节点同样包含这个queue的完整数据)，别的消费者可以到其他节点消费数据。但坏处在于开销过大，消息需要同步到所有机器上，导致网络宽带压力和消耗很重；其次，这样不是分布式的，没有扩展性，如果某个队列负载很重，即使新加机器的机器也要包含这个队列的所有数据，导致无法线性扩展队列，如果queue太大导致机器无法容纳则会出现数据无法存储之类的问题。

  - Kafka的高可用性：

    - Kafka的基础架构——天然的分布式消息队列：由多个broker组成，每个broker是一个节点；当用户创建一个topic的时候，这个topic可以划分为多个partition，每个partition可以存在于不同的broker上，每个partition存放一部分数据。即一个topic的数据，是分散在多个机器上的，每个机器就放一部分数据

    - Kafka 0.8 以前，是没有 HA 机制的，就是任何一个 broker 宕机了，那个 broker 上的 partition 就废了，没法写也没法读，没有什么高可用性可言。比如说，我们假设创建了一个 topic，指定其 partition 数量是 3 个，分别在三台机器上。但是，如果第二台机器宕机了，会导致这个 topic 的 1/3 的数据就丢了，因此这个是做不到高可用的。

      ![kafka-before](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/kafka-before.png)

      Kafka 0.8 以后，提供了 HA 机制，就是 replica（复制品） 副本机制。每个 partition 的数据都会同步到其它机器上，形成自己的多个 replica 副本。所有 replica 会选举一个 leader 出来，那么生产和消费都跟这个 leader 打交道，然后其他 replica 就是 follower。写的时候，leader 会负责把数据同步到所有 follower 上去，读的时候就直接读 leader 上的数据即可。要是用户可以随意读写每个 follower，那么就要 care 数据一致性的问题，系统复杂度太高，很容易出问题。所以Kafka 会均匀地将一个 partition 的所有 replica 分布在不同的机器上，这样才可以提高容错性。

      ![kafka-after](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/kafka-after.png)

      如果某个 broker 宕机了，那个 broker 上面的 partition 在其他机器上都有副本的。如果这个宕机的 broker 上面有某个 partition 的 leader，那么此时会从 follower 中重新选举一个新的 leader 出来，大家继续读写那个新的 leader 即可。这就有所谓的高可用性了。

      写数据的时候，生产者就写 leader，然后 leader 将数据落地写本地磁盘，接着其他 follower 自己主动从 leader 来 pull 数据。一旦所有 follower 同步好数据了，就会发送 ack 给 leader，leader 收到所有 follower 的 ack 之后，就会返回写成功的消息给生产者。（当然，这只是其中一种模式，还可以适当调整这个行为）

      消费的时候，只会从 leader 去读，但是只有当一个消息已经被所有 follower 都同步成功返回 ack 的时候，这个消息才会被消费者读到。

- 如何保证队列消息不被重复消费(本质上是使用消息队列如何保证消息消费的幂等性)

  - 可能出现重复消费的消息队列：RabbitMQ，RocketMQ，Kafka

    - Kafka 实际上有个 offset 的概念，就是每个消息写进去，都有一个 offset，代表消息的序号，然后 consumer 消费了数据之后，每隔一段时间（定时定期），会把自己消费过的消息的 offset 提交一下，表示“我已经消费过了，下次我要是重启啥的，你就让我继续从上次消费到的 offset 来继续消费吧”。

      但是凡事总有意外，比如用户有时候重启系统，情况着急的，直接 kill 进程了，再重启。这会导致 consumer 有些消息处理了，但是没来得及提交 offset，尴尬了。重启之后，少数消息会再次消费一次。

      例子：有这么个场景。数据 1/2/3 依次进入 Kafka，Kafka 会给这三条数据每条分配一个 offset，代表这条数据的序号，我们就假设分配的 offset 依次是 152/153/154。消费者从 Kafka 去消费的时候，也是按照这个顺序去消费。假如当消费者消费了 `offset=153` 的这条数据，刚准备去提交 offset 到 Zookeeper，此时消费者进程被重启了。那么此时消费过的数据 1/2 的 offset 并没有提交，Kafka 也就不知道你已经消费了 `offset=153` 这条数据。那么重启之后，消费者会找 Kafka 说，嘿，哥儿们，你给我接着把上次我消费到的那个地方后面的数据继续给我传递过来。由于之前的 offset 没有提交成功，那么数据 1/2 会再次传过来，如果此时消费者没有去重的话，那么就会导致重复消费。

      注意：新版的 Kafka 已经将 offset 的存储从 Zookeeper 转移至 Kafka brokers，并使用内部位移主题 `__consumer_offsets` 进行存储。

      ![mq-10](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-10.png)

    - 如何保证幂等性：幂等性，通俗点说，就一个数据，或者一个请求，给你重复来多次，你得确保对应的数据是不会改变的，不能出错。

      最基础的做法，假设每次消费就是往数据库中插入一条数据，那么在每次消费的时候，判断一下自己是否已经消费过了，若是就抛弃该数据，确保多次消费只会保留一次数据，数据库中数据不会改变。

    - 如何保证消息队列消费的幂等性：具体实现需要结合业务，在这里该博客提供了如下思路：

      - 比如你拿个数据要写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下好吧。
      - 比如你是写 Redis，那没问题了，反正每次都是 set，天然幂等性。
      - 比如你不是上面两个场景，那做的稍微复杂一点，你需要让生产者发送每条数据的时候，里面加一个全局唯一的 id，类似订单 id 之类的东西，然后你这里消费到了之后，先根据这个 id 去比如 Redis 里查一下，之前消费过吗？如果没有消费过，你就处理，然后这个 id 写 Redis。如果消费过了，那你就别处理了，保证别重复处理相同的消息即可。
      - 比如基于数据库的唯一键来保证重复数据不会重复插入多条。因为有唯一键约束了，重复数据插入只会报错，不会导致数据库中出现脏数据。

      ![mq-11](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/mq-11.png)

- 如何保证消息的可靠性传输：即在数据传输过程中不能把数据弄丢。数据的丢失问题，可能出现在生产者、MQ、消费者中。下面从RabbitMQ和Kafka来分析：

  - RabbitMQ：

    ![rabbitmq-message-lose](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/rabbitmq-message-lose.png)

    **生产者将数据发送到RabbitMQ的时候数据丢失**：可能是网络原因等。此时可以选择使用RabbitMQ提供的事务功能，在生产者发送数据之前开启RabbitMQ事务channel.txSelect，然后发送消息，如果消息没有成功被RabbitMQ接收到，那么生产者会收到异常报错，此时可以回滚事务channel.txRollback，然后重试发送信息；如果收到了消息，那么可以提交事务channel.txCommit。
    
    代码如下：
    
    ```java
    // 开启事务
    channel.txSelect
    try {
        // 这里发送消息
    } catch (Exception e) {
        channel.txRollback
    
        // 这里再次重发这条消息
    }
    
    // 提交事务
    channel.txCommit
    ```
    
    但是RabbitMQ事务机制(同步)因为太耗性能导致吞吐量会下降。所以一般情况下要确保RabbitMQ消息不丢失，可以开启confirm模式，在生产者处设置confirm模式后，用户每次写的消息会分配一个单独的id，然后如果该消息写入了RabbitMQ中，RabbitMQ会返回用户一个ack消息，表明消息已成功传输。如果RabbitMQ没能处理这个消息，会回调用户一个nack接口，告诉用户这个消息接收失败，用户可以重试发送消息。用户可以结合这个机制自己在内存里维护每个消息id的状态，如果超过一定时间还没接收到这个消息的回调，那么用户可以重发。
    
    事务机制和confirm机制最大的不同在于，事务机制是同步的，你提交一个事务之后会阻塞在那儿，但是confirm机制是异步的，你发送个消息之后就可以发送下一个消息，然后那个消息 RabbitMQ 接收了之后会异步回调你的一个接口通知你这个消息接收到了。
    
    所以一般在生产者这块避免数据丢失，都是用 confirm机制的。
    
    **RabbitMQ弄丢了数据**：此时需要开启RabbitMQ的持久化，即消息写入之后会持久化到磁盘，哪怕是RabbitMQ自己挂了，恢复之后会自动读取之前存储的数据，一般数据不会丢失。除非及其罕见的是，RabbitMQ还没持久化就挂了，可能会导致少量的数据丢失，但是这种情况发生的概率较小。
    
    如何设置持久化：
    
    - 创建queue的时候将其设置为持久化，这样就可以保证 RabbitMQ 持久化 queue 的元数据，但是它是不会持久化 queue 里的数据的。
    
    - 发送消息的时候将消息的deliveryMode设置为 2，即将消息设置为持久化的，此时RabbitMQ就会将消息持久化到磁盘上去。必须要同时设置这两个持久化才行，这样RabbitMQ 哪怕是挂了，再次重启，也会从磁盘上重启恢复 queue，恢复这个 queue 里的数据。
    
      **注意**：哪怕是给 RabbitMQ 开启了持久化机制，也有一种可能，就是这个消息写到了 RabbitMQ 中，但是还没来得及持久化到磁盘上，结果不巧，此时 RabbitMQ 挂了，就会导致内存里的一点点数据丢失。
    
      ​			所以，持久化可以跟生产者那边的confirm机制配合起来，只有消息被持久化到磁盘之后，才会通知生产者ack了，所以哪怕是在持久化到磁盘之前，RabbitMQ 挂了，数据丢了，生产者收不到ack，你也是可以自己重发的。
    
    **消费端弄丢了数据**：RabbitMQ消息丢失的主要原因，即刚消费到，还没处理，结果进程挂了，此时RabbitMQ会认为用户已经消费，这样会导致数据丢失。
    
    这时可以使用RabbitMQ提供的ack机制，即用户必须关闭RabbitMQ的自动ack，可以通过一个api来调用，每次用户自己代码里确保处理完的时候，再在程序里执行ack。这样如果用户没处理完，RabbitMQ也会认为用户没处理完成，这个时候 RabbitMQ 会把这个消费分配给别的 consumer 去处理，消息是不会丢的。
    
    ![rabbitmq-message-lose-solution](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/rabbitmq-message-lose-solution.png)
    
  - Kafka

    **消费端弄丢了数据**：

    唯一可能导致消费者弄丢数据的情况，就是说，你消费到了这个消息，然后消费者那边自动提交了 offset，让 Kafka 以为你已经消费好了这个消息，但其实你才刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。

    这不是跟 RabbitMQ 差不多吗，大家都知道 Kafka 会自动提交 offset，那么只要关闭自动提交 offset，在处理完之后自己手动提交 offset，就可以保证数据不会丢。但是此时确实还是可能会有重复消费，比如你刚处理完，还没提交 offset，结果自己挂了，此时肯定会重复消费一次，自己保证幂等性就好了。

    生产环境碰到的一个问题，就是说我们的 Kafka 消费者消费到了数据之后是写到一个内存的 queue 里先缓冲一下，结果有的时候，你刚把消息写入内存 queue，然后消费者会自动提交 offset。然后此时我们重启了系统，就会导致内存 queue 里还没来得及处理的数据就丢失了。

    **Kafka弄丢了数据**：

    这是个比较常见的场景，即Kafka某个broker宕机，然后需要重新选举partition的leader，如果恰好此时其他follower有些数据没有同步，在选取某个follower为leader之后，会出现数据丢失。

    此时一般要求配置如下4个参数：

    - 给 topic 设置 `replication.factor` 参数：这个值必须大于 1，要求每个 partition 必须有至少 2 个副本。
    - 在 Kafka 服务端设置 `min.insync.replicas` 参数：这个值必须大于 1，这个是要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系，没掉队，这样才能确保 leader 挂了还有一个 follower 吧。
    - 在 producer 端设置 `acks=all` ：这个是要求每条数据，必须是写入所有 replica 之后，才能认为是写成功了。
    - 在 producer 端设置 `retries=MAX` （很大很大很大的一个值，无限次重试的意思）：这个是要求一旦写入失败，就无限重试，卡在这里了

    这样配置之后，至少在 Kafka broker 端就可以保证在 leader 所在 broker 发生故障，进行 leader 切换时，数据不会丢失。

    **生产者是否会弄丢数据**：

    如果按照上述的思路设置了 `acks=all` ，一定不会丢，要求是，leader 接收到消息，所有的 follower 都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。

- 如何保证消息的顺序性：

  首先看顺序错乱的两个具体例子：

  - RabbitMQ：一个 queue，多个 consumer。比如，生产者向 RabbitMQ 里发送了三条数据，顺序依次是 data1/data2/data3，压入的是 RabbitMQ 的一个内存队列。有三个消费者分别从 MQ 中消费这三条数据中的一条，结果消费者 2 先执行完操作，把 data2 存入数据库，然后是 data1/data3。明显顺序乱了。

    ![rabbitmq-order-01](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/rabbitmq-order-01.png)

  - Kafka：假设用户建了一个 topic，有三个 partition。生产者在写的时候，其实可以指定一个 key，比如说用户指定了某个订单 id 作为 key，那么这个订单相关的数据，一定会被分发到同一个 partition 中去，而且这个 partition 中的数据一定是有顺序的。
    消费者从 partition 中取出来数据的时候，也一定是有顺序的。到这里，顺序还是 ok 的，没有错乱。接着，我们在消费者里可能会搞多个线程来并发处理消息。因为如果消费者是单线程消费处理，而处理比较耗时的话，比如处理一条消息耗时几十 ms，那么 1 秒钟只能处理几十条消息，这吞吐量太低了。而多个线程并发跑的话，顺序可能就乱掉了。

    ![kafka-order-01](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/kafka-order-01.png)

  解决方法：

  - RabbitMQ：拆分多个 queue，每个 queue 一个 consumer，就是多一些 queue 而已，确实是麻烦点；或者就一个 queue 但是对应一个 consumer，然后这个 consumer 内部用内存队列做排队，然后分发给底层不同的 worker 来处理。

    ![rabbitmq-order-02](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/rabbitmq-order-02.png)

  - Kafka：

    - 一个 topic，一个 partition，一个 consumer，内部单线程消费，单线程吞吐量太低，一般不会用这个。
    - 写 N 个内存 queue，具有相同 key 的数据都到同一个内存 queue；然后对于 N 个线程，每个线程分别消费一个内存 queue 即可，这样就能保证顺序性。

    ![kafka-order-02](https://github.com/doocs/advanced-java/raw/main/docs/high-concurrency/images/kafka-order-02.png)

