---
layout: post
title:  "实习日记84(Java Class文件格式)"
date:   2021-11-29
categories: jekyll update
typora-root-url: ./
---

## Day84

### 4.Class文件格式

本章介绍了java虚拟机的class文件格式。每一个class文件包含一个单独的类或者接口的定义。虽然类和接口不一定都定义在文件中（比如类和接口亦可以通过类加载器直接生成），我们将通俗地将类或接口的任何有效表示称为class文件格式。class文件是由8位的字节流组成。所有16位，32位和64位的数字都是分别通过读取两个、四个和八个连续的8位字节来构成。多字节的数据使用大端模式存储的，也就是高字节在前。在java se平台，这个格式由接口java.io.DataInput和java.io.DataOutput以及例如java.io.DataInputStream和java.io.DataOutputStream等类来支持。

本章定义了它自己的数据类型集来表示class文件数据：类型u1,u2和u4分别表示1字节，2字节和4字节的无符号数。在Java SE平台这些类型通过接口java.io.DataInput的readUnsignedByte，readUnsignerShort和readInt方法来读取。

本章采用类似于c语言接口体的伪结构来表示class文件格式。为了避免类的字段和类的实例等概念混淆，把描述类文件格式结构的内容成为项（item）。连续的项在class文件中是按顺序存放的，之间没有填充和对齐。

表是由0个或者多个可变大小的项组成，用于表示class文件一系列复合结构。尽管我们使用类似C的数组语法来引用表项，但表是不同大小结构的流这一事实意味着无法将表索引直接转换为表中的字节偏移量。

当我们将数据结构表示为一个数组时，数组是由0个或者多个 相邻固定大小的项组成，可以通过数组索引来访问。

本章中对ASCII字符的引用应解释为与ASCII字符对应的Unicode代码点。

#### 4.1ClassFile结构

一个class文件是由单独的ClassFile结构组成的：

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

ClassFile结构中的项描述如下：

- magic

  magic项定义了一个魔数来标识class文件格式，它的值是0xCAFEBABE。

- minor_version,major_version

  minor_version和major_version的值分别表示class文件次版本号和主版本号。次版本号和主版本号一起决定了class文件的版本号。如果一个class文件的主版本号是M，次版本号是m，我们成这个class文件的版本号是M.m。因此类文件格式版本号可以按字典序排序，例如，1.5<2.0<2.1。

  当且仅当v位于某个连续范围Mi.0≤v≤Mj.m时（Mi.0到Mj.m为虚拟机支持的版本号），Java虚拟机实现可以支持版本v的类文件格式。 Java虚拟机所遵循的Java SE平台的发行版级别负责确定范围。

  JDK 1.0.2版中的Oracle Java虚拟机实现支持包含45.0到45.3类的类文件格式。 JDK1.1.*版本支持类文件格式版本范围为45.0到45.65535（含）。 对于k≥2，JDK版本1.k支持45.0到44 + k.0范围内的类文件格式版本。

- constant_pool_count

  constant_pool_count的值为constant_pool表中条目的数量+1。一个constant_pool索引大于0小于constant_pool_count才是有效的，除了long和double类型的常量，具体见4.4.5。

- constant_pool[]

  constant_pool是一个表结构，表示各种字符串常量，类和接口名称，字段名称以及在ClassFile结构及其子结构中引用的其他常量。

　　constant_pool表的索引从1到constant_pool_count_1。

- access_flags

　　access_flags的值是掩码标识，表示这个类或者接口访问权限和属性。每个标识被设置时的含义如下表

| 标识名称       | 值     | 含义                                              |
| -------------- | ------ | ------------------------------------------------- |
| ACC_PUBLIC     | 0x0001 | 声明为public，可以被包以外的类访问                |
| ACC_FINAL      | 0x0010 | 声明为final，不允许有子类                         |
| ACC_SUPER      | 0x0020 | 当带哦用invokspecialial指令时，要特殊处理超类方法 |
| ACC_INTERFACE  | 0x0200 | 是一个接口，不是一个类                            |
| ACC_ABSTRACT   | 0x0400 | 声明为abstract；不能被实例化                      |
| ACC_ANNOTATION | 0x2000 | 是一个注解类型                                    |
| ACC_ENUM       | 0x4000 | 是一个枚举类型                                    |

​		接口是通过设置ACC_INTERFACE标识来区分，如果这个标识没有被设置，那么这个class文件就是定义了一个类，而不是接口

​		如果ACC_INTERFACE标识被设置了，那么ACC_ABSTRACT标识也必须被设置，ACC_FINAL,ACC_SUPER和ACC_ENUM表示不允许被设置。

