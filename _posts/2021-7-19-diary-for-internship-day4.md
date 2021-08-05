---
layout: post
title:  "实习日记4(JPA+MybatisPlus+lombok+编程规范)"
date:   2021-07-19
categories: jekyll update
---

## Day4

- 源码学习：
  - JPA相关：

    - 常见注解：

      - @Id：声明属性为主键

      - @GeneratedValue：表示逐渐是自动生成策略，一般和@Id一起使用

      - @Entity：任何hibernate映射对象需要该注解，将一个类声明为一个实体Bean

      - @Table(name="tablename")：声明此对象映射到哪个表，为实体bean映射指定表，name属性表示实体所对应表的名称，如果没有定义 @Table，那么系统自动使用默认值：实体的类名（不带包名）

      - @Column(name="Name",nullable=false,length=32)：声明数据库字段和类属性对应关系，如果属性名与表中的字段名相同，则可以省略@Column注解，另外有两种方式标记，一是放在属性前，另一种是放在getter方法前

      - @Lob：声明字段为Clob或Blob类型

        - Clob：存储大量的文本数据，大字段的操作通常以流的方式处理

        相关类型如下：

        | 类型       | 最大大小             |
        | ---------- | -------------------- |
        | TinyText   | 255字节              |
        | Text       | 65535字节(约65K)     |
        | MediumText | 16777215字节(约16M)  |
        | LongText   | 4294967295字节(约4G) |

        - Blob：存储二进制数据，通常为图片或音频

        相关类型如下：

        | 类型       | 最大大小             |
        | ---------- | -------------------- |
        | TinyBlob   | 255字节              |
        | Blob       | 65535字节(约65K)     |
        | MediumBlob | 16777215字节(约16M)  |
        | LongBlob   | 4294967295字节(约4G) |

      - @OneToMany(mappedBy=”order”,cascade = CascadeType.ALL, fetch = FetchType.LAZY)
        @OrderBy(value = “id ASC”)：一对多声明

      - @ManyToOne(cascade=CascadeType.REFRESH,optional=false)
        @JoinColumn(name = “order_id”)：声明为双向关联

      - @Temporal(value=TemporalType.DATE) ：做日期类型转换。

      - @OneToOne(optional= true,cascade = CascadeType.ALL, mappedBy = “person”)：一对一关联声明
        @OneToOne(optional = false, cascade = CascadeType.REFRESH)
        @JoinColumn(name = “Person_ID”, referencedColumnName = “personid”,unique = true)：声明为双向关联

      - @ManyToMany(mappedBy= “students”)：多对多关联声明。
         @ManyToMany(cascade = CascadeType.PERSIST, fetch = FetchType.LAZY)
         @JoinTable(name = “Teacher_Student”,
          joinColumns = {@JoinColumn(name = “Teacher_ID”, referencedColumnName = “teacherid”)},
          inverseJoinColumns = {@JoinColumn(name = “Student_ID”, referencedColumnName =
         “studentid”)})：多对多关联，一般有多个关联表， 需要注明

      - @Transiten：表示此属性与表没有映射关系，是一个暂时的属性，指明一个属性或者方法不能持久化

      - @Cache(usage= CacheConcurrencyStrategy.READ_WRITE)：表示此对象应用缓存

      - @TableGenerator：表生成器，将当前主键的值单独保存到一个数据库表中，主键的值每次都是从指定的表中查询来获得，这种生成主键的方式是很常用的。这种方法生成主键的策略可以适用于任何数据库，不必担心不同数据库不兼容造成的问题。

        各属性含义如下：

        - name：表示该表主键生成策略的名称，这个名字可以自定义，它被引用在@GeneratedValue中设置的"generator"值中
        - table：表示表生成策略所持久化的表名，说简单点就是一个管理其它表主键的表
        - pkColumnName：表生成器中的列名，用来存放其它表的主键键名，这个列名是与表中的字段对应的
        - pkColumnValue：实体表所对应到生成器表中的主键名，这个键名是可以自定义
        - valueColumnName：表生成器中的列名，实体表主键的下一个值，假设EMPLOYEE表中的EMPLOYEE_ID最大为2，那么此时，生成器表中与实体表主键对应的键名值则为3
        - allocationSize：表示每次主键值增加的大小，例如设置成1，则表示每次创建新记录后自动加1，默认为50

- java instanceof关键字：二元运算符，测试一个对象是否为一个类的实例，例子：

  ```java
  boolean result = obj instanceof Class
  ```

  其中obj是一个对象，Class表示一个类或者接口，当obj为Class的对象或者是其直接或间接子类，或是其接口的实现类，obj必须是引用类型，result返回true，否则返回false，编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定

  - instanceof实现伪代码：

    ```java
    boolean result;
    if (obj == null) {
      result = false;
    } else {
      try {
          T temp = (T) obj; // checkcast
          result = true;
      } catch (ClassCastException e) {
          result = false;
      }
    }
    ```

    　也就是说有表达式 obj instanceof T，instanceof 运算符的 obj 操作数的类型必须是引用类型或空类型; 否则，会发生编译时错误。 

    　如果 obj 强制转换为 T 时发生编译错误，则关系表达式的 instanceof 同样会产生编译时错误。 在这种情况下，表达式实例的结果永远为false。

    　在运行时，如果 T 的值不为null，并且 obj 可以转换为 T 而不引发ClassCastException，则instanceof运算符的结果为true。 否则结果是错误的

    　简单来说就是：如果 obj 不为 null 并且 (T) obj 不抛 ClassCastException 异常则该表达式值为 true ，否则值为 false 。

