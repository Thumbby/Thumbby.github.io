---
layout: post
title:  "实习日记61(influxdb存储机制)"
date:   2021-10-21
categories: jekyll update
---

## Day61

- InfluxDB机制

  - 数据模型

    在InfluxDB中，时序数据支持多值模型，它的一条典型的时间点数据如下所示：

    ![img](https://img-blog.csdnimg.cn/20190226164857703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

    包括：

    - measurement：指标对象，也即一个数据源对象。每个measurement可以拥有一个或多个指标值，也即下文所述的**field**。在实际运用中，可以把一个现实中被检测的对象（如：“cpu”）定义为一个measurement
    - tags：概念等同于大多数时序数据库中的tags, 通常通过tags可以唯一标示数据源。每个tag的key和value必须都是字符串。
    - field：数据源记录的具体指标值。每一种指标被称作一个“field”，指标值就是 “field”对应的“value”
    - timestamp：数据的时间戳。在InfluxDB中，理论上时间戳可以精确到 **纳秒**（ns）级别

    此外，在InfluxDB中，measurement的概念之上还有一个对标传统DBMS的 **Database** 的概念，逻辑上每个Database下面可以有多个measurement。在单机版的InfluxDB实现中，每个Database实际对应了一个文件系统的 **目录**。

  - Serieskey的概念：InfluxDB中的SeriesKey的概念就是通常在时序数据库领域被称为 **时间线** 的概念, 一个SeriesKey在内存中的表示即为下述字符串(逗号和空格被转义)的 **字节数组**其中，SeriesKey的长度不能超过 65535 字节

  - 支持的Field类型

    InfluxDB的Field值支持以下数据类型:

    | Data Type | Size in Mem | Value Range                                                  |
    | --------- | ----------- | ------------------------------------------------------------ |
    | Float     | 8 bytes     | 1.797693134862315708145274237317043567981e+308 ~ 4.940656458412465441765687928682213723651e-324 |
    | Interger  | 8 bytes     | -9223372036854775808 ～ 9223372036854775807                  |
    | String    | 0~64KB      | String with length less than 64KB                            |
    | Boolean   | 1 byte      | true or false                                                |

    在InfluxDB中，Field的数据类型在以下范围内必须保持不变，否则写数据时会报错 **类型冲突**。即：

    **同一Serieskey + 同一field + 同一shard**

  - Shared的概念：

    在InfluxDB中， 能且只能 对一个Database指定一个 Retention Policy (简称:RP)。通过RP可以对指定的Database中保存的时序数据的留存时间(duration)进行设置。而 Shard 的概念就是由duration衍生而来。一旦一个Database的duration确定后, 那么在该Database的时序数据将会在这个duration范围内进一步按时间进行分片从而时数据分成以一个一个的shard为单位进行保存。

    shard分片的时间 与 duration之间的关系如下：

    | Duration of RP             | Shard Duration |
    | -------------------------- | -------------- |
    | < 2 Hours                  | 1 Hour         |
    | \>= 2 Hours 且 <= 6 Months | 1 Day          |
    | \> 6 Months                | 7 Days         |

    新建的Database在未显式指定RC的情况下，默认的RC为 **数据的Duration为永久，Shard分片时间为7天**

    *注: 在闭源的集群版Influxdb中，用户可以通过RC规则指定数据在基于时间分片的基础上再按SeriesKey为单位进行进一步分片*

  - 存储引擎分析

    时序数据库的存储引擎主要需满足以下三个主要场景的性能需求

    - 大批量的时序数据写入的高性能

    - 直接根据时间线(即Influxdb中的 Serieskey )在指定时间戳范围内扫描数据的高性能

    - 间接通过measurement和部分tag查询指定时间戳范围内所有满足条件的时序数据的高性能

    InfluxDB在结合了1.2所述考量的基础上推出了他们的解决方案，即下面要介绍的 WAL + TSMFile + TSIFile的方案

    - WAL解析

      InfluxDB写入时序数据时为了确保数据完整性和可用性，与大部分数据库产品一样，都是会先写WAL,再写入缓存，最后刷盘。nfluxDB对于时间线数据和时序数据本身分开，分别写入不同的WAL中。

      由于InfluxDB支持对Measurement，TagKey，TagValue的删除操作，当然随着时序数据的不断写入，自然也包括 **增加新的时间线**，因此索引数据的WAL会区分当前所做的操作具体是什么，它的WAL的结构如下图所示：

      ![img](https://img-blog.csdnimg.cn/20190226164930157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

      时序数据的WAL

      由于InfluxDB对于时序数据的写操作永远只有单纯写入，因此它的Entry不需要区分操作种类，直接记录写入的数据即可

      ![img](https://img-blog.csdnimg.cn/20190226164942784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

    - TSM File解析

      TSMFile是InfluxDB对于时序数据的存储方案。在文件系统层面，每一个TSMFile对应了一个 **Shard**。

      TSMFile的存储结构如下图所示:

      ![img](https://img-blog.csdnimg.cn/20190226164951300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

      其特点是在一个TSMFile中将 时序数据（i.e Timestamp + Field value）保存在数据区；将Serieskey 和 Field Name的信息保存在索引区，通过一个基于 Serieskey + Fieldkey构建的形似B+tree的文件内索引快速定位时序数据所在的 数据块

      索引块的构成：

      ![img](https://img-blog.csdnimg.cn/20190226165003892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

      其中 **索引条目** 在InfluxDB的源码中被称为`directIndex`。在TSMFile中，索引块是按照 Serieskey + Fieldkey **排序** 后组织在一起的。

      明白了TSMFile的索引区的构成，就可以很自然地理解InfluxDB如何高性能地在TSMFile扫描时序数据了：

      1. 根据用户指定的时间线（Serieskey）以及Field名 在 **索引区** 利用二分查找找到指定的Serieskey+FieldKey所处的 **索引数据块**
      2. 根据用户指定的时间戳范围在 **索引数据块** 中查找数据落在哪个（*或哪几个*）**索引条目**
      3. 将找到的 **索引条目** 对应的 **时序数据块** 加载到内存中进行进一步的Scan

      *注：上述的1，2，3只是简单化地介绍了查询机制，实际的实现中还有类似扫描的时间范围跨索引块等一系列复杂场景*

      **时序数据的存储**：

      时序数据块的结构：同一个 Serieskey + Fieldkey 的 所有`时间戳 - Field值`对被拆分开，分成两个区：Timestamps区和Value区分别进行存储。它的目的是：**实际存储时可以分别对时间戳和Field值按不同的压缩算法进行存储以减少时序数据块的大小**。采用的压缩算法如下所示：

      - Timestamp： Delta-of-delta encoding
      - Field Value：由于单个数据块的Field Value必然数据类型相同，因此可以集中按数据类型采用不同的压缩算法
        - Float类: Gorrila's Float Commpression
        
        - Integer类型: Delta Encoding + Zigzag Conversion + RLE / Simple8b / None
        
        - String类型: Snappy Compression
        
        - Boolean类型: Bit packing
      
      做查询时，当利用TSMFile的索引找到文件中的时序数据块时，将数据块载入内存并对Timestamp以及Field Value进行解压缩后以便继续后续的查询操作。
      
    - TSI File解析

      nfluxDB的倒排索引依赖于下述两个数据结构

      - `map<SeriesID, SeriesKey>`
      - `map<tagkey, map<tagvalue, List<SeriesID>>>`

      它们在内存中展现如下：

      ![img](https://img-blog.csdnimg.cn/20190226165021423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

      ![img](https://img-blog.csdnimg.cn/20190226165040357.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3MDg5MA==,size_16,color_FFFFFF,t_70)

      但是在实际生产环境中，由于用户的时间线规模会变得很大，因此会造成倒排索引使用的内存过多，所以后来InfluxDB又引入了 **TSIFile**

      TSIFile的整体存储机制与TSMFile相似，也是以 **Shard** 为单位生成一个TSIFile。