　　如果ACC_INTERFACE标识未被设置，那么其它的标识除了ACC_ANNOTATION都可以被设置。然而这样的class文件不能同时设置ACC_FINAL和ACC_ABSTRACT。

　　ACC_SUPER标志指示如果它出现在此类或接口中，则由invokespecial指令（§invokespecial）表示两个备用语义中的哪一个。 Java虚拟机指令集的编译器应设置ACC_SUPER标志。 在Java SE 8及更高版本中，Java虚拟机认为要在每个类文件中设置ACC_SUPER标志，而不管类文件中的标志的实际值和类文件的版本。

　　ACC_SUPER标志的存在是为了向后兼容由较老的Java编程语言编译器编译的代码。在JDK 1.0.2之前的版本中，编译器生成了访问标志，其中现在表示ACC_SUPER的标志没有指定的含义，如果设置了该标志，Oracle的Java虚拟机实现将忽略该标志。

　　ACC_SYNTHETIC标识表示这个类或者接口是由编译器产生的并且没有出现在原码中。

　　注解类型必须设置ACC_ANNOTATION标识，如果ACC_ANNOTATION标识被设置了，那么ACC_INTERFACE标识也必须被设置。

　　ACC_ENUM标识表示这个类或者它的父类被声明为枚举类型。

　　access_flags中所有未在表中分配的位都被保留给未来使用。当生产class文件的时候，它们应该被设置为0，并且java虚拟机实现需要忽略它们。

- this_class

  this_class的值必须是constant_pool表的有效索引。constant_pool在这个索引位置的条目必须是一个CONSTAN_Class_info结构，表示这个class文件定义的类或者接口。

- super_class

  对于类来说，super_class的值必须是0或者constant_pool的有效索引。如果super_class的值不是0，constant_pool在这个索引位置的条目必须是一个CONSTANT_Class_info结构，表示这个class文件定义的类的直接父类。它的直接父类以及它的所有父类的ClassFile结构中的access_flags都不能设置ACC_FINAL标识。

  如果super_class的值是0，那么这个类必须是Object类，这是唯一没有直接支持父类的类或者接口。

  对于接口，super_class的值必须是constan_pool的有效索引，这个索引对应的条目必须是一个CONSTAN_Class_info结构，代表Object类。

- interfaces_count

  interfaces_count的值表示这个类或者接口的直接父接口的数量

- interfaces[]

  interfaces数组的每一个值必须是constant_pool的一个有效索引。constant_pool中interfaces[i]索引值对应的条目必须是一个CONSTANT_Class_info结构，表示这个类或者接口的直接父接口，并且按照原码中从左到有的顺序给出。

- fields_count

  fields_count的值表示fields表中field_info结构的数量。field_info结构表示这个类或者接口声明的所有字段，包括类变量和实例变量。

- fields[]

  fields表中的每一个值必须是一个field_indo结构，给出了这个类或者接口中一个字段的完整描述。fields表仅仅包含声明在这个类或者接口中的字段。不包括继承自父类或者父接口的字段。

- methods_count

  methods_count的值给出类methods表中method_info结构的数量

- methods[]

  methods表中的每一个值必须是一个method_info结构，method_info给出了这个类或者接口中一个方法的完整描述。如果metchod_info结构中access_flags的ACC_NATIVE和ACC_ABSTRACH标识都没有设置，那么实现了这个方法的java虚拟机指令也会提供。

  me thod_info结构表示了这个类或者接口的所有方法，包括实例方法，类方法，实例初始化方法以及任何的类和接口的初始化方法。methods表不包含继承自父类或者父接口的方法。

- atttibutes_count

  attribute_count的值给出了这个类中attributes表中attrubute的数量。

- attributes[]

  attributes表中的每一个值都是attribute_info结构。

#### 4.2名称的内部表示

##### 4.2.1二进制类和接口名称

出现在class文件结构中的类和接口名称始终以称为二进制名称（binary names）的完全限定形式表示（JLS§13.1）。这些名称始终表示为CONSTANT_Utf8_info结构（第4.4.7节），因此可以从整个Unicode字符空间中获取（如果没有进一步的约束）。类和接口的二进制名称还会被CONSTANT_NameAndType_info结构所引用，用于构成它们的描述符（4.3），引用这些名称通过引用它们的CONSTAN_Class_info结构来实现。

由于历史原因，出现在类文件结构中的二进制名称的语法与JLS§13.1中记录的二进制名称的语法不同。 在此内部形式中，通常将构成二进制名称的标识符分隔的ASCII句点（.）替换为ASCII正斜杠（/）。 标识符本身必须是非限定名称（第4.2.2节）。

