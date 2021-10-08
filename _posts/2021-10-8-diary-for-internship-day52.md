---
layout: post
title:  "实习日记52(工作任务)"
date:   2021-10-08
categories: jekyll update
---

## Day52		

- 时间好快啊我的IDEA都过期了，续一下。

- 据说这个项目框架是可以实现自动建表的，但是我找到的能自动建表的方法也就是利用mybatis-plus的那个组件或者spring.jpa.hibernate.ddl-auto设置为update(即没有表新建，有表更新操作,控制台显示建表语句)，但貌似框架里没有使用，现在就是要实现这么一个新功能，所以要继续看源码。

  看了下application.yml，他把spring.jpa.hibernate.ddl-auto给注释掉了，可能是这个项目很久没有更新表了吧。

  顺便在这里说一下spring.jpa.hibernate.ddl-auto几个可用的值及其功能：

  - create：每次应用启动的时候会重新根据实体建立表，之前的表和数据都会被删除。
  - create-drop：和上面的功能一样，但是多了一样，就是在应用关闭的时候，也就是sessionFactory一关闭，会把表删除。
  - update：第一次启动根据实体建立表结构，之后启动会根据实体的改变更新表结构，之前的数据都在。
  - validate：会验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值，运行程序会校验实体字段与数据库已有的表的字段类型是否相同，不同会报错

- 接下来就是弄懂框架中的公共字段了（解释一下，这里的实体类都是继承了一个公共父类，所以有一些公共字段要弄懂）

- Springboot自定义注释：

  例：

  ```java
  @SpringBootApplication
  public @Interface MyApplication{
  }
  ```

- 现在有个问题是哪怕使用了spring.jpa.hibernate.ddl-auto也无法自动建表，因为SOA问题entity层所在的包没有resource目录，但是我直接使用框架的话是可以的，所以现在在看为什么。

- 关于Springboot无法读取.properties文件(例如Environment.properties)，如果是maven项目的话可以采用如下方式：

  在pom文件中添加：

  ```
  <build>
        <finalName>webApi</finalName>
        <plugins>
           <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>3.5.1</version>
              <configuration>
                 <source>1.8</source>
                 <target>1.8</target>
                 <encoding>UTF-8</encoding>
              </configuration>
           </plugin>
        </plugins>
        <resources>
           <resource>
              <directory>src/main/resources</directory>
              <includes>
                 <include>**/*.properties</include>
                 <include>**/*.xml</include>
                 <include>**/*.tld</include>
              </includes>
              <filtering>false</filtering>
           </resource>
           <resource>
              <directory>src/main/java</directory>
              <excludes>
                 <exclude>**/*.java</exclude>
              </excludes>
           </resource>
        </resources>
     </build>
  ```

- @Target注解的作用：

  - @Target(ElementType.TYPE)——接口、类、枚举、注解
  - @Target(ElementType.FIELD)——字段、枚举的常量
  - @Target(ElementType.METHOD)——方法
  - @Target(ElementType.PARAMETER)——方法参数
  - @Target(ElementType.CONSTRUCTOR) ——构造函数
  - @Target(ElementType.LOCAL_VARIABLE)——局部变量
  - @Target(ElementType.ANNOTATION_TYPE)——注解
  - @Target(ElementType.PACKAGE)——包

- @Retention注解的作用：

  - RetentionPolicy.SOURCE:这种类型的Annotations只在源代码级别保留,编译时就会被忽略,在class字节码文件中不包含。
  - RetentionPolicy.CLASS:这种类型的Annotations编译时被保留,默认的保留策略,在class文件中存在,但JVM将会忽略,运行时无法获得。
  - RetentionPolicy.RUNTIME:这种类型的Annotations将被JVM保留,所以他们能在运行时被JVM或其他使用反射机制的代码所读取和使用。

- @Document：说明该注解将被包含在`javadoc`中

- @Inherited：说明子类可以继承父类中的该注解