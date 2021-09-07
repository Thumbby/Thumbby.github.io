---
layout: post
title:  "实习日记36(添加pdf水印问题)"
date:   2021-09-07
categories: jekyll update
---

## Day36

- 目前添加水印会存在水印略微偏移的情况，不过只存在个例

  **可能原因**：pdf内容与图层之间有偏移，现在在学习使用iText读取图层

  **尝试过的解决方案**：

  - 设置showTextAligned()方法的第一个参数设置为PdfContentByte.ALIGN_CENTER，即使用水印文字中心作为轴心，结果**无用**

  - 将showTextAligned()方法替换为showTextAligned()，该方法会根据字间隔进行自行调正，但是**无用**

  - 使用getPageSize()方法，但是该方法同样获取的是一个Rectangle对象，所以**无用**，没有差别。

  - 考虑Rotation情况，但有问题的pdf不存在旋转，**无用**

  - 尝试使用Document对象，但是该对象坐标保证左下角坐标恒为(0,0)，并且其坐标系建立与Rectangle对象坐标系不同，但为找到参考资料对比两者不同，**无果**

    **PS**：关于iText中Document和Rectangle两个类我有点问题，iText是根据pdf页数在每一页上分别进行操作的，同时我可以通过PdfStamper的getOverContent()方法获取到每一页的PdfContentByte对象，再调用getPageSize()方法即可获取到Rectangle对象，那么Document对象是如何获取的，本人见过利用Rectangle对象创建Document对象，那么Document代表着Pdf每一页的对象，即与PdfStamper同级的意思嘛。如果有人看到我的博客并且知道答案欢迎来告诉我。

  - 查看是否是因为Border缘故，但是调用getBorderWidth()方法发现返回的是-1？没有找到答案，此方法**作废**

  **确认问题**：对于出现问题的pdf文件，发现使用getRight()方法读取到的值与实际显示不符，导致水印添加偏移，目前没有找到解决方法。