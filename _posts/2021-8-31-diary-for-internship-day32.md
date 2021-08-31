---
layout: post
title:  "实习日记32(解决pdf水印问题)"
date:   2021-08-31
categories: jekyll update
---

## Day32

- 关于昨天下午的问题，Java8 lambda表达式使用局部变量final问题，需要遵循如下规则：

  - 只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。
  - 局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）
  - 不允许声明一个与局部变量同名的参数或者局部变量。

  其中可以对局部变量做增删改查噔内部改变(如add,removeIf等，但stream的filter()方法由于是返回一个新对象，所以同样会报错)

- pdf水印问题进度

  - 在看itext文档，源代码使用的是模板pdf，通过getOverContent()来获取一个PdfContentByte对象用于在原文件上图层写，原本以为源代码时使用getUnderContent()方法，但目前看来不是这个原因。
  - 源代码添加水印方法没有问题，怀疑是坐标问题，原pdf上的图下移导致坐标出现了问题
  - 确认是坐标问题，先尝试能不能动态获取内容坐标，不行的话只能直接修改pdf