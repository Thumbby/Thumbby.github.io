---
layout: post
title:  "实习日记6(环境搭建)"
date:   2021-07-21
categories: jekyll update
---

## Day6

- 前端环境搭建：
   - 依赖安装时出现exit code 128:远端问题，尝试方案如下：
     - 删除用户缓存和已安装依赖，无果
     - 可能是文件名过长，git有可以创建4096长度的文件名，然而在windows最多是260，因为git用了旧版本的windows api，设置git config --global core.longpaths true，无果
     - 直接复制现有的node_module
   - 运行时出现问题：Missing Binging，尝试方案如下：
     - npm rebuild node-sass，失败
     - 发现本地Node.js为32位，与x64不兼容，下载64位Node.js，成功
- 后端环境搭建：
   - maven安装依赖出现问题，需要查看JDK版本是否一致，否则会出现无法识别依赖的情况
   - 如果maven依赖未识别到，可以删除本地maven仓库，再重新下载。
- mybatisplus学习

