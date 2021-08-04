---
layout: post
title:  "实习日记7"
date:   2021-07-22
categories: jekyll update
---

## Day7

- Mybatis-plus基本配置文档：https://mp.baomidou.com/config/generator-config.html#%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE

- 数据库架构和数据表关系学习

   - 每张表拥有一个字段id或ID_作为唯一主键，存在主键为外键情况，且该主键必须非空，存在有表不以id作为主键的情况，这种情况下表不含id字段（act_id_membership)作为联系集以其两个字段统一作为主键。 	
   - 有关联的数据表：
     - act_id_user和act_id_group,存在联系集act_id_membership,应该是表示用户和某一用户集合（群组）
     - 模型相关一系列:act_re_procdef和其信息act_procdef_info,

- Redis学习：

   - Redis Stream:主要用于消息队列，提供了消息的持久化和主备复制功能，可以让客户端访问任何时候的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。

     - Redis stream结构如下：

       ![img](https://www.runoob.com/wp-content/uploads/2020/09/en-us_image_0167982791.png)

       每个Stream拥有唯一的名称作为Redis的key，在首次使用xadd指令追加消息时会自动创建，其中：

       - **Consumer Group** ：消费组，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者(Consumer)。
       - **last_delivered_id** ：游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。
       - **pending_ids** ：消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id。 pending_ids 记录了当前已经被客户端读取的消息，但是还没有 ack (Acknowledge character：确认字符）。

     - 消息队列相关命令如下：

       - **XADD** - 添加消息到末尾
       - **XTRIM** - 对流进行修剪，限制长度
       - **XDEL** - 删除消息
       - **XLEN** - 获取流包含的元素数量，即消息长度
       - **XRANGE** - 获取消息列表，会自动过滤已经删除的消息
       - **XREVRANGE** - 反向获取消息列表，ID 从大到小
       - **XREAD** - 以阻塞或非阻塞方式获取消息列表

     - 消费者组相关命令：

       - **XGROUP CREATE** - 创建消费者组
       - **XREADGROUP GROUP** - 读取消费者组中的消息
       - **XACK** - 将消息标记为"已处理"
       - **XGROUP SETID** - 为消费者组设置新的最后递送消息ID
       - **XGROUP DELCONSUMER** - 删除消费者
       - **XGROUP DESTROY** - 删除消费者组
       - **XPENDING** - 显示待处理消息的相关信息
       - **XCLAIM** - 转移消息的归属权
       - **XINFO** - 查看流和消费者组的相关信息；
       - **XINFO GROUPS** - 打印消费者组的信息；
       - **XINFO STREAM** - 打印流信息

     - 具体命令详解可参考官方文档和runoob：

       - 官方文档：https://redis.io/topics/streams-intro
       - runoob：https://www.runoob.com/redis/redis-stream.html
       - 概念解释可以看这篇博客：https://www.cnblogs.com/williamjie/p/11201654.html

- Linux学习(因为没事无聊学一点)：

   - Linux启动过程：

     - 内核的引导：当计算机打开电源后，首先是BIOS开机自检，按照BIOS中设置的启动设备（通常是硬盘）来启动。

       操作系统接管硬件以后，首先读入 /boot 目录下的内核文件。

     - 运行 init：init 进程是系统所有进程的起点，没有这个进程，系统中任何进程都不会启动。init 程序首先是需要读取配置文件 /etc/inittab。

       - 运行级别：许多程序需要开机启动。它们在Windows叫做"服务"（service），在Linux就叫做"守护进程"（daemon）。

         init进程的一大任务，就是去运行这些开机启动的程序。

         但是，不同的场合需要启动不同的程序，比如用作服务器时，需要启动Apache，用作桌面就不需要。

         Linux允许为不同的场合，分配不同的开机启动程序，这就叫做"运行级别"（runlevel）。也就是说，启动时根据"运行级别"，确定要运行哪些程序。

         - 运行级别0：系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
         - 运行级别1：单用户工作状态，root权限，用于系统维护，禁止远程登陆
         - 运行级别2：多用户状态(没有NFS)
         - 运行级别3：完全的多用户状态(有NFS)，登陆后进入控制台命令行模式
         - 运行级别4：系统未使用，保留
         - 运行级别5：X11控制台，登陆后进入图形GUI模式
         - 运行级别6：系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动

     - 系统初始化：在init的配置文件中有这么一行： si::sysinit:/etc/rc.d/rc.sysinit　它调用执行了/etc/rc.d/rc.sysinit，而rc.sysinit是一个bash shell的脚本，它主要是完成一些系统初始化的工作，rc.sysinit是每一个运行级别都要首先运行的重要脚本。它主要完成的工作有：激活交换分区，检查磁盘，加载硬件模块以及其它一些需要优先执行任务。

       ```
       l5:5:wait:/etc/rc.d/rc 5
       ```

       这一行表示以5为参数运行/etc/rc.d/rc，/etc/rc.d/rc是一个Shell脚本，它接受5作为参数，去执行/etc/rc.d/rc5.d/目录下的所有的rc启动脚本，/etc/rc.d/rc5.d/目录中的这些启动脚本实际上都是一些连接文件，而不是真正的rc启动脚本，真正的rc启动脚本实际上都是放在/etc/rc.d/init.d/目录下。

       而这些rc启动脚本有着类似的用法，它们一般能接受start、stop、restart、status等参数。

       /etc/rc.d/rc5.d/中的rc启动脚本通常是K或S开头的连接文件，对于以 S 开头的启动脚本，将以start参数来运行。

       而如果发现存在相应的脚本也存在K打头的连接，而且已经处于运行态了(以/var/lock/subsys/下的文件作为标志)，则将首先以stop为参数停止这些已经启动了的守护进程，然后再重新运行。

       这样做是为了保证是当init改变运行级别时，所有相关的守护进程都将重启。

       至于在每个运行级中将运行哪些守护进程，用户可以通过chkconfig或setup中的"System Services"来自行设定。

     - 建立终端 ：rc执行完毕后，返回init。这时基本系统环境已经设置好了，各种守护进程也已经启动了。init接下来会打开6个终端，以便用户登录系统。

     - 用户登录系统：一般来说，用户的登录方式有三种：

       - （1）命令行登录
       - （2）ssh登录
       - （3）图形界面登录

     - 具体流程图：![img](https://www.runoob.com/wp-content/uploads/2014/06/bg2013081706.png)

   - Linux关机：在linux领域内大多用在服务器上，很少遇到关机的操作。毕竟服务器上跑一个服务是永无止境的，除非特殊情况下，不得已才会关机。

     正确的关机流程为：sync > shutdown > reboot > halt

     关机指令为：shutdown ，你可以man shutdown 来看一下帮助文档。

- 明天开始要干活儿，可能会有数据同步的工作，明天再记。

