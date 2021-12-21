---
layout: post
title:  "实习日记94(Java Class文件格式)"
date:   2021-12-21
categories: jekyll update
typora-root-url: ./
---

## Day94

### 4.7属性

属性用于class文件格式中的ClassFile，field_info，method_info和Code_attribute结构。

所有的属性都是下面的格式：

```
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```

对于所有的属性，attribute_name_index是类文件常量池中有效的16位索引。在这个索引位置的常量池条目必须是CONSTANT_Utf8_info结构，表示这个属性的名称。attribute_length表示接下来内容长度有多少字节。长度不包含开始的6个字节（即不包含attribute_name_index和attribute_lenth）。

23个预定义的属性在此规范中定义了，为了便于导航，使用三个不同顺序的表格来列出来。

表4.7-A按本章属性节号排序。每个属性都带有定义它的类文件格式的第一个版本，以及Java SE平台的相应版本。(对于旧的类文件版本，

使用JDK发行版而不是Java SE平台版本)。

表4.7-B是由在其中定义的每个属性的类文件格式的第一版本排序。

表4.7-C是根据类文件中定义每个属性要出现的位置来排序的。

在本规范中使用它们的上下文中，也就是说，在它们出现的类文件结构的属性表中，这些预定义属性的名称是保留的。

属性表中存在预定义属性的任何条件都在描述属性的部分中显式指定。如果没有指定条件，则属性可以在属性表中出现任意次数。

预定义的属性根据其用途分为三组:

1. 五个属性对于Java虚拟机正确解释类文件至关重要:

   - ConstantValue
   - Code
   - StackMapTable
   - Exceptions
   - BootstrapMethods

   在版本为V的类文件中，如果java虚拟机的实现能够识别V版本的类文件，那么这些属性中的每一个都必须被Java虚拟机的实现正确识别和读取，并且版本V至少是属性最初被定义的版本，并且属性出现在它被定义中出现的位置。

2. 12个属性对于Java SE平台类库对类文件的正确解释至关重要:

   - InnerClasses
   - EnclosingMethod
   - Synthetic
   - Signature
   - RuntimeVisibleAnnotations
   - RuntimeInvisibleAnnotations
   - RuntimeVisibleParameterAnnotations
   - RuntimeInvisibleParameterAnnotations
   - RuntimeVisibleTypeAnnotations
   - RuntimeInvisibleTypeAnnotations
   - AnnotationDefault
   - MethodParameters

   如果ava SE平台类库的实现需要识别版本V的类文件，那么版本V的类文件中的这些属性中的每一个都必须被类库的实现识别和正确读取，并且版本V至少是属性最初被定义的版本，并且属性出现在它被定义中出现的位置。

3. 对于Java虚拟机或Java SE平台的类库来说，六个属性对正确解释类文件并不重要，但对工具很有用:

   - SourceFile
   - SourceDebugExtension
   - LineNumberTable
   - LocalVariableTable
   - LocalVariableTypeTable
   - Deprecated

   对于java虚拟机或者Java SE平台的类库的实现来说，使用这些属性是可选的。实现可能使用这些属性包含的信息，也可以直接忽略这些属性。

**Table 4.7-A. 预定义的类文件属性(根据章节排序)**

