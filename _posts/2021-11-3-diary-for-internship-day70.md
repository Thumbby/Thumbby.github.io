---
layout: post
title:  "实习日记70(java Date格式)"
date:   2021-11-03
categories: jekyll update
---

## Day70

- 我感觉其实最近出了在做那个老事情之外也没啥好记录的。

- java UTC时间的String转Date等格式：

  ```java
  public class Test {
  public static void main(String[] args) {
  //此方法是将2017-11-18T07:12:06.615Z格式的时间转化为秒为单位的Long类型。
  String time = "2017-11-30T10:41:44.651Z";
  time = time.replace("Z", " UTC");//UTC是本地时间
  SimpleDateFormat format =new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS Z");
  Date d = null;
  try {
  d = format.parse(time);
  } catch (ParseException e) {
  // TODO Auto-generated catch block
  e.printStackTrace();
  }
  //此处是将date类型装换为字符串类型，比如：Sat Nov 18 15:12:06 CST 2017转换为2017-11-18 15:12:06
  SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  String date = sf.format(d);
  System.out.println(d);//这里输出的是date类型的时间
  System.out.println(d.getTime()/1000);//这里输出的是以秒为单位的long类型的时间。如果需要一毫秒为单位，可以不用除1000.
          System.out.println(sf.format(d));//这里输出的是字符串类型的时间
  }
  }
  
  ```

  结果：

  ```java
  Thu Nov 30 18:41:44 CST 2017
  1512038504
  2017-11-30 18:41:44
  ```

  