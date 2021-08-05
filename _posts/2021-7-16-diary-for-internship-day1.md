---
layout: post
title:  "实习日记1(环境搭建)"
date:   2021-07-16
categories: jekyll update
---

## Day1

- 入职第一天，办理入职手续，签实习合同
- 安装系统环境，具体如下：
   - MYSQL5.7.27，该版本无msi，需下载zip后自行安装，具体安装过程可见如下连接：https://blog.csdn.net/neochan1108/article/details/116046264
   - Redis4.0.11，可直接下载msi版本，链接如下：https://blog.csdn.net/qq_20084101/article/details/82355818
   - Navicat for MYSQL，配合本地MYSQL使用，安装破解版，链接如下：https://blog.csdn.net/mmake1994/article/details/81840878
   - Zookeeper3.4.12，安装后需手动将zkServer注册为windows服务，链接如下：https://blog.csdn.net/xionglangs/article/details/80175721，需要将prungmr.exe更改为注册的服务名（即环境变量）才可运行，不过不影响注册服务，本地开发环境使用单节点即可
   - dubbo2.5.10，github下载地址如下：https://github.com/alibaba/dubbo，安装教程如下：https://blog.csdn.net/djrm11/article/details/83934428