---
layout: post
title:  "实习日记33(iText)"
date:   2021-09-01
categories: jekyll update
---

## Day33

- 添加水印位置不正确问题：

  - 在添加水印前先获取当前pdf的坐标位置，在添加水印的坐标上进行坐标的更改即可。

- 总结一下：

    - 方法一：检查PdfContentByte对象

      查看iText5文档可知，添加水印的方法主要是调用PdfContentByte对象的showTextAligned()方法，而PdfContentByte主要通过一下方法获得：

      - 调用PdfWriter的getDirectCount()方法
      - 调用PdfWriter的getDirectCountUnder()方法，该方法会导致水印在原本内容图层下而被阻挡
      - 调用PdfStamper的getOverContent()方法
      - 调用PdfStamper的getUnderCount()方法，该方法会导致水印在原本内容图层下而被阻挡

      如果需要水印不被遮挡，需要使用getDirectCount()或getOverContent()方法

    - 方法二：检查水印坐标

      在读入每一页的时候，先确定其坐标原点，使用代码如下：

      ```java
      float left = reader.getPageSizeWithRotation(num).getLeft();
      float bottom = reader.getPageSizeWithRotation(num).getBottom();
      ```

      由于不是所有的.pdf文件读取的坐标原点坐标都为(0,0)，所以需要进行坐标补位，插入水印代码例子如下：

      ```java
      int position = -75;
      int interval = -3;
       
      for (int height = (interval + textH) / 120; (float) height < pageRect.getHeight(); height += textH * 26) {
          for (int width = interval + textW - position; (float) width < pageRect.getWidth() + (float) textW; width += textW * 3) {
              under.showTextAligned(0, "thumbby", left + (float) (width - textW), bottom + (float) ((height - textH) * 2 / 3), 35.0F);
          }
       
          ++position;
      }
      ```

      在使用showTextAligned()方法插入水印时，坐标加上之前读取的pdf坐标轴原点坐标即可。

- Java FontMetrics

    - GetAscent()   //ascent表示字体从基线到顶端的距离
    - getDescent（）  //Descent表示字体从基线到下降字符底端的距离
    - getLeading()     //Leading 表示本文行之间的距离
    - getheight()    //字体高度  ascent+Descent+ Leading
    - StringWidth(String)   //字符串宽度

