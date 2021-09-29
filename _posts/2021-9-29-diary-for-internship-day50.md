---
  layout: post
title:  "实习日记50(SpringBoot)"
date:   2021-09-29
categories: jekyll update
---

## Day50

- 第50天了，貌似没啥用，如果今天没事情的话就继续看视频课了https://www.bilibili.com/video/BV1PE411i7CV?p=7&spm_id_from=pageDriver，写这句话的时候电脑又在跑自带的防病毒了，哪天给他关了。

- springboot pom文件

  - <parent>:spring-boot-dependencies：核心依赖
  - <dependency>：
    - spring-boot-starter-xxx：启动器，自动导入对应环境所有依赖，如spring-boot-starter-web等，springboot会将所有的功能场景变成启动器，需要什么功能只需要找到对应启动器就行了。

- 自动装配原理：

  @SpringBootApplication:

  - @SpringBootConfiguration->@Configuration->@Component
  - @EnableAutoConfiguration：自动导入包->
    - @AutoConfigurationPackage->@Import(AutoConfigurationPackages.Registrar.class)：自动注册表
    - @Import(AutoConfigurationImportSelecrot.class)：自动导入包的核心->AutoConfigurationImportSelector：选择器
      - getAutoConfigurationEntry()：获得自动配置的实体
      - getCandidateConfigurations()：获取候选的配置->getSpringFactoriesLoaderFactoryClass()返回标注了EnableAutoConfiguration注解的类
      - loadFactoryNames()：获取所有的加载配置
      - loadSpringFactories()：
        - 项目资源：classLoader.getResources(FACTORIES_RESOURCE_LOCATION)：从META-INF/spring.factories获取配置
        - 系统资源：ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION)
          - 从这些资源中便利了所有的nextElement(自动配置)，遍历完成后，封装为Properties使用
  - @ComponentScan：扫描当前主启动类同级的包

  即：

  - SpringBoot启动会加载大量的自动配置类
  - 查看需要的功能有没有再SpringBott默认写好的自动配置类中
  - 查看配置类中配置了哪些组件(主要要用的组件存在其中，就不需要再手动配置)
  - 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。只需要在配置文件中指定这些属性的值即可；
    - xxxxAutoConfiguration：自动配置类，给容器中添加组件
    - xxxxProperties：封装配置文件中相关属性

- SpringApplication：该方法主要分为两部分，一部分是SpringApplication的实例化，二是run方法的执行

  这个类主要有以下四个用处：

  - 推断应用的类型是普通的项目还是Web项目
  - 查找并加载所有可用初始化器，设置到initializers属性中
  - 找出所有的应用程序监听器，设置到listeners属性中
    - 推断并设置main方法的定义类，找到运行的主类

- yml给实体类赋值：在对应pojo类属性上使用@Value("#{}")字段

  @PropertySource(value = "")：加载指定的配置文件

  |                    | @ConfigurationProperties | @Value     |
  | ------------------ | ------------------------ | ---------- |
  | 功能               | 批量注入配置文件中的属性 | 一个个指定 |
  | 松散绑定(松散语法) | 支持                     | 不支持     |
  | SpEL               | 不支持                   | 支持       |
  | JSR303数据校验     | 支持                     | 不支持     |
  | 复杂类型封装       | 支持                     | 不支持     |

  松散绑定：例如yml中的last-name和lastName一样，-后跟着的字母默认大写

  JSR303数据校验：在字段增加一层过滤器验证，可以保证数据的合法性，使用@Validated注释，并对对应属性使用对应注释判断是否合法

  **如果专门编写了一个JavaBean来配置文件进行映射，就直接使用@ConfigurationProperties**

- springboot多环境配置，可以选择配置文件，或者使用spring-profiles-dev在一个yml文件中选择激活

  如果yml和properties同事都配置了端口，并且没有激活其他环境，默认会使用properties配置文件。

  springboot会扫描一下位置的application.properties或者application.yml文件作为springboot的默认配置文件：

  - 优先级1：项目路径下的config文件夹配置文件
  - 优先级2：项目路径下配置文件
  - 优先级3：资源路径下的config文件夹配置文件
  - 优先级4：资源路径下配置文件

  优先级从高到低，高优先级的配置会覆盖低优先级的配置，同时可以通过spring.config.location来改变默认的配置文件位置。

- @Conditional：自动配置类必须在一定的条件下才能生效，用该注解后，只有其指定的条件成立，才给容器中添加组件，配置里面的所有内容才生效。

  | @Conditional扩展注解            | 作用(判断是否满足当前指定条件)                   |
  | ------------------------------- | ------------------------------------------------ |
  | @ConditionalOnJava              | 系统的java版本是否符合要求                       |
  | @ConditionalOnBean              | 容器中存在指定Bean                               |
  | @ConditionalOnMissingBean       | 容器中不存在指定Bean                             |
  | @ConditionalOnExpression        | 满足SpEL表达式指定                               |
  | @ConditionalOnClass             | 系统中有指定的类                                 |
  | @ConditionalOnMissingClass      | 系统中没有指定的类                               |
  | @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
  | @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
  | @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
  | @ConditionalOnWebApplication    | 当前是web环境                                    |
  | @ConditionalOnNotWebApplication | 当前不是web环境                                  |
  | @ConditionalOnJndi              | JNDI存在指定项                                   |

  