| 属性                                   | 章节                                                         | `class` 文件 | Java SE |
| -------------------------------------- | ------------------------------------------------------------ | ------------ | ------- |
| `ConstantValue`                        | [§4.7.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.2) | 45.3         | 1.0.2   |
| `Code`                                 | [§4.7.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.3) | 45.3         | 1.0.2   |
| `StackMapTable`                        | [§4.7.4](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.4) | 50.0         | 6       |
| `Exceptions`                           | [§4.7.5](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.5) | 45.3         | 1.0.2   |
| `InnerClasses`                         | [§4.7.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.6) | 45.3         | 1.1     |
| `EnclosingMethod`                      | [§4.7.7](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.7) | 49.0         | 5.0     |
| `Synthetic`                            | [§4.7.8](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.8) | 45.3         | 1.1     |
| `Signature`                            | [§4.7.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.9) | 49.0         | 5.0     |
| `SourceFile`                           | [§4.7.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.10) | 45.3         | 1.0.2   |
| `SourceDebugExtension`                 | [§4.7.11](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.11) | 49.0         | 5.0     |
| `LineNumberTable`                      | [§4.7.12](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.12) | 45.3         | 1.0.2   |
| `LocalVariableTable`                   | [§4.7.13](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.13) | 45.3         | 1.0.2   |
| `LocalVariableTypeTable`               | [§4.7.14](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.14) | 49.0         | 5.0     |
| `Deprecated`                           | [§4.7.15](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.15) | 45.3         | 1.1     |
| `RuntimeVisibleAnnotations`            | [§4.7.16](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.16) | 49.0         | 5.0     |
| `RuntimeInvisibleAnnotations`          | [§4.7.17](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.17) | 49.0         | 5.0     |
| `RuntimeVisibleParameterAnnotations`   | [§4.7.18](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.18) | 49.0         | 5.0     |
| `RuntimeInvisibleParameterAnnotations` | [§4.7.19](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.19) | 49.0         | 5.0     |
| `RuntimeVisibleTypeAnnotations`        | [§4.7.20](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.20) | 52.0         | 8       |
| `RuntimeInvisibleTypeAnnotations`      | [§4.7.21](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.21) | 52.0         | 8       |
| `AnnotationDefault`                    | [§4.7.22](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.22) | 49.0         | 5.0     |
| `BootstrapMethods`                     | [§4.7.23](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.23) | 51.0         | 7       |
| `MethodParameters`                     | [§4.7.24](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.24) | 52.0         | 8       |

**Table 4.7-B. 预定的类文件属性(根据类文件版本排序)**

| 属性                                   | `class` 文件 | Java SE | 章节                                                         |
| -------------------------------------- | ------------ | ------- | ------------------------------------------------------------ |
| `ConstantValue`                        | 45.3         | 1.0.2   | [§4.7.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.2) |
| `Code`                                 | 45.3         | 1.0.2   | [§4.7.3](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.3) |
| `Exceptions`                           | 45.3         | 1.0.2   | [§4.7.5](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.5) |
| `SourceFile`                           | 45.3         | 1.0.2   | [§4.7.10](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.10) |
| `LineNumberTable`                      | 45.3         | 1.0.2   | [§4.7.12](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.12) |
| `LocalVariableTable`                   | 45.3         | 1.0.2   | [§4.7.13](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.13) |
| `InnerClasses`                         | 45.3         | 1.1     | [§4.7.6](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.6) |
| `Synthetic`                            | 45.3         | 1.1     | [§4.7.8](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.8) |
| `Deprecated`                           | 45.3         | 1.1     | [§4.7.15](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.15) |
| `EnclosingMethod`                      | 49.0         | 5.0     | [§4.7.7](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.7) |
| `Signature`                            | 49.0         | 5.0     | [§4.7.9](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.9) |
| `SourceDebugExtension`                 | 49.0         | 5.0     | [§4.7.11](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.11) |
| `LocalVariableTypeTable`               | 49.0         | 5.0     | [§4.7.14](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.14) |
| `RuntimeVisibleAnnotations`            | 49.0         | 5.0     | [§4.7.16](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.16) |
| `RuntimeInvisibleAnnotations`          | 49.0         | 5.0     | [§4.7.17](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.17) |
| `RuntimeVisibleParameterAnnotations`   | 49.0         | 5.0     | [§4.7.18](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.18) |
| `RuntimeInvisibleParameterAnnotations` | 49.0         | 5.0     | [§4.7.19](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.19) |
| `AnnotationDefault`                    | 49.0         | 5.0     | [§4.7.22](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.22) |
| `StackMapTable`                        | 50.0         | 6       | [§4.7.4](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.4) |
| `BootstrapMethods`                     | 51.0         | 7       | [§4.7.23](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.23) |
| `RuntimeVisibleTypeAnnotations`        | 52.0         | 8       | [§4.7.20](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.20) |
| `RuntimeInvisibleTypeAnnotations`      | 52.0         | 8       | [§4.7.21](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.21) |
| `MethodParameters`                     | 52.0         | 8       | [§4.7.24](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7.24) |

**Table 4.7-C. 预定义的类文件属性 (根据位置排序)**

