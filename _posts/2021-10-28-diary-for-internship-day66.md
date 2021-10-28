---
layout: post
title:  "实习日记66"
date:   2021-10-28
categories: jekyll update
---

## Day66

- 今天继续写那个工具，不知道能不能写完

  - 本来是想用csv导入导出的，但是貌似用influx cli做数据表measurement删除的时候再次插入会限制结构，也就是说他只是清空了数据并没有改变measurement的结构，但是在influxdb中没有明确的表概念，所以现在在尝试用python的influxdb包来写这个工具

  - 我真的服了，这个问题早在1.5.2版本中就已经有人提出过issue了，结果1.7.6还没有解决，可能这就是在1.8版本之后推出Flux的原因把，为什么要把tags和fields存在Cache里面啊

  - 看到当年开发者的留言了，留言如下：

    ```
    Update
    Hello everyone affected by this issue. Firstly, I would like to apologise that it's been almost a year since this issue was filed. Yesterday I was able to really dig into what was causing this issue, mainly due to the .influxdb directly that @ragnarkurmwunder sent me a while back.
    
    This is a pretty difficult issue to reproduce without an existing dataset. Triggering the issue relies on duplicates of a new series being inserted into a database using the inmem index within the same batch. Further, it looks like they need to sit inside of the WAL so that when the database is restarted they will be replayed in and the problem will continue...
    
    The cause of the issue is that the inmem index, in this rare case over counts how many series belong to the measurement (it counts the duplicate points for the same series as different series). Then, when you go to delete the measurement, the index thinks there are still some series around for the measurement and it does not clean up the fields.idx file. This file contains mappings from measurements to field keys, and if it's not cleaned up properly, then those old field keys can be returned in some cases.
    
    I believe I have fixed this issue in #14266
    
    The fix will be available in the 1.8 release, and also in a future 1.7.8 release.
    
    Operational Mitigation Steps
    Here are some operational steps you could take to try and resolve this issue without waiting for 1.8 or 1.7.8.
    
    Use the TSI index
    I was unable to reproduce this issue using the TSI index. Even when I triggered the issue on the inmem index, and then upgraded to the TSI index, I saw the issue disappear. Whilst we will of course continue to support the inmem index on the 1.x line, from 2.x onwards the TSI index will be the main index InfluxDB uses, and all our development effort will continue on that.
    
    You can find out more information about how to upgrade to TSI here. In the simplest case, you bring your server down and then do something like:
    
    influx_inspect buildtsi -datadir ~/.influxdb/data -waldir ~/.influxdb/wal
    Remove invalid fields.idx files
    The bug is caused because the fields.idx files (there is one file per shard directory) are not properly rebuilt when the measurement is deleted. However, InfluxDB will rebuild these files if they're missing. If you are currently suffering from fields that are appearing in queries when they shouldn't be then I recommend that you delete all of the field.idx files for the problematic database/retention policy. You will need to bring down your server to do this, then:
    
    $ rm -i ~/.influxdb/data/<db_name>/<rp_name>/*/fields.idx
    ```

    Here are some operational steps you could take to try and resolve this issue without waiting for 1.8 or 1.7.8。。。

    我先按照下面的方法处理一下吧。。。

  - **结局**：差不多就是每次删除一张表之后，要么等保留策略过期，要么直接删除对应的field.idx文件，然后重启influxDB，才能更改表结构；或者可以插入一张新表。

- 写完了工具，其实就是对influxDB导出的csv进行重新处理而已，需要用户手动给出tag和非string格式的field，贴一下代码：

  ```python
  WINDOWS_LINE_ENDING = b'\r\n'
  UNIX_LINE_ENDING = b'\n'
  
  
  def to_string_type(value):
      return '"' + value + '"'
  
  
  def influx_csv(file_path, database_name, tags, change_fields):
      raw_file = open(file_path, 'r')
      attributes = raw_file.readline().strip('\n').split(',')
      column = []
      for index in range(len(attributes)):
          column.append(attributes[index])
      datalist = []
      for line in raw_file.readlines():
          line = line.strip().strip('\n')
          data_line = line.split(',')
          datalist.append(data_line)
      change_field_index = []
      tag_index = []
      for tag in tags:
          tag_index.append(column.index(tag))
      for field in change_fields:
          change_field_index.append(column.index(field))
  
      file = open('result.txt', 'w+')
      # 创建数据库，若数据库已有则不需这段代码
      # file.write('# DDL')
      # file.write('\nCREATE DATABASE '+database_name)
      # file.write('\nCREATE RETENTION POLICY oneday ON pirates DURATION 1d REPLICATION 1') 设置保存策略,可不添加
      # file.write('\n\n')
      # 选择数据库
      file.write('# DML')
      file.write('\n# CONTEXT-DATABASE: ' + database_name)
      # file.write('\n# CONTEXT-RETENTION-POLICY: oneday') 设置保存策略,可不添加
      file.write('\n')
  
      for data in datalist:
          line_str = ""
          line_str = line_str + '\n'
          # 表名
          line_str = line_str + data[0] + ','
          # tag
          for index in range(2, len(data)):
              if index in tag_index:
                  line_str = line_str + attributes[index] + '=' + data[index] + ','
          line_str = line_str[:-1] + " "
          # field
          for index in range(2, len(data)):
              if index not in tag_index:
                  if index in change_field_index:
                      line_str = line_str + attributes[index] + '=' + data[index] + ','
                  else:
                      line_str = line_str + attributes[index] + '=' + to_string_type(data[index]) + ','
          line_str = line_str[:-1] + " "
          line_str = line_str + data[1]
          file.write(line_str)
      file.write('\n')  # 最后一行数据后需要回车，否则最后一条无法导入
      file.close()
  
      # 由于influxDB读取的txt格式为UNIX格式,所以需要将文件中的换行符进行修改
      with open('result.txt', 'rb') as open_file:
          content = open_file.read()
  
      content = content.replace(WINDOWS_LINE_ENDING, UNIX_LINE_ENDING)
      with open('result.txt', 'wb') as open_file:
          open_file.write(content)
  ```

  

