---
layout: post
title:  "实习日记35(分布式+微服务)"
date:   2021-09-06
categories: jekyll update
---

## Day35

- 摸会儿鱼清理一下学校那边的材料和事项，希望小木桥路那边别找我。。。

- Java学习，基于https://github.com/doocs/advanced-java

- 分布式事务

  - 分布式事物的实现方案：

    - XA方案(两段提交方案)

      有一个事务管理器，负责协调多个数据库(资源管理器)的事物，事务管理器先问各个数据库是否准备好。如果每个数据库都回复ok，那么就正式提交事务，在各个数据库上执行操作；如果任何其中一个数据库回答不ok，那么久回滚事务。

      这种方案比较适合单块应用中，跨多个库的分布式事务，而且因为严重依赖于数据库层面来搞定复杂的事物，效率很低，绝对不适合高并发的场景。所以很少用这个方案，一般来说某个系统内部出现跨多个库的这么一个操作是不合规的。现在的微服务，一个大的系统分成几十个甚至几百个服务。一般来说，我们的规定和规范，是要求每个服务只能操作自己对应的一个数据库。

      如果要操作别的服务对应的库，不允许直连别的服务的库，违反微服务架构的规范，随便交叉访问，几百个服务的情况下会导致全体乱套，这样的一套微服务是无法管理和治理的，可能会出现数据被别人改错，自己的库被别人写挂等情况。如果要操作别人的服务的库，则必须是通过调用别的服务的接口来实现，绝对不云南徐访问别人的数据库。

    - TTC方案(Try,Confirm,Cancel)

      分为三个阶段：

      1. Try阶段：对各个服务的资源做检测以及对资源进行锁定或者预留。

      2. Confirm阶段：在各个服务中执行实际的操作。

      3. Cancel阶段：如果任何一个服务的业务方法执行出错，那么这里就需要进行补偿，就是执行已经执行成功的业务逻辑的回滚操作。(把那些执行成功的回滚)

      这种方法使用的人很少。因为这个事物回滚实际上是严重依赖于自己写代码来回滚和补偿，会造成补偿代码巨大难以维护。一般会使用在支付、交易相关的场景，严格办证分布式事物要么全部成功，要么全部自动回滚，严格保证资金的正确性，保证在资金上不会出现问题。而且最好是各个业务执行的时间都比较短。

    - Saga方案

      业务流程中每个参与者都提交本地事务，若一个参与者失败，则补偿前面已经成功的参与者。

      **使用场景**：对于一致性要求高、短流程、并发高的场景，例如金融核心系统，会优先考虑TCC方案。而在另外一些场景下，只需要保证最终一致性，如很多金融核心以上的业务(渠道层、产品层、系统集成层)，这些业务最终一致即可、流程多、流程长、还可能要调用其他公司的业务。这种情况如果选择TCC方案开发，一来成本高，二来无法要求其他公司的服务也遵循TCC模式。同时流程长，事物边界太长，加锁时间长，也会影响并发性能。

      所以Saga模式的适用场景是：

      - 业务流程长、业务流程多；
      - 参与者包含其他公司或遗留系统服务，无法提供TCC模式要求的三个接口

      **优点**：

      - 一阶段提交本地事物，无锁，高性能
      - 参与者可异步执行，高吞吐；
      - 补偿服务易于实现，因为一个更新操作的反向操作是比较容易理解的

      **缺点**：不能保证事物的隔离性。

    - 本地消息表

      过程如下：

      - A系统在自己本地一个事务里操作同时，插入一条数据到消息表；
      - 接着A系统将这个消息发送到MQ中去；
      - B系统接收到这个消息之后，在一个事务里，往自己本地消息表里插入一条数据，同时执行其他的业务操作，如果这个消息已经被处理过了，那么此时这个事物会回滚，这样保证不会重复处理消息。
      - B系统执行成功之后，就会更新自己本地消息表的状态以及A系统消息表的状态。
      - 如果B系统处理失败了，那么就不会更新消息表状态，那么此时A系统会定时扫描自己的消息表，如果犹未处理的消息，会再次发送到MQ中去，让B再次处理。
      - 这个方案保证了最终一致性，哪怕B事物失败了，但是A会不断重发消息，直到B那边成功为止。

      但这个方案严重依赖于数据库的消息表来管理事务，很难在高并发场景进行扩展，所以一般也很少使用这种方法。

    - 可靠消息最终一致性方案

      不使用MQ来实现事务，而是直接基于MQ来实现事务。

      流程大致如下：

      1. A系统先发送一个prepared消息到MQ，如果这个prepared消息发送失败那么久直接取消操作不执行。
      2. 如果这个消息发送成功过了，那么接着执行本地事务，如果成功就告诉MQ发送确认消息，如果失败就告诉MQ回滚消息；
      3. 如果发送了确认消息，那么此时B系统会接收到确认消息，然后执行本地的事务；
      4. MQ会自动定时轮询所有prepared消息回调接口，询问用户这个消息是不是本地事务处理失败了；所有没发送确认的消息，是继续重试还是回滚。一般来说这里可以查下数据库看之前本地事务是否执行，如果回滚了那这里也回滚。这个就是避免科恩那个本地事务执行成功了，而确认消息却发送失败了。
      5. 这个方案中要是B的事务失败了，就会自动不断重试直到成功，如果实在不行，要么就是针对重要的资金类业务进行回滚，比如B系统本地回滚后，想办法通知系统A也回滚；或者是发送报警由人工来手动回滚和补偿。
    - 最大努力通知方案

      这个方案意思是：

      1. 系统A本地事务执行完之后，发送个消息到MQ
      2. 这里会有个专门消费MQ的最大努力通知服务，这个服务会消费MQ然后写入数据库中记录下来，或者是放入个内存队列也可以，接着调用系统B的接口
      3. 要是系统B执行成功就ok了；要是系统B执行失败了，那么最大努力通知服务就定时尝试重新调用系统B，反复N次，最后还是不行就放弃。

