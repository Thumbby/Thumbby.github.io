---
layout: post
title:  "实习日记42(工作改接口)"
date:   2021-09-16
categories: jekyll update
---

Day42

- 继续改几个sql

- 我感觉我要重新学下sql语言，很多复杂查询还不太会，如果遇到像用Dataway这种以SQL和自身简单语法进行开发的框架，很多复杂逻辑需要用SQL处理，列入计划。

  - 简单case when函数：

    ```sql
    CASE SCORE WHEN 'A' THEN '优' ELSE '不及格' END
    CASE SCORE WHEN 'B' THEN '良' ELSE '不及格' END
    CASE SCORE WHEN 'C' THEN '中' ELSE '不及格' END
    ```

    等效于

    ```sql
    CASE WHEN SCORE = 'A' THEN '优'
         WHEN SCORE = 'B' THEN '良'
         WHEN SCORE = 'C' THEN '中' ELSE '不及格' END
    ```

     THEN后边的值与ELSE后边的值类型应一致，否则会报错

- 啊需求好乱好烦

