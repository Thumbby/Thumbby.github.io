---
layout: post
title:  "实习日记37(添加pdf水印问题+新项目环境搭建)"
date:   2021-09-08
categories: jekyll update
---

## Day37

- Dubbo应用启动注册到zk获取ip慢：https://blog.csdn.net/caodegao/article/details/119461636

  可能的原因：

  - 第一次思路是我自己下载了zk运行起来注册到本地来看看，也是照样慢。

  - idea慢，我设置了idea的启动最大内存为2G，这之前我没设置过，后发现也无效。

  - 我一度以为我的IP vpn做了转发引起的，或者电脑安装了什么软件导致慢了。最后发现也不是。
  - 要设置hostname和127.0.0.1的映射关系，我就设置了hosts，然后启动确实好了，恢复到几十秒内启动完毕。（hostname是机器名）（成功）

  **注意**：注册接口到zk时打印日志会有个空隙，并不是平滑的日志打印那种感觉，这就是问题的现象。
  
- 本地mysql不能用ip访问解决办法：https://www.cnblogs.com/lc-ant/p/8882326.html、

  - 在运行中输入CMD，确定，进入文本方式。

  - 输入mysql -h localhost -u root -p 回车，使用ROOT用户登录。

  - 输入use mysql; 显示Database changed，选择MYSQL系统库。

  - 假定我们现在增加一个'goldeye2000'用户，密码为'1234567'，让其能够从外部访问MYSQL。输入

    grant all on * to 'goldeye2000' identified by '1234567';

    ALL代表所有权限。

    现在看看用户表内容。输入select user,host from user ; 可以看到"goldeye2000"用户已经加进去了，并且其权限为'% '，'grande'，'localhost '。

    退出MYSQL，输入QUIT；回车

    我们现在可以用goldeye2000用户在局域网或互联网中以IP方式访问了。

    mysql -h 192.168.0.115 -u goldeye2000 -p
  
- 电脑多次蓝屏报错Memory Management，可能内存有问题，先去微软社区看一下

  目前看下来室内存条问题，但是无法具体确定，目前没有再次出现蓝屏，但是这周去售后服务点看一下吧。
