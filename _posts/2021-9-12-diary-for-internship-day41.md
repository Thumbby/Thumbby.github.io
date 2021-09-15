---
layout: post
title:  "实习日记41(DataQL优化)"
date:   2021-09-15
categories: jekyll update
---

## Day41

- 城运大屏做一点维护，改几个sql，没什么好说的，不过感觉像这种长期没怎么干的项目要自己一个个接口去对应前端还是有点烦的

- DataQL感觉还是有点东西的，记录一下：

  - 拼接字符串时发现每次使用已经定义的变量时还是得加var,不然会报错，如下：

    ```js
    var whereSqlDateType = "";
    var whereSqlDateType = whereSqlDateType + "QUARTER(open_ts)=QUARTER(now()) and DATE_FORMAT(open_ts,'%Y')=DATE_FORMAT(now(),'%Y')";
    ```

  - 变换查询结果：

    ```js
    return query(${type}, whereSql) => [
        {
            "id" : id,
            "description" : description,
        }
    ]
    ```

  - 批量操作

    DataQL 的 SQL 执行器支持批量 **Insert\Update\Delete\Select** 操作，最常见的场景是批量插入数据。批量操作必须满足下列几点要求：

    - 入参必须是 List
    - 如果有多个入参。所有参数都必须是 List 并且长度必须一致。
    - **@@sql()<% ... %>** 写法升级为批量写法 **@@sql[]()<% ... %>**
    - 如果批量操作的 SQL 中存在 SQL注入，那么批量操作会自动退化为：**循环遍历模式**

    还是上面插入数据的例子，采用批量模式之后 SQL 部分不变，只是把 **@@sql** 改为 **@@sql[]**。入参数转换为数组即可。

    代码如下：

    ```js
    // 例子数据
    var testData = [
        { "name" : "马一", "age" : 26, "status" : 0 },
        { "name" : "马二", "age" : 26, "status" : 0 },
        { "name" : "马三", "age" : 26, "status" : 0 }
    ]
     
    // insert语句模版
    var insertSqlFun = @@sql[](userInfo) <%
        insert into user_info (
            name,
            age,
            status,
            create_time
        ) values (
            #{userInfo.name},
            #{userInfo.age},
            #{userInfo.status},
            now()
        )
    %>
     
    // 批量操作
    return insertSqlFun(testData);
    ```

- 现在有这么一个需求：返回的是两个列表，分别表示状态1和状态2，这两个状态在数据库字段中只能通过模糊查询得到，在使用DataWay的情况下，由于数据量很大并且目前实现要执行两边SQL，导致效率很慢，DataQL代码如下：

  ```js
  var whereSqlType = "";
  if(${type} != null && ${type} != ""){
      if(${type} == "all"){
          var whereSqlType = whereSql;
      }
      else if(${type}== "local"){
          var whereSqlType = whereSqlType + " and tags like '%地方处置%'" ;
      }
      else if(${type}== "railway"){
          var whereSqlType = whereSqlType + " and tags not like '%地方处置%'";
      }
      else 
          return null;
  }
  
  
  var whereSqlDateType = "";
  if(${dateType} != null && ${dateType} != ""){
      if(${dateType} == "day"){
          var whereSqlDateType = whereSqlDateType+"DATE_FORMAT(open_ts,'%Y-%m-%d')=DATE_FORMAT(now(),'%Y-%m-%d')";
      }
      else if(${dateType}== "month"){
          var whereSqlDateType = whereSqlDateType + "DATE_FORMAT(open_ts,'%Y-%m')=DATE_FORMAT(now(),'%Y-%m')" ;
      }
      else if(${dateType}== "season"){
          var whereSqlDateType = whereSqlDateType + "QUARTER(open_ts)=QUARTER(now()) and DATE_FORMAT(open_ts,'%Y')=DATE_FORMAT(now(),'%Y')";
      }
      else if(${dateType}== "year"){
          var whereSqlDateType = whereSqlDateType + "DATE_FORMAT(open_ts,'%Y')=DATE_FORMAT(now(),'%Y')";
      }
      else 
          return null;
  }
  
  var query_local = @@sql(whereSqlType,whereSqlDateType)<%
  select open_ts,args_event_name,data_address,data_status_name from railwaycurrent where ${whereSqlDateType} 
  ${whereSqlType} and tags like '%地方发现%'
  %>
  
  var result_local = query_local(whereSqlType,whereSqlDateType)
  
  var query_railway = @@sql(whereSqlType,whereSqlDateType)<%
  select open_ts,args_event_name,data_address,data_status_name from railwaycurrent where ${whereSqlDateType} 
  ${whereSqlType} and tags not like '%地方发现%'
  %>
  
  var result_railway = query_railway(whereSqlType,whereSqlDateType)
  
  return {"cases_local":result_local=>[{"open_ts","args_event_name","data_address","data_status_name"}], 
  "cases_railway":result_railway=>["open_ts","args_event_name","data_address","data_status_name"]}
  ```

  现在想办法优化

  **后续**：我真的想不出有什么能在查询语句上面优化的办法了，如果先获取查询结果再遍历分割那效率还不如原来的，就这样吧，一个请求数据交互要个10多秒，我太菜了，DataQL也不熟练（这东西网上也没多少资料，气死我了）