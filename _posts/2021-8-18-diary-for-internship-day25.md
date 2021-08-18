---
layout: post
title:  "实习日记25(EasyExcel)"
date:   2021-08-18
categories: jekyll update
---

## Day25

- EasyExcel官方文档学习

  - 读Excel

    - 最简单的读

      **excel示例**

      ![img](https://cdn.nlark.com/yuque/0/2020/png/553000/1584450793123-1fe49477-0609-4fd8-8ef8-0b907141486f.png?x-oss-process=image%2Fresize%2Cw_332#align=left&display=inline&height=229&originHeight=229&originWidth=332&size=0&status=done&style=none&width=332)

      **对象**

      ```java
      @Data
      public class DemoData {
          private String string;
          private Date date;
          private Double doubleData;
      }
      ```

      **监听器**

      ```java
      // 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
      public class DemoDataListener extends AnalysisEventListener<DemoData> {
          private static final Logger LOGGER = LoggerFactory.getLogger(DemoDataListener.class);
          /**
           * 每隔5条存储数据库，实际使用中可以3000条，然后清理list ，方便内存回收
           */
          private static final int BATCH_COUNT = 5;
          List<DemoData> list = new ArrayList<DemoData>();
          /**
           * 假设这个是一个DAO，当然有业务逻辑这个也可以是一个service。当然如果不用存储这个对象没用。
           */
          private DemoDAO demoDAO;
          public DemoDataListener() {
              // 这里是demo，所以随便new一个。实际使用如果到了spring,请使用下面的有参构造函数
              demoDAO = new DemoDAO();
          }
          /**
           * 如果使用了spring,请使用这个构造方法。每次创建Listener的时候需要把spring管理的类传进来
           *
           * @param demoDAO
           */
          public DemoDataListener(DemoDAO demoDAO) {
              this.demoDAO = demoDAO;
          }
          /**
           * 这个每一条数据解析都会来调用
           *
           * @param data
           *            one row value. Is is same as {@link AnalysisContext#readRowHolder()}
           * @param context
           */
          @Override
          public void invoke(DemoData data, AnalysisContext context) {
              LOGGER.info("解析到一条数据:{}", JSON.toJSONString(data));
              list.add(data);
              // 达到BATCH_COUNT了，需要去存储一次数据库，防止数据几万条数据在内存，容易OOM
              if (list.size() >= BATCH_COUNT) {
                  saveData();
                  // 存储完成清理 list
                  list.clear();
              }
          }
          /**
           * 所有数据解析完成了 都会来调用
           *
           * @param context
           */
          @Override
          public void doAfterAllAnalysed(AnalysisContext context) {
              // 这里也要保存数据，确保最后遗留的数据也存储到数据库
              saveData();
              LOGGER.info("所有数据解析完成！");
          }
          /**
           * 加上存储数据库
           */
          private void saveData() {
              LOGGER.info("{}条数据，开始存储数据库！", list.size());
              demoDAO.save(list);
              LOGGER.info("存储数据库成功！");
          }
      }
      ```

      **持久层**

      ```java
      /**
       * 假设这个是你的DAO存储。当然还要这个类让spring管理，当然你不用需要存储，也不需要这个类。
       **/
      public class DemoDAO {
          public void save(List<DemoData> list) {
              // 如果是mybatis,尽量别直接调用多次insert,自己写一个mapper里面新增一个方法batchInsert,所有数据一次性插入
          }
      }
      ```

      **使用代码**

      ```java
          /**
           * 最简单的读
           * <p>1. 创建excel对应的实体对象 参照{@link DemoData}
           * <p>2. 由于默认一行行的读取excel，所以需要创建excel一行一行的回调监听器，参照{@link DemoDataListener}
           * <p>3. 直接读即可
           */
          @Test
          public void simpleRead() {
              // 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
              // 写法1：
              String fileName = TestFileUtil.getPath() + "demo" + File.separator + "demo.xlsx";
              // 这里 需要指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
              EasyExcel.read(fileName, DemoData.class, new DemoDataListener()).sheet().doRead();
      
              // 写法2：
              fileName = TestFileUtil.getPath() + "demo" + File.separator + "demo.xlsx";
              ExcelReader excelReader = null;
              try {
                  excelReader = EasyExcel.read(fileName, DemoData.class, new DemoDataListener()).build();
                  ReadSheet readSheet = EasyExcel.readSheet(0).build();
                  excelReader.read(readSheet);
              } finally {
                  if (excelReader != null) {
                      // 这里千万别忘记关闭，读的时候会创建临时文件，到时磁盘会崩的
                      excelReader.finish();
                  }
              }
          }
      ```

    - 其余读法参考：https://www.yuque.com/easyexcel/doc/read

  - 写Excel

    **生成测试数据**：

    ```java
        private List<DemoData> data() {
            List<DemoData> list = new ArrayList<DemoData>();
            for (int i = 0; i < 10; i++) {
                DemoData data = new DemoData();
                data.setString("字符串" + i);
                data.setDate(new Date());
                data.setDoubleData(0.56);
                list.add(data);
            }
            return list;
        }
    ```

    - 最简单的写

      **对象**

      ```java
      @Data
      public class DemoData {
          @ExcelProperty("字符串标题")
          private String string;
          @ExcelProperty("日期标题")
          private Date date;
          @ExcelProperty("数字标题")
          private Double doubleData;
          /**
           * 忽略这个字段
           */
          @ExcelIgnore
          private String ignore;
      }
      ```

      **代码**

      ```java
          /**
           * 最简单的写
           * <p>1. 创建excel对应的实体对象 参照{@link DemoData}
           * <p>2. 直接写即可
           */
          @Test
          public void simpleWrite() {
              // 写法1
              String fileName = TestFileUtil.getPath() + "simpleWrite" + System.currentTimeMillis() + ".xlsx";
              // 这里 需要指定写用哪个class去写，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
              // 如果这里想使用03 则 传入excelType参数即可
              EasyExcel.write(fileName, DemoData.class).sheet("模板").doWrite(data());
      
              // 写法2
              fileName = TestFileUtil.getPath() + "simpleWrite" + System.currentTimeMillis() + ".xlsx";
              // 这里 需要指定写用哪个class去写
              ExcelWriter excelWriter = null;
              try {
                  excelWriter = EasyExcel.write(fileName, DemoData.class).build();
                  WriteSheet writeSheet = EasyExcel.writerSheet("模板").build();
                  excelWriter.write(data(), writeSheet);
              } finally {
                  // 千万别忘记finish 会帮忙关闭流
                  if (excelWriter != null) {
                      excelWriter.finish();
                  }
              }
          }
      ```

    - 其余写法参考：https://www.yuque.com/easyexcel/doc/write

    - 图片导出

      **excel示例**

      ![img](https://cdn.nlark.com/yuque/0/2020/png/553000/1584454123364-56f1f13f-810d-4763-a383-5c94c0a8a9be.png?x-oss-process=image%2Fresize%2Cw_448)

      **对象**

      ```java
      @Data
      @ContentRowHeight(100)
      @ColumnWidth(100 / 8)
      public class ImageData {
          private File file;
          private InputStream inputStream;
          /**
           * 如果string类型 必须指定转换器，string默认转换成string
           */
          @ExcelProperty(converter = StringImageConverter.class)
          private String string;
          private byte[] byteArray;
          /**
           * 根据url导出
           *
           * @since 2.1.1
           */
          private URL url;
      }
      ```

      **代码**

      ```java
          /**
           * 图片导出
           * <p>
           * 1. 创建excel对应的实体对象 参照{@link ImageData}
           * <p>
           * 2. 直接写即可
           */
          @Test
          public void imageWrite() throws Exception {
              String fileName = TestFileUtil.getPath() + "imageWrite" + System.currentTimeMillis() + ".xlsx";
              // 如果使用流 记得关闭
              InputStream inputStream = null;
              try {
                  List<ImageData> list = new ArrayList<ImageData>();
                  ImageData imageData = new ImageData();
                  list.add(imageData);
                  String imagePath = TestFileUtil.getPath() + "converter" + File.separator + "img.jpg";
                  // 放入五种类型的图片 实际使用只要选一种即可
                  imageData.setByteArray(FileUtils.readFileToByteArray(new File(imagePath)));
                  imageData.setFile(new File(imagePath));
                  imageData.setString(imagePath);
                  inputStream = FileUtils.openInputStream(new File(imagePath));
                  imageData.setInputStream(inputStream);
                  imageData.setUrl(new URL(
                      "https://raw.githubusercontent.com/alibaba/easyexcel/master/src/test/resources/converter/img.jpg"));
                  EasyExcel.write(fileName, ImageData.class).sheet().doWrite(list);
              } finally {
                  if (inputStream != null) {
                      inputStream.close();
                  }
              }
          }
      ```

  - 填充Excel

    - 最简单的填充

      **模板**：

      ![img](https://cdn.nlark.com/yuque/0/2020/png/553000/1584454713527-4a4500f8-a7ed-46e9-9d68-e282a0221b32.png?x-oss-process=image%2Fresize%2Cw_575)

      **最终效果**：

      ![img](https://cdn.nlark.com/yuque/0/2020/png/553000/1584454912576-3372f8c5-4e94-4960-9910-1c05802d6f26.png?x-oss-process=image%2Fresize%2Cw_490)

      **对象**：

      ```java
      @Data
      public class FillData {
          private String name;
          private double number;
      }
      ```

      **代码**

      ```java
          /**
           * 最简单的填充
           *
           * @since 2.1.1
           */
          @Test
          public void simpleFill() {
              // 模板注意 用{} 来表示你要用的变量 如果本来就有"{","}" 特殊字符 用"\{","\}"代替
              String templateFileName =
                  TestFileUtil.getPath() + "demo" + File.separator + "fill" + File.separator + "simple.xlsx";
      
              // 方案1 根据对象填充
              String fileName = TestFileUtil.getPath() + "simpleFill" + System.currentTimeMillis() + ".xlsx";
              // 这里 会填充到第一个sheet， 然后文件流会自动关闭
              FillData fillData = new FillData();
              fillData.setName("张三");
              fillData.setNumber(5.2);
              EasyExcel.write(fileName).withTemplate(templateFileName).sheet().doFill(fillData);
      
              // 方案2 根据Map填充
              fileName = TestFileUtil.getPath() + "simpleFill" + System.currentTimeMillis() + ".xlsx";
              // 这里 会填充到第一个sheet， 然后文件流会自动关闭
              Map<String, Object> map = new HashMap<String, Object>();
              map.put("name", "张三");
              map.put("number", 5.2);
              EasyExcel.write(fileName).withTemplate(templateFileName).sheet().doFill(map);
          }
      ```

    - 其余填充法参考：https://www.yuque.com/easyexcel/doc/fill

  - 常见API

    ##### **常见类解析**

    - EasyExcel 入口类，用于构建开始各种操作
    - ExcelReaderBuilder ExcelWriterBuilder 构建出一个 ReadWorkbook WriteWorkbook，可以理解成一个excel对象，一个excel只要构建一个

    - ExcelReaderSheetBuilder ExcelWriterSheetBuilder 构建出一个 ReadSheet WriteSheet对象，可以理解成excel里面的一页,每一页都要构建一个
    - ReadListener 在每一行读取完毕后都会调用ReadListener来处理数据

    - WriteHandler 在每一个操作包括创建单元格、创建表格等都会调用WriteHandler来处理数据
    - 所有配置都是继承的，Workbook的配置会被Sheet继承，所以在用EasyExcel设置参数的时候，在EasyExcel...sheet()方法之前作用域是整个sheet,之后针对单个sheet

    ##### **读**

    ###### **注解**

    - `ExcelProperty` 指定当前字段对应excel中的那一列。可以根据名字或者Index去匹配。当然也可以不写，默认第一个字段就是index=0，以此类推。千万注意，要么全部不写，要么全部用index，要么全部用名字去匹配。千万别三个混着用，除非你非常了解源代码中三个混着用怎么去排序的。
    - `ExcelIgnore` 默认所有字段都会和excel去匹配，加了这个注解会忽略该字段

    - `DateTimeFormat` 日期转换，用`String`去接收excel日期格式的数据会调用这个注解。里面的`value`参照`java.text.SimpleDateFormat`
    - `NumberFormat` 数字转换，用`String`去接收excel数字格式的数据会调用这个注解。里面的`value`参照`java.text.DecimalFormat`

    - `ExcelIgnoreUnannotated` 默认不加`ExcelProperty` 的注解的都会参与读写，加了不会参与

    ###### **参数**

    **通用参数**

    `ReadWorkbook`,`ReadSheet` 都会有的参数，如果为空，默认使用上级。

    - `converter` 转换器，默认加载了很多转换器。也可以自定义。
    - `readListener` 监听器，在读取数据的过程中会不断的调用监听器。

    - `headRowNumber` 需要读的表格有几行头数据。默认有一行头，也就是认为第二行开始起为数据。
    - `head`  与`clazz`二选一。读取文件头对应的列表，会根据列表匹配数据，建议使用class。

    - `clazz` 与`head`二选一。读取文件的头对应的class，也可以使用注解。如果两个都不指定，则会读取全部数据。
    - `autoTrim` 字符串、表头等数据自动trim

    - `password` 读的时候是否需要使用密码

    **ReadWorkbook（理解成excel对象）参数**

    - `excelType` 当前excel的类型 默认会自动判断
    - `inputStream` 与`file`二选一。读取文件的流，如果接收到的是流就只用，不用流建议使用`file`参数。因为使用了`inputStream` easyexcel会帮忙创建临时文件，最终还是`file`

    - `file` 与`inputStream`二选一。读取文件的文件。
    - `autoCloseStream` 自动关闭流。

    - `readCache` 默认小于5M用 内存，超过5M会使用 `EhCache`,这里不建议使用这个参数。
    - `useDefaultListener` `@since 2.1.4` 默认会加入`ModelBuildEventListener` 来帮忙转换成传入`class`的对象，设置成`false`后将不会协助转换对象，自定义的监听器会接收到`Map<Integer,CellData>`对象，如果还想继续接听到`class`对象，请调用`readListener`方法，加入自定义的`beforeListener`、 `ModelBuildEventListener`、 自定义的`afterListener`即可。

    **ReadSheet（就是excel的一个Sheet）参数**

    - `sheetNo` 需要读取Sheet的编码，建议使用这个来指定读取哪个Sheet
    - `sheetName` 根据名字去匹配Sheet,excel 2003不支持根据名字去匹配

    ##### 写

    ###### 注解

    - `ExcelProperty` index 指定写到第几列，默认根据成员变量排序。`value`指定写入的名称，默认成员变量的名字，多个`value`可以参照快速开始中的复杂头
    - `ExcelIgnore` 默认所有字段都会写入excel，这个注解会忽略这个字段

    - `DateTimeFormat` 日期转换，将`Date`写到excel会调用这个注解。里面的`value`参照`java.text.SimpleDateFormat`
    - `NumberFormat` 数字转换，用`Number`写excel会调用这个注解。里面的`value`参照`java.text.DecimalFormat`

    - `ExcelIgnoreUnannotated` 默认不加`ExcelProperty` 的注解的都会参与读写，加了不会参与

    ###### 参数

    **通用参数**

    `WriteWorkbook`,`WriteSheet` ,`WriteTable`都会有的参数，如果为空，默认使用上级。

    - `converter` 转换器，默认加载了很多转换器。也可以自定义。
    - `writeHandler` 写的处理器。可以实现`WorkbookWriteHandler`,`SheetWriteHandler`,`RowWriteHandler`,`CellWriteHandler`，在写入excel的不同阶段会调用

    - `relativeHeadRowIndex` 距离多少行后开始。也就是开头空几行
    - `needHead` 是否导出头

    - `head`  与`clazz`二选一。写入文件的头列表，建议使用class。
    - `clazz` 与`head`二选一。写入文件的头对应的class，也可以使用注解。

    - `autoTrim` 字符串、表头等数据自动trim

    **WriteWorkbook（理解成excel对象）参数**

    - `excelType` 当前excel的类型 默认`xlsx`
    - `outputStream` 与`file`二选一。写入文件的流

    - `file` 与`outputStream`二选一。写入的文件
    - `templateInputStream` 模板的文件流

    - `templateFile` 模板文件
    - `autoCloseStream` 自动关闭流。

    - `password` 写的时候是否需要使用密码
    - `useDefaultStyle` 写的时候是否是使用默认头

    **WriteSheet（就是excel的一个Sheet）参数**

    - `sheetNo` 需要写入的编码。默认0
    - `sheetName` 需要些的Sheet名称，默认同`sheetNo`

    **WriteTable（就把excel的一个Sheet,一块区域看一个table）参数**

    - `tableNo` 需要写入的编码。默认0