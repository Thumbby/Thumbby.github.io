---
layout: post
title:  "实习日记3(redis+lua)"
date:   2021-07-16
categories: jekyll update
---

## Day3

- redis学习：

   - redis所有hash命令，见官方文档：https://redis.io/commands#hash

   - redis数据备份和数据恢复：

     - 登录redis客户端后输入 SAVE 即可备份，该命令是阻塞式的，执行 SAVE 之后会在安装目录下创建dump.rdb文件，数据回复只需将对应的dump.rdb文件复制进安装目录即可。
     - 另一种redis备份的方式是执行 BGSAVE 命令，改命令是非阻塞式的，使用redis fork创建一个子进程来负责写入磁盘，这样父进程可以继续处理命令，可以实现热保存，但是内存使用会提高。
     - 可以设置根据多长时间内的写操作数量来自动触发bgsave：save <seconds> <changes>，表示在second秒内如果有changes数次写入操作就保存
     - redis-cli执行shutdown操作同样会更新dump.rdb文件
     - 如果是redis-cli中通过slaveof变成slave redis，同样会生成dump.rdb文件。

   - redis密码相关：

     - redis设置密码：在客户端中执行CONFIG set requirepass "<password>"将密码更新为password，或者修改redis.conf文件，

       ```t
       #requirepass foobared
       requirepass <password>
       ```

     - 查看是否设置了密码验证：在客户端中执行CONFIG get requirepass

     - 在有密码的情况下登录客户端：redis-Cli -p <password>

   - redis性能测试：

     - 在redis目录下(非客户端)执行如下命令：redis-benchmark [option] [option value]，redis性能测试可选工具如下：

       | 序号 | 选项               | 描述                                       | 默认值              |
       | ---- | ------------------ | ------------------------------------------ | ------------------- |
       | 1    | -h                 | 指定服务器主机名                           | 127.0.0.1           |
       | 2    | -p                 | 指定服务器端口                             | 6379(redis默认端口) |
       | 3    | -s                 | 指定服务器socket                           |                     |
       | 4    | -c                 | 指定并发连接数                             | 50                  |
       | 5    | -n                 | 绑定请求数                                 | 10000               |
       | 6    | -d                 | 以字节的形式指定SET/GET值的数据大小        | 2                   |
       | 7    | -k                 | 1=keep；alive 0=reconnect                  | 1                   |
       | 8    | -r                 | SET/GET/INCR 使用随机 key, SADD 使用随机值 |                     |
       | 9    | -P                 | 通过管道传输 <numreq> 请求                 | 1                   |
       | 10   | -q                 | 强制退出 redis。仅显示 query/sec 值        |                     |
       | 11   | --csv              | 以 CSV 格式输出                            |                     |
       | 12   | -l（L 的小写字母） | 生成循环，永久执行测试                     |                     |
       | 13   | -t                 | 仅运行以逗号分隔的测试命令列表。           |                     |
       | 14   | -I（i 的大写字母） | Idle 模式。仅打开 N 个 idle 连接并等待。   |                     |

   - redis客户端连接:redis通过监听一个TCP端口或者Unix Socket方式来接收客户端的连接，当一个连接建立后redis会执行如下操作：

     - 设置客户端socket为非阻塞模式，因为redis在网络事件的处理上采用的是非阻塞多路复用模型
     - 为该socket设置TCP_NODELAY属性，禁用Nagle算法，关于Nagle算法可看：https://www.cnblogs.com/postw/p/9710772.html和https://blog.csdn.net/wdscq1234/article/details/52432095
     - 创建一个可读的文件事件用于监听这个客户端socket的数据发送

     redis客户端连接命令：redis-cli -h host -p port -a password

     ​	host:远程redis服务器host

     ​	port:远程redis服务端口

     ​	password:远程redis服务密码（无密码的的话就不需要-a参数了）	

     redis设置客户端最大连接数：redis-server --maxclients 100000，此处将最大连接数设置为100000

     常见客户端命令：

     | 命令           | 描述                                       |
     | -------------- | ------------------------------------------ |
     | CLIENT LIST    | 返回连接到redis的客户端列表                |
     | CLIENT SETNAME | 设置当前连接的名称                         |
     | CLIENT GETNAME | 获取通过CLIENT SETNAME设置的服务名称       |
     | CLIENT PAUSE   | 挂起客户端连接，指定挂起的时间以毫秒为单位 |
     | CLIENT KILL    | 关闭客户端连接                             |

   - redis管道技术：

     - redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务，redis进行一个请求会遵循如下步骤：
       - 客户端向服务端发送一个查询请求，并监听socket返回，通常是以阻塞模式，等待服务端相应
       - 服务端处理命令， 并将结果返回给客户端
     - 使用redis管道则可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取服务端的响应
       - 原理是redis将多条命令打包，只需要一次网络开销（不固定），在服务端和客户端各一次read和write系统调用，以此来节约时间。
       - 需要注意的是管道中的命令数量要适量，并非越多越好，因为管道中的命令数量过大时会加重读写负担。
       - redis2.6版本后，脚本在大部分场景中表现要优于管道。

   - redis脚本：

     - Redis脚本使用Lua解释器来执行脚本，Redis2.6版本内嵌了Lua环境。执行脚本通常使用EVAL命令，基本语法如下：

       ```
       EVAL script numkeys key [key ...] arg [arg ...]
       ```

       numkeys是key的个数，后边写key1 key2 key3...val1 val2 val3

       redis常用脚本命令如下：

       | 命令                                       | 描述                                               |
       | ------------------------------------------ | -------------------------------------------------- |
       | EVAL script numkeys key[key...]arg[arg...] | 执行Lua脚本                                        |
       | EVALSHA sha1numkeys key[key...]arg[arg...] | 根据缓存码执行Lua脚本                              |
       | SCRIPT EXISTS script [script...]           | 查看指定脚本是否已经被保存在缓存中                 |
       | SCRIPT FLUSH                               | 从脚本缓存中移除所有的脚本                         |
       | SCRIPT KILL                                | 杀死当前正在运行的Lua脚本                          |
       | SCRIPT LOAD script                         | 将脚本script添加到脚本缓存中，但不立即执行这个脚本 |

     - redis脚本的主要优势：

       - 减少网络开销：多个请求通过脚本一次发送，减少网络延迟
       - 原子操作：将脚本作为一个整体执行，中间不会插入其他命令，无需使用事务
       - 复用：客户端发送的脚本永久存在redis中，其他客户端可以复用脚本
       - 可嵌入性：可嵌入JAVA，C#等多种编程语言，支持不同操作系统跨平台交互

     - redis脚本的使用：通常将lua script放到一个lua文件中，然后执行这个lua脚本，例子如下：

       ```lua
       if redis.call("EXISTS",KEYS[1]) == 1 then
            return redis.call("INCRBY",KEYS[1],ARGV[1])
          else
            return nil
          end
       ```

       假设存储位置为/Users/jihite/activeuser.lua，则执行代码如下：

       ```
       $ redis-cli --eval /Users/jihite/activeuser.lua user , 1
       (integer) 1
       
       127.0.0.1:6379> get user
       "1"
       127.0.0.1:6379> exit
       $ redis-cli --eval /Users/jihite/activeuser.lua user , 1
       (integer) 2
       $ redis-cli 
       127.0.0.1:6379> get user
       "2"
       127.0.0.1:6379> exit
       $ redis-cli --eval /Users/jihite/activeuser.lua user , 4
       (integer) 6
       ```

       redis对lua环境做了如下措施：

       - 不提供访问系统状态状态的库（比如系统时间库）
       - 禁止使用 loadfile 函数
       - 如果脚本在执行带有随机性质的命令（比如 RANDOMKEY ），或者带有副作用的命令（比如 TIME ）之后，试图执行一个写入命令（比如 SET ），那么 Redis 将阻止这个脚本继续运行，并返回一个错误。
       - 如果脚本执行了带有随机性质的读命令（比如 SMEMBERS ），那么在脚本的输出返回给 Redis 之前，会先被执行一个自动的字典序排序，从而确保输出结果是有序的。
       - 用 Redis 自己定义的随机生成函数，替换 Lua 环境中 `math` 表原有的 math.random 函数和 math.randomseed 函数，新的函数具有这样的性质：每次执行 Lua 脚本时，除非显式地调用 `math.randomseed` ，否则 `math.random` 生成的伪随机数序列总是相同的。

       lua的学习可见第二部分

