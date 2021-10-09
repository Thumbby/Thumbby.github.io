---
layout: post
title:  "实习日记53(工作任务+java序列化)"
date:   2021-10-09
categories: jekyll update
---

## Day53

- 继续干活

- 在Springboot应用开发中使用JPA时，通常在主应用程序所在包或者其子包的某个位置定义我们的Entity和Repository，这样基于Springboot的自动配置，无需额外配置，我们定义的Entity和Repository即可被发现和使用。但有时候我们需要定义Entity和Repository不在应用程序所在包及其子包，

  @EntityScan：用来扫描和发现指定包及其子包中的Entity定义。如果多处使用`@EntityScan`，它们的`basePackages`集合能覆盖所有被`Repository`使用的`Entity`即可，集合有交集也没有关系。但是如果不能覆盖被`Repository`使用的`Entity`，应用程序启动是会出错,比如：

  ```
  Not a managed type: com.customer.entities.Customer
  ```

  @EnableJpaRepositories：用来扫描和发现指定包及其子包中的`Repository`定义。如果多处使用`@EnableJpaRepositories`，它们的`basePackages`集合不能有交集，并且要能覆盖所有需要的`Repository`定义。

  如果有交集，相应的`Repository`会被尝试反复注册，从而遇到如下错误:

  ```
  The bean ‘OrderRepository’, defined in xxx, could not be registered. A bean with that name has already been defined in xxx and overriding is disabled.
  ```

  如果不能覆盖所有需要的`Repository`定义，会遇到启动错误：

  ```
  Parameter 0 of method setCustomerRepository in com.service.CustomerService required a bean of type ‘come.repo.OrderRepository’ that could not be found.
  ```

