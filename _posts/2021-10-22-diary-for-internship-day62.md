---
layout: post
title:  "实习日记62"
date:   2021-10-22
categories: jekyll update
---

## Day62

- 分配到的任务是研究如何在数据库层面修改influxDB中的field类型，目前找到如下方式：

  - 对原有数据作备份，然后清空该MEASUREMENT并重新插入所有数据，因为每个MEASUREMENT中的Field类型通过其第一个插入的数据类型决定，这种方法可以实现，但是在找有没有更好的方法。
  - 使用Flux进行修改，这种方法同时会修改历史数据类型，但是前提是influxDB版本在1.8及以上，工作环境下的influxDB版本太低，不能使用此方法。
  - 使用infix，是一个开源的influxDB修改工具，但是需要GoLang的编译环境，询问了一下表示不要用此方法
  - 查看influxdb官网：https://docs.influxdata.com/influxdb/v1.8/troubleshooting/frequently-asked-questions/

  问题终结，原文如下：

  Currently, InfluxDB offers very limited support for changing a field’s data type.

  The `<field_key>::<type>` syntax supports casting field values from integers to floats or from floats to integers. See [Cast Operations](https://docs.influxdata.com/influxdb/v1.8/query_language/explore-data/#data-types-and-cast-operations) for an example. There is no way to cast a float or integer to a string or Boolean (or vice versa).

  We list possible workarounds for changing a field’s data type below. Note that these workarounds will not update data that have already been written to the database.

  简单来说就是，可以把整数转化成浮点数或者标准浮点数转化成整数， 但是不能把浮点数或整数转化成string或者布尔值(反之亦然)