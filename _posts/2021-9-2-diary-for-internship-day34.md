---
layout: post
title:  "实习日记34(iText水印坐标)"
date:   2021-09-02
categories: jekyll update
---

## Day34

- 现在需求是根据输入需要的水印行数和列数均分，目前正在努力

- 已经实现大部分功能，代码如下：

  ```java
      public void read() throws IOException, DocumentException {
  
          PdfReader reader = new PdfReader("test.pdf");
          PdfStamper stamper = new PdfStamper(reader, new FileOutputStream("result.pdf"));
  
          //设置水印
          BaseFont base = BaseFont.createFont(BaseFont.HELVETICA, BaseFont.WINANSI, BaseFont.NOT_EMBEDDED);
          PdfGState gs = new PdfGState();
          gs.setFillOpacity(0.6F);
          gs.setStrokeOpacity(0.6F);
  
          int total = reader.getNumberOfPages()+1;
          JLabel label = new JLabel();
          label.setText("thumbby");
          FontMetrics metrics = label.getFontMetrics(label.getFont());
          //字符串长度
          float textH = metrics.getHeight();
          //字符串宽度
          float textW = metrics.stringWidth(label.getText());
  
          System.out.format("textH = %f, textW = %f\n", textH, textW);
  
          for(int num=1; num<total; num++) {
  
              Rectangle pageRect = reader.getPageSizeWithRotation(num);
  
              /*System.out.println("pageTop = " + pageRect.getTop());
              System.out.println("pageBottom = " + pageRect.getBottom());
              System.out.println("pageLeft = " + pageRect.getLeft());
              System.out.println("pageRight = " + pageRect.getRight());
              System.out.println("pageHeight = " + pageRect.getHeight());
              System.out.println("pageWidth = " + pageRect.getWidth());*/
  
              PdfContentByte under = stamper.getOverContent(num);
              under.saveState();
              under.setGState(gs);
              under.beginText();
              under.setColorFill(BaseColor.LIGHT_GRAY);
              under.setFontAndSize(base, 36.0F);
              int xNumber = 3;
              int yNumber = 3;
              float xInterval = pageRect.getWidth()/(xNumber+1);
              float yInterval = pageRect.getHeight()/(yNumber+1);
              System.out.format("xInterval = %f, yInterval = %f\n", xInterval, yInterval);
              /*for(int pointY = 1; pointY < yNumber +1; pointY++){
                  for(int pointX = 1; pointX < xNumber +1; pointX++){
                      under.showTextAligned(PdfContentByte.ALIGN_CENTER, "thumbby", pageRect.getLeft() + pointX*xInterval, pageRect.getBottom() + pointY*yInterval, 35.0F);
                  }
              }*/
              for(float height = yInterval; height < pageRect.getHeight(); height += yInterval){
                  for(float width = xInterval; width < pageRect.getWidth(); width += xInterval){
                      System.out.format("width=%f height=%f\n", pageRect.getLeft() + width, pageRect.getBottom() + height);
                      under.showTextAligned(PdfContentByte.ALIGN_CENTER, "thumbby", pageRect.getLeft() + width, pageRect.getBottom() + height, 35.0F);
                  }
              }
  
              under.endText();
          }
          stamper.close();
          reader.close();
      }
  ```

  - 目前存在如下问题：如果pdf内容与展示页面不符则会造成水印偏移，正在解决问题。
  - 由于该包中各种方法获取的坐标结果一致，所以该问题目前解决方案只能通过在生成pdf文件时尽量避免该种情况发生。

