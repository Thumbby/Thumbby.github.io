---
layout: post
title:  "实习日记38(环境搭建)"
date:   2021-09-09
categories: jekyll update
---

## Day38

- 项目环境搭建：框架更新，其中swagger的url随着版本变化而改变，2.0的swagger的url末尾为swagger-ui.html#/，3.0的swagger的url末尾为swagger-ui。

- 阅读项目代码，看一下框架差异，没什么好记的

- 关系数据库、内存数据库和实时数据库的对比

  | 比较项目                                                     | 关系数据库 | 内存数据库 | 实时数据库 | 说明                                                         |
  | ------------------------------------------------------------ | ---------- | ---------- | ---------- | ------------------------------------------------------------ |
  | 表结构                                                       | 完整       | 完整       | 简化       | 实时数据库不能处理复杂的表关系，但在特定行业的应用中，比如工控监控软件中，不需要复杂的表关系 |
  | 每秒读写速度                                                 | 3000       | 50000      | 500000     | 内存实时数据库比关系数据库快10倍左右，实时数据库比内存数据库快10倍左右 |
  | 历史数据压缩                                                 | 无         | 无         | 有         | 实时数据库比内存数据库的压缩率能达到20~40倍                  |
  | 4G空间能存储30万个测点的每秒变化一次的历史(不带索引)         | 5h         | 5h         | 8.5days    | 在4G内存的情况下，在单服务器处理30万点的情况下，内存数据库只能存贮5小时以内的历史数据，在带索引时，只能保存3小时以内的历史数据。 |
  | 128G空间能存贮30万个测点的每秒变化一次的历史数据（不带索引） | 7days      | 7days      | 269days    | 内存数据库有般用在电信行业，国内电信行业应用的最大项目也就使用了90G内存，在128G内存下，内存数据库也只能保存7天的历史数据 |
  | 是否需要历史数据库                                           | 不需要     | 需要       | 不需要     | 内存数据库还需要配套使用历史数据库，且历史数据库同样存在不能压缩、不能保存长时间海量历史数据的问题 |

- influxdb搭建与学习

  **搭建**：

  官网：https://www.influxdata.com/

  下载地址：选择对应平台的influxdb：https://dl.influxdata.com/influxdb/releases/influxdb-1.7.6_windows_amd64.zip

  下载之后直接解压在对应位置就可以使用，不需要安装。

  解压之后的样子：

  ![img](https://img-blog.csdnimg.cn/20200511164712401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JoeTU2ODM=,size_16,color_FFFFFF,t_70)

  influx.exe 表示客户端，influxd.exe 表示服务端，influxdb.conf 是配置文件，

  有的文章说我们需要修改该文件，主要是三个路径修改：改成你的influx解压的路径，后边的meta、data、wal是默认的文件夹的名字

  ```conf
  [meta]
    # Where the metadata/raft database is stored
    dir = "D:/influxdb-1.8.0-1/meta"
  [data]
    # The directory where the TSM storage engine stores TSM files.
    dir = "D:/influxdb-1.8.0-1/data"
   
    # The directory where the TSM storage engine stores WAL files.
    wal-dir = "D:/influxdb-1.8.0-1/wal"
  ```


  但是经过我测试这个位置的路径设置在windows下是不生效的，真实的数据存储路径并不是我们指定的这个，而是

  C:/Users/用户/.influxdb/下，会自动创建这三个文件夹。

  所以此处路径可以不做修改，直接跳过就好了

  启动：

  打开Influxd.exe服务端，然后打开influx.exe客户端就可以进入命令行模式了

- kafka安装：

  保证本地安装了java和zookeeper并且zookeeper在启动中

  - 下载安装文件： http://kafka.apache.org/downloads.html

  -  解压文件

  - 打开kafka_2.11-2.0.0\config

  - 从文本编辑器里打开 server.properties

  - 把 log.dirs的值改成D:\\kafka\\kafka_2.12-2.6.0\\logs(自己需要的日志目录，在windows环境下需要使用双反斜杠)

  - 进入kafka文件目录，输入并执行: .\bin\windows\kafka-server-start.bat .\config\server.properties

  - 创建topics

    - 打开cmd 并进入cd kafka目录
    - 创建一个topic： kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

  - 打开一个producer

    ```
    kafka-console-producer.bat --broker-list localhost:9092 --topic test
    ```

  - 打开一个 consumer

    ```
    kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
    ```

    

  

