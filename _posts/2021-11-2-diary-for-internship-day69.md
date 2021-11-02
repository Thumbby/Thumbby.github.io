---
layout: post
title:  "实习日记69(java stream)"
date:   2021-11-02
categories: jekyll update
---

## Day69

- 继续弄那个问题，这东西感觉已经弄了一周多了。

- 现在是获取的数据tag一直为空，网上教程说更改查询语句如下：

  ```sql
  SELECT * FROM MEASUREMENT GROUP BY *
  ```

  这样就可以返回tag，但是返回的数据会只有第一项，需要更根据index返回不同的项，而且目前没有找到能返回field的键值对方法。

- 学习java stream操作：

  - stream()：为集合创建串行流

  - parallelStream()：为集合创建并行流

    ```java
    List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
    List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
    ```

  - forEach()：迭代流中的每个数据

    ```java
    Random random = new Random();
    random.ints().limit(10).forEach(System.out::println);
    ```

  - map：映射每个元素到对应的结果

    ```java
    List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
    // 获取对应的平方数
    List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
    ```

  - filter：通过设置的条件过滤出元素：

    ```java
    List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
    // 获取空字符串的数量
    long count = strings.stream().filter(string -> string.isEmpty()).count();
    
    ```

  - limit：获取指定数量的流

    ```java
    Random random = new Random();
    random.ints().limit(10).forEach(System.out::println);
    ```

  - sorted：用于对流的排序

    ```java
    Random random = new Random();
    random.ints().limit(10).sorted().forEach(System.out::println);
    ```

  - 并行(parallel)程序

    使用 parallelStream 来输出空字符串的数量：

    ```java
    List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
    // 获取空字符串的数量
    long count = strings.parallelStream().filter(string -> string.isEmpty()).count();
    ```

  - Collectors:

    Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

    ```java
    List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
    List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
     
    System.out.println("筛选列表: " + filtered);
    String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
    System.out.println("合并字符串: " + mergedString);
    ```

  - 统计：一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

    ```java
    List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
     
    IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
     
    System.out.println("列表中最大的数 : " + stats.getMax());
    System.out.println("列表中最小的数 : " + stats.getMin());
    System.out.println("所有数之和 : " + stats.getSum());
    System.out.println("平均数 : " + stats.getAverage());
    ```

    