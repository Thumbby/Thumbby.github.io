---
layout: post
title:  "实习日记13"
date:   2021-08-02
categories: jekyll update
---

## Day13

- 继续城运大屏定时任务的开发

- 赶回东方体育中心

- Swagger学习：

   1. 使用SpringBoot集成Swagger2构建Restful API

      1. 引用相关jar包，使用maven在pom.xml中引入相关依赖：

         ```html
         <dependency>
             <groupId>io.springfox</groupId>
             <artifactId>springfox-swagger2</artifactId>
             <version>2.4.0</version>
         </dependency>
         <dependency>
             <groupId>io.springfox</groupId>
             <artifactId>springfox-swagger-ui</artifactId>
             <version>2.4.0</version>
         </dependency>
         ```

      2. 创建swagger的配置类

         ```java
         package com.xingguo.springboot;
         
         import org.springframework.context.annotation.Bean;
         import org.springframework.context.annotation.Configuration;
         
         import springfox.documentation.builders.ApiInfoBuilder;
         import springfox.documentation.builders.PathSelectors;
         import springfox.documentation.builders.RequestHandlerSelectors;
         import springfox.documentation.service.ApiInfo;
         import springfox.documentation.service.Contact;
         import springfox.documentation.spi.DocumentationType;
         import springfox.documentation.spring.web.plugins.Docket;
         import springfox.documentation.swagger2.annotations.EnableSwagger2;
         
         @Configuration
         @EnableSwagger2
         public class Swagger2Configuration {
         
             @Bean
             public Docket buildDocket(){
                 return new Docket(DocumentationType.SWAGGER_2)
                         .apiInfo(buildApiInf())
                         .select()
                         .apis(RequestHandlerSelectors.basePackage("com.xingguo.springboot.controller"))
                         .paths(PathSelectors.any())
                         .build();
             }
         
             private ApiInfo buildApiInf(){
                 return new ApiInfoBuilder()
                             .title("标题")
                             .description("springboot swagger2")
                             .termsOfServiceUrl("网址链接")
                             .contact(new Contact("diaoxingguo", "http://blog.csdn.net/u014231523", "diaoxingguo@163.com"))
                             .build();
         
             }
         
         }
         ```

         在原来2.2.0的版本中使用new ApiInfo()的方法已经过时，使用new ApiInfoBuilder()进行构造，需要什么参数就添加什么参数。当然也可以什么都添加。如：

         ```java
         private ApiInfo buildApiInfo(){
          return new ApiInfoBuilder().build();
         }
         ```

   2. 注解说明

      1. 用于controller上：

         | 注解 | 说明           |
         | ---- | -------------- |
         | @Api | 对请求类的说明 |

      2. 用于方法上(说明参数的含义)：

         | 注解                                  | 说明                                                        |
         | ------------------------------------- | ----------------------------------------------------------- |
         | @ApiOperation                         | 方法的说明                                                  |
         | @ApiImplicitParams、@ApiImplicitParam | 方法的参数的说明；@ApiImplicitParams 用于指定单个参数的说明 |

      3. 用于方法上(返回参数或对象的说明)
      
         | 注释                                        | 说明                                 |
         | ------------------------------------------- | ------------------------------------ |
         | @ApiResponses、@ApiResponse方法返回值的说明 | @ApiResponses 用于指定单个参数的说明 |
      
      4. 对象类
      
         | 注解              | 说明                                         |
         | ----------------- | -------------------------------------------- |
         | @ApiModel         | 用在JavaBean类上，说明JavaBean的用途         |
         | @ApiModelProperty | 用在JavaBean类的属性上面，说明此属性的的含议 |
      
   3. 各注解的参数及说明：

      1. @Api：请求类的说明

         ```java
         @Api：放在 请求的类上，与 @Controller 并列，说明类的作用，如用户模块，订单类等。
         	tags="说明该类的作用"
         	value="该参数没什么意义，所以不需要配置"
         ```

         属性配置：

         | 属性名称       | 备注                                    |
         | -------------- | --------------------------------------- |
         | value          | url的路径值                             |
         | tags           | 如果设置这个值，value的值会被覆盖       |
         | description    | 对api资源的描述                         |
         | basePath       | 基本路径                                |
         | position       | 如果配置多个Api可以改变展示的顺序       |
         | produces       | 如, “application/json, application/xml” |
         | consumes       | 如, “application/json, application/xml” |
         | protocols      | 协议类型，如: http, https, ws, wss.     |
         | authorizations | 高级特性认证时配置                      |
         | hidden         | 配置为true则在文件中隐藏                |

      2. @ApiOperation：方法的说明

         ```java
         @ApiOperation："用在请求的方法上，说明方法的作用"
         	value="说明方法的作用"
         	notes="方法的备注说明"
         ```

      3. @ApiImplicitParams，@ApilmplicitParam：方法参数的说明

         ```java
         @ApiImplicitParams：用在请求的方法上，包含一组参数说明
         	@ApiImplicitParam：对单个参数的说明	    
         	    name：参数名
         	    value：参数的说明、描述
         	    required：参数是否必须必填
         	    paramType：参数放在哪个地方
         	        · query --> 请求参数的获取：@RequestParam
         	        · header --> 请求参数的获取：@RequestHeader	      
         	        · path（用于restful接口）--> 请求参数的获取：@PathVariable
         	        · body（请求体）-->  @RequestBody User user
         	        · form（普通表单提交）	   
         	    dataType：参数类型，默认String，其它值dataType="Integer"	   
         	    defaultValue：参数的默认值
         ```

      4. @ApiResponses、@ApiResponse：方法返回值的状态码说明：

         ```java
         @ApiResponses：方法返回对象的说明
         	@ApiResponse：每个参数的说明
         	    code：数字，例如400
         	    message：信息，例如"请求参数没填好"
         	    response：抛出异常的类
         ```

      5. @ApiModel：用于JavaBean上，表示对JavaBean的功能描述

         主要用处有两个：

         1. 当请求数据描述，即@RequestBody时，用于封装请求(包括数据的各种校验)数据；

            例：

            ```
            @ApiModel(description = "用户登录")
            public class UserLoginVO implements Serializable {
            
            	private static final long serialVersionUID = 1L;
            
            	@ApiModelProperty(value = "用户名",required=true)	
            	private String username;
            
            	@ApiModelProperty(value = "密码",required=true)	
            	private String password;
            	
            	// getter/setter省略
            }
            ```

            ```java
            @Api(tags="用户模块")
            @Controller
            public class UserController {
            
            	@ApiOperation(value = "用户登录", notes = "")	
            	@PostMapping(value = "/login")
            	public R login(@RequestBody UserLoginVO userLoginVO) {
            		User user=userSerivce.login(userLoginVO);
            		return R.okData(user);
            	}
            }
            ```

            ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191227165035627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpYW9qaW4yMWNlbg==,size_16,color_FFFFFF,t_70)

         2. 当响应值是对象时，即 @ResponseBody时，用于返回值对象的描述。

            例：

            ```java
            @ApiModel(description= "返回响应数据")
            public class RestMessage implements Serializable{
            
            	@ApiModelProperty(value = "是否成功",required=true)
            	private boolean success=true;	
            	
            	@ApiModelProperty(value = "错误码")
            	private Integer errCode;
            	
            	@ApiModelProperty(value = "提示信息")
            	private String message;
            	
                @ApiModelProperty(value = "数据")
            	private Object data;
            		
            	/* getter/setter 略*/
            }
            ```

   4. Markdown语法学习

      1. 标题

         在想要设置为标题的文字前面加#来表示
         一个#是一级标题，二个#是二级标题，以此类推。支持六级标题。

         注：标准语法一般在#后跟个空格再写文字，貌似简书不加空格也行。

         示例：

         ```
         # 这是一级标题
         ## 这是二级标题
         ### 这是三级标题
         #### 这是四级标题
         ##### 这是五级标题
         ###### 这是六级标题
         ```

      2. 字体

         - 加粗

         要加粗的文字左右分别用两个*号包起来

         - 斜体

         要倾斜的文字左右分别用一个*号包起来

         - 斜体加粗

         要倾斜和加粗的文字左右分别用三个*号包起来

         - 删除线

         要加删除线的文字左右分别用两个~~号包起来

         示例：

         ```
         **这是加粗的文字**
         *这是倾斜的文字*`
         ***这是斜体加粗的文字***
         ~~这是加删除线的文字~~
         ```

      3. 引用

         在引用的文字前加>即可。引用也可以嵌套，如加两个>>三个>>>n个...

         示例：

         ```
         >这是引用的内容
         >>这是引用的内容
         >>>>>>>>>>这是引用的内容
         ```

      4. 分割线

         三个或者三个以上的 - 或者 * 都可以。

         示例：

         ```
         ---
         ----
         ***
         *****
         ```

      5. 图片

         语法：

         ```
         ![图片alt](图片地址 ''图片title'')
         
         图片alt就是显示在图片下面的文字，相当于对图片内容的解释。
         图片title是图片的标题，当鼠标移到图片上时显示的内容。title可加可不加
         ```

      6. 超链接

         语法：

         ```
         [超链接名](超链接地址 "超链接title")
         title可加可不加
         ```

      7. 列表

         1. 无序列表

            无序列表用 - + * 任何一种都可以

            语法：

            ```
            - 列表内容
            + 列表内容
            * 列表内容
            
            注意：- + * 跟内容之间都要有一个空格
            ```

         2. 有序列表

            语法：
            数字加点

            语法：

            ```
            1. 列表内容
            2. 列表内容
            3. 列表内容
            
            注意：序号跟内容之间要有空格
            ```

         3. 列表嵌套

            上一级和下一级之间敲三个空格即可

      8. 表格

         语法：

         ```
         表头|表头|表头
         ---|:--:|---:
         内容|内容|内容
         内容|内容|内容
         
         第二行分割表头和内容。
         - 有一个就行，为了对齐，多加了几个
         文字默认居左
         -两边加：表示文字居中
         -右边加：表示文字居右
         注：原生的语法两边都要用 | 包起来。此处省略
         ```

      9. 代码

         语法：
         单行代码：代码之间分别用一个反引号包起来

         ```
             `代码内容`
         ```

         代码块：代码之间分别用三个反引号包起来，且两边的反引号单独占一行

         ```
         (```)
           代码...
           代码...
           代码...
         (```)
         ```

      10. 流程图

          语法：

          ```
          ```flow
          st=>start: 开始
          op=>operation: My Operation
          cond=>condition: Yes or No?
          e=>end
          st->op->cond
          cond(yes)->e
          cond(no)->op
          &```
          ```

          结果：![img](https://upload-images.jianshu.io/upload_images/6860761-9d9524ba31047696.png?imageMogr2/auto-orient/strip|imageView2/2/w/751/format/webp)