- java序列化和反序列化(我好像以前记录过，但忘了= =)

  - 定义：Java序列化就是指把Java对象转换为字节序列的过程

    ​           Java反序列化就是指把字节序列恢复为Java对象的过程

  - 序列化的作用：在传递和保存对象时.保证对象的完整性和可传递性。对象转换为有序字节流,以便在网络上传输或者保存在本地文件中。

    反序列化的作用：根据字节流中保存的对象状态及描述信息，通过反序列化重建对象。

    总结：核心作用就是对象状态的保存和重建。（整个过程核心点就是字节流中所保存的对象状态及描述信息）

  - 序列化优点：

    - 将对象转为字节流存储到硬盘上，当JVM停机的话，字节流还会在硬盘上默默等待，等待下一次JVM的启动，把序列化的对象，通过反序列化为原来的对象，并且序列化的二进制序列能够减少存储空间（永久性保存对象）。
    - 序列化成字节流形式的对象可以进行网络传输(二进制形式)，方便了网络传输。
    - 通过序列化可以在进程间传递对象。

  - 序列化算法要做的事：

    - 将对象实例相关的类元数据输出。
    - 递归地输出类的超类描述直到不再有超类。
    - 类元数据输出完毕后，从最顶端的超类开始输出对象实例的实际数据值。
    - 从上至下递归输出实例的数据。

  - 实现序列化的必备要求：只有实现了Serializable或者Externalizable接口的类的对象才能被序列化为字节序列。(不是则会抛出异常)，但讲道理把对象转换成json属于序列化么？

  - 序列化和反序列化的API：

    - java.io.ObjectInputStream：对象输入流。

      该类的readObject()方法从输入流中读取字节序列，然后将字节序列反序列化为一个对象并返回。

    - java.io.ObjectOutputStream：对象输出流。

      该类的writeObject(Object obj)方法将将传入的obj对象进行序列化，把得到的字节序列写入到目标输出流中进行输出。

  - 序列化和反序列化的三种实现：

    - 若Student类仅仅实现了Serializable接口，则可以按照以下方式进行序列化和反序列化。

      ObjectOutputStream采用默认的序列化方式，对Student对象的非transient的实例变量进行序列化。 
      ObjcetInputStream采用默认的反序列化方式，对Student对象的非transient的实例变量进行反序列化。

    - 若Student类仅仅实现了Serializable接口，并且还定义了readObject(ObjectInputStream in)和writeObject(ObjectOutputSteam out)，则采用以下方式进行序列化与反序列化。

      ObjectOutputStream调用Student对象的writeObject(ObjectOutputStream out)的方法进行序列化。 
      ObjectInputStream会调用Student对象的readObject(ObjectInputStream in)的方法进行反序列化。

    - 若Student类实现了Externalnalizable接口，且Student类必须实现readExternal(ObjectInput in)和writeExternal(ObjectOutput out)方法，则按照以下方式进行序列化与反序列化。

      ObjectOutputStream调用Student对象的writeExternal(ObjectOutput out))的方法进行序列化。 
      ObjectInputStream会调用Student对象的readExternal(ObjectInput in)的方法进行反序列化。

  - 代码示例

    ```java
    public class SerializableTest {
            public static void main(String[] args) throws IOException, ClassNotFoundException {
                //序列化
                FileOutputStream fos = new FileOutputStream("object.out");
                ObjectOutputStream oos = new ObjectOutputStream(fos);
                Student student1 = new Student("lihao", "wjwlh", "21");
                oos.writeObject(student1);
                oos.flush();
                oos.close();
                //反序列化
                FileInputStream fis = new FileInputStream("object.out");
                ObjectInputStream ois = new ObjectInputStream(fis);
                Student student2 = (Student) ois.readObject();
                System.out.println(student2.getUserName()+ " " +
                        student2.getPassword() + " " + student2.getYear());
        }
     
    }
    ```

    ```java
    public class Student implements Serializable{                             
                                                                              
        private static final long serialVersionUID = -6060343040263809614L;   
                                                                              
        private String userName;                                              
        private String password;                                              
        private String year;                                                  
                                                                              
        public String getUserName() {                                         
            return userName;                                                  
        }                                                                     
                                                                              
        public String getPassword() {                                         
            return password;                                                  
        }                                                                     
                                                                              
        public void setUserName(String userName) {                            
            this.userName = userName;                                         
        }                                                                     
                                                                              
        public void setPassword(String password) {                            
            this.password = password;                                         
        }                                                                     
                                                                              
        public String getYear() {                                             
            return year;                                                      
        }                                                                     
                                                                              
        public void setYear(String year) {                                    
            this.year = year;                                                 
        }                                                                     
                                                                              
        public Student(String userName, String password, String year) {       
            this.userName = userName;                                         
            this.password = password;                                         
            this.year = year;                                                 
        }                                                                     
    }                                                                         
    ```

  - 图示

    - 序列化

      ![img](https://img-blog.csdn.net/20180919090545190?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RyZWVfaWZjb25maWc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    - 反序列化

      ![img](https://img-blog.csdn.net/20180919090621522?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RyZWVfaWZjb25maWc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  - 注意点

    - 序列化时，只对对象的状态进行保存，而不管对象的方法；
    - 当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；
    - 当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；
    - 并非所有的对象都可以序列化，比如：
      - 安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行RMI传输等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的；
      - 资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现；
    - 声明为static和transient类型的成员数据不能被序列化。因为static代表类的状态，transient代表对象的临时数据。
    - 序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。为它赋予明确的值。显式地定义serialVersionUID有两种用途：
      - 在某些场合，希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有相同的serialVersionUID；
      - 在某些场合，不希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有不同的serialVersionUID。
    - Java有很多基础类已经实现了serializable接口，比如String,Vector等。但是也有一些没有实现serializable接口的；
    - 如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存！这是能用序列化解决深拷贝的重要原因；

- IDEA自动生成serialVersionUID

  - 设置自动生成 serialVersionUID

    ![img](https://img-blog.csdn.net/20180821112058718?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hldG9uZ3Vu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  - 选中对应的类名，然后按 alt+enter 快捷键

    ![img](https://img-blog.csdn.net/20180821112152756?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hldG9uZ3Vu/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    点击即可自动生成

  - 在java类中显示添加serialVersionUID的作用：

    serialVersionUID适用于java序列化机制。简单来说，JAVA序列化的机制是通过 判断类的serialVersionUID来验证的版本一致的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID于本地相应实体类的serialVersionUID进行比较。如果相同说明是一致的，可以进行反序列化，否则会出现反序列化版本一致的异常，即是InvalidCastException。

    **具体序列化的过程是这样的：**序列化操作时会把系统当前类的serialVersionUID写入到序列化文件中，当反序列化时系统会自动检测文件中的serialVersionUID，判断它是否与当前类中的serialVersionUID一致。如果一致说明序列化文件的版本与当前类的版本是一样的，可以反序列化成功，否则就失败；

    **serialVersionUID有两种显示的生成方式：**
    一是默认的1L，比如：private static final long serialVersionUID = 1L;    

    二是根据包名，类名，继承关系，非私有的方法和属性，以及参数，返回值等诸多因子计算得出的，极度复杂生成的一个64位的哈希字段。基本上计算出来的这个值是唯一的。比如：private static final long serialVersionUID = xxxxL;

    **注意**：显示声明serialVersionUID可以避免对象不一致，

    当实现java.io.Serializable接口中没有显示的定义3serialVersionUID变量的时候，JAVA序列化机制会根据Class自动生成一个serialVersionUID作序列化版本比较用，这种情况下，如果Class文件(类名，方法明等)没有发生变化(增加空格，换行，增加注释等等)，就算再编译多次，serialVersionUID也不会变化的。

    如果我们不希望通过编译来强制划分软件版本，即实现序列化接口的实体能够兼容先前版本，就需要显示的定义一个serialVersionUID，类型为long的变量。不修改这个变量值的序列化实体，都可以相互进行序列化和反序列化。

- 实体类名和表名保持一致，但实体类名使用大驼峰，表名使用小写和下划线(_)进行分割，字段名以表现出属性为主。

