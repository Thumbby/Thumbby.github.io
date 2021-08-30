---
layout: post
title:  "实习日记31(zookeeper)"
date:   2021-08-30
categories: jekyll update
---

## Day31

- 修改了一下自己的秋招简历，增加了实习经历

- Java学习，基于https://github.com/doocs/advanced-java

- zookeeper使用场景

  利用zookeeper来实现分布式锁。

  - zookeeper的常见使用场景：

    - 分布式协调

      假设A系统发送一个请求到MQ，然后B系统消息消费之后处理了，为了让A系统知道B系统的处理结果，使用zookeeper实现分布式系统之间的协调工作。A系统发送请求之后可以在zookeeper上对某个节点的值注册一个监听器，一旦B系统处理完了就修改zookeeper那个节点的值，这样A系统能立刻收到通知。

    - 分布式锁

      对于某一个数据连续发出两个修改操作，两台机器同时收到了请求，但是只能一台机器先执行完，然后另一个机器再执行。那么此时可以使用zookeeper分布式锁，一个机器接收到了请求之后先获取zookeeper上的一把分布式锁，创建一个znode，接着执行操作；然后另外一个机器执行时会尝试创建那个znode，但是因为该znode已经被创建了自己无法创建，只能等第一个机器执行完了再自己执行。

    - 元数据/配置信息管理

      zookeeper可以用作许多系统的配置信息管理，比如kafka、storm等分布式系统会选用zookeeper来做一些元数据、配置信息的管理，包括dubbo注册中心也支持zookeeper。

    - HA高可用性

      如hadoop、hdfs、yarn等很多大数据系统，都选择基于zookeeper来开发HA高可用机制，即一个重要进程一般会做主备两个，主进程挂了可以立刻通过zookeeper感知到并切换到备用进程。

- 分布式锁设计
  - Redis分布式锁

      Redis官方支持的分布式锁算法：RedLock

      这和分布式锁有三个重要考量点：

      - 互斥(只能有一个客户端获取锁)
      - 不能死锁
      - 容错(只要大部分Redis节点创建了这把锁就可以)

      **Redis最普通的分布式锁**

      最普通的实现方式，就是在Redis中使用`SET key value [EX seconds] [PX milliseconds] NX`创建一个key，这样就算加锁，其中：

      - `NX`：表示只有`key`不存在的时候才会设置成功，如果此时Redis中存在这个key，那么设置失败，返回`nil`
      - `EX seconds`：设置`key`的过期时间，精确到秒级。意思是`seconds`秒后锁自动释放，别人创建的时候如果发现已经有了就不能加锁。
      - `PX milliseconds`：同样是设置`key`的过期时间，精确到毫秒级。

      释放锁就是删除`key`，但是一啊不能可以使用lua脚本删除，判断value一样菜删除：

      ```lua
      -- 删除锁的时候，找到 key 对应的 value，跟自己传过去的 value 做比较，如果是一样的才删除。
      if redis.call("get",KEYS[1]) == ARGV[1] then
          return redis.call("del",KEYS[1])
      else
          return 0
      end
      ```

      为什么要使用`random_value`随机值：因为如果某个客户端获取到了锁，但是阻塞了很长时间才执行完，比如说超过了30s，此时可能已经自动释放锁了，此时可能别的客户端已经获取到了这个锁，要是这个时候直接删除`key`的话会有问题，所以需要用随机值和上述lua脚本来释放锁。

      使用这种方法的话，如果时普通的Redis单实例，那就是单点故障；或者是Redis普通主从，那Redis主从异步复制，如果主节点挂了(`key`没有了)，`key`还没同步到从节点，此时从节点切换到主节点，别人就可以`set key`拿到锁。

      **RedLock算法**

      这个场景假设有一个Redis cluster，有5个Redis master实例。然后执行如下步骤获取一把锁：

      - 获取当前时间戳，单位是毫秒；
      - 轮流尝试在每个master节点上创建所，超时时间较短，一般就几十毫秒(客户端为了获取锁而使用的超时时间比自动释放锁的总时间要小。例如，如果自动释放的时间是10秒，那么超时时间可能在5~50毫秒范围内)
      - 尝试在大多数节点上建立一个锁(`n/2+1`)
      - 客户端计算计算好锁的时间，如果建立锁的时间小于超时时间，就算建立成功了；
      - 要是锁建立失败了，那么就一次之前建立过的锁删除；
      - 只要别人建立了一把分布式锁，每个用户就得不断轮询去尝试获取锁。

      Redis官方给出了以上两种基于Redis实现分布式锁的方法，详细说明可以查看：https://redis.io/topics/distlock 

