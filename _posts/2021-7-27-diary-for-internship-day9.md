---
layout: post
title:  "实习日记9"
date:   2021-07-27
categories: jekyll update
---

## Day9

- 城运大屏项目sql修改
- DataQL学习:https://www.hasor.net/doc/pages/viewpage.action?pageId=1573215
- 手动数据同步更改为自动数据同步,目前还没有思路,学习到的逻辑如下:
   - 使用spring中schedule关键字
   - 确定同步的数据周期,即开始时间和结束时间,其中结束时间为当时,开始时间根据数据库中是否已经有数据进行确定:
     - 如果数据库中没有数据,则须从其他数据库中导入数据,开始时间确定为其他数据库中最早的一条数据的更新时间
     - 如果数据库中有对应数据,则根据update_time进行选择,可能需要记录上次同步的结束时间