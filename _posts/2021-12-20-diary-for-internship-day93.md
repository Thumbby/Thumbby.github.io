---
layout: post
title:  "实习日记93(Java Class文件格式)"
date:   2021-12-20
categories: jekyll update
typora-root-url: ./
---

## Day93

### Java Class文件格式

#### 4.5字段

字段使用field_info结构来描述。

在同一个class文件中的两个字段不能有相同的名称和描述符。

结构的格式如下：

```
field_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

field_info结构中的项目如下：

- access_flags

　　access_flags的值是标识的掩码，用于表示对该字段的访问权限和属性。下表指定了设置每个标志含义。

| Flag Name       | Value  | Interpretation                                 |
| --------------- | ------ | ---------------------------------------------- |
| `ACC_PUBLIC`    | 0x0001 | 申明为 `public`; 可以从包外访问                |
| `ACC_PRIVATE`   | 0x0002 | `申明为private`; 仅能在定义的class中使用       |
| `ACC_PROTECTED` | 0x0004 | `申明为protected`; 可以在子类中访问            |
| `ACC_STATIC`    | 0x0008 | `申明为static`.                                |
| `ACC_FINAL`     | 0x0010 | `申明为final`; 对象构造好之后无法再分配        |
| `ACC_VOLATILE`  | 0x0040 | `申明为volatile`; 不能被缓存.                  |
| `ACC_TRANSIENT` | 0x0080 | `申明为transient`; 持久化对象管理器不会读和写. |
| `ACC_SYNTHETIC` | 0x1000 | 申明为synthetic; 不是由源代码产生的            |
| `ACC_ENUM`      | 0x4000 | 申明为枚举中的一员.                            |

​	类字段可能被设置上表中的任意标志位。然而，类中的每个字段只能设置ACC_PUBLIC，ACC_PRIVATE和ACC_PROTECTED中的一个，并且ACC_FINAL和ACC_VOLATILE不能同事被设置。

​	接口字段必须设置ACC_PUBLIC，ACC_STATIC和ACC_FINAL标志位；可以设置ACC_SYNTHETIC标志位，除此之外的其它标志位都不能设置。

​	ACC_SYNTHETIC标志表明这个字段是由编译器产生的，不存在源码中。

​	ACC_ENUM标志表面这个字段存储枚举类型中的一个元素。

​	上表中没有提到的标志位都是保留的供未来使用。在生成class文件的时候，它们应该设置为0，并且java虚拟机实现应该忽略这些位。

- name_index

  name_index的值必须是常量池表中的有效索引。该索引处的常量池条目必须是一个CONSTANT_Utf8_info结构，它表示一个表示字段的有效非限定名称.

- descriptor_index

  descriptor_index的值必须是常量池表中的有效索引。该索引处的常量池条目必须是一个CONSTANT_Utf8_info结构，它表示一个有效的字段描述符。

- arrtibutes_count

  attributes count的值指明此字段的附加属性的数量。

- attributes[]

  attributes表的每个值必须是一个attribute_info结构。

  字段可以包含任意数量的与之关联的可选属性。

  表4.7-C列出了本规范定义的属性，它们出现在field_info结构的attributes表中。

  有关定义出现在field_info结构的attributes表中的属性的规则在§4.7中给出。

  有关field_info结构的attributes表中的非预定义属性的规则在§4.7.1中给出。

#### 4.6方法

每个方法（包括每个实例初始化方法（2.9）和类或接口初始化方法（2.9））都由method_info结构描述。

同一个class文件不能有两个相同名称和描述符的方法。

method_info结构的格式如下：

```
method_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

method_info结构的项目如下：