- lua的基础学习：

   - 数据类型：lua语言有以下8种基础数据类型：

     | 数据类型 | 描述                                                         |
     | -------- | ------------------------------------------------------------ |
     | nil      | 只有值nil属于该类，表示一个无效值(在条件表达式中相当于false)，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉 |
     | boolean  | 布尔值，有true和false两种值                                  |
     | number   | 双精度的实浮点数                                             |
     | string   | 字符串，由一对双引号或者单引号表示，或者用[[]]表示一块字符串（保存字符串格式），在一串数字字符串上进行算数操作时，lua会尝试将数字字符串转成一个数字 |
     | function | 由C或lua编写的函数，function可以通过匿名函数的方式进行参数传递，function可以返回多个值，并且可以接收可变数目的参数，在参数列表中用...表示且必须放在固定参数之后，可以用select("#",...)来获取可变参数的数量 |
     | userdata | 表示任意存储在变量中的C数据结构，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。 |
     | thread   | 表示独立的执行线路(线程)，用于执行协同程序                   |
     | table    | Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。没初始的table都是nil |

     lua中可以使用type()测定给定变量或者值的类型，该函数返回的结果是一个string，所以在与nil做判断时需要给nil打上双引号

   - 变量类型：lua有三种变量类型：全局变量(lua中的所有变量，除非使用local声明)、局部变量(用local声明)和表中的域

   - 循环方式：while循环、for循环、repeat...util和循环嵌套

     循环控制语句：break语句、goto语句（需要将label格式变成：：label：：)

   - 运算符：算数运算符，关系运算符，逻辑运算符，其他运算符

     - 算数运算符：

       | 操作符 | 描述      |
       | ------ | --------- |
       | +      | 加法      |
       | -      | 减法/负号 |
       | *      | 乘法      |
       | /      | 除法      |
       | %      | 取余      |
       | ^      | 乘幂      |

     - 关系运算符：

       | 操作符 | 描述                                                         |
       | ------ | ------------------------------------------------------------ |
       | ==     | 等于，检测两个值是否相等，相等返回 true，否则返回 false      |
       | ~=     | 不等于，检测两个值是否相等，不相等返回 true，否则返回 false  |
       | >      | 大于，如果左边的值大于右边的值，返回 true，否则返回 false    |
       | <      | 小于，如果左边的值大于右边的值，返回 false，否则返回 true    |
       | >=     | 大于等于，如果左边的值大于等于右边的值，返回 true，否则返回 false |
       | <=     | 小于等于， 如果左边的值小于等于右边的值，返回 true，否则返回 false |

     - 逻辑运算符：

       | 操作符 | 描述                                                         |
       | ------ | ------------------------------------------------------------ |
       | and    | 逻辑与操作符。 若 A 为 false，则返回 A，否则返回 B。         |
       | or     | 逻辑或操作符。 若 A 为 true，则返回 A，否则返回 B。          |
       | not    | 逻辑非操作符。与逻辑运算结果相反，如果条件为 true，逻辑非为 false。 |

     - 其他运算符：

       | 操作符 | 描述                             |
       | ------ | -------------------------------- |
       | ..     | 连接两个字符串                   |
       | #      | 一元运算符，返回字符串或表的长度 |

   - lua模块与包：从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

     Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。

     调用一个模块只需要使用require函数来加载，执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。

     luna模块加载机制：require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

     当然，如果没有 LUA_PATH 这个环境变量，也可以自定义设置，在当前用户根目录下打开 .profile 文件（没有则创建，打开 .bashrc 文件也可以）

   - lua元方法： Lua 提供了元表(Metatable)，允许我们改变table的行为，每个行为关联了对应的元方法。 lua中的每个值都可以有一个元表，只是table和userdata可以有各自独立的元表，而其他类型的值则共享其类型所属的单一元表。lua代码中只能设置table的元表，至于其他类型值的元表只能通过C代码设置。多个table可以共享一个通用的元表，但是每个table只能拥有一个元表。 我们称元表中的键为事件（event），称值为元方法（metamethod）。

     有两个很重要的函数来处理元表：

     - **setmetatable(table,metatable):** 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 __metatable 键值，setmetatable 会失败。
     - **getmetatable(table):** 返回对象的元表(metatable)。

     lua查找表规则：

     ​	1.在表中查找，如果找到，返回该元素，找不到则继续

     ​	2.判断该表是否有元表，如果没有元表，返回nil，有元表则继续

     ​	3.判断元表有没有index方法，如果index方法为nil，则返回nil；如果_index方法是一个表，则重复1、2、3；如果__index方法是一个函数，则返回该函数的返回值

     关于lua中元表这篇帖子讲的不错：https://blog.csdn.net/zhaixh_89/article/details/84301882

   - lua协同程序：

     - 与线程的共同点：拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其他协同程序共享全局变量和其他大部分东西。

     - 与线程的区别：一个具有多线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确被要求挂起的时候才会被挂起。协同程序有点类似同步的多线程，在等待同一个线程锁的几个线程有点类似。

     - 基本语法：

       | 方法                 | 描述                                                         |
       | -------------------- | ------------------------------------------------------------ |
       | corountine.create()  | 创建 coroutine，返回 coroutine， 参数是一个函数，当和 resume 配合使用的时候就唤醒函数调用 |
       | corountine.resume()  | 重启 coroutine，和 create 配合使用                           |
       | corountine.yield()   | 挂起 coroutine，将 coroutine 设置为挂起状态，这个和 resume 配合使用能有很多有用的效果 |
       | corountine.status()  | 查看 coroutine 的状态<br/>注：coroutine 的状态有三种：dead，suspended，running |
       | corountine.wrap()    | 创建 coroutine，返回一个函数，一旦你调用这个函数，就进入 coroutine，和 create 功能重复 |
       | corountine.running() | 返回正在跑的 coroutine，一个 coroutine 就是一个线程，当使用running的时候，就是返回一个 corouting 的线程号 |

   - lua文件I/O：

     - 简单模式，拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作
     - 完全模式：使用外部文件句柄来实现，以一种面对对象的方式，将所有的文件操作定义为文件句柄的方式。通常用于需要同时处理多个文件的情况。

   - lua错误处理：

     - assert和error：assert用于抛出错误信息；error会终止正在执行的函数，并返回message的内容作为错误信息(error函数永远都不会返回)，通常情况下，error会附加一些错误位置的信息到message头部。

     - pcall，xpcall和debug：

       - pcall接收一个函数和要传递给后者的参数，并执行，执行结果：有错误、无错误；返回值true或者或false, errorinfo。语法格式如下：

         ```
         if pcall(function_name, ….) then
         -- 没有错误
         else
         -- 一些错误
         end
         ```

         pcall以一种"保护模式"来调用第一个参数，因此pcall可以捕获函数执行中的任何错误。

         通常在错误发生时，希望落得更多的调试信息，而不只是发生错误的位置。但pcall返回时，它已经销毁了调用桟的部分内容。

     - xpcall接收第二个参数——一个错误处理函数，当错误发生时，Lua会在调用桟展开（unwind）前调用错误处理函数，于是就可以在这个函数中使用debug库来获取关于错误的额外信息了。

       debug库提供了两个通用的错误处理函数:

       - debug.debug：提供一个Lua提示符，让用户来检查错误的原因
       - debug.traceback：根据调用桟来构建一个扩展的错误消息

   - lua垃圾回收：lua采用了自动内存管理，运行了一个垃圾收集器来收集所有死对象 （即在 Lua 中不可能再访问到的对象）来完成自动内存管理的工作。 Lua 中所有用到的内存，如：字符串、表、用户数据、函数、线程、 内部结构等，都服从自动管理。

     Lua 实现了一个增量标记-扫描收集器。 它使用这两个数字来控制垃圾收集循环： 垃圾收集器间歇率和垃圾收集器步进倍率。 这两个数字都使用百分数为单位 。

     垃圾收集器间歇率控制着收集器需要在开启新的循环前要等待多久。 增大这个值会减少收集器的积极性。 当这个值比 100 小的时候，收集器在开启新的循环前不会有等待。 设置这个值为 200 就会让收集器等到总内存使用量达到 之前的两倍时才开始新的循环。

     垃圾收集器步进倍率控制着收集器运作速度相对于内存分配速度的倍率。 增大这个值不仅会让收集器更加积极，还会增加每个增量步骤的长度。 不要把这个值设得小于 100 ， 那样的话收集器就工作的太慢了以至于永远都干不完一个循环。 默认值是 200 ，这表示收集器以内存分配的"两倍"速工作。

     如果你把步进倍率设为一个非常大的数字 （比你的程序可能用到的字节数还大 10% ）， 收集器的行为就像一个 stop-the-world 收集器。 接着你若把间歇率设为 200 ， 收集器的行为就和过去的 Lua 版本一样了： 每次 Lua 使用的内存翻倍时，就做一次完整的收集。

     Lua 提供了以下函数**collectgarbage ([opt [, arg]])**用来控制自动内存管理:

     - **collectgarbage("collect"):** 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：
     - **collectgarbage("count"):** 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。
     - **collectgarbage("restart"):** 重启垃圾收集器的自动运行。
     - **collectgarbage("setpause"):** 将 arg 设为收集器的 间歇率。 返回 间歇率 的前一个值。
     - **collectgarbage("setstepmul"):** 返回 步进倍率 的前一个值。
     - **collectgarbage("step"):** 单步运行垃圾收集器。 步长"大小"由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。
     - **collectgarbage("stop"):** 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。

