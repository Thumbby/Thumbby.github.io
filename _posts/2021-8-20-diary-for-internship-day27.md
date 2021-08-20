---
layout: post
title:  "实习日记27(EasyExcelDemo)"
date:   2021-08-20
categories: jekyll update
---

## Day27

- 观看视屏课，学习EasyExcel并尝试手动实现图片导出demo

- IDEA使用技巧(是的这些我才知道所以别骂我了)

  - 创建类自动生成注释

    1.File->setting->File and Code Templates -> Includes

    2.![img](https://img-blog.csdnimg.cn/20200424180857303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjU3OTU5,size_16,color_FFFFFF,t_70)

    3.个人配置：

    ```
    /**
    * @author: Thumbby
    * @description: ${description}
    * @date: ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}
    **/
    ```

- IDEA使用tomcat时控制台出现中文乱码

  尝试如下修改：

  - 配置启动参数

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072110050513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MzM1ODM3,size_16,color_FFFFFF,t_70)

  - 修改IDEA配置，idea64.exe.vmoptions，追加如下：

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721104546230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MzM1ODM3,size_16,color_FFFFFF,t_70)

  - 修改tomcat配置

    Tomcat安装目录`...\apache-tomcat-8.5.69\conf\logging.properties`，将所有的UTF-8修改为GBK，保存重启

  - 配置IDEA FileEncoding

    从File->Setting ,设置File Encodings ,检查Default Encodings 是否是UTF-8

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721105430486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MzM1ODM3,size_16,color_FFFFFF,t_70)

  - 配置JAVA_TOOL-OPTION

    Name：`JAVA_TOOL_OPTION`
    Value：`-Dfile.encoding=UTF-8`

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721110653439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MzM1ODM3,size_16,color_FFFFFF,t_70)

  - 个性化设置修改VmOption

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200721111626689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE4MzM1ODM3,size_16,color_FFFFFF,t_70)

  全部修改之后成功

- 自己尝试写个demo，现在在tomcat部署后连接处有问题，下周解决