- access_flags

  access_flags的值是标识的掩码，用于表示对该方法的访问权限和属性。下表指定了设置每个标志含义。

  | Flag Name          | Value  | Interpretation                                               |
  | ------------------ | ------ | ------------------------------------------------------------ |
  | `ACC_PUBLIC`       | 0x0001 | Declared `public`; may be accessed from outside its package. |
  | `ACC_PRIVATE`      | 0x0002 | Declared `private`; accessible only within the defining class. |
  | `ACC_PROTECTED`    | 0x0004 | Declared `protected`; may be accessed within subclasses.     |
  | `ACC_STATIC`       | 0x0008 | Declared `static`.                                           |
  | `ACC_FINAL`        | 0x0010 | Declared `final`; must not be overridden.                    |
  | `ACC_SYNCHRONIZED` | 0x0020 | Declared `synchronized`; invocation is wrapped by a monitor use. |
  | `ACC_BRIDGE`       | 0x0040 | A bridge method, generated by the compiler.                  |
  | `ACC_VARARGS`      | 0x0080 | Declared with variable number of arguments.                  |
  | `ACC_NATIVE`       | 0x0100 | Declared `native`; implemented in a language other than Java. |
  | `ACC_ABSTRACT`     | 0x0400 | Declared `abstract`; no implementation is provided.          |
  | `ACC_STRICT`       | 0x0800 | Declared `strictfp`; floating-point mode is FP-strict.       |
  | `ACC_SYNTHETIC`    | 0x1000 | Declared synthetic; not present in the source code.          |

  类的方法可以具有上表中的任何标志。 但是，类的每个方法最多可以设置ACC_PUBLIC，ACC_PRIVATE和ACC_PROTECTED标志中的一个。

  接口方法可以具有上表中除ACC_PROTECTED，ACC_FINAL，ACC_SYNCHRONIZED和ACC_NATIVE之外的任何标志。 在版本号小于52.0的类文件中，接口的每个方法必须设置ACC_PUBLIC和ACC_ABSTRACT标志; 在版本号为52.0或更高版本的类文件中，接口的每个方法必须只设置ACC_PUBLIC和ACC_PRIVATE标志中的一个。

  如果类或接口的方法设置了ACC_ABSTRACT标志，则它不能设置任何ACC_PRIVATE，ACC_STATIC，ACC_FINAL，ACC_SYNCHRONIZED，ACC_NATIVE或ACC_STRICT标志。

  每个实例初始化方法最多可以设置一个ACC_PUBLIC，ACC_PRIVATE和ACC_PROTECTED标志，并且还可以设置其ACC_VARARGS，ACC_STRICT和ACC_SYNTHETIC标志，但不得具有上表中的任何其他标志。

  类和接口的初始化方法由Java虚拟机隐式调用。 除了ACC_STRICT标志的设置外，它们的access_flags项的值被忽略。

  ACC_BRIDGE标志用于指示编译器为Java编程语言生成的桥接方法。

  ACC_VARARGS标志指示此方法在源代码级别采用可变数量的参数。 声明采用可变数量参数的方法必须在ACC_VARARGS标志设置为1

  的情况下进行编译。所有其他方法必须在ACC_VARARGS标志设置为0的情况下编译。

  ACC_SYNTHETIC标志表示此方法是由编译器生成的，并且不会出现在源代码中，除非它是§4.7.8中指定的方法之一。

  上表中未分配的access_flags项的所有位都保留供将来使用。 它们应该在生成的类文件中设置为零，并且应该被Java虚拟机实现忽略。

- name_index

  name_index项的值必须是constant_pool表的有效索引。 该索引处的constant_pool条目必须是CONSTANT_Utf8_info结构（4.4.7），表示特殊方法名称<init>或<clinit>（2.9）之一，或表示方法的有效非限定名称（4.2.2）。

- descriptor_index

  descriptor_index项的值必须是constant_pool表的有效索引。 该索引处的constant_pool条目必须是表示有效方法描述符的CONSTANT_Utf8_info结构（4.3.3）。

- arrtibutes_count

  attributes_count项的值指示此方法的其他属性的数量。

- attributes[]　

  attributes表的每个值必须是attribute_info结构（第4.7节）。

  方法可以具有与之关联的任意数量的可选属性。

  4.7表中列出了此规范定义的属性出现在method_info结构的attributes表中。

  有关定义出现在method_info结构的attributes表中的属性的规则在§4.7中给出。

  关于method_info结构的attributes表中的非预定义属性的规则在第4.7.1节中给出。
