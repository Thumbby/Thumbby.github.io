---
layout: post
title:  "实习日记28(EasyExcelDemo)"
date:   2021-08-24
categories: jekyll update
---

Day28

- 记点别的，23号因为牙齿疼看牙齿请了一天假，现在基本就是忍着牙齿疼来了，难受

- 继续easyExcel的demo，上周遗留的404问题依旧没解决，如果今天还没弄好就只能现在test中写一个测试，之后再去搞这个接口的问题吧，差不多到最晚下午三点的样子开始写导出图片的demo

- 原来是我url搞错了。。。。我是个傻逼(记录于上一条10min后)

- EasyExcel注意事项

  - 为了节省内存，EasyExcel没有采用把整个文档在内存中组织号之后再整体写入到文件的做法，而是采用一行一行写入的方法，不能实现删除和移动行，也不支持备注写入。多组数据写入的时候，如果需要新增行，只能在最后一行添加，不能再中间位置添加。

    代码示例：

    ```java
    		//创建工作簿对象
            ExcelWriter writerBook = EasyExcel.write("组合数据填充.xlsx", FillData.class).withTemplate(template).build();
    
            //创建工作表
            WriteSheet sheet =EasyExcel.writerSheet().build();
    
            //换行
            FillConfig fillConfig = FillConfig.builder().forceNewRow(true).build();
            
            //准备数据
            List<FillData> fillData = initFillData();
    
            HashMap<String, String> dateAndTotal = new HashMap<String, String>();
            dateAndTotal.put("date", "2114-5-14");
            dateAndTotal.put("total", "114514");
    
            //多组填充
            writerBook.fill(fillData, fillConfig, sheet);
    
            //单组填充
            writerBook.fill(dateAndTotal, sheet);
    
            //手动关闭流(因为没有使用doFill)
            writerBook.finish();
    ```

- EasyExcel导出图片，任务完成，记录于下午14:07，由于easyExcel无法读取图片所以只需导出图片即可，代码如下：

  ```
      @Test
      public void imageWrite() throws Exception{
          String fileName = "测试用表1_学生.xlsx";
          InputStream inputStream = null;
          try {
              List<Student> list = new ArrayList<Student>();
              Student student = new Student();
              list.add(student);
              student.setId("114514");
              student.setGender("男");
              student.setBirth(new Date());
              student.setName("田所浩二");
              //使用url插入图片,图片来自网络(确信
              student.setUrl(new URL("https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fi0.hdslb.com%2Fbfs%2Farticle%2F68db50227a2044998526d6ab55f93e9f60dae645.jpg&refer=http%3A%2F%2Fi0.hdslb.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1632376625&t=c6f93dabcf01320ed71482679be2278d"));
              EasyExcel.write(fileName, Student.class).sheet().doWrite(list);
          } finally {
              if (inputStream != null) {
                  inputStream.close();
              }
          }
      }
  ```

- EasyExcel不创建对象即可进行读写和下载的工具类

  参考https://blog.csdn.net/qq_34706514/article/details/105734770

  ```java
  package com.thumbby.easyexcel.dto;
  
  import com.alibaba.excel.EasyExcel;
  import com.alibaba.excel.write.style.column.LongestMatchColumnWidthStyleStrategy;
  
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  import java.io.OutputStream;
  import java.net.MalformedURLException;
  import java.net.URL;
  import java.net.URLEncoder;
  import java.util.ArrayList;
  import java.util.List;
  import java.util.Map;
  
  /**
   * @author: Thumbby
   * @description: EasyExcel导出工具类
   * @date: 2021-08-24 15:36
   **/
  public class EasyExcelUtils {
      /**
       * web浏览器写 自动列宽
       * 导出图片只支持单张，需要导出就在listkey的图片字段拼接showImg
       * @param response
       * @param fileName  文件名称
       * @param headArray 文件头
       * @param listKey   list key
       * @param dataList  数据list
       */
      public static void download(HttpServletResponse response, String fileName, String[] headArray, String[] listKey, List<Map<String, Object>> dataList) {
          try {
              EasyExcel.write(EasyExcelUtils.outputStream(response, fileName)).head(EasyExcelUtils.head(headArray))
                      .registerWriteHandler(new LongestMatchColumnWidthStyleStrategy()).sheet()
                      .doWrite(EasyExcelUtils.dataList(dataList, listKey));
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
  
      public static List<List<String>> head(String[] array) {
          List<List<String>> list = new ArrayList<List<String>>();
          for (String s : array) {
              List<String> head = new ArrayList<String>();
              head.add(s);
              list.add(head);
          }
          return list;
      }
  
      public static List<List<Object>> dataList(List<Map<String, Object>> list, String[] listKey) throws MalformedURLException {
          List<List<Object>> dataList = new ArrayList<List<Object>>();
          for (Map<String, Object> map : list) {
              List<Object> data = new ArrayList<Object>();
              for (String s : listKey) {
                  if (map.get(s) == null) {
                      data.add("");
                  } else {
                      //数据格式处理,可以根据自己的需求进行格式化操作都放在这里
                      Object obj = map.get(s);
                      if(s.contains("profile")  && (obj.toString().contains("http") || obj.toString().contains("https"))){
                          data.add(new URL(obj.toString()));
                      }else {
                          data.add(obj.toString());
                      }
                  }
              }
              dataList.add(data);
          }
          return dataList;
      }
  
      public static OutputStream outputStream(HttpServletResponse response, String fileName) throws IOException {
          response.setContentType("application/vnd.ms-excel");
          response.setCharacterEncoding("utf-8");
          //防止中文乱码
          fileName = URLEncoder.encode(fileName, "UTF-8");
          response.setHeader("Content-disposition", "attachment;filename=" + fileName + ".xlsx");
          return response.getOutputStream();
      }
  
  }
  ```

  具体使用可以看本人github上对应demo

- EasyExcel的demo遗留问题：现在插入了图片之后图片无法固定在单元格内，目前已知可能有如下解决方案等待测试：

  - 使用createFreezePane方法固定单元列

    **结果**：好像这个固定不是将图片固定在单元格内，可参考资料太少，我甚至都没找到这个方法

  - 其他方法后天再说吧，牙疼，明天想去医院再看下

