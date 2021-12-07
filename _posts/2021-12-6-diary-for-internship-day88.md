---
layout: post
title:"实习日记88"
date:2021-12-06(Influx2插入数据并修改数据类型)
categories: jekyll update
typora-root-url: ./
---

## Day88

### 插入数据

- maven配置需要的依赖

  ```
  <dependency>
  	<groupId>com.influxdb</groupId>
  	<artifactId>influxdb-client-java</artifactId>
  	<version>3.1.0</version>
  </dependency>
  ```

- 通过UI获取账号token等信息

  ![](asset/2021-12-6-diary-for-internship-day88/QQ%E6%88%AA%E5%9B%BE20211206134019.png)

- 创建链接：

  相关参数在influxDB的UI界面上都有显示

  ```java
   // You can generate an API token from the "API Tokens Tab" in the UI
          String token = "your token";
          String bucket = "bucket-name";
          String org = "org-name";
  
          InfluxDBClient client = InfluxDBClientFactory.create("http://localhost:8086", token.toCharArray());
  ```

- 插入数据

  - 方法一：

    ```java
    String data = "mem,host=host1 used_percent=23.43234543";
    WriteApiBlocking writeApi = client.getWriteApiBlocking();
    writeApi.writeRecord(bucket, org, WritePrecision.NS, data);
    ```

  - 方法二：

    ```java
    Point point = Point
                    .measurement("mem")
                    .addTag("host", "host1")
                    .addField("used_percent", 23.43234543)
                    .time(Instant.now(), WritePrecision.NS);
    
    WriteApiBlocking writeApi = client.getWriteApiBlocking();
    writeApi.writePoint(bucket, org, point);
    ```

  - 方法三：

    ```java
    Mem mem = new Mem();
    mem.host = "host1";
    mem.used_percent = 23.43234543;
    mem.time = Instant.now();
    
    WriteApiBlocking writeApi = client.getWriteApiBlocking();
    writeApi.writeRecord(bucket, org, WritePrecision.NS, mem);
    
    @Measurement(name = "mem")
    public static class Mem {
      @Column(tag = true)
      String host;
      @Column
      Double used_percent;
      @Column(timestamp = true)
      Instant time;
    }
    
    ```

- 简单的查询

  其中query使用Flux语法，具体语法可参考：https://docs.influxdata.com/flux/v0.x/

  ```java
  String query = "from(bucket: \"zhgd\") |> range(start: -1h)";
  List<FluxTable> tables = client.getQueryApi().query(query, org);
  
  for (FluxTable table : tables) {
    for (FluxRecord record : table.getRecords()) {
      System.out.println(record);
    }
  }
  ```

- 关闭连接：

  ```java
  client.close();
  ```

### 转换field字段数据类型

Flux提供了几种转换方法，具体可参考相关文档https://docs.influxdata.com/flux/v0.x/function-types/#type-conversions

例子如下：

```
from(bucket: "zhgd")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "mem")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")
  |> toString()
```

此语句会返回所有相关字段能转化为string的数据并转为string类型，但是并不会改变measurement表中的数据。