- Mybatisplus文档阅读：https://mp.baomidou.com/guide/


- lombok文档阅读：https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok
  - 如果一个父类无默认构造器，那他的子类不能在没有实现父类构造器时使用@Data注释

- Java编程规范学习：
  - 使用Object.equals(a,b)进行对象的判断，因为这种方法不会抛出异常，若使用Object.equals(obj)，则会在Object为空时抛出异常

  - 进行浮点数之间的等值判断时，若用==则需指定一个误差范围，若两个浮点数的差值在此范围之内则认为是相等的；或者使用BigDecimal定义值，再进行浮点数的运算操作(对超过16位有效位的数进行操作，创建出的是类的对象)。

    - 常用构造函数：
    - **BigDecimal(int)**

    创建一个具有参数所指定整数值的对象

    - **BigDecimal(double)**

    创建一个具有参数所指定双精度值的对象

    - **BigDecimal(long)**

    创建一个具有参数所指定长整数值的对象

    - **BigDecimal(String)**

    创建一个具有参数所指定以字符串表示的数值的对象

  - 构造问题：

    假设：

    ```java
    BigDecimal a =new BigDecimal(0.1);
    System.out.println("a values is:"+a);
    System.out.println("=====================");
    BigDecimal b =new BigDecimal("0.1");
    System.out.println("b values is:"+b);
    ```

    该程序运行结果为：

    ```java
    a values is:0.1000000000000000055511151231257827021181583404541015625
    =====================
    b values is:0.1
    ```

    原因分析：

    1. 参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。
    2. String 构造方法是完全可预知的：写入 newBigDecimal(“0.1”) 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言， 通常建议优先使用String构造方法。
    3. 当double必须用作BigDecimal的源时，请注意，此构造方法提供了一个准确转换；它不提供与以下操作相同的结果：先使用Double.toString(double)方法，然后使用BigDecimal(String)构造方法，将double转换为String。要获取该结果，请使用static valueOf(double)方法。

  - 常用方法：

    - **add(BigDecimal)**

    BigDecimal对象中的值相加，返回BigDecimal对象

    - **subtract(BigDecimal)**

    BigDecimal对象中的值相减，返回BigDecimal对象

    - **multiply(BigDecimal)**

    BigDecimal对象中的值相乘，返回BigDecimal对象

    - **divide(BigDecimal)**

    BigDecimal对象中的值相除，返回BigDecimal对象

    - **toString()**

    将BigDecimal对象中的值转换成字符串

    - **doubleValue()**

    将BigDecimal对象中的值转换成双精度数

    - **floatValue()**

    将BigDecimal对象中的值转换成单精度数

    - **longValue()**

    将BigDecimal对象中的值转换成长整数

    - **intValue()**

    将BigDecimal对象中的值转换成整数

    - **compareTo(BigDecimal)**

    比较大小，返回值为-1表示小于，0表示等于，1表示大于

    - **format(String)**

    格式化BigDecimal对象

  - 常见异常：

    - java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result

    通过BigDecimal的divide方法进行除法时当不整除，出现无限循环小数时，就会抛异常，解决方法为设置精确的小数点，如：divide(xxxxx,2)

