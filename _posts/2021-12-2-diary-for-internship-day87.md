---
layout: post
title:  "实习日记87"
date:   2021-12-02(Java Class文件格式)
categories: jekyll update
typora-root-url: ./
---

## Day87

### 4.Class文件格式

#### 4.4常量池

java虚拟机指令并不依赖类、接口、类实例或者数组的运行时布局。相反，指令依靠常量池中的符号信息。

所有的常量池条目都有如下的通用结构：

```
cp_info {
    u1 tag;
    u1 info[];
}
```

常量池表中的每一个项目是以1比特的标识位开始，指示是哪种cp_info条目。info数组的内容由标志位来决定。有效的标识以及对应的值见下表。每个标识位后面必须跟2个或更多字节，这些字节给出了这些指定常量的信息。额外信息的格式由标识值来决定。

| Constant Type                 | Value |
| ----------------------------- | ----- |
| `CONSTANT_Class`              | 7     |
| `CONSTANT_Fieldref`           | 9     |
| `CONSTANT_Methodref`          | 10    |
| `CONSTANT_InterfaceMethodref` | 11    |
| `CONSTANT_String`             | 8     |
| `CONSTANT_Integer`            | 3     |
| `CONSTANT_Float`              | 4     |
| `CONSTANT_Long`               | 5     |
| `CONSTANT_Double`             | 6     |
| `CONSTANT_NameAndType`        | 12    |
| `CONSTANT_Utf8`               | 1     |
| `CONSTANT_MethodHandle`       | 15    |
| `CONSTANT_MethodType`         | 16    |
| `CONSTANT_InvokeDynamic`      | 1     |

##### 4.4.1CONSTANT_Class_info结构

CONSTANT_Class_info结构用于表示类或者接口：

```
CONSTANT_Class_info {
    u1 tag;
    u2 name_index;
}
```

CONSTANT_Class_info结构的项目如下：

- tag

  标志位的值为CONSTANT_Class（7）

- name_index

  name_index的值必须是常量池表中的一个有效索引。常量池在这个索引位置的条目必须是一个CONSTANT_Utf8_info结构，表示一个有效的二进制类或者接口名称，这个名称使用内部形式进行编码。

因为数组是对象，字节码anewarray和multianewarray（不包括字节码new）能够通过常量池中的CONSTANT_Class_info结构来引用一个数组“类”。对于这样的数组“类”，类的名称就是数组类型来的描述符（4.3.2）。

