---
layout: post
title:  "实习日记40
date:   2021-09-13
categories: jekyll update
---

## Day40

- 关闭Microsoft Compatibility telemetry

  - 打开计划任务程序，定位到Microsoft-Windows-Application Experience，禁用Microsoft Compatibility Appraiser。
  - 打开服务，禁用Connected User Experiences and Telemetry。

  这玩意儿能上来把我CPU内存磁盘统统吃满，直接给禁了，该服务用于windows反馈，对实际使用无影响

- windows下Telegraf+Influxdb+Grafana环境搭建

  - Influxdb搭建见Day39的日记

  - Telegraf搭建

    - wget [https://dl.influxdata.com/telegraf/releases/telegraf-1.5.0_windows_amd64.zip](https://link.jianshu.com/?t=https%3A%2F%2Fdl.influxdata.com%2Ftelegraf%2Freleases%2Ftelegraf-1.5.0_windows_amd64.zip)

    - 修改telegraf.conf文件，设置日志文件目录 ## Specify the log file name. The empty string means to log to stdout.

      ```
      logfile = "F:/Grafana/server/telegraf/telegraf.log"   ##你修改为自己定义的目录路径，其他的配置不要乱动。
    
  - Grafana环境搭建
    
    - wget [https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-](https://link.jianshu.com/?t=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fgrafana-releases%2Frelease%2Fgrafana-4.6.3.windows-x64.zip)
    
    由于grafana仅仅只是提供界面显示, 所以他需要从influxdb中获取数据, 而influxdb中的数据又需要从其他地方收集过来, 常用的收集工具是collectd和telegraf, 其中collectd这里不做介绍, 有些数据不是太适合, 而 influxdb 自身集成 telegraf插件, 不需要进行专门的配置，即
    
    collectd/telegraf(收集数据) -------> influxdb(保存数据) -------> grafana(显示数据)
    
  - 启动

    - 启动InfluxDB

      通过cmd命令窗口，切换到influxdb安装目录，执行如下命令：
      
       influxd -config influxdb.conf

    - 启动Telegraf

      通过cmd命令窗口，切换到Telegraf安装目录，执行如下命令：

      telegraf -config telegraf.conf

    - 启动Grafana

      切换到Grafana安装目录中的bin目录下，双击grafana-server.exe启动程序

    - 登入[http://localhost:3000](https://link.jianshu.com/?t=http%3A%2F%2Flocalhost%3A3000) , 使用admin/admin登录本机Grafana

- influxdb使用问题：登录之后use database会报错

  ERR: Database database test doesn't exist. Run SHOW DATABASES for a list of existing databases.
  DB does not exist!

  **尝试的解决方案**：

  - 设置用户全线，**无效**

  - 更改conf文件：

    ```
      # Determines whether user authentication is enabled over HTTP/HTTPS.
      # auth-enabled = true
    ```

    **无效**
