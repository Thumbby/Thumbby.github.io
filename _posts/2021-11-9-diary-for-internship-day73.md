---
layout: post
title:  "实习日记73(JVM结构)"
date:   2021-11-09
categories: jekyll update
---

## Day73

- 学习JVM虚拟机规范-Java8，基于https://docs.oracle.com/javase/specs/jvms/se8/html/index.html

- ### Java虚拟机结构

  - java执行的编译代码使用一个独立于硬件和操作系统的二进制格式，通常(不是必须)存在于class文件中。class文件格式定义了类和接口的表示，如byte ordering的细节。

  - java虚拟机操作两种数据类型：

    - 基本类型
    - 引用类型

    java虚拟机有两种类型的数据可用于变量赋值、参数传递和方法返回：

    - 基本值
    - 引用值

    java虚拟机希望几乎所有类型的数据检查在运行前通常由编译器完成。基本类型的值不需要特殊标记或者特殊的方法在运行时确定他们的类型，也不需要它们和引用类型区分开。而JVM的指令集对不同的操作数使用不同的操作指令来区分操作数类型，例如iadd、ladd指明了操作数分别是int和long

    JVM包含对对象的显示支持。对象是动态分配的类实例或数组。一个对象的引用可以认为是Java虚拟机的引用（reference）类型。引用（reference）的值可以被认为是指向对象的指针。可能存在多个对象的引用。对象始终通过引用（reference）类型的值来进行的值的操作，传递和检查。

  - #### 基本数据类型和值

    - ##### 整数类型：

      - byte，其值为8位有符号二进制补码整数，其默认值为零 (-2^7 to 2^7 - 1,inclusive)
      - short，其值为16位有符号二进制补码整数，其默认值为零 (-2^15 to 2^15 - 1,inclusive)
      - int，其值为32位有符号二进制补码整数，其默认值为零 (-2^31 to 2^31 - 1,inclusive)
      - long，其值为64位有符号二进制补码整数，其默认值为零(-2^63 to 2^63 - 1,inclusive)
      - char，其值为16位无符号整数，表示基本多文本平面（Basic Multilingual Plane）中的Unicode代码点，使用UTF-16编码，其默认值为空代码点（'\ u0000'）（0 ~ 2^16-1）

    - ##### 浮点数类型：

      - fload，值为单精度浮点数集中的元素，或者（如果虚拟机支持的话）是单精度扩展指数（Float-Extended-Exponent）集合中的元素。默认值为正数零。
      - double，取值范围是双精度浮点数集合中的元素，或者（如果虚拟机支持的话）是双精度扩展指数（Double-Extended-Exponent）集合中的元素。默认值为正数零。

      整数类型和浮点数类型统称数字类型

    - ##### 布尔类型：取值范围是true和false，默认值是false

    - ##### returnAddress类型

      值是指向Java虚拟机指令的操作码（opcodes）的指针。在基本类型中，除了returnAddress类型，其它类型都与Java编程语言类型直接相关联。

  - #### 浮点数类型和值

    任何有限非零浮点数可以表示为 *s* ⋅ *m* ⋅ 2^(*e* − *N* + 1)，其中s为+1或者-1，m为小于2N的正整数，e为一个介于*Emin，* −(2^(K−1)−2)和 *Emax，* 2^(K−1)−1之间的一个整数，N和K的取值范围取决于当前的浮点数值集合。部分浮点数使用这种规则得到的表示形式可能不是唯一的，例如在指定的数值集合内，可以存在一个数字 v，它能找到特定的 s、m 和 e 值来表示，使得其中 m 是偶数，并且 e 小于  2^(K-1)，这样我们就能够通过把 m 的值减半再将 e 的值增加 1 来的方式得到 v 的另外一种不同的表示形式。在这些表示形式中，如果其中某种表示形式中 m 的值满足条件 *m* ≥ 2*N*-1的话，那就称这种表示为标准表示（Normalized Representation），不满足这个条件的其他表示形式就称为非标准表示（Denormalized Representation）。如果某个数值不存在任何满足 *m* ≥ 2^(*N*-1)的表示形式，即不存在任何标准表示，那就称这个数字为非标准值（Denormalized Value）

    | 参数 | float | 单精度扩展指数集合 | double | 双精度扩展指数集合 |
    | ---- | ----- | ------------------ | ------ | ------------------ |
    | N    | 24    | 24                 | 53     | 53                 |
    | K    | 8     | ≥ 11               | 11     | ≥ 15               |
    | Emax | +127  | ≥ 1023             | +1023  | ≥ 16383            |
    | Emin | -127  | ≤ -1022            | -1022  | ≤ -16382           |

    当虚拟机实现一个或者都实现的扩展指数集，这里有一个和实现无关的限制参数K，见上表中的具体限制。参数K同时决定了*Emin*和*Emax*的范围。

    单精度浮点数集中的元素可以精确的表示为IEEE 754标准中的单精度浮点格式，除了NaN（IEEE 754中描述了2^24-2中不同的NaN值）。双精度浮点数集中的元素可以精确的表示为IEEE 754标准中的双精度浮点格式，除了N   aN（IEEE 754中描述了2^53-2中不同的NaN值）。注意，这里定义的单精度扩展指数和双精度扩展指数和IEEE 754中对应的单精度扩展和双精度扩展并不对应。不过除了 Class 文件格式中必要的浮点数表示描述以外，本规范并不特别要求表示浮点数值表示形式。　

    除了 NaN 以外，浮点数集合中的所有元素都是有序的。如果把它们从小到大按顺序排列好，那顺序将会是：负无穷，可数负数、正负零、可数正数、正无穷。
    浮点数中，正数零和负数零是相等的，但是它们有一些操作会有区别。例如 1.0 除以 0.0 会产生正无穷大的结果，而 1.0 除以-0.0 则会产生负无穷大的结果。
    NaN 是无序的，对它进行任何的数值比较和等值测试都会返回 false 的比较结果。值得一提的是，有且只有 NaN 一个数与自身比较是否数值上相等时会得到 false 的比较结果，任何数字与NaN 进行非等值比较都会返回 true。

  - #### returnAddress类型和值

    returnAddress类型被java虚拟机使用在jsr，ret和jsr_w指令中。returnAddress的值是java虚拟机指令操作码的指针。和基本的数字类型不同，returnAddress在java编程语言中没有对应的类型，同时在运行的程序中无法被修改。
    
  - #### 布尔类型

    尽管java虚拟机定义了boolean类型，但是仅仅提供了非常有限的支持。java虚拟机没有单独专门的指令来操作布尔值。相反，java编程语言中的表达式涉及到boolean类型的值会被编译为java虚拟机中的int类型。

    java虚拟机直接支持布尔数组。java虚拟机中的newarray指令允许创建boolean类型的数组。boolean类型数组的元素通过byte数组指令baload和bastore来访问和修改。

    java虚拟机将boolean数组元素编码成1和0分别表示true和false。java编程语言中boolean值会被编译器映射为java虚拟机中类型int，编译器必须使用相同的编码方式。

  - #### 引用类型和值

    总共又三种类型的引用类型(reference)：类类型（class types），数组类型（array types）以及接口类型（interface types），它们的值分别表示动态创建的类实例，数组和类实例或者数组实现的接口。

    数组类型由具有单个维度的组件类型（*component type*）组成（其长度不是由类型给出的）。数组的组件类型本身也可以是数组类型。但从任意一个数组开始，如果发现其组件类型也是数组类型的话，继续重复取这个数组的组件类型，这样操作不断执行，最终一定可以遇到组件类型不是数组的情况，这时就把这种类型成为数组类型的元素类型（Element Type）。数组的元素类型必须是原始类型、类类型或者接口类型之中的一种。

    引用值也可以是特殊的空引用，对无对象的引用，这里将用null表示。null引用本质上没有任何的运行时类型，但是可以转变为任意类型。引用类型的默认值时null。

    本规范没有强制将null编程成一个具体的值。

  - #### 运行时数据区域

    - **pc寄存器**：java虚拟机支持多个线程同时执行。每个线程拥有自己的pc(program counter)寄存器。在任何时刻，一个线程只会执行一个方法，即线程的当前方法。如果执行的方法不是本地（native）方法，pc寄存器包含当前正在执行的虚拟机指令的地址。如果当前线程执行的是本地方法，虚拟机的pc寄存器中的值是未定义的。Java虚拟机的pc寄存器足够保存一个一个特定平台上的returnAddress或本地指针。

    - **java虚拟机栈**

      每个线程都拥有一个私有的java虚拟机栈，和线程同时被创建。java虚拟机栈保存的栈是帧（frames）。java虚拟机栈和传统语言（比如c）的栈类似：它保存局部变量和部分结果，并在方法调用和返回中起作用。由于除了栈帧的入栈和出栈，Java虚拟机栈永远不会被直接操作，因此栈帧可以在堆中进行分配。Java虚拟机栈的内存不需要是连续的。

      本规范允许Java虚拟机堆栈具有固定大小或根据计算的需要动态扩展和收缩。如果Java虚拟机栈具有固定大小，则可以在创建该栈时独立选择每个Java虚拟机栈的大小。

      Java虚拟机的一个具体实现可以提供程序员或用户更改Java虚拟机栈初始大小的手段，以及在动态扩展或收缩Java虚拟机栈的情况下，控制最大和最小容量。

      虚拟机栈可能出现以下异常：

      - 如果一个线程需要的虚拟机栈大小超过允许的最大容量时，虚拟机栈将抛出stackOverflowError。
      - 如果java虚拟机栈可以动态扩展，并且当进行扩展时没有申请到足够的内存完成扩展，或者没有足够的内存为新线程创建虚拟机栈，则Java 虚拟机抛出OutOfMemoryError。

    - **java堆**

      Java虚拟机堆被所有的线程共享。java堆是运行时数据区域，用来分配类实例和数组的内存区域。

      java堆在虚拟机启动时创建。对象的堆存储由自动存储管理系统（称为垃圾收集器）回收;对象永远不会被显式释放。java虚拟机并不假定任何特定的自动存储管理系统类型，存储管理技术会根据实现者的系统需求来选定。堆可以是固定大小，或者根据需要扩展和压缩不需要的容量。堆使用的内存不需要时连续的。

      java堆和虚拟机栈类似，一个具体的实现可以提供开发者或者用户去控制堆的初始大小，也可以去控制一个动态扩展堆的最大和最小容量。

      下面的异常条件和堆有关：

      **如果计算需要堆的容量超过了自动存储管理系统能给的最大容量，java虚拟机则抛出OutOfMermoryError。**

    - **方法区**

      java虚拟机的方法区（method area）被所有Java虚拟机线程共享。方法区类似于传统语言的编译代码的存储区或类似于操作系统进程中的“文本”段。它存储每个类的结构，例如运行时常量池，字段和方法数据，以及方法和构造函数的代码，包括类和实例初始化以及接口初始化中使用的特殊方法。

      方法区在虚拟机启动时创建。尽管方法区在逻辑上是堆的一部分，但是简单的实现可以选择不进行垃圾回收和压缩。本规范未规定方法区域的位置或用于管理编译代码的策略。方法区可以是固定大小，或者根据需要扩展和压缩不需要的容量。堆使用的内存不需要时连续的。

      java虚拟机实现可以提供开发者或者用户去控制方法区的初始大小，也可以去控制一个动态扩展方法区的最大和最小容量。

      下面的异常条件和方法区有关：

      **如果方法区的没有可用的内存去满足一次内存分配请求，java虚拟机会抛出OutOfMermoryError。**

    - **运行时常量池**

      运行时常量池（*run-time constant pool*）是每一个类或者接口对应的class文件中constant_pool表的运行时表示形式。它包含几种常量，从编译时已知的数字的字面量到必须在运行时解析的方法和字段引用。运行时常量池提供类似于传统编程语言的符号表的功能，只是它包含比典型符号表范围更广的数据。

      每个运行时常量池都是分配在java虚拟机的方法区。当Java虚拟机加载类或接口时，将构造类或接口的运行时常量池。

      下面的异常条件和构建一个类或接口的运行时常量池有关：

      **当加载一个类或者接口时，如果构建运行时常量池需要的内存超过java虚拟机方法区能够提供的最大容量，虚拟机将抛出OutOfMermoryError。**

    - **本地方法栈**

      Java虚拟机的实现可以使用传统的栈，俗称“C stacks”，以支持本地方法（native methods）（用Java编程语言以外的语言编写的方法）。当Java虚拟机使用其他语言如C语言来实现java虚拟机指令集的解释器时，也可能用到本地方法栈。如果java虚拟机实现不能加载本地方法，它们自己也不需要依赖传统的栈，则没必要提供本地方法栈。如果支持的话，本地方法栈通常在线程创建的时候按照每个线程分配。

      本规范允许本地方法栈具有固定大小或根据计算的需要动态扩展和收缩。如果本地方法栈具有固定大小，则可以在创建该栈时独立选择每个本地方法栈的大小。

      java虚拟机实现可以提供开发者或者用户去控制本地方法栈的初始大小，也可以去控制一个支持动态扩展的本地方法栈的最大和最小容量。

      下面的异常条件和本地方法栈有关：

      - 如果一个线程需要的本地方法栈的容量超过允许的最大容量，那么虚拟机抛出`StackOverflowError。`
      - 如果本地方法栈支持动态扩展，当尝试扩展时没有足够的容量可用，或者没有足够的容量去为一个新线程创建一个初始的本地方法栈，java虚拟机抛出`OutOfMemoryError。`

  - #### 栈帧(Frames)

    栈帧用于存储数据和部分结果，同样也用于执行动态链接，返回方法的值和分派异常。

    当方法被调用的时候会创建一个新的栈帧。当一个方法调用结束时，它对应的栈帧就被销毁了，不管是正常调用结束还是意外结束（抛出了未被捕获的异常）。栈帧分配在线程创建的虚拟机栈中。每个栈帧都有自己的局部变量表，操作数栈，以及当前方法的类的运行时常量池的引用。

    可以使用附加的特定于实现的信息来扩展帧，例如调试信息

    局部变量表和操作数栈在编译时期就确定了，并且通过栈帧关联的方法的code提供。因此栈帧的大小仅仅取决于虚拟机的实现，和方法调用时这些结构所分配的内存。

    在一个给定的线程上，任何时刻只有正在执行的方法的栈帧时活动的。这个帧被称之为当前帧，它的方法就是当前方法，对应的类就是当前类。局部变量和操作数堆栈的操作通常是指当前帧上的操作。

    一个帧的方法调用另外一个方法，或者这个帧的方法结束了，那么这个帧不再是当前帧了。当一个方法被调用，一个新的栈帧就会被创建，当控制权转移到这个新方法时，这个帧就变成当前帧。在方法返回时，当前帧将其方法调用的结果（如果有）传递回前一帧，然后当前一帧成为当前帧时丢弃当前帧。

    注意线程创建的栈帧是属于这个线程的，不可能被别的线程引用。

    - ##### 局部变量表

      每个栈帧都包含一组变量，称之为它的局部变量表。帧的局部变量数组的长度在编译时确定，并以类或接口的二进制表示形式提供，存储在帧相关的方法的code属性中。

      一个局部变量可以保存boolean，byte，char，short，int，float，reference或者returnAddress的值。一对局部变量可以保存long或者double的值。

      局部变量表根据索引定位。第一个局部变量的索引是0。如果一个整数大于等于0并且小于局部变量表的长度，就是这个局部变量表的索引值。

      long或者double的值占据两个连续的局部变量，这样的值只用两个变量中最小的索引值来定位。例如，double的值存在局部变量表的索引n处，那么实际是它也占据了n+1的局部变量位置，但是无法根据n+1的索引来加载这个变量。索引值为n+1的变量可以被写入，但是，这样做会使局部变量n的内容无效。

      上文中提及的局部变量n的n值并不要求一定是偶数，Java虚拟机也不要求double和long类型数据采用 64 位对齐的方式存放在连续的局部变量中。虚拟机实现者可以自由地选择适当的方式，通过两个局部变量来存储一个 double 或 long 类型的值。

      java虚拟机使用局部变量表在方法调用时传递参数。当类的方法被调用时，所有参数都被传递到索引从0开始的连续的局部变量表中。当一个实例的方法调用时，索引为0的局部变量总是被用来传递这个实例对象的引用（java编程语言中this）。后续其他参数将会传递到从1开始的连续的局部变量表中。

    - ##### 操作数栈

      每个栈帧都有一个后进先出(LIFO)的栈，即操作数栈。一个栈帧的操作数栈的最大深度在编译器确定，由这个帧所在的方法的code属性提供。

      当上下文清晰的适合，我们有时会把当前帧的操作数栈简称操作数栈。

      栈帧刚创建时，他的操作数栈时空的。java虚拟机提供一系列指令从局部变量表或者字段中去加载常量或者变量到操作数栈中。其他的一些虚拟机指令从操作数栈中取出操作数，对它们进行操作，然后将结果推入到操作数栈中。操作数栈也被用户准备参数参数传递给方法和接受方法的结果。

      例如，iadd指令用户将两个int数相加。它需要操作数栈顶部的两个int数字进行相加，这两个数字需要被之前的指令提前入栈。两个int值从操作数栈中出栈，然后它们相加，将他们之和入栈到操作数栈。子计算可以操作数栈中嵌套进行，它们的结果能够被外围的计算使用。

      每个入栈的成员都能保存任何一个虚拟机类型，包括long类型和double类型。

      必须使用符合它们类型的操作去操作操作数栈中的值。例如，不可能入栈两个int值，然后将它们当成long类型来操作，或者入栈两个两个float值，接着使用iadd命令来操作。只有一小部分虚拟机指令（如dub和swap指令）操作运行时数据区域时，将其中的值当作原始值，而不用考虑它们的特殊类型；这些指令不能用于修改或者分解单个值。这些操作数栈上面的操作限制时用class文件验证来强制保证的。

      任何时刻，一个操作数栈都有确定的深度，long或者double占用两个单元的深度，其他类型只会占用一个类型的深度。

    - ##### 动态连接

      每一个帧都包含一个运行时常量池的引用，用于当前方法支持方法代码的动态链接（dynamic linking）。一个方法在class文件中的code，描述一个方法调用和变量访问时通过符号引用（symbolic references）。动态链接将这些方法的符号引用转换为具体的方法引用，根据需要加载类以解析尚未定义的符号，并将变量访问转换为与这些变量的运行时位置相关的存储结构中的适当偏移。

      方法和变量的这种后期绑定使得，当前方法使用了其他类中的方法或者变量发生变化时，不太可能破坏当前方法的代码。

    - ##### 正常方法调用完成

      一个方法调用正常完成是指，这个调用没有导致异常出现，既不是java虚拟机直接抛出也不是执行了一个具体的throw语句。如果当前方法的调用正常完成，然后可能返回一个值给调用者。当调用的方法执行了某一种return指令，才会出现返回值，return指令必须选择符合返回值类型的指令。

      在这种情况下，当前帧的目的是用于恢复调用者的状态，包括其局部变量和操作数堆栈；然后调用者的程序计数器适当地递增以跳过方法调用指令。然后在调用者的方法的帧中继续正常执行，返回值（如果有）被推送到该帧的操作数堆栈。

    - ##### 异常方法调用完成

      一个方法异常调用结束是指，jiava虚拟机指令执行该方法时导致了虚拟机抛出一个异常，并且这个异常没有在该方法中处理。执行athrow指令同样会导致一个异常被显示的抛出，如果当前方法没有捕获这个异常，该方法调用会异常完成。方法调用异常结束不会返回值给它的调用者。
