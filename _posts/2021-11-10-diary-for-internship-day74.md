---
layout: post
title:  "实习日记74(JVM结构)"
date:   2021-11-10
categories: jekyll update
---

## Day74

- 继续学习JVM规范 JavaSE8版，基于https://docs.oracle.com/javase/specs/jvms/se8/html

- ### Java虚拟机结构

  - #### 对象的表示

    java虚拟机并不要求对象满足任何特定的内部结构。

     *在Oracle的一些Java虚拟机实现中，对类实例的引用是指向句柄的指针，该句柄本身是一对指针：一个指向包含对象方法的表和指向表示Class对象的指针对象的类型，另一个是从堆为对象数据分配的内存。*

  - #### 浮点算法

    Java虚拟机包含IEEE二进制浮点运算标准（ANSI / IEEE Std.754-1985，New York）中规定的浮点运算的子集。

    ##### Java浮点算法和IEEE 754

    Java虚拟机支持的浮点运算与IEEE 754标准之间的主要区别是：

    -  IEEE 754中的定义的无效操作，如被0除，溢出，向下溢出或者不精确，在java虚拟机中不会抛出异常，或者其它IEEE 754中规定的表示方式。java虚拟机没有型号NaN值。（The Java Virtual Machine has no signaling NaN value. ）
    - JAVA虚拟机中不支持IEEE 754中的信号浮点比较。（The Java Virtual Machine does not support IEEE 754 signaling floating-point comparisons. ）
    - Java虚拟机的舍入操作始终使用IEEE 754舍入到最近模式。不精确的结果四舍五入到最接近的可表示值，并且保证此值的最低有效位为0，这是IEEE 754默认模式。但Java虚拟机指令将浮点类型的值转换为整数类型的值，向零舍入。Java虚拟机没有提供任何改变浮点舍入模式的方法。
    - Java虚拟机不支持IEEE 754单扩展或双扩展格式，但是在双精度浮点数集合和双精度扩展指数集合范围与单精度扩展指数格式的表示会有重叠。单精度扩展指数和双精度扩展指数值集（可选择支持）与IEEE 754扩展格式的值不对应：IEEE 754扩展格式规定了扩展精度以及扩展指数范围。

    ##### 浮点格式

    每一个方法都有一个浮点模式，要么是FP-strict，要么时not FP-strict。方法的浮点格式由方法的method_info结构中access_flags项的ACC_STRICT标志来决定。如果设置了这个标志位就是FP-strict，否则就是not FP-strict。

    请注意，ACC_STRICT标志的这种映射意味着由JDK 1.1版或更早版本中的编译器编译的类中的方法实际上时not FP-strict。

    当调用创建包含操作数堆栈的帧的方法具有浮点模式时，我们将操作数堆栈称为具有给定的浮点模式。类似地，当包含该指令的方法具有该浮点模式时，我们将该Java虚拟机指令称为具有给定的浮点模式。

    如果支持单精度扩展指数集（第2.3.2节），则非FP-strict的操作数堆栈上float类型的值可以超出该值集，除非值集转换禁止（§2.8.3））。如果支持双精度扩展指数集（第2.3.2节），则非FP-strict的操作数堆栈上的double类型值可以超出该值集，除非值集转换禁止。

    在所有其他上下文中，无论是在操作数堆栈上还是在其他位置上，无论浮点模式如何，float和double类型的浮点值只能分别在单精度浮点集和双精度浮点集的范围内。特别是，类和实例字段、数组元素、局部变量和方法参数只能从标准值集中取值。

    ##### 数值集转换

    在一些特定场景下，支持扩展指数集合的 Java 虚拟机实现数值在标准浮点数集合与扩展指数集合之间的映射关系是允许和必要的，这种映射操作就称为数值集合转换数值集合转换并非数据类型转换，而是在同一种数据类型之中不同数值集合的映射操作。

    在指示值集转换的情况下，允许实现对值执行以下操作之一：

    - 如果一个值的类型为float，并且不是float值集的元素，则它将值映射到float值集的最近元素。
    - 如果值的类型为double且不是double值集的元素，则它将值映射到double值集的最近元素。

    　　另外，如果如果要进行数值集转换，则需要执行下面的操作：

    - 假设非FP-strict的Java虚拟机指令执行时导致了float类型的值被推送到FP-strict的操作数堆栈，如作为参数传递，或存储到局部变量，字段或数组的元素。如果该值不是floa值集的元素，则将该值映射到float值集的最近元素。
    - 假设非FP-strict的Java虚拟机指令执行时导致了double类型的值被推送到FP-strict的操作数堆栈，如作为参数传递，或存储到局部变量，字段或数组的元素。如果该值不是double值集的元素，则将该值映射到double值集的最近元素。

    在方法调用期间传递浮点类型的参数（包括本地方法调用）可能会发生这种所需的值集转换，如：将浮点类型的值从非FP-strict的方法返回到FP-strict的方法、或者将浮点类型的值存储到非FP严格的方法中的局部变量，字段或数组中。

    并非扩展指数值集中的所有值都可以精确映射到相应标准值集中的值。如果映射的值太大而无法准确表示（其指数大于标准值集允许的值），则将其转换为相应类型的（正或负）无穷大。如果映射的值太小而不能精确表示（其指数小于标准值集所允许的值），则将其舍入到最接近的可表示的非规范化值或相同符号的零。

    数值集转换不会转换无限大、无限小和NaN并且不会改变值的符号。

  - #### 特殊方法
  
    从java虚拟机的维度来看，每一个使用java编程语言编写的构造器都是一个实例初始化方法（*instance initialization method*），并且有一个特殊的名字<init>。这个名称有编译器提供，因为<init>在java编程语言中不是一个合法的标识符，无法使用java编程语言编写。实例初始化方法只能被java虚拟机通过指令调用，并且只会被调用在未初始化的实例上。实例初始化方法和构造器的访问权限一致。
  
    一个类或者接口最多只有一个类或接口初始化方法（*class or interface initialization method*），通过调用这个初始化方法来初始化这个类或者接口。类或者接口的初始化方法有一个特殊的名称<clinit>，它没有参数，并且返回值为void。
  
    *类文件中名为<clinit>的其他方法无关紧要。 它们不是类或接口初始化方法。 它们不能被任何Java虚拟机指令调用，并且永远不会被Java虚拟机本身调用。*
  
    在版本号为51.0或以上的类文件中，方法必须另外设置ACC_STATIC标志(§4.6)，以便成为类或接口初始化方法。
  
    *这个需求是在Java SE 7中引入的。在版本号为50.0或更低的类文件中，一个名为<clinit>的方法返回值是void的，不接受任何参数，不管它的ACC_STATIC标志设置如何，都被认为是类或接口初始化方法。*
  
    <clinit>名称由编译器提供，因为这不是一个有效的标识符，它不能在java编程语言直接使用。类和接口的初始化方法被java虚拟机隐式调用，它们从来不会被java虚拟机指令直接调用，而是作为类初始化过程中一部分被间接调用。
  
    如果下列条件都为真，则方法为签名多态（*signature polymorphic*）:
  
    - 它在java.lang.invoke.MethodHandle类中声明
    - 它只有一个Object[] 参数
    - 它的返回类型是Object
    - 它的设置了`ACC_VARARGS和``ACC_NATIVE`标志位
  
    *在Java SE 8中，仅有的签名多态方法（ signature polymorphic）是`java.lang.invoke.MethodHandle类中`的`invoke` 和 `invokeExact方法。`*
  
    Java虚拟机在invokevirtual指令中对签名多态方法进行了特殊处理，以实现方法句柄（*method handle*）的调用。方法句柄是对底层方法、构造函数、字段或类似低级操作(§5.4.3.5)的强类型，直接可执行的引用，具有可选的参数或返回值转换。这些转换非常普遍，包括转换、插入、删除和替换等模式。可以从Java SE平台API中的java.lang.invoke包中获取更多信息。
  
  - #### 异常
  
     java虚拟机中的异常用Throwable类或者它的子类的实例来表示。抛出一个异常会导致立即非本地（an inmediate nolocal）的控制转移，从发生异常的地方跳到处理异常的地方。
  
    大多数异常是在当前线程执行某些操作时同步发生的。对应的，非同步异常可能发生在程序执行的任何阶段。java虚拟机会由于下面三个原因中的一个抛出异常：
  
    - 执行athrow指令
    - java虚拟机同步检测到非正常的执行情况。这些异常不是在程序中的任意点抛出的，而是在执行以下指令后同步抛出:
      - 指明异常作为一种可能的结果，例如
        - 当指令包含违反Java编程语言语义的操作时，例如在访问数组边界外的元素。
        - 程序在加载或者连接的过程中出现错误
      - 导致了使用了超过限制的资源，例如使用了太多的内存。
    - 异步异常出现的原因：
      - 调用了Thread或者ThreadGroup类的stop方法，或者
      - java虚拟机实现出现了内部错误
  
    一个线程调用stop方法会影响另外一个线程，或者一个限制组的所有线程。它们是异步的异常，因为它们可能出现线程执行的任何位置。内部异常被认为是异步的。
  
    Java虚拟机可能允许在抛出异步异常之前执行少量但有限制的指令。这种延迟允许优化代码检测并抛出这些异常，这些异常可以在符合Java编程语言语义的情况下处理。
  
    *一个简单的实现可能在每个控件传输指令处轮询异步异常。由于程序的大小是有限的，这就为检测异步异常的总延迟提供了一个界限。由于控制传输之间不会发生异步异常，代码生成器具有一定的灵活性，可以在控制传输之间重新排序计算，以获得更好的性能。The paper Polling Efficiently on Stock Hardware by Marc Feeley, Proc. 1993 Conference on Functional Programming and Computer Architecture, Copenhagen, Denmark, pp. 179–187, is recommended as further reading.* 
  
    ​	java虚拟机抛出的异常是精确的：当发生控制转移时，在抛出异常之前执行的指令的所有效果必须可以被观察到。异常抛出之后的指令应当是没有被执行过。如果虚拟机进行了代码优化，导致了异常抛出之后的代码可能被执行了，那么必须保证执行这些代码造成的影响对用户是不可见的。
  
    java虚拟机中的每一个方法都会关联0个或者多个异常处理器（exception handlers）。异常处理器描述了其在方法代码中的有效作用范围（通过字节码偏移量来描述）、能处理的异常类型以及处理异常的代码所在的位置。如果导致异常的指令的偏移量在异常处理程序的偏移范围内，并且异常类型与异常处理器处理的异常类的子类相同，则异常与异常处理器匹配。。当抛出异常时，Java虚拟机在当前方法中搜索匹配的异常处理器。如果找到匹配的异常处理器，系统将跳转到异常处理器指定的异常处理代码处执行。
  
    如果当前方法没有产生异常所对应的异常处理器，当前方法调用会立即结束，当前方法中的操作数栈和局部变量表会被丢弃，栈帧被出栈，恢复调用者方法的栈帧。然后这个异常沿着方法调用链，在调用者栈帧等的上下文中被重新抛出。在到达方法调用链顶层前没有找到合适的异常处理器，异常抛出的线程将被终止。
  
    方法的异常处理器的顺序在搜索匹配时重要的。在class文件中，每个方法的异常处理器时存在表中。在运行时，当异常抛出时，java虚拟机按照顺序搜索当前方法的异常处理器，顺序是根据它们出现在class文件表的位置，从表的起始处开始。
  
    请注意，Java虚拟机不强制嵌套或对方法的异常表项进行任何排序。所以java语言中对异常处理的语义，实际上是通过编译器适当安排异常处理器在表中的顺序来协助完成的。在class文件中定义了明确的异常处理器查找顺序，才能确保无论class文件时通过何种途径产生的，java续集及执行时都能有一致的行为表现。
  
  - #### 指令集简介
  
    java虚拟机指令由一个字节的操作码，接着时0个或多个操作数组成，操作码描述了执行的操作，操作数提供了操作所需的参数或者数据。许多指令没有操作数只包含一个操作码。
  
    如果忽略异常处理，那java虚拟机使用下面的伪代码循环即可有效工作：
  
    ```java
    do {
        atomically calculate pc and fetch opcode at pc;
        if (operands) fetch operands;
        execute the action for the opcode;
    } while (there is more to do);
    ```
  
    操作数的数量和大小都有操作码决定。如果一个操作数大于一个字节，那么它将以大端顺序存储——高位在前。例如一个16位无符号整数使用两个无符号字节存储，byte1和byte2，它的值位(byte1<<8)|byte2.
  
    字节码指令流是单字节对齐的，除了lookupswitch和tableswitch指令。由于它们的操作数比较特殊，都是4字节为界划分的，所以这两条指令也需要预留响应的空位来实现对齐。
  
    限制java虚拟机操作码长度为一个字节，并且放弃了编译后的代码的参数长度对齐，是为了尽可能的获得短小精悍的编译代码，即使这可能让java虚拟机的具体实现付出一定的性能成本为代价。由于每个操作码只有一个字节，所以限制了整个指令集的数量，又因为没有假设数据是对齐的，意味着处理超过一个字节的数据的时候，需要运行时从字节中构建具体的数据结构，这样会损失一定的性能。
  
    ##### 类型和Java虚拟机
  
    java虚拟机指令集中的大多数指令都包含了其操作数的类型信息。例如，iload指令加载一个局部变量到操作数栈中，这个局部变量必须是int类型。fload指令对float值执行相同的操作。这两个指令可以具有相同的实现，但是不能使用相同的操作码。
  
    对于大多数类型相关的指令，指令类型在操作码助记符中用一个字母显示的表示：i表示int，l表示long，s表示short，b表示byte，c表示char，f表示float，d表示double，a表示reference。一些指令的类型是不确定的，这些指令的助记符中不包含类型字母。例如，arraylength总是操作数组对象。一些指令，例如goto是一个无条件控制转移指令，不操作一个有类型的操作数。
  
    由于java虚拟机的操作码长度只有一个字节，所以包含了数据类型的操作码对指令集的设计带来很大的压力。如果每个类型指令都支持所有的java虚拟机运行时数据类型，那么只用一个字节表示所有的指令就不太可能了。相应的，java虚拟机的指令集对于特定的操作提供了有限的类型支持，换句话说，不是每种类型的每个操作都有对应的类型指令。根据需要，可以使用单独的指令在不支持和支持的数据类型之间进行转换。
  
    下表汇总了java虚拟机的指令集支持的类型。具体的指令具有类型信息，通过将opcode列的指令模板中的字母T替换为对应类型列的字母来得到。如果一些指令模板对应的类型列是空的，就表示不存在支持这个类型操作的指令。例如，存在加载int类型的指令iload，但是没有加载byte的指令。
  
    注意到表中的大部分指令都没有整数类型byte，char和short的形式，也没有一个有boolean类型的形式。编译器在编译期和运行期将byte和shot类型带符号位扩展为int类型，将boolean和char类型进行零扩展为int类型，从而使用int类型指令进行操作。同样的，使用java虚拟机指令处理boolean，byte，char，short类型的数组时也会转换为int类型。因此，大多数对于boolean，byte，short和char类型的操作，实际上都是作为int类型作为运算（computational ）类型。
  
    | opcode    | byte    | short   | int       | long    | float    | double  | char    | reference |
    | --------- | ------- | ------- | --------- | ------- | -------- | ------- | ------- | --------- |
    | Tipush    | bipush  | sipush  |           |         |          |         |         |           |
    | Tconst    |         |         | iconst    | lconst  | fconst   | dconst  |         | aconst    |
    | Tload     |         |         | iload     | lload   | fload    | dload   |         | aload     |
    | Tstore    |         |         | istore    | lstore  | fstore   | dstore  |         | astore    |
    | Tinc      |         |         | iinc      |         |          |         |         |           |
    | Taload    | baload  | saload  | iaload    | laload  | faload   | daload  | caload  | aaload    |
    | Tastore   | bastore | sastore | iastore   | lastore | fastore  | dastore | castore | aastore   |
    | Tadd      |         |         | iadd      | ladd    | fadd     | dadd    |         |           |
    | Tsub      |         |         | isub      | lsub    | fsub     | dsub    |         |           |
    | Tmul      |         |         | imul      | lmul    | fmul     | dmul    |         |           |
    | Tdiv      |         |         | idiv      | ldiv    | fdiv     | ddiv    |         |           |
    | Trem      |         |         | irem      | lrem    | frem     | drem    |         |           |
    | Tneg      |         |         | ineg      | lneg    | fneg     | dneg    |         |           |
    | Tshl      |         |         | ishl      | lshl    |          |         |         |           |
    | Tshr      |         |         | ishr      | lshr    |          |         |         |           |
    | Tushr     |         |         | iushr     | lushr   |          |         |         |           |
    | Tand      |         |         | iand      | land    |          |         |         |           |
    | Tor       |         |         | ior       | lor     |          |         |         |           |
    | Txor      |         |         | ixor      | lxor    |          |         |         |           |
    | i2T       | i2b     | i2s     |           | i2l     | i2f      | i2d     |         |           |
    | l2T       |         |         | l2i       |         | l2f      | l2d     |         |           |
    | f2T       |         |         | f2i       | f2l     |          | f2d     |         |           |
    | d2T       |         |         | d2i       | d2l     | d2f      |         |         |           |
    | Tcmp      |         |         |           | lcmp    |          |         |         |           |
    | Tcmpl     |         |         |           |         | fcmpl    | dcmpl   |         |           |
    | Tcmpg     |         |         |           |         | fcmpg    | dcmpg   |         |           |
    | if_TcmpOP |         |         | if_icmpOP |         |          |         |         | if_acmpOP |
    | Treturn   |         |         | ireturn   | lreturn | frenturn | dreturn |         | atreturn  |
  
    java虚拟机实际类型和java虚拟机运算类型之间的映射见下表。
  
    一些java虚拟机指令如pop和swap在操作数栈上操作时忽略类型，不过这些指令也必须受到运算类型分类的限制，这些分类也在下表中给出。
  
    | Actual type   | Computational type | Category |
    | :------------ | ------------------ | -------- |
    | boolean       | int                | 1        |
    | byte          | int                | 1        |
    | char          | int                | 1        |
    | short         | int                | 1        |
    | int           | int                | 1        |
    | float         | float              | 1        |
    | reference     | reference          | 1        |
    | returnAddress | returnAddress      | 1        |
    | long          | long               | 2        |
    | double        | double             | 2        |
  
    ##### 加载和存储指令
  
    加载和存储指令将值在java虚拟机栈帧的局部变量表和操作数栈之间进行转移。
  
    - 加载一个局部变量到操作数栈：*iload*, *iload_<n>*, *lload*, *lload_<n>*, *fload*, *fload_<n>*, *dload*, *dload_<n>*, *aload*, *aload_<n>*. 
    - 从操作数栈中存储一个值到局部变量表：*istore*, *istore_<n>*, *lstore*, *lstore_<n>*, *fstore*, *fstore_<n>*, *dstore*, *dstore_<n>*, *astore*, *astore_<n>*
    - 加载一个常量到操作数栈：*bipush*, *sipush*, *ldc*, *ldc_w*, *ldc2_w*, *aconst_null*, *iconst_m1*, *iconst_<i>*, *lconst_<l>*, *fconst_<f>*, *dconst_<d>*.
    - 扩充局部变量表的访问索引的指令：*wide*
  
    访问对象的属性或者数组的元素也需要和操作数栈进行数据转移。
  
    上面列出的指令助记符中，有一部分是以尖括号结尾的（例如，iload_<n>），这些指令助记符实际上是代表了一组指令（例如iload_<n>有iload_0,iload_1,iload_2,iload3组成）。这几组指令都是某个带有一个操作数的通用指令（如iload）的特殊形式，它们的操作数是隐含的，不需要存储和获取。除此之外，它们的语义和原生通用指令完全一致（如iload_0的语义和操作数为0的iload指令语义完全一致）。尖括号之间的字母指定该指令族的隐式操作数的类型：对于<n>，表示非负整数; 对于<i>，表示int; 对于<l>，表示long; 对于<f>，表示float; 对于<d>，表示double。 在许多情况下，类型int的形式用于对byte，char和short类型的值执行操作。
  
    指令族的概念贯穿这个规范。
  
    ##### 运算指令
  
    算数指令将操作数栈上的两个元素进行计算，然后将结果压入操作数栈。主要有两种类型的运算指令，操作整数和操作浮点数。无论哪种类型的运算指令都使用java虚拟机的数字类型。对于byte，short，char以及boolean类型的值没有直接的整型指令支持它们；这些运算操作使用int类型的指令来操作。整型和浮点型指令在溢出和除零行为上同样不同。运算指令如下：
  
    - 加：*iadd*, *ladd*, *fadd*, *dadd*.
    - 减：*isub*, *lsub*, *fsub*, *dsub*. 
    - 乘：*imul*, *lmul*, *fmul*, *dmul*. 
    - 除：*idiv*, *ldiv*, *fdiv*, *ddiv*. 
    - 求余：*irem*, *lrem*, *frem*, *drem*. 
    - 取反：*ineg*, *lneg*, *fneg*, *dneg*. 
    - 位移：*ishl*, *ishr*, *iushr*, *lshl*, *lshr*, *lushr*. 
    - 位或：*ior*, *lor*. 
    - 位与：*iand*, *land*. 
    - 位异或：*ixor*, *lxor*. 
    - 局部变量自增：*iinc*. 
    - 比较：*dcmpg*, *dcmpl*, *fcmpg*, *fcmpl*, *lcmp*. 
  
    java虚拟机指令集的语义直接支持java编程语言对于整型和浮点型值操作的语义。
  
    java虚拟机没有明确表示操作整数数值时的溢出情况。整型操作能抛出异常的指令只有整数除指令（idiv，ldiv）和整数求余指令（irem和lrem），当它们除0时，将抛出`ArithmeticException`。
  
    java虚拟机操作浮点型数行为和IEEE 754中描述的一样。java虚拟机要求完全支持IEEE 754中定义的非正规（*denormalized*）数值和逐级下溢（*gradual underflow*）。这些特征将会使得某些数值算法处理起来变得更容易一些。
  
    java虚拟机要求浮点数运算时，所有的运算结果都必须舍入到适当精度，非精确的结果必须舍入到可被表示的最接近的精确值，如果两种可表示的形式与该值一样接近，将优先选择最低有效位为零的。这种舍入模式也是IEEE 754规范中的默认舍入模式，称为向最近数舍入模式。
  
    java虚拟机使用IEEE 754中的向零舍入（round towards zero）模式来将浮点数转换为整数。这种模式的舍入结果会导致数字被截断，所有小鼠部分的数字字节都被丢掉。向零舍入模式将在目标数值中选择一个最近的，但是不大于原值的数字作为最精确的舍入结果。
  
    java虚拟机在处理浮点数运算时，不会抛出任何运行时异常（这里说的时java的异常，不要和IEEE 754规范中的浮点异常混淆），当一个操作产生溢出时，将会使用有符号的无穷大来表示，如果某个操作没有明确的定义的话，将会使用NaN来表示。所有使用NaN值作为操作数的算数操作，其结果都是NaN。
  
    对与long类型的比较（lcmp）使用带符号的比较方式。对于浮点型类型(*dcmpg*, *dcmpl*, *fcmpg*, *fcmpl*) 使用IEEE 754规定非信号比较方式（nonsignaling comparisons）。
  
    ##### 类型转换指令
  
    类型转换指令允许java虚拟机中的数字类型相互转换。这些转换操作一般用于实现用户代码的显示类型转换操作，或者用来处理Java虚拟机字节码指令集中指令非完全独立的问题。（部分类型没有对应的操作指令，具体参考2.11.1）
  
    java虚拟机直接支持以下数值的宽化类型转换：
  
    - int到long，float或者double
    - long到float或者double
    - float到double
  
    宽化数值转化指令有i2l,i2f,i2d,l2f,l2d以及f2d。这些操作码的助记符直接从名称上给出了类型转换的内容，使用2来作为双关语表示to。例如，i2d指令表示将int值转换为double值。
  
    大多数宽化数字转化不会丢失数值的数量级信息。相应的，将int转换为long，将int转换为double不会丢失任何信息。在FP-strict模式下将float转换为double会保留精确的数值，只有在非FP-strict模式下转化可能丢失数值的数量级发生变化。
  
    从int到float，或从long到float，或从long到double的转换可能会失去精度，也就是说，可能会丢失该值的一些最低有效位; 生成的浮点值是整数值的精确舍入版本，使用IEEE 754舍入到最近模式。
  
    不考虑可能丢失精度的事实情况，宽化数值转换不会导致虚拟机抛出运行时异常（不要和IEEE 754中的浮点异常混淆）。
  
    将int宽化数值转换为long只需对int值的二进制补码表示进行扩展，以填充为更宽的格式。 将char宽化数值转换为int类型需要对char值进行零 - 扩展以填充为更宽的格式。
  
    注意到宽化数值类型转化不包括将整型类型byte,char,short转化为int。如2.11.1中所述，byte，char和short类型的值在内部扩展为int类型，这使得这些转换是隐式的。
  
    java虚拟机也直接支持下面的窄化数值转化（narrowing numeric conversions）：
  
    - int 到 byte，short 或者 char
    - long 到 int
    - float 到 int 或者 long
    - double 到 int，long 或者 float
  
    上面的窄化数值转换是：i2b,i2c,i2s,l2i,f2l,d2i,d2l和d2f。窄化数值转换可能导致结果具有不同的符号，不同的数量级，或者两者都有；转换过程可能导致数值丢失精度。
  
    将int或者long窄化数值转换为整型T，只是简单的丢弃除最低n位之外的所有位，n指类型T的长度。这回导致结果和输入具有不同的符号位。
  
    当将浮点型数值窄化转换为类型T时，T为int或者long，遵循以下规则：
  
    - 如果浮点数为NaN，那么转换结果为int 0或者long 0。
    - 否则，如果浮点型数值不是无穷大，那么浮点数将使用IEEE 754的舍入到0模式转换为整数V，并且下面符合下面两种情况：
      - 如果T是long类型，并且这个转换的整数值V在long的范围之内，那么结果就是V。
      - 如果T是int类型，并且整数值V在int范围之内，那么结果就是V。
    - 否则：
      - 如果数值非常小（比如一个具有非常大数量级的负数或者负无穷大），那么将用int或者long中的最小的数来表示。
      - 如果数值太大（比如一个具有非常大数量级的正数或者正无穷大），那么结果将用int或者long中最大的数来表示。
  
    将double窄化数值转换为float过程和IEEE 754中一致。结果使用IEEE 754的舍入到零模式转换。如果数值太小无法表示为float，那么使用float的正负0来表示；如果数值太大，那么使用float的正负无穷大来表示。double的NaN总是转换为float的NaN。
  
    不考虑可能溢出，下溢出或者丢失精度的可能情况，窄化数值转换不会导致虚拟机抛出运行时异常（不要和IEEE 754中的浮点异常混淆）。
  
    ##### 创建和操作对象
  
    尽管类实例和数组都是对象，但是java虚拟机创建和操作类实例和数组使用不用指令集：
  
    - 创建一个类实例：new
    - 创建一个数组：newarray，anewarray，multianewarray。
    - 访问类字段（static fields）和类实例的字段（非静态域）：getstatic,putstatic,getfield,putfield。
    - 加载数组元素到操作数栈：baload，caload，saload，iaload，laload，faload，daload，aaload。
    - 从操作数栈中存储一个元素到数组中：bastore，castore，sastore，iastore，lastore，fastore，dastore，aastore
    - 获取数组的长度：arraylength
    - 检测类实例或者数组的属性：instanceof，checkcast
  
    ##### 操作数栈管理命令
  
    提供了一系列指令去直接操作操作数栈：*pop*, *pop2*, *dup*, *dup2*, *dup_x1*, *dup2_x1*, *dup_x2*, *dup2_x2*, *swap*. 
  
    ##### 控制转移指令
  
    控制转移指令可以让java虚拟机有条件或者无条件的跳转到指定指令处执行，而不是继续执行下一条指令。控制转移指令包括有：
  
    条件分支：*ifeq*, *ifne*, *iflt*, *ifle*, *ifgt*, *ifge*, *ifnull*, *ifnonnull*, *if_icmpeq*, *if_icmpne*, *if_icmplt*, *if_icmple*, *if_icmpgt* *if_icmpge*, *if_acmpeq*, *if_acmpne*. 
  
    组合条件分支：*tableswitch*, *lookupswitch*. 
  
    无条件分支：*goto*, *goto_w*, *jsr*, *jsr_w*, *ret*. 
  
    java虚拟机在条件分支中比较int和reference类型的值使用不同的指令集。它还有不同的条件分支指令，用于测试null引用，因此不需要为null指定具体的值（参考2.4）。
  
    条件分支在boolean，byte，char，和short之间比较时，使用int的比较指令（2.11.1）。对于long类型，float类型和double类型的条件分支比较操作，则会先执行相应类型的比较运算指令（2.11.3）,运算指令会返回一个int结果。随后执行int类型的条件分支比较操作来完成整个分支的跳转。由于各种类型的比较最终都会转换为int类型的比较操作，基于int类型比较的重要性，java虚拟机提供了非常丰富的int类型的条件分支指令。
  
    所有int类型的条件分支转移指令都是有符号的比较操作。
  
    ##### 方法调用和返回指令
  
    下面五个指令用于方法调用：
  
    - invokevitual调用对象的实例方法，根据对象的实际类型进行分派（虚方法分派），这也是java语言中最常见的方法分派方式。
    - invokeinterface调用接口的方法，它会在运行时搜索一个实现了这个接口方法的对象，找出合适的方法进行调用。
    - invokespecial指令用于调用一些需要特殊处理的实例方法，包括实例初始化方法（2.9），私有方法或者父类的方法。
    - invokestatic指令用于调用一个类的静态方法。
    - invokedynamic指令用于调用的方法是绑定到invokedynamic指令的调用端对象的目标。Java虚拟机将调用端对象绑定到invokedynamic指令的特定词法，这是在第一次执行该指令之前运行引导方法的结果。因此，与调用方法的其他指令不同，invokedynamic指令的每次出现都有一个惟一的链接状态。
  
    方法的返回指令通过返回类型来区分，包括：*ireturn* (用于返回类型是 `boolean`, `byte`, `char`, `short`,或者 `int`), *lreturn*, *freturn*, *dreturn*, and *areturny*以及return指令用于返回值是void的方法，实例初始化方法，类或者接口初始化方法的返回。
  
    ##### 抛出异常
  
    在程序中抛出异常使用athrow指令。多个java虚拟机指令在检测到异常情况是也会抛出异常。
  
    ##### 同步
  
    java虚拟机支持方法同步和方法中一系列指令同步，同步使用同步结构：monitor
  
    方法级别的同步是隐式的，作为方法调用和返回的一部分。方法调用指令检查运行时常量池中的method_info结构中的ACC_SYNCHRONIZED标识来区分一个方法是不是同步方法。当调用一个设置了ACC_SYNCHRONIZED标识的方法，执行线程进入monitor，调用这个方法，当方法不管是正常或者异常调用结束时，线程退出monitor。在执行这个方法的时间段内，只有执行的线程拥有这个monitor，没有其他线程可以进入。当调用同步方法时抛出异常，并且在同步方法内部没有处理这个异常，那么这个方法的monitor将在同步方法向外抛出异常前自动退出。
  
    指令序列的同步通常使用java编程语言中的synchronized块。java虚拟机提供monitorenter和monitorexit指令来支持这个语法。正确实现synchronized需要编译器和java虚拟机合作完成。
  
    结构化锁定（*Structured locking*）是指在方法调用期间每一个给定的monitor退出都将匹配这个这个monitor之前的进入的情形。由于没法保证所有提交给java虚拟机的代码都会满足结构化锁定，java虚拟机的实现者允许（不是必需的）强制执行下面两条规则来保证结构化锁定。其中T表示一个线程，M表示一个monitor。
  
    　　1、当方法调用完成时，不管时正常结束还是异常退出，T进入M的次数必需和T离开M的次数相等。
  
    　　2、在方法调用过程中，任何时刻都不会出现T离开M的次数多余T进入M的次数。
  
    注意，当调用一个同步方法时，java虚拟机自动执行monitor的进入和退出也被认为时在方法调用期间完成。
  
  - #### 类库
  
    java虚拟机对于java se平台实现类库必须提供足够的支持。这些类库中的一些类如果没有java虚拟机的协作将无法实现。
  
    支持以下功能的类可能需要java虚拟机的特殊支持：
  
    - 反射，例如java.lang.reflect中的类和Class类。
    - 加载和创建一个类或者接口。上面提到的列子也适用于这点
    - 链接和初始化一个类或者接口。上面提到的列子也适用于这点
    - 安全，例如java.security包中的类和其它例如SecurityManager的类
    - 多线程，例如Thread类
    - 弱引用，例如java.lang.ref包中的类。
  
    上面的列表是为了说明问题，而不是全面的介绍类库。这些类或它们提供的功能的详尽列表超出了本规范的范围。有关详细信息，请参阅Java SE平台类库的规范。
  
  - #### 公有设计，私有实现
  
    到目前为止，该规范概述了Java虚拟机的公共框架：类文件格式和指令集。这些组件对Java虚拟机的硬件、操作系统和实现独立性至关重要。实现者可能更愿意将它们看作是在每个实现Java SE平台的主机之间安全地通信程序片段的一种方法，而不是作为要严格遵循的蓝图。
  
    理解公共设计和私有实现之间的界线是很重要的。Java虚拟机实现必须能够读取类文件，并且必须准确地实现其中Java虚拟机代码的语义。实现此目的的一种方法是将此文档作为规范并按字面意思实现该规范。但是，对于实现者来说，在此规范的约束下修改或优化实现也是完全可行和可取的。只要能够读取类文件格式并维护其代码的语义，实现者就可以以任何方式实现这些语义。只要小心维护正确的外部接口，“底层”是实现者的事情。
  
    也有一些例外:调试器、分析器和即时代码生成器都需要访问Java虚拟机的元素，这些元素通常被认为是“隐藏在底层”的。在适当的情况下，Oracle与其他Java虚拟机实现者和工具供应商合作，开发Java虚拟机的公共接口，供这些工具使用，并在整个行业推广这些接口。
  
    实现者可以使用这种灵活性为高性能、低内存使用或可移植性定制Java虚拟机实现。在给定的实现中什么是有意义的取决于该实现的目标。实施方案的范围包括:
  
    在加载时或执行期间将Java虚拟机代码转换为另一个虚拟机的指令集。
  
    在加载时或执行期间将Java虚拟机代码转换为主机CPU的本机指令集(有时称为即时代码生成，或JIT)。
  
    精确定义的虚拟机和目标文件格式的存在不需要显著限制实现者的创造力。Java虚拟机的设计目的是支持许多不同的实现，提供新的和有趣的解决方案，同时保持实现之间的兼容性。

