---
layout: post
title:  "实习日记44(interface)"
date:   2021-09-22
categories: jekyll update
---

## Day44

- 尝试写个influxDB+Kafka的 demo

  - 先解决Day40最后的那个use database报错问题，现在命令同mysql，使用库直接用use [表名]，而非use database[表名]

  - influxDB的常用操作：

    - 数据库操作：

      ```
      # 查看所有的数据库
      show databases;
      #创建数据库！
      create database test;
      #删除数据库
      drop database test;
      #使用数据库
      use xk_name;
      ```

    - 表操作：

      ```
      # 显示所有的表
      SHOW MEASUREMENTS;
      #InfluxDB中没有显式的新建表的语句，只能通过insert数据的方式来建立新表
      #disk_free是表名,hostname=server01是tag，属于索引，value=xx是field，这个可以随意写，随意定义。
      #InfluxDB规定格式，首先是表名，后面是tags，然后是field，最后是时间戳。tags、field和时间戳三者之间以空格相分隔
      insert disk_free,hostname=server01 value=442221834240i 1435362189575692182;
      ```

    - 常见字段说明

      ```
      # influxdb的数据都有一列名为time的列，里面存储UTC时间戳
      tag  #索引字段 ，根据这个字段查询会很快,如果不根据这个字段查询相当于遍历表。
      field key，field value，field set  #字段键key,值value 两者合并称之为 set
      field key，      #它们为string类型
      field value     #可以为string，float，integer或boolean类型
      retention policy   #指数据保留策略,默认的autogen,它表示数据一直保留永不过期，副本数量为1
      series     #共享同一个retention policy，measurement以及tag set的数据集合
      point      #point则是同一个series中具有相同时间的field set，points相当于SQL中的数据行
      ```

    - influx数据保存策略
      1.InfluxDB的数据保留策略（RP） 用来定义数据在InfluxDB中存放的时间，或者定义保存某个期间的数据。
      2.一个数据库可以有多个保留策略，但每个策略必须是独一无二的。
      3.InfluxDB本身不提供数据的删除操作，因此用来控制数据量的方式就是定义数据保留策略。
      4.数据保留策略的目的是让InfluxDB能够知道可以丢弃哪些数据，从而更高效的处理数据
      
      ```
      # 对于 数据库 mydb 创建一个策略 '2_hours' duration(持续时间)为2小时，副本为1，设置为默认策略。
      create retention policy "2_hours" on "mydb" duration 2h replication 1 default
      # 对于数据库 mydb 展示出所有的策略
      show retention policies on mydb;
      # 修改策略 2_hours 为默认策略 ,持续时间为 2h
      alter retention policy "2_hours" on "mydb" duration 4h default;
      # 查看使用 default 策略的 cpu 表信息
      select * from "default".cpu limit 2
      ```
      
      