- RPC方法和POJO类属性都必须使用包装数据类型

  - RPC：远程过程调用，简单的理解是一个节点请求另一个节点提供的服务

    - 本地过程调用：如果需要将本地student对象的age+1，可以实现一个addAge()方法，将student对象传入，对年龄进行更新之后返回即可，本地方法调用的函数体通过函数指针来指定。

    - 远程过程调用：上述操作的过程中，如果addAge()这个方法在服务端，执行函数的函数体在远程机器上，如此告诉机器需要调用这个方法：

      - 首先客户端需要告诉服务器，需要调用的函数，这里函数和进程ID存在一个映射，客户端远程调用时，需要查一下函数，找到对应的ID，然后执行函数的代码。

      - 客户端需要把本地参数传给远程函数，本地调用的过程中，直接压栈即可，但是在远程调用过程中不再同一个内存里，无法直接传递函数的参数，因此需要客户端把参数转换成字节流，传给服务端，然后服务端将字节流转换成自身能读取的格式，是一个序列化和反序列化的过程。

      - 数据准备好了之后，如何进行传输？网络传输层需要把调用的ID和序列化后的参数传给服务端，然后把计算好的结果序列化传给客户端，因此TCP层即可完成上述过程，gRPC中采用的是HTTP2协议。

        具体过程如下：

        ```
        // Client端 
        //    Student student = Call(ServerAddr, addAge, student)
        1. 将这个调用映射为Call ID。
        2. 将Call ID，student（params）序列化，以二进制形式打包
        3. 把2中得到的数据包发送给ServerAddr，这需要使用网络传输层
        4. 等待服务器返回结果
        5. 如果服务器调用成功，那么就将结果反序列化，并赋给student，年龄更新
        
        // Server端
        1. 在本地维护一个Call ID到函数指针的映射call_id_map，可以用Map<String, Method> callIdMap
        2. 等待客户端请求
        3. 得到一个请求后，将其数据包反序列化，得到Call ID
        4. 通过在callIdMap中查找，得到相应的函数指针
        5. 将student（params）反序列化后，在本地调用addAge()函数，得到结果
        6. 将student结果序列化后通过网络返回给Client
        ```

        ![img](https://upload-images.jianshu.io/upload_images/7632302-ca0ba3118f4ef4fb.png?imageMogr2/auto-orient/strip|imageView2/2/w/560/format/webp)

      具体可参加这篇博客:https://www.jianshu.com/p/7d6853140e13

    - 构造方法中禁止加入任何业务逻辑，如果有初始化逻辑则放在init方法中

    - 当使用索引访问用String的split方法得到的数组时，需要在最后一个分隔符后做有无内容的检查，否则会有抛出IndexOutOfBoundsException的风险

    - 不允许在程序的任何地方使用java.sql.Data、java.sql.Time、java.sql.Timestamp

    - ArrayList的subList结果不可强转成ArrayList，否则会抛出ClassCastException异常，因为subList()返回的时ArrayList的内部类SubList，并不是ArrayList本身，二十ArrayList的一个视图，对于SubList的所有操作最终会反映到原列表上。

    - 当使用工具类Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的add/remove/clear方法会抛出UnsupportedOperationException异常，因为asList的返回对象是一个Arrays内部类，并没有实现集合的修改方法。Arrays.asList体现的是适配器模式，只是转换接口，后台的数据仍是数组。

    - Map类集合K/V存储

      | 集合类             | Key          | Value        | Super       | 说明                 |
      | ------------------ | ------------ | ------------ | ----------- | -------------------- |
      | Hashtable          | 不允许为null | 不允许为null | Dictionary  | 线程安全             |
      | Concurrent HashMap | 不允许为null | 不允许为null | AbstractMap | 锁分段技术(JDK8:CAS) |
      | TreeMap            | 不允许为null | 允许为null   | AbstractMap | 线程不安全           |
      | HashMap            | 允许为null   | 允许为null   | AbstractMap | 线程不安全           |

    - 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程，可以减少在创建和销毁线程上消耗的时间及系统资源，解决资源不足的问题.如果不使用线程池,则有可能造成系统创建大量同类线程而导致消耗完内存或者"过度切换"的问题.

    - 线程池不允许使用Executors创建,而是通过ThreadPoolExecutor的方式创建,这样的处理方式能让编写代码的工程师更加明确线程池的运行规则,规避资源耗尽的风险.

      Executors返回线程池对象的弊端如下:

      - FixedThreadPool和SingleThreadPool:允许的请求队列长度为Integer的最大值,可能会堆积大量的请求,从而导致OOM.
      - CachedThreadPool:允许创建的线程数量为Integer的最大值,可能会创建大量的线程,从而导致OOM.

    - 避免Random实力被多线程使用,虽然共享该实例时线程安全的,但会因为竞争同一seed导致性能下降.Random实例包括java.拉.Random的实例或者Math.random()的方式.在JDK7之后,可以直接使用API ThreadLocalRandom;而在JDK7之前,需要编码保证每个线程持有一个单独的Random实例.

    - 在一个switch块内,每个case要么通过continue/break/return等终止,要么注释说明程序将继续执行到哪一个case为止.一个switch块内必须包含一个default语句并且放在最后,即使它什么代码也没有.当switch括号内的变量类型为String并且此变量为外部参数时,必须先进行null判断.

    - 在高并发场景中,避免使用"等于"判断作为中断或退出的条件.

    - 对于需要使用超大整数的场景,服务端一律使用String字符串类型返回,禁止使用Long类型.

    - 在使用正则表达式时,利用好其预编译功能,可以有效加快正则匹配速度.

      代码例子:

      ```java
      private static final Pattern pattern = Pattern.compile(regexRule);
       
      private void func(...) {
          Matcher m = pattern.matcher(content);
          if (m.matches()) {
              ...
          }
      }
      ```

      反例:

      ```java
      // 没有使用预编译
      private void func(...) {
          if (Pattern.matches(regexRule, content)) {
              ...
          }
      }
      // 多次预编译
      private void func(...) {
          Pattern pattern = Pattern.compile(regexRule);
          Matcher m = pattern.matcher(content);
          if (m.matches()) {
              ...
          }
      }
      ```

      