| 属性                                                         | 位置                                             | `class` 文件 |
| ------------------------------------------------------------ | ------------------------------------------------ | ------------ |
| `SourceFile`                                                 | `ClassFile`                                      | 45.3         |
| `InnerClasses`                                               | `ClassFile`                                      | 45.3         |
| `EnclosingMethod`                                            | `ClassFile`                                      | 49.0         |
| `SourceDebugExtension`                                       | `ClassFile`                                      | 49.0         |
| `BootstrapMethods`                                           | `ClassFile`                                      | 51.0         |
| `ConstantValue`                                              | `field_info`                                     | 45.3         |
| `Code`                                                       | `method_info`                                    | 45.3         |
| `Exceptions`                                                 | `method_info`                                    | 45.3         |
| `RuntimeVisibleParameterAnnotations`, `RuntimeInvisibleParameterAnnotations` | `method_info`                                    | 49.0         |
| `AnnotationDefault`                                          | `method_info`                                    | 49.0         |
| `MethodParameters`                                           | `method_info`                                    | 52.0         |
| `Synthetic`                                                  | `ClassFile`, `field_info`, `method_info`         | 45.3         |
| `Deprecated`                                                 | `ClassFile`, `field_info`, `method_info`         | 45.3         |
| `Signature`                                                  | `ClassFile`, `field_info`, `method_info`         | 49.0         |
| `RuntimeVisibleAnnotations`, `RuntimeInvisibleAnnotations`   | `ClassFile`, `field_info`, `method_info`         | 49.0         |
| `LineNumberTable`                                            | `Code`                                           | 45.3         |
| `LocalVariableTable`                                         | `Code`                                           | 45.3         |
| `LocalVariableTypeTable`                                     | `Code`                                           | 49.0         |
| `StackMapTable`                                              | `Code`                                           | 50.0         |
| `RuntimeVisibleTypeAnnotations`, `RuntimeInvisibleTypeAnnotations` | `ClassFile`, `field_info`, `method_info`, `Code` | 52.0         |

#### 4.7.1定义和命名新属性

允许编译器定义和发布的class文件在class文件结构体、field_info结构体、method_info结构体和Code结构体中的attributes表中包含新的属性。允许java虚拟机识别和使用attributes表中的新属性。但是，任何没有在class文件规范中定义的属性都不能影响class文件的语义。java虚拟机的实现需要忽略它们不能识别的属性。

例如，允许定义一个新属性来支持特定供应商的调试。因为java虚拟要需要忽略它们不能识别的属性，为特殊java虚拟机实现所使用的class文件也可以用于其它的java虚拟机实现，尽管这些实现不能使用class文件中包含的额外调试信息。

禁止java虚拟机实现仅因为存在一些新属性就抛出异常或者拒绝使用class文件。当然，运行class文件的工具可能不能正确工作，如果定的class文件中没有包含它们需要的属性。

两个本来是不同的属性，但是碰巧使用了相同的属性名并且长度相同，虚拟机实现在识别这两个属性时会发生冲突。除本规范中定义的属性外，其他属性的名称必须根据《Java语言规范，Java SE 8版》(JLS 6.1)中描述的包命名约定进行选择。

这个规范的未来版本可能定义额外的属性。

#### 4.7.2常量值属性

ConstantValue属性的长度时固定的，它在field_info结构的attributes表中。ConstantValue属性表示常量表达式的值，用于以下场景：

- 如果在field_info结构的access_flags项中设置了ACC_STATIC标志，那么field_info结构所表示的字段将被分配其ConstantValue属性所表示的值，作为声明该字段的类或接口初始化的一部分(5.5)。这发生在调用该类或接口的类或接口初始化方法之前(2.9)。
  - 否则，java虚拟机必须忽略这个属性

　field_info结构中的attributes表中最多只能有一个ConstantValue属性。

　ConstantValue属性的格式如下：

```
ConstantValue_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 constantvalue_index;
}
```

ConstantValue_attribute结构的各条目如下：

- attribute_name_index

　　attribute_name_index的值必须是常量池表中的有效索引。该索引处的常量池条目必须是一个表示字符串“ConstantValue”的常量Utf8信息结构(4.4.7)。

- attribute_length	

　　它的值必须是2.

- constantvalue_index

  　　constantvalue_index的值必须是常量池表中的有效索引。该索引处的常量池条目给出由该属性表示的常量值。常量池条目必须是与字段相适应的类型，如表4.7.2-A所指定。

