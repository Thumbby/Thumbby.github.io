---
layout: post
title:  "实习日记55(工作任务)"
date:   2021-10-12
categories: jekyll update
---

## Day55

- 一直在搞工作，没啥记的

- springBoot搭建时遇到的坑之Failed to configure a DataSource: 'url' attribute is not specified and no embedded

  - 因为我在pom文件中添加了mybatis依赖，但是我没有配置连接数据库的url、用户名user 、和密码 password
  - **解决方法**：添加数据库配置，或者直接删除pom中依赖，这次情况由于在web层的pom文件中添加了依赖，故删除即可。

- mybatisplus模糊查询:

  ```sql
  name LIKE CONCAT('%',#{ew.entity.name},'%')
  ```

  