- 集群部署时的分布式Session如何实现

  - Session：当访问服务器否个网页的时候，会在服务器端的内存里开辟一块内存，这块内存就叫做session，而这个内存是跟浏览器关联在一起的。这个浏览器指的是浏览器窗口，或者是浏览器的子窗口，意思就是，只允许当前这个session对应的浏览器访问，就算是在同一个机器上新启的浏览器也是无法访问的。而另外一个浏览器也需要记录session的话，就会再启一个属于自己的session

  - 分布式系统中维护Session状态的方式

    - 完全不用Session

      使用JWT Token存储用户身份，然后再从数据库或者cache中获取其他的信息。这样无论请求分配到哪个服务器都无所谓。

    - Tomcat+Redis

      使用基于Tomcat原生的Session支持，通过`TomcatRedisSessionManager`让所有部署的Tomcat都将Session存储到Redis。

      Tomcat的配置文件：

      ```xml
      <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
      
      <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
               host="{redis.host}"
               port="{redis.port}"
               database="{redis.dbnum}"
               maxInactiveInterval="60"/>
      ```

      然后指定Redis的host和port

      ```xml
      <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
      <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
      	 sentinelMaster="mymaster"
      	 sentinels="<sentinel1-ip>:26379,<sentinel2-ip>:26379,<sentinel3-ip>:26379"
      	 maxInactiveInterval="60"/>
      
      ```

      还可以用上面这种方式基于Redis哨兵支持的Redis高可用集群来保存Session数据。

    - Spring Session+Redis

      Tomcat+Redis方法会与Tomcat容器重耦合，虽然好用但是严重依赖于Web容器，不好讲代码移植到其他Web容器上去，尤其是换了技术栈之后。所以现在比较流行的是基于Java一站式解决方案，也就是Spring。使用Spring Session是一个很好的选择。

      pom配置如下：

      ```xml
      <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
        <version>1.2.1.RELEASE</version>
      </dependency>
      <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.8.1</version>
      </dependency>
      ```

      Spring配置文件如下：

      ```xml
      <bean id="redisHttpSessionConfiguration"
           class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
          <property name="maxInactiveIntervalInSeconds" value="600"/>
      </bean>
      
      <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
          <property name="maxTotal" value="100" />
          <property name="maxIdle" value="10" />
      </bean>
      
      <bean id="jedisConnectionFactory"
            class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" destroy-method="destroy">
          <property name="hostName" value="${redis_hostname}"/>
          <property name="port" value="${redis_port}"/>
          <property name="password" value="${redis_pwd}" />
          <property name="timeout" value="3000"/>
          <property name="usePool" value="true"/>
          <property name="poolConfig" ref="jedisPoolConfig"/>
      </bean>
      ```

      web.xml配置如下：

      ```xml
      <filter>
          <filter-name>springSessionRepositoryFilter</filter-name>
          <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>springSessionRepositoryFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>
      ```

      示例代码：

      ```java
      @RestController
      @RequestMapping("/test")
      public class TestController {
      
          @RequestMapping("/putIntoSession")
          public String putIntoSession(HttpServletRequest request, String username) {
              request.getSession().setAttribute("name",  "leo");
              return "ok";
          }
      
          @RequestMapping("/getFromSession")
          public String getFromSession(HttpServletRequest request, Model model){
              String name = request.getSession().getAttribute("name");
              return name;
          }
      }
      ```

      给Spring Session配置基于Redis来存储Session数据，然后配置了一个Spring Session的过滤器，这样的话，Session相关操作都会交给Spring Session来管了。接着在代码中，就用原生的Session操作，就是基于Spring Session从Redis中获取操作了。
  