例如，Thread类的普通二进制名称是java.lang.Thread。 在类文件格式的描述符中使用的内部形式中，使用表示字符串java/lang/Thread的CONSTANT_Utf8_info结构来实现对类Thread的名称的引用。

##### 4.2.2非限定名称

方法，字段，局部变量和形参的名称使用非限定名称（unqualified names）存储。非全限定名中必须至少包含一个Unicode代码点，并且不得包含ASCII字符. ; [ /(也就是点号，分号，左方括号，正斜杠)。

方法名称进一步受到限制，除了特殊方法名称<init>和<clinit>（§2.9）之外，它们不能包含ASCII字符<或>（即左尖括号或右尖括号） 。

请注意，字段名称或接口方法名称可以是<init>或<clinit>，但是没有方法调用指令可以引用<clinit>，只有invokespecial指令（§invokespecial）可以引用<init>。

#### 4.3描述符

描述符是表示字段或者方法类型的字符串。描述符使用改进的UTF-8字符串（第4.4.7节）以class文件格式表示，因此可以使用整个Unicode字符空间中的字符（如果没有进一步限制的话）。

##### 4.3.1语法符号

描述符使用特定的语法来表示。语法是一组描述了字符序列如何形成各种语法正确的描述符的产物。语法中的终止符使用固定宽度的字体，非终止符使用斜体。非终止符的定义由被定义的非终止名后跟随一个冒号表示。冒号右侧的一个或多个可交换的非终止符连续排列。

出现在右侧的{x}表示出现0次或多次。

右侧的短语（one of）表示下面一行或者多行上的每个终止符都是可选的定义。

##### 4.3.2字段描述符

一个字段描述符表示一个类，实例或者局部变量的类型。

```
FieldDescriptor:
　　FieldType
FieldType:
　　BaseType
　　ObjectType
　　ArrayType
BaseType:
　　(one of)
　　B C D F I J S Z
ObjectType:
　　L ClassName ;
ArrayType:
　　[ ComponentType
ComponentType:
　　FieldType
```

BaseType的字符，ObjecType的L和;以及ArrayType的[都是ASCII字符。

ClassName表示一个二进制类或者接口的内部形式的名称。

字段描述符的类型含义见下表

表示数组类型的字段描述符仅在表示具有255或更少维度的类型时才有效。

| 字段类型项          | 类型        | 含义                                                         |
| ------------------- | ----------- | ------------------------------------------------------------ |
| `B`                 | `byte`      | signed byte                                                  |
| `C`                 | `char`      | Unicode character code point in the Basic Multilingual Plane, encoded with UTF-16 |
| `D`                 | `double`    | double-precision floating-point value                        |
| `F`                 | `float`     | single-precision floating-point value                        |
| `I`                 | `int`       | integer                                                      |
| `J`                 | `long`      | long integer                                                 |
| `L` *ClassName* `;` | `reference` | an instance of class *ClassName*                             |
| `S`                 | `short`     | signed short                                                 |
| `Z`                 | `boolean`   | `true` or `false`                                            |
| `[`                 | `reference` | one array dimension                                          |

int类型的实例变量的字段描述符就是I.

Object类型的实例变量的字段描述符是Ljava/lang/Object;。 请注意，使用类Object的二进制名称的内部形式。

多维数组类型double [] [] []的实例变量的字段描述符是[[[D.

##### 4.3.3方法描述符

方法描述符包含零个或多个参数描述符，表示方法采用的参数类型，以及返回描述符，表示方法返回的值的类型（如果有）。

```
MethodDescriptor:
　　( {ParameterDescriptor} ) ReturnDescriptor
ParameterDescriptor:
　　FieldType
ReturnDescriptor:
　　FieldType
　　VoidDescriptor
VoidDescriptor:
　　V
```

符号V表示方法没有返回值（它的结果是void）。

```
这个方法的描述符:

Object m(int i, double d, Thread t) {...}
是:

(IDLjava/lang/Thread;)Ljava/lang/Object;
注意，使用了Thread和Objec的二进制名称的内部形式。
```

如果一个方法描述符是有效的，那么它对应的方法的参数列表总长度小于255，对于实力方法和接口方法，需要考虑额外的this。参数列表长度的计算规则如下：每个long和double类参数长度为2，其余为1，方法参数列表的总长度等于所有参数长度之和。

无论方法描述的方法是类方法还是实例方法，方法描述符都是相同的。 虽然实例方法被传递了this，指向被调用方法的对象的引用，除了它的预期参数之外，该事实不会反映在方法描述符中。 this的引用由调用实例方法的Java虚拟机指令隐式传递

