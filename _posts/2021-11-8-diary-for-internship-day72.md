---
layout: post
title:  "实习日记72(InfluxDB数据删除流程)"
date:   2021-11-08
categories: jekyll update
---

## Day72

- 成功了，只要保留策略过期了之后，就可以通过删除measurement再重新插入数据来实现修改field类型。

- InfluxDB数据删除流程：

  - 根据DELETE SQL中指定的WHERE条件，从tsi/tsl文件中过滤得到满足条件的seriesID

  - 通过查找series index以及series segment文件得到seriesID对应的seriesKey

  - 查找TSM文件中的IndexBlock，判断该TSM是否存储seriesKey对应的数据（由于IndexBlock中保存的Key是由seriesKey + fieldKey组成，故只需要判断Key中的seriesKey与待删除的seriesKey是否相同就可以得到该TSM文件中是否存在seriesKey对应的时序数据），若存在，则将该Key对应的删除记录写入到该TSM对应的tombstone文件中，删除记录如下所示：
    ![tsm.tonestone](https://img-blog.csdnimg.cn/20200113200055880.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI3OTQ5MTU=,size_16,color_FFFFFF,t_70)

    minTime和maxTime表示DELTE SQL中WHERE中指定的时间范围，表示删除指定时间范围的数据。

    - 如果Key在TSM中的数据需要全部删除（DELETE SQL中指定的时间范围完全覆盖Key在TSM中对应的时间范围），则将indirectIndex中的offsets数组中IndexBlock对应的的entry删除，这样在进行数据查找时由于在offsets中已经将对应项删除的原因，从而会跳过读取该部分惰性删除的数据；如果只需删除TSM中Key某段时间范围的数据（DELETE SQL中指定的时间范围与Key在TSM中对应的时间范围有重叠），则在indirectIndex中添加tombstone（[startTime, endTime)]，从而select查询时通过这些删除标志能够判断这部分数据是否已经删除。当TSM文件compaction时进行实际数据的删除生成新的TSM文件，TSM compaction完成后将tombstone文件删除

  - InfluxDB采用LSM结构，删除内存中seriesKey在DELETE SQL指定时间范围的数据

  - 将DeleteRangeWALEntry写入wal文件中

  - 从series index以及series segment中查找待删除的seriesKey所对应你的series id

  - 将SeriesTombstoneLogEntry写入TSL文件中，并在内存中TSL对应的结构体中标记该seriesKey已经删除

  - 判断删除seriesKey所在的measurement中所有seriesKey是否已经删除，如果measurement中所有seriesKey均已删除，则从index中删除该measurement的索引信息：

    - 从TSL以及TSI中查找除所有该measurement包含的tagKey，将tagKey对应的TagKeyTombstoneLogEntry写入TSL文件，同时在内存中TSL对应的结构体中标记该tagKey已经删除
    - 从TSL以及TSI中查找上述tagKey对应的所有tagValue，将tagValue对应的TagValueTombstoneLogEntry写入TSL文件，并在TSL对应的结构体中标记该tagValue已经删除
    - 从fields.idx中删除该measurement的field信息

  - 将待删除的series id对应的SeriesTombStoneEntry写入series segment文件中，同时在series index的内存结构体重将该series id标记成删除