- 迁移单体式应用带微服务架构策略：

  一个主要的策略是**不要大规模重写代码**

  - 策略1：停止挖掘

    应当停止让单体式应用继续变大，也就是说当开发新功能时不应该为就单体应用添加新代码，最佳方法就是将新功能开发成独立微服务。

    将新功能以轻量级微服务方式实现有很多优点，例如可以阻止单体应用变的更加无法管理。微服务本身可以开发、部署和独立扩展。采用微服务架构会给开发者带来不同的切身感受。然而，这方法并不解决任何单体式本身问题，为了解决单体式不慎问题必须深入单体应用，做出改变。

  - 策略2：将前端和后端分离：

    减小单体式应用复杂度的策略是将表现层和业务逻辑、数据访问层分开。典型的企业应用至少有三个不同元素构成：

    - 表现层：处理HTTP请求，要么相应一个RESTAPI请求，要么是提供一个基于HTML的图形接口。
    - 业务逻辑层：完成业务逻辑的应用核心
    - 数据访问层：访问基础元素，例如数据库和消息代理。

    在表现层和数据访问层之间有清晰的隔离。业务层由若干方面组成的粗粒度API，内部包含了业务逻辑元素。API是可以将单体业务分割成两个更小应用的天然边界，其中一个应用是表现层，另外一个是业务和数据访问逻辑。分割后，表现逻辑应用远程调用业务逻辑应用。

    单体应用这么分割有两个好处，其一是的应用两部分开发、部署和扩展各自独立，特别地，允许表现层开发者在用户界面上快速选择，进行A/B测试；其二，使得一些远程API可以被微服务调用。

    然后，这种策略只是部分的解决方案。很可能应用的两部份之一或者全部都是不可管理的，因此需要使用第三种策略来消除剩余的单体架构。
    
  - 策略3：抽出服务
  
    第三种迁移策略就是从单体应用种抽取出某些模块成为独立微服务。每当抽取一个模块变成微服务，单体应用就变得简单一些；一旦转换足够多的模块，单体应用本身已经不成为问题了，要么消失了，要么简单到成为一个服务。
  
    - 排序哪个模块应该被转成微服务
  
      一个巨大的复杂单体应用由成千上百个模块构成，每个都是被抽取对象。决定第一个被抽取模块一般都是挑战，一般最好是从最容易抽取的模块开始，这会让开发者积累足够经验，这些经验可以为后续模块化工作带来巨大好处。
  
      转换模块成为微服务一般很耗时间，一般可以根据获益成都来排序，一般从经常变化模块开始会收益最大。一旦转换一个模块为微服务，就可以将其开发部署成独立模块，从而加速开发进程。
  
      将资源消耗大户先抽取出来也是排序标准之一。例如，将内存数据库抽取出来成为一个微服务会非常有用，可以将其部署在大内存主机上。同样的，将对计算资源很敏感的算法应用抽取出来也是非常有益的，这种服务可以被部署在有很多CPU的主机上。通过将资源消耗模块转换成微服务，可以使得应用易于扩展。
  
      查找现有粗粒度边界来决定哪个模块应该被抽取，也是很有益的，这使得一直工作更容易和简单。例如，只与其他应用异步同步消息的模块就是一个明显边界，可以很简单容易地将其转换为微服务。
      
    - 如何抽取模块
      
      抽取模块第一步就是定义好模块和单体应用之间粗粒度接口，由于单体应用需要微服务的数据，反之亦然，因此更像是一个双向API。因为必须在负责依赖关系和细粒度接口模式之间做好平衡，因此开发这种API很有挑战性，尤其对使用域模型模式的业务逻辑层来说更具有挑战，因此经常需要改变代码来解决依赖性问题。
      
      一旦完成粗粒度接口，也就将此模块转换成独立微服务。为了实现，必须写代码使得单体应用和微服务之间通过使用进程间通信(IPC)机制的API来交换信息。
      
      迁移第二部就是将模块转换成独立服务。内部和外部接口都是用基于IPC机制的代码，一般都会将Z模块整合成一个微服务基础框架，来出来割接过程中的问题，例如服务发现。
      
      抽取完模块，也就可以开发、部署和扩展另外一个服务，此服务独立于单体应用和其他服务。可以从头写代码实现服务；这种情况下，将服务和单体应用整合的API代码成为容灾层，在两种域模型之间进行翻译工作。每抽取一个服务，就朝着微服务方向前进一步。随着时间推移，单体应用将会越来越简单，用户就可以增加更多独立的微服务。将现有应用迁移成微服务架构的现代化应用，不应该通过从头重写代码的方式实现，相反，应该通过逐步迁移的方式。有三种策略可以考虑：将新功能以微服务方式实现；将表现层与业务数据访问层分离；将现存模块抽取便成为服务。随着时间推移，微服务数量会增加，开发团队的弹性和效率会大大增加。
      
      
      
      
      
      
