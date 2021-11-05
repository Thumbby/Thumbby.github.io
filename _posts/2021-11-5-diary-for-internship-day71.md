---
layout: post
title:  "实习日记71(InfluxDB机制)"
date:   2021-11-05
categories: jekyll update
---

## Day71

- 记录一下修改InfluxDB中修改field类型可能出现的结果
  - 删除表后插入不同类型的数据报field类型冲突，此时idx文件存在，可能唯一也可能不唯一
  - 删除表后成功插入数据但出现相同名不同类型的field，此时idx文件存在且不唯一，删除某一idx文件后field类型不变但数据丢失
  - 删除表后成功插入数据并且替换了field类型，这个是希望达到的预期效果，此时idx可能存在且唯一，可能不存在。
- 我明白了，其实替换field类型的操作依旧不可行，之前出现的可行情况只是因为field存储在了不同的shard中或者之前的shard过期了，对于修改field类型，官方给出了如下方案：
  - 写不同的field，即将data写入不同类型的field
  - 希望更改字段数据类型的用户可以使用`SHOW SHARDS`查询来识别`end_time`当前分片的 。InfluxDB 将接受具有不同数据类型的写入到现有字段，如果该点具有在此之后发生的时间戳`end_time`。
- 在一个RP中，如果指定的保留时间为24小时，那么每个shard的duration为1小时，即每个shard的时间跨度为1小时，那么总共会有24个跨度为1小时的shard，在触发数据的RP后，删除最早时间跨度的shard。
- 我可能需要等到下周一shard过期了再尝试看下结果。
