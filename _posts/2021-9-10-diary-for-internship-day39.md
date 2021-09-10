---
layout: post
title:  "实习日记39(influxDB+Kafka)
date:   2021-09-10
categories: jekyll update
---

## Day39

- 配置Chronograf

  - 解压文件后，直接进入安装目录，执行chronograf.exe后；
  - 输入：[http://localhost:8888](http://localhost:8888/)（chronograf默认是8888端口）
  - influxDB数据源连接

  ![img](https://img-blog.csdn.net/20180914164654631?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1N3VGVzdGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  - 输入数据库密码，点击下一步，其他不用变，第二个页面也一样，进入成功

    ![img](https://img-blog.csdn.net/20180914171735937?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1N3VGVzdGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

- influxdb设计原则(v1.0)：

  - 把元数据(meta data)设计进tag中
  
    tags会被索引，但是fields不会，因此基于tags的检索会比基于fields的检索更快。以下几条通用的判断是否使用tag还是field来存储数据的方法：
  
    - 如果字段是经常要被作为检索条件的元数据，设计为tag；
    - 如果字段经常要作为group by的参数，设计为tag；
    - 如果字段需要被用来放在influxQL的函数的参数，设计为field；
    - 如果处于某些考虑，字段不方便作为string类型保存，则应该使用field，因为tags总是以string类型来存储
  
  - 避免使用influxQL的关键字作为字段名
  
    这个并不是必须的，但是这样做会简化写influxQL的过程，这样可以不必给字典加上双引号就能检索
  
  - 不要有太多的series，和mysql的索引一样，否则会造成大量的内存占用以及io负载。
  
  - 不要再measurement的名字里携带数据，设计数据分类的时候，分类的数据最好作为一个字段，编入一个measurement中，而不要通过给measurement加后缀的方式，编入多个measurements中。
  
  - 不要在一个tag字段里存放多于一条信息，为了方便检索，一个tag中只存放单一的、纯粹的数据，如果存放多于一个数据点，在做检索的时候还需要手动解析tag再分类，这样会极大地削弱tags地索引作用。
  
- 下午去修电脑
  
  
  
  
  

