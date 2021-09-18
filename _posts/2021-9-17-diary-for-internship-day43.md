---
layout: post
title:  "实习日记43(DataQL)"
date:   2021-09-18
categories: jekyll update
---

## Day43

- 学习到了分库分表的重要性，sql查询语句级别的优化是有极限的。

- 我不搞优化啦JOJO！

- 好像Dataway返回的结果重新包装一下会快一点，我也不知道为什么

- DataQL中某个集合函数库中的函数：size()

  - 函数定义：**int size(target)**

    - 参数定义：**target** 类型：**任意**
    - 返回类型：**int**
    - 作用：返回对象或数组的长度。

    说明：

    - 如果是空：返回 **0**
    - 如果是 **Map** 返回字段数量
    - 如果是数组：返回数组长度

  - 使用例：

    ```js
    collect.size(null)   = 0
    collect.size([]),    = 0
    collect.size([null]) = 1
    collect.size({}),    = 0
    collect.size({"key1":1, "key2":2, "key3":3 }) = 3
    ```

  - **注意**：该函数需要在 4.1.10 版本及其后续版本中才可以使用。

  - **注意**：目前这个项目使用的DataQL版本为4.1.7。。。。。

- 顺便记录下DataQL中FunctionX库函数

  - 转换函数库

    - toInt

      函数定义：**Number toInt(target)**

      - 参数定义：**target** 类型：**Object**
      - 返回类型：**Number**
      - 作用：将参数转换为 **Number**

      例子：

      ```js
      convert.toInt("12")     = 12
      convert.toInt(12)       = 12
      convert.toInt("0x12")   = 18
      convert.toInt("1.2e10") = 12000000000
      convert.toInt("")       = 0
      convert.toInt(null)     = 0
      convert.toInt("abc")    = throw Error
      ```

    - toString

      函数定义：**String toString(target)**

      - 参数定义：**target** 类型：**Object**
      - 返回类型：**String**
      - 作用：将参数转换为 **String**

      内部实现逻辑为： **String.valueOf(target)**

      例子:

      ```js
      convert.toString("12")        = "12"
      convert.toString(12)          = "12"
      convert.toString("0x12")      = "0x12"
      convert.toString("1.2e10")    = "1.2e10"
      convert.toString("")          = ""
      convert.toString(null)        = "null"
      convert.toString("abc")       = "abc"
      convert.toString([1,2,3,4])   = "[1, 2, 3, 4]"
      convert.toString({"tet":123}) = "{tet=123}"
      ```

    - toBlooean

      函数定义：**Boolean toBoolean(target)**

      - 参数定义：**target** 类型：**Object**
      - 返回类型：**Boolean**
      - 作用：将参数转换为 **Boolean**

      例子：

      ```js
      convert.toBoolean(null)    = null
      convert.toBoolean("true")  = Boolean.TRUE
      convert.toBoolean("false") = Boolean.FALSE
      convert.toBoolean("on")    = Boolean.TRUE
      convert.toBoolean("ON")    = Boolean.TRUE
      convert.toBoolean("off")   = Boolean.FALSE
      convert.toBoolean("oFf")   = Boolean.FALSE
      convert.toBoolean("blue")  = null
      ```

    - byteToHex

      函数定义：**String byteToHex(target)**

      - 参数定义：**target** 类型：**List<Byte>**
      - 返回类型：**String**
      - 作用：将二进制数据转换为16进制字符串

      例子：

      ```js
      convert.byteToHex([123]) = '7B'
      convert.byteToHex([])    = ''
      convert.byteToHex(null)  = null
      ```

    - hexToByte

      函数定义：**List<Byte> hexToByte(target)**

      - 参数定义：**target** 类型：**String**
      - 返回类型：**List<Byte>**
      - 作用：将16进制字符串转换为二进制数据

      例子：

      ```js
      convert.hexToByte('7B7B') = [123,123]
      convert.hexToByte('')     = []
      convert.hexToByte(null)   = null
      ```

    - stringToByte

      函数定义：**List<Byte> stringToByte(target, charset)**

      - 参数定义：**target** 类型：**String**；**charset** 类型：**String**，字符集
      - 返回类型：**List<Byte>**
      - 作用：字符串转换为二进制数据。

      例子：

      ```js
      convert.stringToByte('1234','utf-8')  = [49,50,51,52]
      convert.stringToByte('1234','utf-16') = [-2,-1,0,49,0,50,0,51,0,52]
      ```

    - byteToString

      函数定义：**String byteToString(target, charset)**

      - 参数定义：**target** 类型：**List<Byte>**；**charset** 类型：**String**，字符集
      - 返回类型：**String**
      - 作用：二进制数据转换为字符串。

      例子：

      ```js
      convert.byteToString([49,50,51,52], 'UTF-8')                = '1234'
      convert.byteToString([-2,-1,0,49,0,50,0,51,0,52], 'UTF-16') = '1234'
      ```

  - 集合函数库

    - isEmpty

      函数定义：**boolean isEmpty(target)**

      - 参数定义：**target** 类型：**List/Map**
      - 返回类型：**boolean**
      - 作用：判断集合或对象是否为空。si

      说明：

      - **ObjectModel** 等价于 **Map**
      - **ListModel** 等价于 **List**
      - 数组 等价于 **List**

      \- 如果是一个空 Map 那么返回 true，否则返回 false
      \- 如果是一个空 List 那么返回 true，否则返回 false

      例子：

      ```js
      collect.isEmpty([])          = true  // 空集合
      collect.isEmpty({})          = true  // 空对象
      collect.isEmpty([0,1,2])     = false // 集合不为空
      collect.isEmpty({'key':123}) = false // 对象含有至少一个属性
      collect.isEmpty(null)        = false // 不支持的基本类型会返回 false
      ```

    - size(见上文)

    - merge

      函数定义：**List merge(target_1, target_2, target_3, ..., target_n)**

      - 参数定义：**target** 类型：**任意**
      - 返回类型：**List**
      - 作用：合并多个对象或者集合成为一个新的集合。

      说明：

      \- 将多个 **List** 入参合并成一个集合，或者将多个对象合并成一个集合。

      例子：

      ```js
      collect.merge([0], [1,2], [3,4,5]) = [0,1,2,3,4,5] // 将三个集合合并成一个集合。
      collect.merge(0,1,2,3,4) = [0,1,2,3,4]             // 将多个对象合并成一个集合
      collect.merge([0,1,2], 3, 4, 5) = [0,1,2,3,4,5]    // 集合和对象混合存在会自动归并。
      ```

    - mergeMap

      函数定义：**Map mergeMap(target_1, target_2, target_3, ..., target_n)**

      - 参数定义：**target** 类型：**Map**
      - 返回类型：**Map**
      - 作用：合并多个对象合成为一个新的对象，当 **Key** 冲突会覆盖老数据。

      说明：

      \- 将多个对象合并成一个对象。或者将多个对象合并成一个集合。
      \- 如果入参不是 Map / ObjectModel，会引发错误，并终止后续查询的执行。

      例子：

      ```js
      var data1 = {"key1":1, "key2":2, "key3":3 }
       
      var data2 = {"key4":4, "key5":5, "key3":6 }
      var result = collect.mergeMap(data1,data2)
      // result = { "key1":1, "key2":2, "key3":6, "key4":4, "key5":5} // 合并两个Map，由于key3冲突，后面的会覆盖前面的。
       
      collect.mergeMap(data1,data2,[]) // throw "all args must be Map."
      ```

    - filter

      函数定义：**List filter(dataList, filterUDF)**

      - 参数定义：**dataList** 类型：**List**，待过滤的原始数据； **filterUDF** 类型：**Udf/Lambda**，过滤的规则函数；
      - 返回类型：**List**
      - 作用：根据规则函数来对集合进行过滤。

      说明：

      \- 根据一个规则来过滤集合中的数据。

      例子：

      ```js
      var dataList = [
          {"name" : "马一" , "age" : 18 },
          {"name" : "马二" , "age" : 28 },
          {"name" : "马三" , "age" : 30 },
          {"name" : "马四" , "age" : 25 }
      ]
       
      var result = collect.filter(dataList, (dat) -> {
          return dat.age > 20;
      });
       
      // result = [
      //     {"name" : "马二" , "age" : 28 },
      //     {"name" : "马三" , "age" : 30 },
      //     {"name" : "马四" , "age" : 25 }
      // ]
      ```

    - filterMap

      函数定义：**Map filterMap(dataMap, keyFilterUDF)**

      - 参数定义：**dataMap** 类型：**Map**，待过滤的原始数据； **keyFilterUDF** 类型：**Udf/Lambda**，过滤 **Key** 的规则函数；
      - 返回类型：**Map**
      - 作用：根据规则函数来对 **Map** 进行过滤。

      说明：

      \- 根据一个规则来过滤Map中的数据。

      例子：

      ```js
      var dataMap = {
          "key1" : "马一",
          "key2" : "马二",
          "key3" : "马三",
          "key4" : "马四"
      }
       
      var result = collect.filterMap(dataMap, (key) -> {
          return key == 'key1' || key == 'key3' || key == 'key5'
      });
       
      // result = { "key1": "马一", "key3": "马三" }
      ```

    - limit

      函数定义：**List limit(dataList, start, limit)**

      - 参数定义：**dataList** 类型：**List**，原始数据；**start** 类型：**Integer**，截取的起始位置； **limit** 类型：**Integer**，截取长度；
      - 返回类型：**List**
      - 作用：截取 List 的一部分，返回一个集合。

      说明：

      \- 截取List的一部分，返回一个新的子数据集。

      例子：

      ```js
      var dataList = [0,1,2,3,4,5,6,7,8,9]
      var result = collect.limit(dataList, 3,4);
       
      // result = [3,4,5,6] -> start从0开始
      var result = collect.limit(dataList, 3,0);
       
      // result = [3,4,5,6,7,8,9] -> limit 小于等于0表示全部
      ```

    - newList

      函数定义：**Map newList(target)**

      - 参数定义：**target** 类型：**任意**，初始化数据或集合；
      - 返回类型：**Map**
      - 作用：创建一个带有状态的 **List**。

      说明：

      \- 带有状态的 List ，类似于 ArrayList 对象。
      \- 提供三个子方法来使用：addFirst(target)、addLast(target)、data()、size()
      \- 提示：由于 DataQL 只能表示无状态的数据，并不能表示有状态的对象。因此为了表示一个带有状态的对象，通常是创建一组UDF，这些 UDF 内部共享同一个对象。

      例子：

      ```js
      // 多维数组打平成为一纬
      var data = [
          [1,2,3,[4,5]],
          [6,7,8,9,0]
      ]
       
      var foo = (dat, arrayObj) -> {
          // 无论 dat 是什么都将其转换为数组（符号 '#' 相当于在循环 dat 数组期间的当前元素）
          var tmpArray = dat => [ # ];
          // 如果 dat 是最终元素，在将其转换为 List 的时会作为第一个元素存在。这里判断可以断言dat是末级元素。
          if (tmpArray[0] == dat) {
              run arrayObj.addLast(dat); // 末级元素直接加到最终的集合中，否则就继续遍历集合
          } else {
              run tmpArray => [ foo(#,arrayObj) ]; // 继续递归遍历，直至末级。
          }
          return arrayObj;
      }
       
      var newList = collect.newList();
      var result = foo(data, newList).data();
       
      // result = [1,2,3,5,6,7,8,9,0]
      ```

    - newMap

      函数定义：**Map newMap(target)**

      - 参数定义：**target** 类型：任意，初始化数据或集合；
      - 返回类型：**Map**
      - 作用：创建一个带有状态的 **List**。

      说明：

      \- 该函数需要在 4.1.10 版本及其后续版本中才可以使用。
      \- 带有状态的 Map，类似于 **LinkedHashMap** 对象。
      \- 提供三个子方法来使用：**put(target)**、**putAll(target)**、**data()**、**size()
      **- 提示：由于 DataQL 只能表示无状态的数据，并不能表示有状态的对象。因此为了表示一个带有状态的对象，通常是创建一组UDF，这些 UDF 内部共享同一个对象。

      例子：

      ```js
      var mapData = collect.newMap({'key':123 });
      // 调用 sss.data() 的结果是
      // {
      //     "key": 123
      // }
       
      var mapData = mapData.put('sss','sss')
      // 调用 sss.data() 的结果是
      // {
      //     "key": 123,
      //     "sss": "sss"
      // }
       
      var mapData = mapData.putAll({'id':1, 'parent_id':null, 'label': 't1'})
      // 调用 sss.data() 的结果是
      // {
      //     "key": 123,
      //     "sss": "sss",
      //     "id": 1,
      //     "parent_id": null,
      //     "label": "t1"
      // }
      ```

    - mapJoin

      函数定义：**List mapJoin(data_1, data_2, joinMapping)**

      - 参数定义：**data_1** 类型：**List**，左表数据；**data_2** 类型：**List**，右表数据；**joinMapping** 类型：**Map**，两表的 **join** 关系；
      - 返回类型：**List**
      - 作用：将两个 **Map/List** 进行左链接，行为和 sql 中的 **left join** 相同。

      说明：

      \- 左连接形式，连接两个数据集。
      \- 提示：目前 **mapJoin** 函数只支持一个连接条件。

      例子：

      ```js
      var year2019 = [
          { "pt":2019, "item_code":"code_1", "sum_price":2234 },
          { "pt":2019, "item_code":"code_2", "sum_price":234 },
          { "pt":2019, "item_code":"code_3", "sum_price":12340 },
          { "pt":2019, "item_code":"code_4", "sum_price":2344 }
      ];
       
      var year2018 = [
          { "pt":2018, "item_code":"code_1", "sum_price":1234.0 },
          { "pt":2018, "item_code":"code_2", "sum_price":1234.0 },
          { "pt":2018, "item_code":"code_3", "sum_price":1234.0 },
          { "pt":2018, "item_code":"code_4", "sum_price":1234.0 }
      ];
       
      var result = collect.mapJoin(year2019,year2018, { "item_code":"item_code" }) => [
          {
              "商品Code": data1.item_code,
              "去年同期": data2.sum_price,
              "今年总额": data1.sum_price,
              "环比去年增长": ((data1.sum_price - data2.sum_price) / data2.sum_price * 100) + "%"
          }
      ]
       
      // result = [
      //     {"商品Code":"code_1", "去年同期":1234.0, "今年总额":2234, "环比去年增长":"81.04%"},
      //     {"商品Code":"code_2", "去年同期":1234.0, "今年总额":234, "环比去年增长":"-81.04%"},
      //     {"商品Code":"code_3", "去年同期":1234.0, "今年总额":12340,"环比去年增长":"900.0%"},
      //     {"商品Code":"code_4", "去年同期":1234.0, "今年总额":2344, "环比去年增长":"89.95%"}
      // ]
      ```

    - mapKeyToLowerCase

      函数定义：**Map mapKeyToLowerCase(dataMap)**

      - 参数定义：**dataMap** 类型：**Map**，准备要转换的 **Map** 对象；
      - 返回类型：**Map**
      - 作用：将 **Map** 的 Key 全部转为小写，如果 Key 有冲突会产生覆盖。

      例子：

      ```js
      var mapData = {
          "abc" : "aa",
          "ABC" : "bb",
          "test_abc" : "cc"
      }
       
      var result = collect.mapKeyToLowerCase(mapData)
      // result = { "abc": "bb", "test_abc": "cc" }
      ```

    - mapKeyToUpperCase

      函数定义：**Map mapKeyToUpperCase(dataMap)**

      - 参数定义：**dataMap** 类型：**Map**，准备要转换的 **Map** 对象；
      - 返回类型：**Map**
      - 作用：将 **Map** 的 Key 全部转为大写，如果 Key 有冲突会产生覆盖。

      例子：

      ```js
      var mapData = {
          "abc" : "aa",
          "ABC" : "bb",
          "test_abc" : "cc"
      }
       
      var result = collect.mapKeyToUpperCase(mapData)
      // result = { "ABC": "bb", "TEST_ABC": "cc" }
      ```

    - mapKeyToHumpCases

      函数定义：**Map mapKeyToHumpCase(dataMap)**

      - 参数定义：**dataMap** 类型：**Map**，准备要转换的 **Map** 对象；
      - 返回类型：**Map**
      - 作用：将 **Map** 的 Key 中下划线做驼峰转换。

      例子：

      ```js
      var mapData = {
          "abc" : "aa",
          "ABC" : "bb",
          "test_abc" : "cc"
      }
       
      var result = collect.mapKeyToHumpCase(mapData)
      // result = { "ABC": "bb", "testAbc": "cc" }
      ```

- 还在和前端联调ing，DataQL后续再更