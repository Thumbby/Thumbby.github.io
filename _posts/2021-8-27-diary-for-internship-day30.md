---
layout: post
title:  "实习日记30(EasyExcelDemo)"
date:   2021-08-27
categories: jekyll update
---

## Day30

- @Override注解是作用于源代码的注解，用于表明注解的方法重写了父类型的方法，但是这个注解在1.5和1.6及以后是有区别的。1.5中，只能用于在继承某个类时，重写父类中的方法，而在实现一个接口中的方法时，是不能使用该注解的，从1.6开始，才支持实现父接口的方法使用该注解。但是在@Override源代码文档中，1.6没有对这个变化进行说明，到1.7才进行了说明。

  - 在1.5中源码如下：

  ```java
  package java.lang;
   
  import java.lang.annotation.*;
   
  /**
   * Indicates that a method declaration is intended to override a
   * method declaration in a superclass.  If a method is annotated with
   * this annotation type but does not override a superclass method,
   * compilers are required to generate an error message.
   *
   * @author  Joshua Bloch
   * @since 1.5
   */
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.SOURCE)
  public @interface Override {
  }
  ```

  意思是：表示一个方法声明打算重写超类中的某个方法声明。如果方法利用此注解类型进行注解但没有重写超类方法，则编译器会生成一条错误消息。
  
  - 在1.7中源码如下：
  
    ```java
    package java.lang;
     
    import java.lang.annotation.*;
     
    /**
     * Indicates that a method declaration is intended to override a
     * method declaration in a supertype. If a method is annotated with
     * this annotation type compilers are required to generate an error
     * message unless at least one of the following conditions hold:
     *
     * <ul><li>
     * The method does override or implement a method declared in a
     * supertype.
     * </li><li>
     * The method has a signature that is override-equivalent to that of
     * any public method declared in {@linkplain Object}.
     * </li></ul>
     *
     * @author  Peter von der Ahé
     * @author  Joshua Bloch
     * @jls 9.6.1.4 Override
     * @since 1.5
     */
    @Target(ElementType.METHOD)
    @Retention(RetentionPolicy.SOURCE)
    public @interface Override {
    }
    ```
  
    意思是：表示一个方法声明打算重写父类型中的某个方法声明。如果方法利用此注解类型进行注解时，编译器必须生成一条错误消息，除非满足下列两个条件之一：
  
    - 该方法重写或实现了父类型中声明的某个方法
    - 该方法的签名与Object类中声明的某个公共方法重写等价（override-equivalent）的
  
- 今天把demo整合完了，confluen也写了，快乐周五刷点leetcode

  

