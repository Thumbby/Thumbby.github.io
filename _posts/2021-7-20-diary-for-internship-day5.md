---
layout: post
title:  "实习日记5"
date:   2021-07-20
categories: jekyll update
---

## Day5

- 开发手册学习：
   - 数据库索引规约：
     - 唯一索引影响的insert速度损耗可以忽略，但会明显提高查找速度，另外，即使在应用层做了非常完善的校验控制，只要没有唯一索引，根据墨菲定律，就必然有脏数据生成。
     - 在varchar字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据实际文本区分度决定索引长度即可。索引的长度与区分度是一对矛盾体，一般对于字符串类型数据，长度为20的索引，区分度会高达90%以上，可以使用count(distinct left(列名，索引长度))/count(*)的区分度来确定
     - 页面所搜严禁做模糊或者全模糊，如果需要，通过搜索引擎来解决，因为索引文件具有B-Tree的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索引。
     - SQL性能优化的目标：至少达到range级别，要求是ref级别，最好是consts级别。
       - consts级别是指单表中最多只有一个匹配行(主键或者唯一索引，在优化阶段即可读取到数据)
       - ref级别是指使用普通的索引(normal index)
       - range级别是指对索引进行范围检索
     - count()会值为NULL的行，而count(列名)不会统计此列值为NULL的行，所以不要使用count(列名)或count(常量)来替代count(*)
     - SQL中使用ISNULL()判断是否为NULL值，因为NULL与任何值的直接比较都为NULL。
- 源代码学习：
   - mybatisplus:
     - 条件构造器(wrapper):
       - 说明：
         - 以下出现的第一个入参`boolean condition`表示该条件**是否**加入最后生成的sql中，例如：query.like(StringUtils.isNotBlank(name), Entity::getName, name) .eq(age!=null && age >= 0, Entity::getAge, age)
         - 以下代码块内的多个方法均为从上往下补全个别`boolean`类型的入参,默认为`true`
         - 以下出现的泛型`Param`均为`Wrapper`的子类实例(均具有`AbstractWrapper`的所有方法)
         - 以下方法在入参中出现的`R`为泛型,在普通wrapper中是`String`,在LambdaWrapper中是**函数**(例:`Entity::getId`,`Entity`为实体类,`getId`为字段`id`的**getMethod**)
         - 以下方法入参中的`R column`均表示数据库字段,当`R`具体类型为`String`时则为数据库字段名(**字段名是数据库关键字的自己用转义符包裹!**)!而不是实体类数据字段名!!!,另当`R`具体类型为`SFunction`时项目runtime不支持eclipse自家的编译器!!!
         - 以下举例均为使用普通wrapper,入参为`Map`和`List`的均以`json`形式表现!
         - 使用中如果入参的`Map`或者`List`为**空**,则不会加入最后生成的sql中!!!
   - 根据数据表类型分别调用不同的DAO实现，如果是树形结构则调用ITree，反之调用IAdv