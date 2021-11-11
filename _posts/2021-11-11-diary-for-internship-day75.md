---
layout: post
title:  "实习日记75(JVM编译)"
date:   2021-11-11
categories: jekyll update
---

## Day75

- 学习Java虚拟机规范，基于https://docs.oracle.com/javase/specs/jvms/se8/html/

- ### JVM编译

  java虚拟机是设计用来支持java编程语言的。Oracle的JDK软件包含了一个将Java源代码编译成java虚拟机指令集的编译器，以及一个用于java虚拟机本身的运行时系统。了解编译器如何使用java虚拟机对编译器作者来说是有用的，同样也有助于理解java虚拟机本身。本章中编号的部分不是规范性的。

  注意，术语“编译器”有时用于指从Java虚拟机的指令集到特定CPU的指令集的转换程序。这种转换器的一个例子是即时(just-in-time, JIT)代码生成器，它只在加载Java虚拟机代码之后生成特定于平台的指令。本章不讨论与代码生成相关的问题，只讨论与将用Java编程语言编写的源代码编译为Java虚拟机指令相关的问题。

  - #### 示例的格式

    本章主要由源代码示例和带注释的Java虚拟机代码清单组成，这些代码是由Oracle的JDK 1.0.2版本中的javac编译器为这些示例生成的。Java虚拟机代码是用非正式的“虚拟机汇编语言”编写的，由Oracle的javap工具生成，随JDK发行版一起发布。您可以使用javap生成其他已编译方法的例子。

    如果读者阅读过汇编代码，都应该熟悉示例中的格式。每个指令的格式如下：

    ```bash
    <index> <opcode> [ <operand1> [ <operand2>... ]] [<comment>]
    ```

    <index>是包含该方法的Java虚拟机代码字节的数组中指令的操作码的索引。<index>可以被认为是从方法起始处的字节偏移量。<opcode>是指令操作码的助记符，零或更多<operandN>是指令的操作数。 可选的<comment>以行尾注释语法给出：

    ```base
    8   bipush 100     // Push int constant 100
    ```

    注释中的一部分是有javap产生的，剩余部分由作者添加的。每条指令前的<index>可以被用于控制转移指令的目标。例如，goto 8这条指令表示跳转到索引为8的指令处执行。需要注意的是，java虚拟机的控制转移指令的实际操作数是当前指令的操作码集合中的地址偏移量，这些操作数会被javap工具按照更容易被人阅读的方式来显示。

    我们在表示运行时常量池索引的操作数的前面加上一个#符号，然后接着指令之后有一条注释来标识引用的运行时常量池项，如下所示:

    ```base
    10  ldc #1         // Push float constant 100.0
    ```

    或者：

    ```base
    9   invokevirtual #4    // Method Example.addTwo(II)I
    ```

    本章节主要目的是描述虚拟机的编译过程，我们将忽略一些诸如操作数容量等细节问题。

  - #### 常量、局部变量和而控制结构的使用

    java虚拟机代码中展示了java虚拟机设计和使用所遵循的一些通用特性。在第一个例子中，我们遇到了许多这样的情况，我们对它们进行了详细的考虑。

    spin方法简单的进行了100次空循环：

    ```java
    void spin() {
        int i;
        for (i = 0; i < 100; i++) {
            ;    // Loop body is empty
        }
    }
    ```

    编译器可能将其编译为下面的代码：

    ```base
    0   iconst_0       // Push int constant 0
    1   istore_1       // Store into local variable 1 (i=0)
    2   goto 8         // First time through don't increment
    5   iinc 1 1       // Increment local variable 1 by 1 (i++)
    8   iload_1        // Push local variable 1 (i)
    9   bipush 100     // Push int constant 100
    11  if_icmplt 5    // Compare and loop if less than (i < 100)
    14  return         // Return void when done
    ```

    Java虚拟机是面向堆栈的，大多数操作从Java虚拟机当前帧的操作数堆栈中获取一个或多个操作数，或者将结果推回到操作数堆栈中。任何时候当一个方法被调用时，一个新的栈帧就会被创建出来，同时创建一个新的操作数栈和局部变量表供这个方法使用。因此在计算的任何一点，每个控制线程可能存在许多栈帧和相同数量的操作数堆栈，对应于许多嵌套方法调用。 只有当前帧中的操作数堆栈处于活动状态。

    java虚拟机指令集使用不同的字节码来区分不同的操作数类型，用于操作各种类型的操作数。spin方法仅仅操作了int类型的值。它编译后的代码中的指令都选择了针对int型的数据类型操作指令（*iconst_0*, *istore_1*, *iinc*, *iload_1*, *if_icmplt*）。

    spin方法中的两个常量0和100，使用两个不同的指令压入操作数栈。压入0使用了iconst_0指令，是iconst_<i>指令家族之一。压入100使用了bipush指令，这个指令获取它的立即数压入栈中。

    java虚拟机经常使用操作码隐式的包含操作数（如整型常量-1,0,1,2,3,4和5在iconst_<i>指令的例子）。因为iconst_0指令知道它将要压入一个整数0，iconst_0不再需要存储一个操作数来告诉它应该压入哪个值，也不需要获取和解析一个操作数。将压入0编译成bipush 0也是正确的，凡是会导致spin编译后的代码长度增加一个字节。一个简单的虚拟机也会在每次循环中花费额外的时间来获取和解码显式的操作数。使用隐含的操作数使得编译后的代码更加紧凑和高效。

    spin方法中的i存在java虚拟机局部变量1中。因为大多数java虚拟机指令操作从操作数栈中弹出的值，而不是直接使用局部变量，在为Java虚拟机编译的代码中，在局部变量和操作数堆栈之间传输值的指令很常见。这些操作同样被指令集特殊的支持。在spin方法中，值在局部变量表中传输使用istore_1和iload_1指令，每个指令都隐式的操作局部变量表中位置为1的值。istore_1指令从操作数栈中弹出一个int值，然后存入局部变量1中。iload_1指令将局部变量1的值压入操作数栈。

    使用和重用局部变量是编译器作者决定的。特殊的load和store指令应该鼓励编译器作者尽可能的重用局部变量。这样编译后的代码会更快，更紧凑，并且使用栈帧更少的空间。

    对局部变量的某些非常频繁的操作由Java虚拟机专门处理。iinc指令为局部变量增加一个长度为1字节有符号的值。spin中的iinc指令将第一个局部变量（这个指令的第一个操作数）加1（这个指令的第二个操作数）。iinc指令很适合实现循环结构。

    spin中的fou循环主要由以下指令来实现：

    ```
    5   iinc 1 1       // Increment local variable 1 by 1 (i++)
    8   iload_1        // Push local variable 1 (i)
    9   bipush 100     // Push int constant 100
    11  if_icmplt 5    // Compare and loop if less than (i < 100)
    ```

    bipush指令将100作为int值压入操作数栈，然后if_icmplt指令将操作数栈中的值弹出冰河和i进行比较。如果满足条件（变量i<100），将跳转到索引为5的位置，然后到for循环的开始处进行下一次迭代。否则将继续执行if_icmplt指令后面的指令。

    如果spin例子中的循环计数器使用了int职位的数据类型，那么编译后的代码也会随之改成相应的类型。例如，将spin例子int改成double：

    ```
    void dspin() {
        double i;
        for (i = 0.0; i < 100.0; i++) {
            ;    // Loop body is empty
        }
    }
    ```

    编译后的代码为：

    ```
    Method void dspin()
    0   dconst_0       // Push double constant 0.0
    1   dstore_1       // Store into local variables 1 and 2
    2   goto 9         // First time through don't increment
    5   dload_1        // Push local variables 1 and 2 
    6   dconst_1       // Push double constant 1.0 
    7   dadd           // Add; there is no dinc instruction
    8   dstore_1       // Store result in local variables 1 and 2
    9   dload_1        // Push local variables 1 and 2 
    10  ldc2_w #4      // Push double constant 100.0 
    13  dcmpg          // There is no if_dcmplt instruction
    14  iflt 5         // Compare and loop if less than (i < 100.0)
    17  return         // Return void when done
    ```

    现在指令操作的数据类型是专门针对double的（ldc2_w指令稍后会在本章讨论）。

    回想一下，double类型的值将占据两个局部变量，尽管只使用最小的索引值去访问这两个局部变量。这同样对longleix生效。再看一个例子：

    ```
    double doubleLocals(double d1, double d2) {
        return d1 + d2;
    }
    ```

    变成：

    ```
    Method double doubleLocals(double,double)
    0   dload_1       // First argument in local variables 1 and 2
    1   dload_3       // Second argument in local variables 3 and 4
    2   dadd
    3   dreturn
    ```

    注意局部变量表使用了一对变量来存储doubleLocals中的double值，这对变量绝不能单独操作。

    java虚拟机使用一字节大小的操作码的结果是编译后代码非常紧凑。但是一字节操作码也意味着java虚拟机的指令集非常小。作为折中，java虚拟机并不为每种数据类型提供相等的支持：他们并非完全正交的。

    例如，在spin的例子中使用了单独的if_icmplt指令来实现for语句中的int值的比较；然而，java虚拟机指令集中对于double类型并没有单独的指令来实现同样的效果。因此在dspin中比较double类型的值，必须在*iflt*指令之后使用*dcmpg*指令。

    java虚拟机对于int类型中的大多操作提供了直接支持。这在一定程度上是考虑到了java虚拟机操作数栈和局部变量表的实现效率。当然也考虑了大多数程序都会对int进行频繁操作的原因。对于其他的整型数据只有很少的直接支持。例如，没有byte, char和short版本的store，load和add指令。下面的例子使用short类型重写了spin：

    ```
    void sspin() {
        short i;
        for (i = 0; i < 100; i++) {
            ;    // Loop body is empty
        }
    }
    ```

    下面是为java虚拟机编译的代码，使用对另一种类型(很可能是int)进行操作的指令，在必要时在short和int值之间进行转换，以确保对short的操作结果保持在适当的范围内:

    ```
    Method void sspin()
    0   iconst_0
    1   istore_1
    2   goto 10
    5   iload_1        // The short is treated as though an int
    6   iconst_1
    7   iadd
    8   i2s            // Truncate int to short
    9   istore_1
    10  iload_1
    11  bipush 100
    13  if_icmplt 5
    16  return
    ```

    Java虚拟机中缺少对byte，char和short类型的直接支持并没有大的问题，因为这些类型的值在内部被提升为int（byte和short被符号扩展为int，char是零扩展）。 因此，可以使用int指令对字节，字符和短数据执行操作。 唯一的额外成本是将int操作的值截断为有效范围。

    Java虚拟机对于long和浮点类型（float和double）提供了中等程度的支持，仅缺少条件转移指令部分。

  - #### 算术运算

    java虚拟机通常在操作数栈上进行算术运算（例外情况是iinc指令，它直接增加一个局部变量的值）。例如下面的align2grain()方法，它的作用是将int值对齐到2的指定次幂：

    ```java
    int align2grain(int i, int grain) {
        return ((i + grain-1) & ~(grain-1));
    }
    ```

    算术运算的操作数是从操作数栈中弹出的，运算结果会压回操作数栈。因此，算术子计算的结果可以作为嵌套计算的操作数。例如。~(grain-1)的计算结果就是被这样使用的：

    ```
    5   iload_2        // Push grain
    6   iconst_1       // Push int constant 1
    7   isub           // Subtract; push result
    8   iconst_m1      // Push int constant -1
    9   ixor           // Do XOR; push result
    ```

    首先使用局部变量2的值和一个int类型的立即数1来计算grain-1的值。这些操作数被操作数栈弹出，然后他们的差值压回操作数栈，这个差值可以立即作为一个操作数被ixor指令使用（回想，~x == -1 ^ x）。同样的，ixor指令的计算结果成为接下来iand指令的一个操作数。

    这个方法的代码如下：

    ```
    Method int align2grain(int,int)
    0   iload_1
    1   iload_2
    2   iadd
    3   iconst_1
    4   isub
    5   iload_2
    6   iconst_1
    7   isub
    8   iconst_m1
    9   ixor
    10  iand
    11  ireturn
    ```

  - #### 访问运行时常量池

    许多数值类型常量，以及对象，字段和方法是通过当前类的运行时常量池来访问的。对象的访问在3.8节讨论。数据类型为int，long，float和double以及String类示例的引用通过ldc，ldc_w和ldc2_w来管理。

    ldc和ldc_w指令用于访问运行时常量池中除了double和long类型的值（但是包括String类型实例）。当使用的运行时常量池的项目过多时（多余256个，一个字节能表示的范围），需要使用ldc_w来代替。ldc2_w用于访问所有double和long类型的值，它没有对应的非宽版本，即没有ldc2指令。

    整型常量类型byte，char，short，以及小的int值，可能使用bipush，sipush或者iconst_<i>指令来编译。某些小的浮点型常量也可以使用fconst_<f>和dconst_<d>指令来编译。

    在所有这些情况下，编译都是非常直观的，例如，下面这些常量：

    ```java
    void useManyNumeric() {
        int i = 100;
        int j = 1000000;
        long l1 = 1;
        long l2 = 0xffffffff;
        double d = 2.2;
        ...do some calculations...
    }
    ```

    编译后：

    ```
    Method void useManyNumeric()
    0   bipush 100   // Push small int constant with bipush
    2   istore_1
    3   ldc #1       // Push large int constant (1000000) with ldc
    5   istore_2
    6   lconst_1     // A tiny long value uses small fast lconst_1
    7   lstore_3
    8   ldc2_w #6    // Push long 0xffffffff (that is, an int -1)
            // Any long constant value can be pushed with ldc2_w
    11  lstore 5
    13  ldc2_w #8    // Push double constant 2.200000
            // Uncommon double values are also pushed with ldc2_w
    16  dstore 7
    ...do those calculations...
    ```

  - #### 更多控制结构示例

    for语句的编译已经在前面的3.2节中展示过了。绝大多数java编程语言的其他控制结构(if-then-else,do,while,break和continue)也同样使用明细的方式来编译。switch语句的编译在单独的章节（3.10）介绍，异常的编译（3.12），finally从句的编译（3.13）。

    作为进一步的示例，while循环以一种明显的方式编译，尽管Java虚拟机提供的特定控制传输指令因数据类型的不同而不同。像往常一样，对int类型的数据有更多的支持，例如:

    ```
    void whileInt() {
        int i = 0;
        while (i < 100) {
            i++;
        }
    }
    ```

    编译为：

    ```
    Method void whileInt()
    0   iconst_0
    1   istore_1
    2   goto 8
    5   iinc 1 1
    8   iload_1
    9   bipush 100
    11  if_icmplt 5
    14  return
    ```

    注意，while语句的条件判断(使用if_icmplt指令实现)位于Java虚拟机代码循环的的底部。(在前面的spin例子中也是如此)。位于循环底部的条件判断强制使用goto指令，以便在循环的第一次迭代之前进行条件判断。如果不满足条件，并且循环体从未进入，那么这个额外的指令就被浪费了。不过while循环通常用于期望循环体会执行的场景中。对于后续的迭代，将判断条件放在循环的底部每次循环时都会节省一条Java虚拟机指令:如果判断条件位于循环的顶部，则循环体将需要一条尾随的goto指令才能回到顶部。

    使用其他数据类型的控制结构都使用相同的方式来编译，但是必须使用对应数据类型的指令。这会导致一些效率不高的代码，因为需要更多的java虚拟机指令，例如：

    ```java
    void whileDouble() {
        double i = 0.0;
        while (i < 100.1) {
            i++;
        }
    }
    ```

    编译成：

    ```
    Method void whileDouble()
    0   dconst_0
    1   dstore_1
    2   goto 9
    5   dload_1
    6   dconst_1
    7   dadd
    8   dstore_1
    9   dload_1
    10  ldc2_w #4      // Push double constant 100.1
    13  dcmpg          // To compare and branch we have to use...
    14  iflt 5         // ...two instructions
    17  return
    ```

    每一种浮点类型有两个比较指令：float的fcmpl和fcmpg，double的dcmpl和dcmpg。这些指令语义相似，仅仅在对待NaN变量时有所区别。NaN是无序的，所以只要有一个操作数是NaN，浮点指令的比较结果都会失败。无论比较操作是否会因为遇到NaN值而失败，编译器都会根据不用的操作类型来选择不同的比较指令。