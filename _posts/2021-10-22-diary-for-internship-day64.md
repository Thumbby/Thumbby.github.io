---
layout: post
title:  "实习日记64(infix)"
date:   2021-10-26
categories: jekyll update
---

## Day64

- 问了一下可以装，那么就尝试一下infix了

- 这东西资料太少了吧，现在检出在本地后需要用dep工具编译，但是会报如下错误：

  ```
   Could not introduce github.com/influxdata/influxdb@valuesWritten, as it is not allowed by constraint ^1.5.5 from project github.com/Abc-Arbitrage/infix.
  ```

  查询之后貌似是版本问题，但这两个包都是从该git仓库拉下来的，不是本地的golang或者dep问题啊。。。

  infix是个开源的golang model包，但该仓库的issue下面没有人反映过该问题，甚至有其他人反映了其他的问题但贡献者还没有进行更新，这项目是有多新啊。

- 不弄这个了，明天会记录一下在InfluxDB1.8版本前如何通过备份数据重新插入来修改field type，以及在1.8版本后如何通过Flux来修改field type。