- zk分布式锁

  zk分布式锁就是某个节点尝试创建临时znode，此时创建成功了就获取这个锁；这个时候别的客户端来创建锁会失败，只能注册个监听器监听这个锁。释放锁就是删除这个znode，一旦释放掉就会通知客户端，然后有一个等待着的客户端就可以再次重新加锁。

  ```java
  /**
   * ZooKeeperSession
   */
  public class ZooKeeperSession {
  
      private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
  
      private ZooKeeper zookeeper;
      private CountDownLatch latch;
  
      public ZooKeeperSession() {
          try {
              this.zookeeper = new ZooKeeper("192.168.31.187:2181,192.168.31.19:2181,192.168.31.227:2181", 50000, new ZooKeeperWatcher());
              try {
                  connectedSemaphore.await();
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
  
              System.out.println("ZooKeeper session established......");
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 获取分布式锁
       *
       * @param productId
       */
      public Boolean acquireDistributedLock(Long productId) {
          String path = "/product-lock-" + productId;
  
          try {
              zookeeper.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
              return true;
          } catch (Exception e) {
              while (true) {
                  try {
                      // 相当于是给node注册一个监听器，去看看这个监听器是否存在
                      Stat stat = zk.exists(path, true);
  
                      if (stat != null) {
                          this.latch = new CountDownLatch(1);
                          this.latch.await(waitTime, TimeUnit.MILLISECONDS);
                          this.latch = null;
                      }
                      zookeeper.create(path, "".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
                      return true;
                  } catch (Exception ee) {
                      continue;
                  }
              }
  
          }
          return true;
      }
  
      /**
       * 释放掉一个分布式锁
       *
       * @param productId
       */
      public void releaseDistributedLock(Long productId) {
          String path = "/product-lock-" + productId;
          try {
              zookeeper.delete(path, -1);
              System.out.println("release the lock for product[id=" + productId + "]......");
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      /**
       * 建立 zk session 的 watcher
       */
      private class ZooKeeperWatcher implements Watcher {
  
          public void process(WatchedEvent event) {
              System.out.println("Receive watched event: " + event.getState());
  
              if (KeeperState.SyncConnected == event.getState()) {
                  connectedSemaphore.countDown();
              }
  
              if (this.latch != null) {
                  this.latch.countDown();
              }
          }
  
      }
  
      /**
       * 封装单例的静态内部类
       */
      private static class Singleton {
  
          private static ZooKeeperSession instance;
  
          static {
              instance = new ZooKeeperSession();
          }
  
          public static ZooKeeperSession getInstance() {
              return instance;
          }
  
      }
  
      /**
       * 获取单例
       *
       * @return
       */
      public static ZooKeeperSession getInstance() {
          return Singleton.getInstance();
      }
  
      /**
       * 初始化单例的便捷方法
       */
      public static void init() {
          getInstance();
      }
  
  }
  ```

  也可以采用另一种方式，创建临时顺序节点：

  如果有一把锁，被多个人给竞争，此时多个人会排队，第一个拿到锁的人会执行，然后释放锁；后面的每个人都会区监听排在自己前面的那个人创建的node上，一旦某个人释放了锁，排在自己后面的人就会被ZooKeeper给通知，一旦被通知了之后，自己就获取到了锁，就可以执行代码了。

  ```java
  public class ZooKeeperDistributedLock implements Watcher {
  
      private ZooKeeper zk;
      private String locksRoot = "/locks";
      private String productId;
      private String waitNode;
      private String lockNode;
      private CountDownLatch latch;
      private CountDownLatch connectedLatch = new CountDownLatch(1);
      private int sessionTimeout = 30000;
  
      public ZooKeeperDistributedLock(String productId) {
          this.productId = productId;
          try {
              String address = "192.168.31.187:2181,192.168.31.19:2181,192.168.31.227:2181";
              zk = new ZooKeeper(address, sessionTimeout, this);
              connectedLatch.await();
          } catch (IOException e) {
              throw new LockException(e);
          } catch (KeeperException e) {
              throw new LockException(e);
          } catch (InterruptedException e) {
              throw new LockException(e);
          }
      }
  
      public void process(WatchedEvent event) {
          if (event.getState() == KeeperState.SyncConnected) {
              connectedLatch.countDown();
              return;
          }
  
          if (this.latch != null) {
              this.latch.countDown();
          }
      }
  
      public void acquireDistributedLock() {
          try {
              if (this.tryLock()) {
                  return;
              } else {
                  waitForLock(waitNode, sessionTimeout);
              }
          } catch (KeeperException e) {
              throw new LockException(e);
          } catch (InterruptedException e) {
              throw new LockException(e);
          }
      }
  
      public boolean tryLock() {
          try {
   		    // 传入进去的locksRoot + “/” + productId
  		    // 假设productId代表了一个商品id，比如说1
  		    // locksRoot = locks
  		    // /locks/10000000000，/locks/10000000001，/locks/10000000002
              lockNode = zk.create(locksRoot + "/" + productId, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
  
              // 看看刚创建的节点是不是最小的节点
  	 	    // locks：10000000000，10000000001，10000000002
              List<String> locks = zk.getChildren(locksRoot, false);
              Collections.sort(locks);
  
              if(lockNode.equals(locksRoot+"/"+ locks.get(0))){
                  //如果是最小的节点,则表示取得锁
                  return true;
              }
  
              //如果不是最小的节点，找到比自己小1的节点
  	  int previousLockIndex = -1;
              for(int i = 0; i < locks.size(); i++) {
  		if(lockNode.equals(locksRoot + “/” + locks.get(i))) {
  	         	    previousLockIndex = i - 1;
  		    break;
  		}
  	   }
  
  	   this.waitNode = locks.get(previousLockIndex);
          } catch (KeeperException e) {
              throw new LockException(e);
          } catch (InterruptedException e) {
              throw new LockException(e);
          }
          return false;
      }
  
      private boolean waitForLock(String waitNode, long waitTime) throws InterruptedException, KeeperException {
          Stat stat = zk.exists(locksRoot + "/" + waitNode, true);
          if (stat != null) {
              this.latch = new CountDownLatch(1);
              this.latch.await(waitTime, TimeUnit.MILLISECONDS);
              this.latch = null;
          }
          return true;
      }
  
      public void unlock() {
          try {
              // 删除/locks/10000000000节点
              // 删除/locks/10000000001节点
              System.out.println("unlock " + lockNode);
              zk.delete(lockNode, -1);
              lockNode = null;
              zk.close();
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (KeeperException e) {
              e.printStackTrace();
          }
      }
  
      public class LockException extends RuntimeException {
          private static final long serialVersionUID = 1L;
  
          public LockException(String e) {
              super(e);
          }
  
          public LockException(Exception e) {
              super(e);
          }
      }
  }
  ```

  但是，使用zk临时节点会存在另一个问题：由于 zk 依靠 session 定期的心跳来维持客户端，如果客户端进入长时间的 GC，可能会导致 zk 认为客户端宕机而释放锁，让其他的客户端获取锁，但是客户端在 GC 恢复后，会认为自己还持有锁，从而可能出现多个客户端同时获取到锁的情形。针对这种情况，可以通过JVM调优。

- redis分布式锁和zk分布式锁的对比

  - redis 分布式锁，其实**需要自己不断去尝试获取锁**，比较消耗性能。
  - zk 分布式锁，获取不到锁，注册个监听器即可，不需要不断主动尝试获取锁，性能开销较小。

  另外一点就是，如果是 Redis 获取锁的那个客户端 出现 bug 挂了，那么只能等待超时时间之后才能释放锁；而 zk 的话，因为创建的是临时 znode，只要客户端挂了，znode 就没了，此时就自动释放锁。
  
- 同学问了个问题感觉很有意思记录一下：

  **问题**：Arrays.stream().forEach()用lambda遍历出现Variable used in lambda expression should be final or effectively final报错

  **解决过程**：首先发现报错信息意思是`lambda 表达式中使用的变量应该是 final 或者有效的 final`

  今天没时间了明天解决

