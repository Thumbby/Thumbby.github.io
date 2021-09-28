---
layout: post
title:  "实习日记49(改下demo+springboot学习)"
date:   2021-09-28
categories: jekyll update
---

## Day49

- 也不知道干啥，可能把demo再改改，学学spring，如果有活的话就干活吧，最近没啥动力，就这样。

- Mybatis-plus实现自增id,在pojo类对应属性上添加如下注释：

  ```java
  @TableId(type = IdType.AUTO)
  ```

  同时在数据库中设定对应主键为自动递增，即可实现自增id

- Mybatis-plus实现自动填充：

  - 在pojo类对应属性上添加如下注释：

    ```java
    @TableField(fill = FieldFill.INSERT)
    ```

  - 重写MetaObjectHandler，代码如下：

    ```java
    @Component
    public class MyMetaObjectHandler implements MetaObjectHandler {
    
        @Override
        public void insertFill(MetaObject metaObject) {
    
            this.setFieldValByName("record_create_time", new Date(), metaObject);
            this.setFieldValByName("record_update_time", new Date(), metaObject);
    
        }
    
        @Override
        public void updateFill(MetaObject metaObject) {
    
            this.setFieldValByName("record_update_time", new Date(), metaObject);
    
        }
    } 
    ```

- 从头开始学springboot，看视屏课https://www.bilibili.com/video/BV1PE411i7CV?from=search&seid=14980182282387504719&spm_id_from=333.337.0.0

- springboot笔记

  - spring	关键策略：

    - 基于POJO的轻量级和最小侵入性编程；
      - 通过IOC，依赖注入(DI)和面向接口实现松耦合；	
    - 基于切面(AOP)和惯例进行声明式编程
    - 通过切面和模板减少样式代码

  - 注解

    - @SpringBootConfiguration：springboot的配置
      - @Configuration：spring配置类
      - @Component：说明是一个spring组件
    - @EnableAutoConfiguration：自动配置
      - @AutoConfigurationPackage：自动配置包
        - @Import(AutoConfigurationPackages.Registrar.class)：导入选择器‘包注册'
      - @Import(AutoConfigurationImportSelector.class)：自动配置导入选择
    - @SpringBootApplication：标记这个类为启动类

  - springboot所有自动配置都是在启动的时候扫描并加载：spring.factories所有的自动配置类都在里面，但是不一定生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器，然后配置装配就会生效，步骤如下：

    - springboot在启动的时候，从类路径下/META-INF/spring.factories获取指定的值
    - 将这些自动配置类导入容器，自动配置就会生效
    - 解决方案和自动配置的东西都在spring-boot-autoconfigure-2.2.0.RELEASE.jar包下
    - springbott会把所有需要导入的组件，以类名的的方式返回，这些组件就会被添加到容器中
    - 容器中也会存在非常多的xxxAutoConfiguration的文件(@Bean)，就是这些类给容器中导入了这个场景需要的所有组件，并自动装配。

    