*例如，二维数组类型int[][]的类名表示为[[I，然而Thread[]的类名表示为[Ljava/lang/Thread。*

只有数组的维度为255或者更少时，数组类型描述符才是有效的

##### 4.4.2CONSTANT_Fieldred_info,CONSTANT_Methodref_info,以及CONTANT_InterfaceMethodref_info结构

字段，方法和接口方法使用相似的结构来表示：

```
CONSTANT_Fieldref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}

CONSTANT_Methodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}

CONSTANT_InterfaceMethodref_info {
    u1 tag;
    u2 class_index;
    u2 name_and_type_index;
}
```

这个结构的条目如下：

- tag

  CONTANT_Fieldref_info的tag值为CONSTANT_Fieldref(9)。

  CONTANT_Methodref_info的tag值为CONSTANT_Methodref(10)。

  CONTANT_InterfaceMethodref_info的tag值为CONSTANT_InterfaceMethodref(11)。

- class_index

  class_index的值必须是constant_pool表中的一个有效索引。索引位置的条目必须是一个CONSTANT_Class_info结构，表示一个有这个字段或者方法作为成员的类或者接口类型。

  CONSTANT_Methodref_info结构的class_index必须是一个类类型，不能是接口类型

  CONSTANT_InterfaceMethodref_info的class_index表示的条目必须是一个接口类型

  CONSTANT_Fieldref_info结构的class_index条目可以是类类型或者接口类型。

- name_and_type_index

  name_and_type_index的值必须是constant_pool表中的一个有效索引。索引位置的条目必须是一个CONSTANT_NameAndType_info结构，指示这个字段或者方法的名称和描述符。

在CONSTANT_Fieldref_info中，指示的描述符必须是一个字段描述符。其它的必须是方法描述符。

如果CONSTANT_Methodref_info结构的方法的名称以'<' （'\u003c'）开始，那么这个名称必须是特殊名称<init>表示一个实例的初始化方法。它的返回类型必须是void。

##### 4.4.3CONSTANT_String_info结构

CONSTANT_String_info结构用来表示String类型的常量对象

```
CONSTANT_String_info {
    u1 tag;
    u2 string_index;
}
```

CONSTANT_String_info结构的项目如下：

- tag

　　CONSTANT_String_info的tag值为CONSTANT_String（8）。

- string_index

　　class_index的值必须是constant_pool表中的一个有效索引。索引位置的条目必须是一个CONSTANT_Utf8_info结构，表示需要初始化的String对象的Unicode代码点的序列。

##### 4.4.4CONSTANT_Integer_info和CONSTANT_Float_info结构

CONSTANT_Integer_info和CONSTANT_Float_info结构表示4字节数值常量(int 和 float)。

```
CONSTANT_Integer_info {
    u1 tag;
    u4 bytes;
}

CONSTANT_Float_info {
    u1 tag;
    u4 bytes;
}
```

这些架构的项目如下：

- tag

　　CONSTANT_Integer_info结构的tag值为CONSTANT_Integer（3）.

　　CONSTANT_Float_info结构的tag值为CONSTANT_Float（4）.

- bytes

　　CONSTANT_Integer_info的bytes项目表示int常量的值，int值的字节存储为大端模式。

　　CONSTANT_Float_info的bytes项目表示float常量的值，使用IEEE 754单精度浮点格式。单精度格式的字节存储为大端模式。

　　CONSTANT_Float_info结构表示的值由下面这些规则决定。值的字节首先按照int形式转换，然后：

　　　　1、如果转换的比特位为0x7f800000，那么float的值为正无穷大

　　　　2、如果转换的比特位为0xff800000，那么float的值为负无穷大

　　　　3、如果转换的比特位在0x7f800001到0x7fffffff或者在0xff800001到0xffffffff之间，那么float的值为NaN。

　　　　4、其它情况下，flaot的值根据比特位计算如下：

```
int s = ((bits >> 31) == 0) ? 1 : -1;
int e = ((bits >> 23) & 0xff);
int m = (e == 0) ?
          (bits & 0x7fffff) << 1 :
          (bits & 0x7fffff) | 0x800000;
```

那么float的值等于数学表达式`s · m · 2e-150`.的结果

##### 4.4.5CONSTANT_Long_info和CONSTANT_Double_info结构

CONSTANT_long_info和CONSTANT_double_info结构表示8字节数值常量：

```
CONSTANT_Long_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}

CONSTANT_Double_info {
    u1 tag;
    u4 high_bytes;
    u4 low_bytes;
}
```

所有8字节常量都占用常量池表中的两个Class条目。如果CONSTANT_Long_info或CONSTANT_Double_info结构是常量池表中索引`n`处的项目，则池中的下一个可用项目位于索引`n+2`处。该常量池指数`n+1`必须是有效的但通常认为是不可用的。

*回顾一下，让8字节常量占用常量池两个实体不是一个好主意*

这些架构的项目如下：

- tag

  CONSTANT_Long_info结构的tag值为 CONSTANT_Long(5)

  CONSTANT_Double_info结构的tag值为CONSTANT_Double(6)

- high_bytes,low_bytes

  CONSTANT_Long_info结构的无符号high_bytes和low_bytes项目一同表示了long类型常量的值。

  ```
  ((long) high_bytes << 32) + low_bytes
  ```

  其中high_bytes和low_bytes按照大端(高字节在线)顺序存储
  
  CONSTANT_Double_info结构的high_bytes和low_bytes项目一同表示了double IEEE754浮点双精度类型常量的值，每个项目的字节以大端(高字节在先)顺序存储
  
  CONSTANT_Double_info结构表示的值如下确定。high_bytes和low_ bytes项被转化成long恒定比特，它等于：
  
  ```
  ((long) high_bytes << 32) + low_bytes
  ```
  
  并且：
  
  - 如果比特位为`0x7ff0000000000000L`，则该`double` 值将为正无穷大。
  
  - 如果比特位为`0xfff0000000000000L`，则该`double` 值将为负无穷大。
  
  - 如果比特位是`0x7ff0000000000001L`到 `0x7fffffffffffffffL`或在`0xfff0000000000001L`到`0xffffffffffffffffL`，双值将为NaN。
  
  - 在所有其他情况下， let `s`、`e`、 和`m`是可能从*bits*计算出的三个值：
  
    ```
    int s = ((bits >> 63) == 0) ? 1 : -1;
    int e = (int)((bits >> 52) & 0x7ffL);
    long m = (e == 0) ?
               (bits & 0xfffffffffffffL) << 1 :
               (bits & 0xfffffffffffffL) | 0x10000000000000L;
    ```
  
    浮点值等于`double`数学表达式的值。 `s · m · 2e-1075`

##### 4.4.6CONSTANT_NameAndType_info结构

CONSTANT_NameAndType_info结构用来表示字段和方法，但是不指明属于哪个类或者接口类型：

```
CONSTANT_NameAndType_info {
    u1 tag;
    u2 name_index;
    u2 descriptor_index;
}
```

CONSTANT_NameAndType_info结构的项目如下：

- tag

　   CONSTANT_NameAndType_info结构的tag项目的值为CONSTANT_NameAndType（12）.

- name_index

　   name_index的值必须是常量池表中的有效索引。这个索引位置必须是一个CONSTANT_Utf8_info结构，表示一个特殊的方法名<init>或者一个有效的非限定名称指示一个字段或者方法。

- desciptor_index

　　name_index的值必须是常量池表中的有效索引。这个索引位置必须是一个CONSTANT_Utf8_info结构，表示一个有效的字段描述符或者方法描述符。

##### 4.4.7 CONSTANT_Utf8_info结构

CONSTANT_Utf8_info结构用来表示一个字符串常量值：

```
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```

结构中的项目如下：

- tag

　　CONSTANT_Utf8_info的tag值为CONSTANT_Utf8（1）.

- length

　　length的值bytes数组中字节的数量（不是string的长度）。

- bytes[]

　　bytes数组包含的字符串的字节。

　　字节的值不能为(byte)0

　　字节的值不能为(byte)0xf0 到 (byte)0xff.

字符串的内容使用修正的UTF-8编码。使用修正的UTF-8编码以便每个代码点仅使用一个字节来表示仅包含非空ASCII字符的代码点序列，也可以表示Unicode代码空间的所有代码点。修正的UTF-8不以空值结束。编码过程如下：

代码点在'\u0001' 到 '\u007F'范围间，使用一个单独的字节来表示：

| *0*  | *bits 6-0* |
| ---- | ---------- |

7位的数值给出了代码点表示的值。

null代码点（'\u0000'）和代码点在'\u0080'到'\u07FF'范围的使用一对字节x和y来表示：

x:

| *1*  | *1*  | *0*  | *bits 10-6* |
| ---- | ---- | ---- | ----------- |

y:

| *1*  | *0*  | *bits 5-0* |
| ---- | ---- | ---------- |

这两个字节的代码点的值为：

```
((x & 0x1f) << 6) + (y & 0x3f)
```

代码点在'\u0800'到 '\uFFFF'范围的使用三个字节x,y,z来表示：

x:

| *1*  | *1*  | *1*  | *0*  | *bits 15-12* |
| ---- | ---- | ---- | ---- | ------------ |

y:

| *1*  | *0*  | *bits 11-6 * |
| ---- | ---- | ------------ |

z:

| *1*  | *0*  | *bits 5-0 * |
| ---- | ---- | ----------- |

三字节表示的代码点的值为：

```
((x & 0xf) << 12) + ((y & 0x3f) << 6) + (z & 0x3f)
```

代码点高于U + FFFF的字符（所谓的补充字符）通过分别编码其UTF-16表示的两个代理代码单元来表示。每个代理代码单元由三个字节表示。这意味着补充字符由六个字节u，v，w，x，y和z表示：

| `u`: | 11101101          |
| ---- | ----------------- |
| `v`: | 1010(bits 20-16)- |
| `w`: | 10bits 15-10      |
| `x`: | 11101101          |
| `y`: | 1011bits9-6       |
| `z`: | 10bits5-0         |

六字节表示的代码点的值为：

```
0x10000 + ((v & 0x0f) << 16) + ((w & 0x3f) << 10) +
((y & 0x0f) << 6) + (z & 0x3f)
```

多字节字符的字节以大端模式（高字节优先）顺序存储在类文件中。

这种格式与“标准”UTF-8格式有两点不同。首先，使用2字节格式而不是1字节格式对空字符（char）0进行编码，以便修正的UTF-8字符串永远不会嵌入空值。其次，仅使用标准UTF-8的1字节，2字节和3字节格式。 Java虚拟机无法识别标准UTF-8的四字节格式;它使用自己的2*三字节格式。

##### 4.4.8CONSTANT_MethodHandle_info结构

CONSTANT_MethodHandle_info结构用于表达一个方法句柄：

```
CONSTANT_MethodHandle_info {
    u1 tag;
    u1 reference_kind;
    u2 reference_index;
}
```

结构中项目如下：

- tag

  CONSTANT_MethodHandle_info的tag值为CONSTANT_MethodHandle（15）.

- reference_kind

  reference_kind的值必须在1~9之间，该值表示此方法句柄的种类，其表示字节码行为。

- reference_index

  reference_index的值必须是常量池表中的一个有效索引，该索引处条目如下所示：

  - 如果项目的值`reference_kind`是 1 ( `REF_getField`)、2 ( `REF_getStatic`)、3 ( `REF_putField`) 或 4 ( `REF_putStatic`)，则该`constant_pool`索引处的 条目必须是 `CONSTANT_Fieldref_info`( [§4.4.2](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.2) ) 结构，表示方法句柄要对其进行处理的字段被创建。
  - 如果`reference_kind`项的值为 5 ( `REF_invokeVirtual`) 或 8 ( `REF_newInvokeSpecial`)，则该`constant_pool`索引处的条目必须是一个 `CONSTANT_Methodref_info`结构（第[4.4.2 节](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.2)），表示要为其创建方法句柄的类的方法或构造函数（第[2.9 节](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.9)） .
  - 如果该项的值为`reference_kind`6 ( `REF_invokeStatic`) 或 7 ( `REF_invokeSpecial`)，则如果`class`文件版本号小于 52.0，则`constant_pool`该索引处的 条目必须是`CONSTANT_Methodref_info`表示要为其创建方法句柄的类方法的 结构体；如果 `class`文件版本号为 52.0 或更高，则`constant_pool`该索引处的 条目必须是 `CONSTANT_Methodref_info`结构或 `CONSTANT_InterfaceMethodref_info`结构（第[4.4.2 节](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.2)）表示要为其创建方法句柄的类或接口的方法。
  - 如果该项的值为`reference_kind`9 ( `REF_invokeInterface`)，则该`constant_pool` 索引处的条目必须是`CONSTANT_InterfaceMethodref_info`表示要为其创建方法句柄的接口方法的 结构。

  如果该项的值为`reference_kind`5( `REF_invokeVirtual`)、6( `REF_invokeStatic`)、7( `REF_invokeSpecial`)或9( `REF_invokeInterface`)，则`CONSTANT_Methodref_info` 结构或`CONSTANT_InterfaceMethodref_info`结构所表示的方法名称不得为`<init>`或`<clinit>`。

  如果值为 8 ( `REF_newInvokeSpecial`)，则`CONSTANT_Methodref_info`结构所表示的方法的名称必须为`<init>`。

##### 4.4.9CONSTANT_MethodType_info结构

CONSTANT_MethodType_info结构表示方法种类

```
CONSTANT_MethodType_info {
    u1 tag;
    u2 descriptor_index;
}
```

结构中项目如下：

- tag

  CONSTANT_MethodType_info结构的tag值是CONSTANT_MehtodType（16）

- descriptor_index

  descriptor_index的值必须是常量池中的一个有效索引。该constant_pool索引处的条目必须是表示方法描述符的CONSTANT_Utf8_info结构。

##### 4.4.10CONSTANT_InvokeDynamic_info结构

CONSTANT_InvokeDynamic_info结构用于通过*invokedynamic* 指令来指定一个自举方法，动态调用名，参数和返回类型呼叫的，并且可选地，附加的常数的序列称为*静态参数*到引导方法。

```
CONSTANT_InvokeDynamic_info { 
    u1 标签；
    u2 bootstrap_method_attr_index; 
    u2 name_and_type_index; 
}
```

结构中项目如下：

- tag

  CONSTANT_InvokeDynamic_info的tag值是 CONSTANT_InvokeDynamic(18)。

- bootstrap_method_attr_index

项目的值`name_and_type_index`必须是`constant_pool`表中的有效索引。该`constant_pool`索引处的 条目必须是一个 `CONSTANT_NameAndType_info`结构，表示方法名称和方法描述符。