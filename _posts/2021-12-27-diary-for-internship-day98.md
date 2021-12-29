---
layout: post
title:  "实习日记98(Maven profile依赖问题)"
date:   2021-12-27
categories: jekyll update
typora-root-url: ./

---

## Day98

研究问题：maven在切换profile时并没有重新导入依赖

1. development是自带的profile

2. 按照如下写法在切换分支后是可以刷新的

   ```
   <profiles>
           <profile>
               <id>dev</id>
               <dependencies>
                   <dependency>
                       <groupId>junit</groupId>
                       <artifactId>junit</artifactId>
                       <version>${junit.version}</version>
                       <scope>test</scope>
                   </dependency>
               </dependencies>
           </profile>
           <profile>
               <id>test</id>
               <dependencies>
                   <dependency>
                       <groupId>org.springframework.boot</groupId>
                       <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
                   </dependency>
               </dependencies>
           </profile>
       </profiles>
   ```

3. <properties>标签是用于pom文件中赋值，并不能改依赖