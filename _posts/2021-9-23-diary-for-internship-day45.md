---
layout: post
title:  "实习日记45(写kafka+influxdb+mysql的demo)"
date:   2021-09-23
categories: jekyll update
---

## Day45

- 在写demo，等写好之后把demo传到github上，感觉今天没什么好记的

- @SpringbootApplication和@ComponentScan

  @SpringBootApplication=@Configuration+@EnableAutoConfiguration+@ComponentScan，其中扫描包的范围为启动类所在包和子包，不包括第三方的jar包。如果我们需要扫描通过maven依赖添加的jar，我们就要单独使用@ComponentScan注解扫描第三方包。
  但是，如果@SpringBootApplication和@ComponentScan注解共存，那么@SpringBootApplication注解的扫描的作用将会失效，也就是说不能够扫描启动类所在包以及子包了。因此，我们必须在@ComponentScan注解配置本工程需要扫描的包范围。

  存在如下三种情况：

  - 只存在@SpringBootApplication注解

    **源码**：

    ```java
    //AnnotatedElementUtils.java
    //第967行
    private static <T> T searchWithGetSemanticsInAnnotations(@Nullable AnnotatedElement element,
                                                             List<Annotation> annotations, Set<Class<? extends Annotation>> annotationTypes,
                                                             @Nullable String annotationName, @Nullable Class<? extends Annotation> containerType,
                                                             Processor<T> processor, Set<AnnotatedElement> visited, int metaDepth) {
        //第一个for循环作用是检测是否有@ComponentScan注解
        //显然第一种情况下是没有这个注解的，所以直接省略掉第一个循环的代码
        
        //省略部分代码
        
        //第二个循环，递归寻找注解中是否存在@ComponentScan和@ComponentScans注解
        //这里就会找到@SpringBootApplication中的@ComponentScan然后返回
        //如果还想继续深入下到底是怎么找的，各位可以自己看下代码
        for (Annotation annotation : annotations) {
            Class<? extends Annotation> currentAnnotationType = annotation.annotationType();
            if (!AnnotationUtils.hasPlainJavaAnnotationsOnly(currentAnnotationType)) {
                T result = searchWithGetSemantics(currentAnnotationType, annotationTypes,
                                                  annotationName, containerType, processor, visited, metaDepth + 1);
                if (result != null) {
                    processor.postProcess(element, annotation, result);
                    if (processor.aggregates() && metaDepth == 0) {
                        processor.getAggregatedResults().add(result);
                    }
                    else {
                        return result;
                    }
                }
            }
        }
    }
    ```

    日志正常打印，@SpringBootApplication默认会扫描同包及子包

  - 存在@SpringBootApplication和一个@ComponentScan注解

    **源码**:

    ```java
    //AnnotatedElementUtils.java
    //第967行
    private static <T> T searchWithGetSemanticsInAnnotations(@Nullable AnnotatedElement element,
                                                             List<Annotation> annotations, Set<Class<? extends Annotation>> annotationTypes,
                                                             @Nullable String annotationName, @Nullable Class<? extends Annotation> containerType,
                                                             Processor<T> processor, Set<AnnotatedElement> visited, int metaDepth) {
        for (Annotation annotation : annotations) {
            Class<? extends Annotation> currentAnnotationType = annotation.annotationType();
            if (!AnnotationUtils.isInJavaLangAnnotationPackage(currentAnnotationType)) {
                //判断是否是@ComponentScan注解
                if (annotationTypes.contains(currentAnnotationType) ||
                    currentAnnotationType.getName().equals(annotationName) ||
                    processor.alwaysProcesses()) {
                    T result = processor.process(element, annotation, metaDepth);
                    if (result != null) {
                        //这个判断的作用是啥咱也不知道
                        //有大佬知道的话请赐教~
                        //但是这里的判断是false，所以会直接返回结果
                        if (processor.aggregates() && metaDepth == 0) {
                            processor.getAggregatedResults().add(result);
                        }
                        else {
                            return result;
                        }
                    }
                }
                
                //省略部分代码
                
            }
        }
        
        //省略第二个循环
    }
    ```

    可以看到当有一个@ComponentScan注解的时候，它会直接返回这个注解，不会再去解析@SpringBootApplication中的配置

  - 存在@SpringBootApplication和多个@ComponentScan注解

    这里就要介绍一个新的注解——@Repeatable，我们来看下@ComponentScan的代码

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.TYPE)
    @Documented
    @Repeatable(ComponentScans.class)
    public @interface ComponentScan {
    }
    ```

    可以看到上边有个@Repeatable注解，这是JDK8新增的注解，作用就是当有多个@ComponentScan注解时，将它们转成数组作为@ComponentScans的value。

    所以第三种情况就是@SpringBootApplication和一个@ComponentScans注解，注意这里是带s的，所以@SpringBootApplication中的包扫描依然会生效。

    所以解决办法是：

    使用@ComponentScans注解，而不是直接使用@ComponentScan注解

    这是最完美方案，既不会影响SpringBoot本身的配置，你也可以随意自定义自己的配置
