---
layout: post
title:  "实习日记51(springboot+新工作读源码)"
date:   2021-09-30
categories: jekyll update
---

## Day51

- 继续看视频课

- 讲道理现在还有人用thmeleaf么，暂且简单过一下吧。

- SpringBoot扩展SpringMVC

  ```java
  @Configuration
  public class MyMvcConfig implements WebMvcConfig{
  	//实现试图解析器
  	@Bean
  	public ViewResolver myViewResolver(){
  		return new myViewResolver();
  	}
  	public static class MyViewResolver implements ViewResoler{
  		@Override
  		public View resolveViewName(String viewName, Locale locale) throws Exception{
  			return null;
  		}
  	}
  }
  ```

- 看到一半来活了，看原型去了

- 研究任务，用框架设计表，然后还要读源码

- IDEA发现使用maven时有些依赖找不到，更换了仓库后因为仓库中原来就有对应依赖所以问题解决，但为什么无法从头下载依赖未知，今天把这个问题解决了吧。
  - 册那不知道为啥可以下载了，本问题暂且完结，下次遇到了相同的问题再说

- Mybatisplus自动建表

  - 引入依赖

    ```
    <!--mybatis-plus自动建表功能-->
        <dependency>
    	<groupId>com.gitee.sunchenbin.mybatis.actable</groupId>
        <artifactId>mybatis-enhance-actable</artifactId>
        <version>${actable.version}</version>
    </dependency>
    ```

  - 配置yml

    ```
    actable:
      table:
        auto: update
      model:
        pack: com.wteam.dragon.security.bean,com.wteam.dragon.pay.bean #扫描用于创建表的对象的包名
      database:
        type: mysql
        
    #mybatisplus的配置	
    mybatis-plus: 	
    	classpath*:xxxxxx/*.xml,classpath*:com/gitee/sunchenbin/mybatis/actable/mapping/*/*.xml
    ```

    1）auto
    当auto=create时，系统启动后，会将所有的表删除掉，然后根据model中配置的结构重新建表，该操作会破坏原有数据。
    当auto=update时，系统会自动判断哪些表是新建的，哪些字段要修改类型等，哪些字段要删除，哪些字段要新增，该操作不会破坏原有数据。
    当auto=none时，系统不做任何处理。

    2）pack
    扫描用于创建表的对象的包名（这里换成你自己的创建表的包名，支持多路径，用逗号隔开即可）

    3）type
    目前只支持mysql

    4）mapper-locations
    配置mapper路径

    5）如果使用的是mybatis
    将mapper-locations放到mybatis配置中

  - 在启动类中加上

    ```java
    @MapperScan({"com.gitee.sunchenbin.mybatis.actable.dao.*"} )
    @ComponentScan("com.gitee.sunchenbin.mybatis.actable.manager.*") 
    ```

  - 使用例

    ```java
    @Table(name = "role")
    public class Role extends BaseModel{
        /**
         * 自增id
         */
        @Column(name = "role_id", type = MySqlTypeConstant.INT, isNull = false,
                isKey = true, isAutoIncrement = true, comment = "自增id")
        private Long roleId;
        /**
         * 角色名字
         */
        @Column(name = "name", type = MySqlTypeConstant.VARCHAR, isNull = false, length = 20, comment = "角色名字")
        private String name;
        /**
         * 角色的中文名字
         */
        @Column(name = "name_zh", type = MySqlTypeConstant.VARCHAR, isNull = true, length = 20, comment = "角色的中文名字")
        private String nameZh;
    }
    ```

    

