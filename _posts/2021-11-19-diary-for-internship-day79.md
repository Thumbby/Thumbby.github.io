---
layout: post
title:  "实习日记79(Flux使用)"
date:   2021-11-19
categories: jekyll update
typora-root-url: ./
---

## Day79

- 第一个问题：使用Flux会报错：

  ```
  Error: 404 page not found
  : 404 Not Found
  ```

  解决方法：在启动flux命令时添加如下命令：

  ```
  influx -precision rfc3339 -type=flux -path-prefix /api/v2/query
  ```

  使用-path-prefix来规定url，因为influx client默认post路径为/

- 第二个问题：上述情况下使用from()语句报错：

  ```
  Error: Flux query service disabled. Verify flux-enabled=true in the [http] section of the InfluxDB config.
  : 403 Forbidden
  ```

  此外，在config文件中已经将flux-enabled设置为true，但仍然会报该错误

