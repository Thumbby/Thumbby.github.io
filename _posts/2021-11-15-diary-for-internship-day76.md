---
layout: post
title:  "实习日记76(JVM编译)"
date:   2021-11-15
categories: jekyll update
---

## Day76

- 继续学习Java虚拟机规范，基于https://docs.oracle.com/javase/specs/jvms/se8/html/

- ### JVM编译

  - #### 接收参数

    如果n个参数传给一个实例的方法，按照约定，它们被接受并放在这个新方法创建的栈帧中的局部变量表里，在局部变量表中的序号从1到n。这些参数按照它们传递过来的顺序存放。例如：

    ```java
    int addTwo(int i, int j) {
        return i + j;
    }
    ```

      编译为

    ```
    Method int addTwo(int,int)
    0   iload_1        // Push value of local variable 1 (i)
    1   iload_2        // Push value of local variable 2 (j)
    2   iadd           // Add; leave int result on operand stack
    3   ireturn        // Return int result
    ```

    按照约定，一个实例的引用要传递到当前实例方法的局部变量0。在java编程语言中这个实例可以通过关键词this来访问。

    类（static）方法不拥有实例，所以对他们来说没有必要使用局部变量0。类方法的参数是的从局部这些变量0开始的。如果addTwo是一个类方法，它的参数传递和前面类似：

    ```java
    static int addTwoStatic(int i, int j) {
        return i + j;
    }
    ```

    编译为

    ```
    Method int addTwoStatic(int,int)
    0   iload_0
    1   iload_1
    2   iadd
    3   ireturn
    ```

    与前面的唯一不同是参数从局部变量0开始而不是1。

  - #### 调用方法

    对普通实例方法调用是在运行时根据对象类型进行分派的（相当于在 C++中所说的“虚方法”）。这样的调用是通过invokevitual指令来实现的，该指令的参数是一个指向运行时常量池条码的索引，索引指向的内容给出了对象的类类型的二进制名称的内部形式、要调用的方法名称以及该方法的描述符（4.3.3）。

    为了调用之前定义的实例方法addTwo，我们可以这么写：

    ```
    int add12and13() {
        return addTwo(12, 13);
    }
    ```

    编译为

    ```
    Method int add12and13()
    0   aload_0             // Push local variable 0 (this)
    1   bipush 12           // Push int constant 12
    3   bipush 13           // Push int constant 13
    5   invokevirtual #4    // Method Example.addtwo(II)I
    8   ireturn             // Return int on top of operand stack;
                            // it is the int result of addTwo()
    ```

    方法调用首先需要将当前实例的引用（this）压入操作数栈。这个方法的调用参数，int值12和13，紧接着也被压入操作数栈。当addTwo方法的栈帧被创建时，这些传递到这个方法的参数成为新的栈帧中局部变量中的初始值。也就是说，被调用者压入操作数栈中的this引用和两个参数，将成为被调用方法中局部变量0,1,2的初始值。

    最终，addTwo方法被调用。当它返回时，它的int类型的返回值被压入调用者（add12and13方法）的栈帧的操作数栈中。然后它的返回值会被立即返回给add12and13的调用者。

    add12and13的返回是由ireturn指令来实现的。ireturn指令在当前栈帧的操作数栈中拿到addTwo的int类型返回值，然后压入调用者的栈帧的操作数栈中。ireturn然后将控制权返回给调用者，将调用者的栈帧变成当前帧。对于许多的数值类型和引用类型，java虚拟机提供了对应的返回指令，同样也包括没有返回值的return指令。所有的方法调用的所有变量都使用这一组返回指令。

    invokevirtual指令的操作数（在例子中是运行时常量池索引 #4）不是类实例中方法的偏移量。编译器不知道类实例的内部布局。作为替代，它产生符号引用来指向实例的方法，这些符号引用存储在运行时常量池中。那些运行时常量池中的元素在运行时被解析，从而确定实际的方法位置。其他java虚拟机指令访问类实例也采用相同的方式。

    调用addTwoStatic(一个类中的addTwo的静态变体)，也是类似的：

    ```java
    int add12and13() {
        return addTwoStatic(12, 13);
    }
    ```

    尽管使用了一个不同的java虚拟机方法调用指令：

    ```
    Method int add12and13()
    0   bipush 12
    2   bipush 13
    4   invokestatic #3     // Method Example.addTwoStatic(II)I
    7   ireturn
    ```

    编译一个类静态方法的调用和编译一个实例方法的调用是非常相似，除了this没有被调用者传递。因此将要接收到的方法参数从局部变量0开始。invokestatic指令总是用于调用类方法。

    invokespecial指令必须被用于调用实例的初始化方法。它同样用于调用父类的方法（super），以及调用private方法。例如，对于给定的类Near和Far定义如下：

    ```java
    class Near {
        int it;
        public int getItNear() {
            return getIt();
        }
        private int getIt() {
            return it;
        }
    }
    
    class Far extends Near {
        int getItFar() {
            return super.getItNear();
        }
    }
    ```

    Near.getItNear方法（调用了一个private方法），编译为：

    ```
    Method int getItNear()
    0   aload_0
    1   invokespecial #5    // Method Near.getIt()I
    4   ireturn
    ```

    Far.getItFar(调用了一个父类的方法)，编译为：

    ```
    Method int getItFar()
    0   aload_0
    1   invokespecial #4    // Method Near.getItNear()I
    4   ireturn
    ```

    注意，使用invokespecial指令调用方法总是传递this到被调用的方法作为它的第一个参数。照例使用局部变量0来接收。

    为了调用一个方法，编译器必须产生一个方法描述符，这个方法描述符中记录了实际的参数和返回类型。编译器在方法调用时不会处理参数的类型转换问题，只是简单的将参数压入操作数栈，且不改变其类型。通常，编译器会把方法所在的对象的引用压操作数栈，接着按顺序压入方法参数。编译器生成一个invokevitual指令，它引用了一个描述符，这个描述符描述了参数和返回类型。作为方法解析时的特殊处理过程，一个用于调用java.lang.invoke.MethodHandle的invoke或者invokeExact方法的invokevirtual指令会提供一个方法描述符，这个方法描述符符合语法规则，并且在描述符中确定的类型将会被解析。

  - 使用类实例

    java虚拟机通过new指令来创建java虚拟机类实例。回想一下，在Java虚拟机级别，构造函数作为方法出现，它使用所编译器提供的名称<init>。这个特殊名字的方法被称为实例初始化方法。多个实例初始化方法，对应于多个构造器，可能存在已一个给定的类中。一旦创建了类实例并且其实例变量（包括该类及其所有超类的实例变量）已初始化为其默认值，就会调用新类实例的实例初始化方法。 例如：

    ```java
    Object create() {
        return new Object();
    }
    ```

    编译为：

    ```
    Method java.lang.Object create()
    0   new #1              // Class java.lang.Object
    3   dup
    4   invokespecial #4    // Method java.lang.Object.<init>()V
    7   areturn
    ```

    类实例的传递和返回（通过引用类型）和数值类型非常相似，尽管引用类型有其完备的指令，例如：

    ```
    int i;                                  // An instance variable
    MyObj example() {
        MyObj o = new MyObj();
        return silly(o);
    }
    MyObj silly(MyObj o) {
        if (o != null) {
            return o;
        } else {
            return o;
        }
    }
    ```

    编译为：

    ```
    Method MyObj example()
    0   new #2              // Class MyObj
    3   dup
    4   invokespecial #5    // Method MyObj.<init>()V
    7   astore_1
    8   aload_0
    9   aload_1
    10  invokevirtual #4    // Method Example.silly(LMyObj;)LMyObj;
    13  areturn
    
    Method MyObj silly(MyObj)
    0   aload_1
    1   ifnull 6
    4   aload_1
    5   areturn
    6   aload_1
    7   areturn
    ```

    类实例的字段（实例变量）通过getfield和putfield指令来访问。假设i是一个int类型的实例变量，它的setIt和getIt定义如下：

    ```
    void setIt(int value) {
        i = value;
    }
    int getIt() {
        return i;
    }
    ```

    编译为：

    ```
    Method void setIt(int)
    0   aload_0
    1   iload_1
    2   putfield #4    // Field Example.i I
    5   return
    
    Method int getIt()
    0   aload_0
    1   getfield #4    // Field Example.i I
    4   ireturn
    ```

    和方法调用指令的操作数一样，putfield和getfield指令的操作数（运行时常量池索引#4）不是类实例字段的偏移量。编译器产生这个实例字段的符号引用，存储在运行时常量池。这些运行时常量池中的元素在运行时被解析从而决定引用对象的字段的位置。

  - #### 数组

    java虚拟机的数组同样是对象。数组的创建和操作使用不同的指令集。newarray指令用于创建一个数值类型的数组。代码如下：

    ```java
    void createBuffer() {
        int buffer[];
        int bufsz = 100;
        int value = 12;
        buffer = new int[bufsz];
        buffer[10] = value;
        value = buffer[11];
    }
    ```

    会编译成：

    ```
    Method void createBuffer()
    0   bipush 100     // Push int constant 100 (bufsz)
    2   istore_2       // Store bufsz in local variable 2
    3   bipush 12      // Push int constant 12 (value)
    5   istore_3       // Store value in local variable 3
    6   iload_2        // Push bufsz...
    7   newarray int   // ...and create new int array of that length
    9   astore_1       // Store new array in buffer
    10  aload_1        // Push buffer
    11  bipush 10      // Push int constant 10
    13  iload_3        // Push value
    14  iastore        // Store value at buffer[10]
    15  aload_1        // Push buffer
    16  bipush 11      // Push int constant 11
    18  iaload         // Push value at buffer[11]...
    19  istore_3       // ...and store it in value
    20  return
    ```

    anewarray指令用于创建一维对象引用数组，例如：

    ```java
    void createThreadArray() {
        Thread threads[];
        int count = 10;
        threads = new Thread[count];
        threads[0] = new Thread();
    }
    ```

    编译成：

    ```
    Method void createThreadArray()
    0   bipush 10           // Push int constant 10
    2   istore_2            // Initialize count to that
    3   iload_2             // Push count, used by anewarray
    4   anewarray class #1  // Create new array of class Thread
    7   astore_1            // Store new array in threads
    8   aload_1             // Push value of threads
    9   iconst_0            // Push int constant 0
    10  new #1              // Create instance of class Thread
    13  dup                 // Make duplicate reference...
    14  invokespecial #5    // ...for Thread's constructor
                            // Method java.lang.Thread.<init>()V
    17  aastore             // Store new Thread in array at 0
    18  return
    ```

    anewarray指令同样可以用于创建多维数组的第一维。multianewarray指令可以用于一次创建多维数组。例如，三维数组：

    ```java
    int[][][] create3DArray() {
        int grid[][][];
        grid = new int[10][5][];
        return grid;
    }
    ```

    会被这样创建：

    ```
    Method int create3DArray()[][][]
    0   bipush 10                // Push int 10 (dimension one)
    2   iconst_5                 // Push int 5 (dimension two)
    3   multianewarray #1 dim #2 // Class [[[I, a three-dimensional
                                 // int array; only create the
                                 // first two dimensions
    7   astore_1                 // Store new array...
    8   aload_1                  // ...then prepare to return it
    9   areturn
    ```

    multianewarray指令的第一个操作数是运行时常量池中的索引，这个索引指向被创建的数组类型。第二个操作数表示实际创建出来的数据具有的维数。multianewarray指令可用于创建该类型的所有维度，例如create3DArray代码中的那样。注意多维数组也仅仅是一个对象，所以加载和返回使用了aload_1指令和areturn指令。更多关数组类名的信息，参考4.4.1.

    所有的数组都有长度，可以通过arraylength指令来过去。

  - #### 编译Switches

    switch语句的编译使用tableswitch和lookupswitch指令。当switch的case可以有效地表示为目标偏移表中的索引时，使用tableswitch指令。如果switch表达式的值超出了有效索引的返回，则使用default的值。例如：

    ```
    int chooseNear(int i) {
        switch (i) {
            case 0:  return  0;
            case 1:  return  1;
            case 2:  return  2;
            default: return -1;
        }
    }
    ```

    编译为：

    ```
    Method int chooseNear(int)
    0   iload_1             // Push local variable 1 (argument i)
    1   tableswitch 0 to 2: // Valid indices are 0 through 2
          0: 28             // If i is 0, continue at 28
          1: 30             // If i is 1, continue at 30
          2: 32             // If i is 2, continue at 32
          default:34        // Otherwise, continue at 34
    28  iconst_0            // i was 0; push int constant 0...
    29  ireturn             // ...and return it
    30  iconst_1            // i was 1; push int constant 1...
    31  ireturn             // ...and return it
    32  iconst_2            // i was 2; push int constant 2...
    33  ireturn             // ...and return it
    34  iconst_m1           // otherwise push int constant -1...
    35  ireturn             // ...and return it
    ```

    java虚拟机的tableswtich和lookupswitch指令只操作int类型的数据。因为对于byte，char或者short的操作会内部提升到int，switch表达式中使用了这些类型将会编译为int类型。如果chooseNear方法使用short类型来写，将会使用int类型来生成相同的java虚拟机指令。使用switch时，其他的数值类型必须窄化为int类型。

    如果switch的case比较分散，使用tableswitch指令来表示将会在空间上低效。可以使用lookupswitch指令来代替。lookupswitch指令将int键（case标签的值）与表中的目标偏移量配对。当lookupswitch指令运行时，switch表达式的值将会和表中的key进行比较。如果其中一个key和switch表达式的值匹配上，将从其关联的目标便宜位置继续执行。如果没有匹配上，将从default处继续执行。例如，下面的代码：

    ```
    int chooseFar(int i) {
        switch (i) {
            case -100: return -1;
            case 0:    return  0;
            case 100:  return  1;
            default:   return -1;
        }
    }
    ```

     看起来和chooseNear很像，除了lookupswitch指令：

    ```
    Method int chooseFar(int)
    0   iload_1
    1   lookupswitch 3:
             -100: 36
                0: 38
              100: 40
          default: 42
    36  iconst_m1
    37  ireturn
    38  iconst_0
    39  ireturn
    40  iconst_1
    41  ireturn
    42  iconst_m1
    43  ireturn
    ```

    Java虚拟机指定lookupswitch指令的表必须按键排序，以便实现可以使用比线性扫描更高效的搜索。即便如此，lookupswitch指令必须搜索其键以进行匹配，而不是简单地执行边界检查并索引到像tableswitch这样的表。因此，当空间条件允许时，tableswitch指令可能比lookupswitch指令更高效。

  - #### 操作数栈上的操作

    java虚拟机拥有大量的指令可以将操作数栈上的内容当做无类型数据进行操作。这是非常有用的，因为java虚拟机依赖于其对操作数栈的灵活操作。例如：

    ```java
    public long nextIndex() { 
        return index++;
    }
    
    private long index = 0;
    ```

    编译为

    ```
    Method long nextIndex()
    0   aload_0        // Push this
    1   dup            // Make a copy of it
    2   getfield #4    // One of the copies of this is consumed
                       // pushing long field index,
                       // above the original this
    5   dup2_x1        // The long on top of the operand stack is 
                       // inserted into the operand stack below the 
                       // original this
    6   lconst_1       // Push long constant 1 
    7   ladd           // The index value is incremented...
    8   putfield #4    // ...and the result stored in the field
    11  lreturn        // The original value of index is on top of
                       // the operand stack, ready to be returned
    ```

  - #### 抛出和处理异常

    在程序中使用throw关键字来抛出异常。编译结果很简单。

    ```java
    void cantBeZero(int i) throws TestExc {
        if (i == 0) {
            throw new TestExc();
        }
    }
    ```

    编译为

    ```
    Method void cantBeZero(int)
    0   iload_1             // Push argument 1 (i)
    1   ifne 12             // If i==0, allocate instance and throw
    4   new #1              // Create instance of TestExc
    7   dup                 // One reference goes to its constructor
    8   invokespecial #7    // Method TestExc.<init>()V
    11  athrow              // Second reference is thrown
    12  return              // Never get here if we threw TestExc
    ```

    使用try-catch构建的编译时直观的，例如：

    ```java
    void catchOne() {
        try {
            tryItOut();
        } catch (TestExc e) {
            handleExc(e);
        }
    }
    ```

    编译为

    ```
    Method void catchOne()
    0   aload_0             // Beginning of try block
    1   invokevirtual #6    // Method Example.tryItOut()V
    4   return              // End of try block; normal return
    5   astore_1            // Store thrown value in local var 1
    6   aload_0             // Push this
    7   aload_1             // Push thrown value
    8   invokevirtual #5    // Invoke handler method: 
                            // Example.handleExc(LTestExc;)V
    11  return              // Return after handling TestExc
    Exception table:
    From    To      Target      Type
    0       4       5           Class TestExc
    ```

    自己观察，try代码块编译后，就和try不存在一样：

    ```
    Method void catchOne()
    0   aload_0             // Beginning of try block
    1   invokevirtual #6    // Method Example.tryItOut()V
    4   return              // End of try block; normal return
    ```

    在执行try代码块的时候，如果没有异常抛出，就和try不存在一样：tryItOut被调用然后catchOne返回。

    在try块后面是实现单个catch语句的Java虚拟机代码。

    ```
    5   astore_1            // Store thrown value in local var 1
    6   aload_0             // Push this
    7   aload_1             // Push thrown value
    8   invokevirtual #5    // Invoke handler method: 
                            // Example.handleExc(LTestExc;)V
    11  return              // Return after handling TestExc
    Exception table:
    From    To      Target      Type
    0       4       5           Class TestExc
    ```

    catch语句中的的内容——调用handlwExc，同样被编译为一个普通的方法调用。但是，catch语句的存在导致编译器生成一个异常表条目。catchOne方法的异常表有一个条目对应于catchOne的catch子句可以处理的一个参数（类TestExc的实例）。如果TestExc的某一个实例在指令执行catchOne的索引0到4之间抛出了，那么控制将转移到代码所有为5的位置，也就是实现了catch语句的代码块。如果抛出的异常不是TestExc的实例，catchOne的catch语句无法处理它。相应的，异常会抛给catchOne的调用者。

    try可以有多个catch语句：

    ```java
    void catchTwo() {
        try {
            tryItOut();
        } catch (TestExc1 e) {
            handleExc(e);
        } catch (TestExc2 e) {
            handleExc(e);
        }
    }
    ```

    通过简单地为每个catch子句依次附加Java虚拟机代码并将条目添加到异常表中来编译给定try语句的多个catch子句，如下所示：

    ```
    Method void catchTwo()
    0   aload_0             // Begin try block
    1   invokevirtual #5    // Method Example.tryItOut()V
    4   return              // End of try block; normal return
    5   astore_1            // Beginning of handler for TestExc1;
                            // Store thrown value in local var 1
    6   aload_0             // Push this
    7   aload_1             // Push thrown value
    8   invokevirtual #7    // Invoke handler method:
                            // Example.handleExc(LTestExc1;)V
    11  return              // Return after handling TestExc1
    12  astore_1            // Beginning of handler for TestExc2;
                            // Store thrown value in local var 1
    13  aload_0             // Push this
    14  aload_1             // Push thrown value
    15  invokevirtual #7    // Invoke handler method:
                            // Example.handleExc(LTestExc2;)V
    18  return              // Return after handling TestExc2
    Exception table:
    From    To      Target      Type
    0       4       5           Class TestExc1
    0       4       12          Class TestExc2
    ```

    如果在执行try子句期间（在索引0和4之间）抛出一个值，该值与一个或多个catch子句的参数匹配（值是一个或多个参数的实例），则选择第一个匹配的catch语句。 控制转移到该catch语句的块的Java虚拟机代码。 如果抛出的值与catchTwo的任何catch子句的参数不匹配，则Java虚拟机将重新抛出该值，而不调用catchTwo的任何catch语句中的代码。

    嵌套的try-catch语句编译结果和try语句跟随多个catch是非常相似的：

    ```java
    void nestedCatch() {
        try {
            try {
                tryItOut();
            } catch (TestExc1 e) {
                handleExc1(e);
            }
        } catch (TestExc2 e) {
            handleExc2(e);
        }
    }
    ```

    编译为：

    ```
    Method void nestedCatch()
    0   aload_0             // Begin try block
    1   invokevirtual #8    // Method Example.tryItOut()V
    4   return              // End of try block; normal return
    5   astore_1            // Beginning of handler for TestExc1;
                            // Store thrown value in local var 1
    6   aload_0             // Push this
    7   aload_1             // Push thrown value
    8   invokevirtual #7    // Invoke handler method: 
                            // Example.handleExc1(LTestExc1;)V
    11  return              // Return after handling TestExc1
    12  astore_1            // Beginning of handler for TestExc2;
                            // Store thrown value in local var 1
    13  aload_0             // Push this
    14  aload_1             // Push thrown value
    15  invokevirtual #6    // Invoke handler method:
                            // Example.handleExc2(LTestExc2;)V
    18  return              // Return after handling TestExc2
    Exception table:
    From    To      Target      Type
    0       4       5           Class TestExc1
    0       12      12          Class TestExc2
    ```

    catch语句的嵌套仅表现在异常表中，java虚拟机本身不要求异常表条目的顺序。但是，因为try-catch构造是结构化的，所以编译器总是可以对异常处理程序表的条目进行排序，这样，对于任何抛出的异常，都会被最接近异常位置的、可处理该异常的catch语句块所处理。

    例如，如果tryItOut（在索引1处）的调用抛出TestExc1的实例，则它将由调用handleExc1的catch语句处理。 即使异常发生在外部catch语句能够处理的范围之内（捕获TestExc2的catch语句），并且外部的catch能够处理这个异常，也不会分配给外部的catch处理，因为异常表的顺序。

    作为一个微妙的点，请注意catch子句的范围包含在“from”端，不包括“to”端（第4.7.3节）。 因此，捕获TestExc1的catch子句的异常表条目不包括偏移量为4的return指令。但是，捕获TestExc2的catch子句的异常表条目确实覆盖了偏移量11处的返回指令。因此，如果嵌套catch语句的return指令抛出异常，将由外出的catch语句处理。

