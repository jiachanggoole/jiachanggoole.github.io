##   数据库

- 设计原则 三大范式
  - 字段原子性，不能再分
  - 一个表必须有主键，每行数据唯一区分
  - 一个表不能包含其他表非关键字段信息，不能有冗余字段(一般设计的时候，都会有冗余字段)
- 数据库事务 [MySQL事务隔离级别和实现原理](https://zhuanlan.zhihu.com/p/117476959)
  - 基本要素ACID （原子性、一致性、隔离性、持久性）

  - 事务并发问题（脏读，丢失修改，不可重复读(注重修改)，幻读(注重新增删除)）

  - 隔离级别
    - 读未提交
    - 读提交
    - 可重复读
    - 串行化
    
  - MySQL事务隔离实现
    - 实现可重复读(快照)
    - 并发写(更新时，会锁住记录，当前事务提交后才会释放锁；行锁)
      - 有索引，直接定位到这些数据，加上锁
      - 无索引，所有数据加上锁，然后再过滤，不符合条件的释放锁；大表，性能会有问题
    - 解决幻读(间隙锁，锁住两边范围的数据；无索引情况，则整个表加上间隙锁)
    - Next-Key锁(行锁和间隙锁的结合)
    
  - [**如何实现事务？**](https://www.cnblogs.com/kismetv/p/10331633.html)
  - 原子性
    
    - 原子性和隔离性实现的基础 undo log (回滚日志)
    
    - 当事务对数据库进行修改时，innoDB会生成相应的undo log,如果执行失败，就会根据undo log中的信息将数据回滚到之前的样子
    
  - 持久性

    - 读取数据默认从缓存Buffer Pool中读取，没有的话，从磁盘读取后放入buffer中
    - 写入数据默认先写入buffer pool中，然后buffer中修改数据定期刷入磁盘中(刷脏)；带来问题，如果MySQL宕机，buffer中没有刷新到磁盘，数据丢失；
    - 引入redo log**(InnoDB引擎的)**，修改数据先写入日志，其次更新buffer中；如果MySQL宕机，可以根据redo log中数据，对数据库进行恢复；提交事务的时候调用fsync接口对redo log进行刷盘
      - redo log 刷盘相对于buffer刷脏的优点
        - 刷脏是随机io，redo log是追加操作，顺序IO
        - 刷脏是按数据页（Page）为单位，一点小修改，整页数据都要更新；redo log只包含真正写入的部分，无效IO减少
    - redo log 与 binlog 区别
        -  作用、层次、内容、时机

  - MVCC 多版本并发控制协议
- 数据结构（多叉树）

  - B树 

    - 所有的key数据存储在各个节点上
    - 每次IO会把key和数据都读取出来，浪费IO
    - 所有叶子节点存在同一层，升序排列

  - B+树

    - 非叶子节点不存放数据,可以存放更多的key，减少IO数据量读取
    - 所有叶子节点存在同一层，升序排列，双向链接；适合范围查询,排序查找,分组查找等
    - 高度差不超过1
    - 搜索过程
      - 1. 二级索引数找到主键 2. 回主键索引树进行搜索
        2. 每个磁盘块查找属于哪个区间，是用链表循环查询
    - 覆盖索引
      - 索引列就包含所查询的结果数据，不需要回表查询
    - 联合索引最左原则
- 大表如何加索引 （加索引会锁表，容易出事故）
  - 1.先创建一张跟原表A数据结构相同的新表B。
  - 2.在新表B添加需要加上的新索引。
  - 3.把原表A数据导到新表B
  - 4.rename新表B为原表的表名A，原表A换别的表名；
- 存储引擎
  - MyISAM
  - InnoDb
    - 支持事务
    - 支持行锁
- [分布式id生成](https://tech.meituan.com/2017/04/21/mt-leaf.html)
  - Redis自增
  - MySQL自增
  - UUID
    - 太长
    - 造成Mac地址泄露
    - MySQL不适用，影响性能
  - 类雪花算法
    - 机器时钟回拨问题
    - 64位，分段，性能高；最后生成id是通过各种左移或运算完成的；每毫秒限制4096个id；
    - 第一位0代表正数；时间戳转成二进制是41位;机房id  5位；机器id 5位
    - ![image](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/01888770c8f84b1df258ddd1d424535c68559.png@1112w_282h_80q)
  - 美团算法
    - Leaf-segment数据库方案 一次拿多个号，用完了再拿，减少数据库调用；业务字符分库
    - 双buffer方案，维护两个号段，当前号段用到一定程度，提前取号段缓存起来
    - Leaf-snowflake方案
      - 去zk上取自己的workId,持久节点
      - 本地将workId缓存，防止zk出问题，弱依赖
      - 启动过程，判断自己本地时间跟zk上节点时间大小；同时跟其他所有机器时间平均值校验；
      - 本地时间定时往zk上同步
- 慢查询分析
  - 查看是否开启 show variables like '%slow_query_log%'
  - 开启慢查询日志 set global slow_query_log=1  slow_query_log_file=/tmp/mysql_slow.log
  - long_query_time 时间设置
- **like是否走索引** 【只要包含左%都不走索引】
  - like 'a' 或者 like'a%'  走索引
  - like '%a%' 或者like '%a' 不走索引

## Java集合

### List

- Arraylist `Object[]`数组
- Vector  数组 线程安全
- LinkedList 双向链表，
  - 默认add复杂度为O(1),指定位置add，复杂度为O(n),要遍历先定位到这个位置
  - 每个元素占用的空间比arraylist大，要维护着前后指针数据

- arraylist的扩容机制
  - 构造方法 3种，默认不传，空数组，真正加数据的时候，扩容到10
  - 每次扩容变为原来1.5倍左右(右移一位除以2)；奇偶数不同
  - 每次加元素的时候，看下数据元素个数+1是否大于数组大小；> 则扩容
  - 确定数据量多的时候，创建list的时候，指定容量，防止频繁扩容

### Set

- hashset 
  - 无序 唯一
  -  底层hashmap存储，value是个固定值 
  - 查找key是否存在速度快，O(1)
- LinkedHashSet 内部通过LinkedHashMap实现
- TreeSet 
  - 有序 唯一
  - 红黑树  (自平衡的排序二叉树)

### Map

- LinkedHashMap  在hashmap的基础上，加一个双向链表，维护一个头结点
- Hashtable  锁全表
- TreeMap    红黑树 排序，重写compareTo方法

### HashMap

- 底层实现
  - JDK1.7 数组+链表
  - JDK1.8 优化点：链表长度大于阈值，转为红黑树(转之前，数组长度<64,优先数组扩容)
- 长度为何是2的幂次方

  - hash%length==hash&(length-1)条件成立前提是length是2的幂次方
  - &的效率比%高
- 多线程死循环问题(rehash导致)
- 默认16，扩容*2；指定容量时，会自动变成2的幂次方，阈值0.75
- ConcurrentHashMap线程安全实现

  - JDK1.7,分段锁对数据分割分段(Segment)，每把锁只锁部分数据；Segment数组，每个Segment对象包含HashEntry数组
  - <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210308102931257.png" alt="image-20210308102931257" style="zoom:33%;" />
  - JDK1.8,摒弃Segment,并发控制使用 **synchronized** 和 **CAS** 来操作;只锁当前链表or红黑树首节点
  - TreeNode,红黑树的数据存储结构；TreeBin,存储树形结构的容器，提供红黑树转换or锁的一些操作
- Hashtable

  - 使用synchronized锁方法，效率低下(锁全表数据)
  - key value不允许空值

##框架
### Spring
- IOC 控制反转
- **手写spring ioc思想**
- **循环依赖如何解决？**
  - 一级缓存： 保存所有的singletonBean的实例 <String, Object> singletonObjects 
  - 二级缓存： 保存所有早期创建的Bean对象，这个Bean还没有完成依赖注入  Map<String, Object> earlySingletonObjects 
  - 三级缓存：singletonBean的生产工厂 Map<String, ObjectFactory<?>> singletonFactories 
- AOP
  - 动态代理，接口JDK Proxy,类使用Cglib
  - Spring Aop 集成AspectJ(功能强大)
  - Spring Aop 运行时增强，基于代理；AspectJ编译时增强，基于字节码操作
- Spring Bean
  - Bean作用域
    - singleton 
    - prototype 每次请求都创建
    - request 每次Http请求都创建一个新的，当前Http request 内有效
    - Session 每次Http请求都创建一个新的，当前Http session 内有效
    - global-session
  - 单例安全问题（可变成员变量存储在ThreadLocal中）
  - **@Component** 和 **@Bean** 的区别是什么
    - 作用到类，作用到方法
    - 前者自定义扫描；后者主要是告诉spring这是某个类的实例
    - @bean 自定义更强，引用第三方包的时候只能通过这个实现
  - Bean生命周期
- Spring MVC
  
  - 使用到的设计模式 工厂 单例 代理 观察者 适配器等
- Spring 事务
  - 隔离级别 
    - 读未提交
    - 读已提交
    - 可重复读
    - 串行
    - 默认(mysql 可重复读 Oracle 读已提交)
  - 事务传播行为
  - PROPAGATION_REQUIRED 有事务，加入当前事务；无事务，创建一个新的事务
    - PROPAGATION_SUPPORTS 有事务，加入当前事务；无事务，非事务运行
    - PROPAGATION_MANDATORY 有事务，加入当前事务；无事务，报错
- @Transactional的实现原理
  - 作用就是让Spring为我们管理事务，免去事务管理的逻辑，专注于业务实现
  - 利用Aop实现；
  - @Transactional，作用是定义代理植入点，如果需要代理，将代理对象放入到容器中；同时把事务管理相关的属性信息带过去
  - Interceptor invoke 方法，在执行目标前后执行特定的逻辑；抛异常就回滚，否则提交
- 过滤器 拦截器 切面的顺序

  - 过滤器 -> 拦截器 -> 切面

  - filter只能获取request response，基于serverlet容器

  - 拦截器 preHandle/postHandle/afterCompletion ;报错的话，postHanle不进去；能获取类名与方法名

    - preHandle --> controller方法 --> postHandle --> afterCompletion 

    - 实现 WebMvcConfigurer，将自定义的拦截器加入其中


  - 切面 能获取具体的参数信息
  - @Before→@After→@AfterRunning(如果有异常→@AfterThrowing)
  - @Around→@Before→@Around→@After执行 ProceedingJoinPoint.proceed() 之后的操作→@AfterRunning(如果有异常→@AfterThrowing)
  - <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210310104445315.png" alt="image-20210310104445315" style="zoom:33%;" />
  - ![image-20210310105217419](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210310105217419.png)

### Mybatis

- ```
  # {} 与 ${} 区别
  ```

  - 防止sql注入
  - $简单替换 #当字符串，动态解析为？
  - $用于表名这些

- xml里的标签有哪些

- xml与Dao接口如何对应？Dao接口的原理如何？能否重载

  - namespace与接口全名匹配
  - 方法名与MappedStatement对应,每一个 select update  delete insert 解析为一个MappedStament
  - 不可重载
  - Dao接口原理是JDK动态代理，代理对象拦截接口方法，转而执行与之对应的Sql

- sql结果封装为目标对象返回的方式

  - <resultMap>标签 一一对应
  - 别名设置为对象属性名，不区分大小写

- 映射枚举 TypeHandler



### [DUBBO](https://snailclimb.gitee.io/javaguide/#/docs/system-design/distributed-system/rpc/%E5%85%B3%E4%BA%8EDubbo%E7%9A%84%E9%87%8D%E8%A6%81%E7%9F%A5%E8%AF%86%E7%82%B9?id=%e4%b8%80-%e9%87%8d%e8%a6%81%e7%9a%84%e6%a6%82%e5%bf%b5) 

- 学习资源 [敖丙dubbo学习](https://juejin.cn/post/6870276943448080392)

- 架构图解

  ![Dubbo 架构](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/46816446.jpg)

  ![Dubbo 工作原理](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/64702923.jpg)

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/2398618e72fb4332a6b704eaac2b1e7d~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom: 67%;" />

  - **Provider：** 暴露服务的服务提供方

  - **Consumer：** 调用远程服务的服务消费方

  - **Registry：** 服务注册与发现的注册中心

  - **Monitor：** 统计服务的调用次数和调用时间的监控中心
  
  - **Container：** 服务运行容器
  
  - 分层结构
  
    - Business 、RPC、 Remoting
    - API 、 SPI
  
  - **每层功能分析**
  
    - Service，业务层，就是咱们开发的业务逻辑层。
  
    - Config，配置层，主要围绕 ServiceConfig 和 ReferenceConfig，初始化配置信息。
  
    - Proxy，代理层，服务提供者还是消费者都会生成一个代理类，使得服务接口透明化，代理层做远程调用和返回结果。
  
    - Register，注册层，封装了服务注册和发现。
  
    - Cluster，路由和集群容错层，负责选取具体调用的节点，处理特殊的调用要求和负责远程调用失败的容错措施。
  
    - Monitor，监控层，负责监控统计调用时间和次数。
  
    - Portocol，远程调用层，主要是封装 RPC 调用，主要负责管理 Invoker，Invoker代表一个抽象封装了的执行体，之后再做详解。
  
    - Exchange，信息交换层，用来封装请求响应模型，同步转异步。
  
    - Transport，网络传输层，抽象了网络传输的统一接口，这样用户想用 Netty 就用 Netty，想用 Mina 就用 Mina。
  
    - Serialize，序列化层，将数据序列化成二进制流，当然也做反序列化。
  
- **重要知识点**

  - 提供方和消费方只在服务启动时与注册中心交互，消费方在本地将提供者信息缓存起来
  - 三者之间均是长连接，监控中心除外
  - 服务宕机，注册中心通过长连接推送事件给消费者
  - 注册中心宕机，不影响已有功能的使用
  - 服务端宕机，消费方将无限重连
  - 消费者和提供者在内存记录接口调用次数等信息，定时发送给监控中心

  - 默认dubbo协议，单一长连接，NIO、性能高、支持高并发；适用于数据量小(100kb以内)，但是并发量高

  - 序列化协议：hessian(dubbo)、java序列化(rmi协议) 、json(http通讯协议)、saop文本序列化协议(webservice)

#### 负载均衡策略 

- 默认 基于权重的随机负载均衡策略
- 基于权重的轮询负载机制
- 一致性hash，比如某个用户id的请求，每次都要转发到固定的服务器
- 最少活跃调用数 慢的提供者提供少的服务

#### 集群容错策略

- failover cluster : 失败自动切换，自动重试其他服务器；默认这个，读操作
- falifast cluster :  调用失败就立马报错失败；写操作
- failsafe cluster : 出现异常忽略；记录日志
- failbackc cluster : 失败了后台自动记录请求，定时重发；适合写消息队列场景
- forking cluster : 并行调用多个provider,一个成功，立即返回
- broadcast cluster : 逐个调用所有的provider



#### **服务暴露过程**



####**SPI机制（重点）**

- java spi机制实现  --> ExtensionLoader扩展点加载器

  - 约定好在META-INF/services/下，建立对应接口全路径名的文件
  - 文件中放入需要加载的实现类
  - 缺点：文件中配置的类全部加载，无法按需加载

- Dubbo SPI

  - <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/778df4f10e0a4a7ba054f621241ae329~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom:67%;" />

  - 配置文件存的是键值对，接口上@SPI注解，可以指定名字实例化具体实现的类，

    > optimusPrime = org.apache.spi.OptimusPrime
    >
    > bumblebee = org.apache.spi.Bumblebee

  - 目录

    - META-INF/services/ 兼容Java SPI
    - META-INF/dubbo/  用户自定义SPI配置文件
    - META-INF/dubbo/internal/ Dubbo内部使用的SPI配置文件

  - 自适应扩展--**Adaptive 注解** 

  - AOP 包装器

    Wrapper类：放在一个set里；存在多个的话，层层嵌套

    - 构造方法参数是目标接口类型
    - 给所有的实现类加个切面，先执行wrapper的方法
    - 配置文件里直接配置wrapper类全路径即可

  - IOC  injectExtension

- Filter

  - 基于SPI机制实现
  - 自定义的filter在默认的之后
  - 配置方式
    -  在某个service上配置 filter=""
    - <dubbo:provider filter=""/>或<dubbo:consumer filter=""/> 针对所有的
    - 在Filter实现类上加@Activate，@Activate(group = {Constants.PROVIDER, Constants.CONSUMER})

- [实现原理](https://blog.csdn.net/chicaoxia5444/article/details/100851391)  --> ProtocolFilterWrapper  
  
  - 将实现了filter的类找到， 取<dubbo:service/>中找到filter属性中配置的，同时是provider类型的filter，组成链
  - 为每个节点创建一个Invoker对象，该Invoker对象调用Filter节点
  - ![过滤器](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/e3d709f5e33620536ddec0a8189e2dd8da4.jpg)

#### 服务治理、降级、失败超时重试

- 服务治理： 调用链路生成；服务访问压力时长统计 ；监控报警等
- 降级  服务不可用的时候，返回的数据，mock，return true xxxMock类
- 失败、超时重试 

### Zookeeper

- 节点类型
- 临时节点原理
  - 客户端与服务端连接后，会话建立，生产全局的会话id
  - 保持一个长连接，在session_timeout 时间内，服务器会检查客户端是否正常(客户端会定期给服务端发送一个心跳包，服务器更新下session_timeout时间)
  - 连接断开  (客户端感知到服务端挂了，获取一个新的ZK地址进行重连)
  - 会话超时 
    - 服务端感知不到客户端了，可能是主动断开，也可能是重连失败，那么开始清除这个会话相关的数据
    - 临时节点 +  watcher
    - 同时注册中心通过长连接将变更数据推送给消费者

### 消息队列

#### Rabbit MQ

- 集群模式
  - demo
  - 普通集群模式   
    - 数据只存储在一台机器上，元数据是所有机器都存储；
    - 多机器只是提高吞吐量，从目标机器拉取数据；集群内部大量数据传输
  - 镜像集群模式
    - 每个节点包含所有的数据，高可用
    - 各个节点数据进行数据同步
    - 缺点：不是分布式的；消息同步网络带宽压力与消耗大；

#### Kafaka

- broken进程，kafka在每台机器上启动的一个进程

- topic partition,分布在不同的机器上；生产者的数据分布在不同的机器；分布式的

- 消费者定期将offset提交给kafka; 基于zk实现，zk上记录的消费者目前消费到的offset

- **高可用架构实现**

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/kafka-after.png" alt="kafka-after" style="zoom: 67%;" />

  - 每个partition，有两个副本，信息完全一样，这两个会选举一个为leader，另一个为follwer;
  - 生产和消费数据都连接leader；
  - 写数据，leader数据落地后，其他follewer会向leader来pull数据；所有follower同步好数据，发送ack给leader，leader收到所有的ack后，返回成功的消息给生产者（其中一种，可以调整）
  - leader宕机，kafka自动感知到，自动将follewer变为leader;生产者和消费者都连接新的leader

#### Active MQ

- 消息队列的作用：解耦，异步，削峰
- 架构中引入MQ,可能带来的缺陷
  - 系统可用性降低  MQ故障，整个系统无法运转
  - 系统复杂性变高   消息丢失、消息重复、消息顺序、消息积压等等
  - 一致性问题

- MQ之间的一个对比

  | 特性       | Active MQ                                                    | Rabbit MQ                                        | Rocket MQ                                                    | Kafka                                                        |
  | ---------- | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | 单机吞吐量 | 万级，吞吐量比Rocket MQ和Kafka低了一个数量级                 | 万级，吞吐量比Rocket MQ和Kafka低了一个数量级     | 10万级，支撑高吞吐量的MQ                                     | 10万级，吞吐量高<br/>一般配合大数据的系统进行实时数据计算、日志采集等 |
  | 时效性     | ms级                                                         | 微秒级，这是他的特性，延迟最低                   | ms级                                                         | ms级以内                                                     |
  | 可用性     | 高，基于主从架构实现高可用                                   | 高，基于主从架构实现高可用                       | 非常高，分布式架构                                           | 非常高，分布式的，一个数据多个副本，少数机器宕机，不会求实数据，不会不可用 |
  | 消息可靠性 | 较低概率丢数据                                               |                                                  | 参数优化，可以做到0丢失                                      | 参数优化，可以做到0丢失                                      |
  | 核心特点   | MQ领域的功能极其完备                                         | 基于erlang开发，并发性好，性能极其好，延时非常低 | MQ功能较为完善，分布式的，扩展好                             | 功能较为简单，主要支持简单的MQ功能，在大数据领域使用较多     |
  | 优缺点总结 | 优点：非常成熟，功能强大<br />缺点：小概率丢消息；维护越来越少；主要用于解耦、异步；基本不用于大并发的场景 | 优点:延时低，界面管理很棒，社区相对活跃          | 优点：大规模吞吐，性能非常好，分布式扩展；topic 可以达到几百/几千的级别 <br />缺点：社区活跃一般，做好社区不维护的准备，有实力维护的公司推荐使用 | 大数据领域的标杆                                             |

  

- 点对点、发布订阅模式

- 消息发送失败
  - 点对点，消息一直会保存到服务端，等待消费者消费
  - 发布订阅：一定要送达的话，要配置持久订阅，直到消费者消费
  
- 消息重复消费
  
  - 增加状态表，看看消息有没有消费
  
- 丢消息如何处理
  - 持久化消息 **对数据进行持久化JDBC，AMQ(日志文件)，KahaDB和LevelDB**
  - 启动事务，commit等待服务器的返回
  - 非持久化消息及时处理不要堆积
  - 消息消费不匀称
    - prefetch设置为1，每次取一条消费，处理完成再去取；默认是取1000条
  
- 持久化消息慢如何处理
  
  -  开启事务，会异步发送
  
- 服务宕机
  - 持久化消息会从文件中恢复
  - 非持久化的临时文件(内存不够的时候，会临时存储到临时文件中)会删除



### Quartz

- 关键组件
  - job 任务是什么
  - trigger 表示何时触发任务
  - scheduler 调度器
- 整体流程
  - 先获取线程池中可用线程数量(没有可用，阻塞，直到有可用的)
  - 获取30ms内有要执行的trigger，获取trigger的锁， for update 加锁

## JVM

- 内存区域(运行是数据区)
  - 线程共享  堆 方法区
  - 线程独有  计数器 Jave虚拟机栈(方法如何调用、栈帧结构) 本地方法栈
  - 1.6 运行时常量池在方法区
  - 1.7 运行时常量池在堆，逐渐去方法区
  - 1.8 运行时常量池在元空间(直接内存)
  - 堆的结构(分代是为了回收)
    - 新生代 老年代 永久代(1.7之后废除，元空间)
    - 新生代(Eden、From Survivor、To Survivor),From To来回切换
    - 新生代对象到一定年龄进入老年代,默认15
  - 方法区
    - 与永久代关系 接口与实现类的关系 永久代是HotSpot的概念
    - 永久代被替换为元空间的原因
    - 常用参数设置
  
- 类加载过程
  - 加载  二进制class文件转换成虚拟机所需的格式，加载到方法区
  - 验证  验证文件信息是否符合要求，对虚拟机是否有伤害
  - 准备  类变量在方法区分配内存，初始零值(哪怕代码赋值了，现在也是0)
  - 解析  符号引用转换为直接引用
  - 初始化 赋值，静态块执行等
  
- Java对象创建过程
  - 类加载检查
  - 内存分配
    - 分配方式：指针碰撞 空闲列表
    - 分配并发问题：CAS+失败重试  TLAB(本地内存分配缓冲)
  - 初始化零值
  - 设置对象头
  - 执行init方法
  
- 对象头信息

  - 对象在内存中布局： 对象头信息+实例数据+对齐填充 P47
  - mark word

  ![image-20210314205416075](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210314205416075.png)

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210314214748664.png" alt="image-20210314214748664" style="zoom: 50%;" />

- 相关锁
  
  - 纤程 --> 用户态线程/轻量级线程    可以很多用户态线程对应内核态线程1个
  - 重量级线程：就是经过操作系统，上下文切换/线程切换
  - 偏向锁 --> 偏向第一个线程，将线程id放在对象头
  - 自旋锁  --> 自旋竞争
  
- 对象访问定位方式
  
  - 句柄访问 引用句柄地址不变，句柄指向指针修改即可
  - 直接指针 速度快
  
- 判断对象死亡
  - 引用计数法 互相引用无法回收
  - 可达性分析法
    - GC Root 虚拟机栈-局部变量，类静态属性引用变量，常量引用变量，本地方法栈引用变量

- 类是否为无用类
  - 类的所有实例都已回收
  - 加载该类ClassLoader已回收
  - 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

- 什么时候触发垃圾回收

- 垃圾回收的算法
  - 标记-清除
    - 效率
    - 内存碎片
  - 复制算法 每次用一块
  - 标记-整理
  - 分代收集
    - Minor GC / Young GC 新生代回收
    - Major GC / Old GC 老年代回收
    - Full GC 收集整个堆和方法区
  
- 常见垃圾回收器
  
  - 并行与并发区别   并发(用户线程回收进程同时运行)

<img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210301111136175.png" alt="垃圾回收器总结" style="zoom:50%;" />

- 垃圾回收过程
  - 怎么判断对象已死
  - 垃圾回收算法 讲讲分代回收
  - 垃圾回收器
  
- 常用命令

  - 栈内存溢出排查
    - top -Hp 进程id  
    -  printf '%x%n' 线程id 
    - Jastack 进程id | grep 十六进制
  
  - 堆内存溢出
    -  jmap -heap pid 查看堆内存使用情况
    - Jstat 监控 然后用jmap导出为文件，使用jhat或者mat进行查看
  
  
  
  
  

## 计算机基础

- TCP/IP五层结构

  - 应用层
  - 传输层
  - 网络层
  - 数据链路层
  - 物理层

- TCP三次握手，四次挥手

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210301153706623.png" alt="image-20210301153706623" style="zoom:25%;" />

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210301153752349.png" alt="image-20210301153752349" style="zoom:25%;" />

- TCP如何保证安全传输

  - 数据切割成TCP认为最快的数据块
  - 发送方对数据编号，接收方对数据包进行排序
  - 检验和 首部和数据检验和；有差错，丢弃or不接收
  - 接收端去重复数据
  - 流量控制 滑动窗口协议
  - 拥塞控制
  - ARQ协议->自动重传请求 发送完一个分组，收到确认后，再发送下一个分组
  - 超时重传

- ARQ协议

  - 停止等待ARQ协议
  - 连续ARQ协议

- 浏览器输入地址到显示内容过程

  - DNS解析  域名-->IP地址，DNS缓存
  - TCP建立连接
  - 发送HTTP请求 cookie
  - 服务端处理请求，返回数据
  - 浏览器解析渲染页面
  - 结束连接

- HTPP长连接 短连接

- Http无状态，如何保存用户状态？  Session

- Session与Cookie区别 一个客户端 一个服务端

- http 1.0/1.1区别

  - 长连接支持
  - 状态码变多
  - 缓存处理
  - 带宽优化



## Redis

- [IO多路复用模型](https://mp.weixin.qq.com/s/3gC-nUnFGv-eoSBsEdSZuA)

  - read函数阻塞IO --> 用户态多线程处理 

    --> 操作系统提供非阻塞的read函数 (非阻塞IO),数据没到内核缓冲区时，立马返回 -1；到内核后，read阻塞等待，返回到用户缓冲区 
  
    -->  接收到客户端连接后，放到数组里，一个新的线程不断的遍历这个数组，调用每一个的read方法（用户态的无用的系统调用过多）

    --> 操作系统提供这样的函数，将这个数组发送给操作系统，操作系统遍历，确定哪些有数据返回；用户态再去遍历  select函数 [ **可优化点**  1：数组需要从用户态复制到内核  2：用户态需要遍历检查文件就绪状态，优化为异步事件通知  3：内核只返回就绪的个数，优化为只返回就绪的文件描述符，用户态无需遍历  ]  

    --> poll : 去掉了select函数只能监听1024的限制

    --> epoll : 解决select和poll的一些问题，主要优化了上述的三个问题

- [数据类型结构](https://www.cnblogs.com/ysocean/p/9080942.html)

  - string  

    - 简单动态字符串SDS,保存文本数据、二进制数据
    - length：字符长度  free：buffer数组中未使用的长度   buf[]:保存字符串的每个元素
    - ![image-20210317093731466](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210317093731466.png)

  - list 双向链表

    ![image-20210317093952760](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210317093952760.png)

  - hash 对象

    ![image-20210317095433286](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210317095433286.png)

  - set 无重复数据，交集、差集等

    - intset和hashtable

  - zset 权重值，排序 对应java的treeset
  
    - 跳跃表   --> 分层
      - 一个元素包含向下和向后的指针；
      - 每层是个有序链表,第一层是双向的；
      - 一个元素在level i，肯定也在level i之下的层里
  
- 哪些命令禁用

  - keys    flushdb   flushall   config
  
- 设置过期时间的作用

- 如何判断时间过期   过期字典存储

- 过期数据如何删除
  - 惰性删除 用的时候再查 对CPU友好
  - 定期删除 对内存友好
  
- 内存淘汰机制
  - 持久化机制
  
    - RDB bgsave

    - AOF appendonly yes (三种同步)
  - <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210306103344380.png" alt="image-20210306103344380" style="zoom: 33%;" align=left />
  
- 事务
  
  - MULTI exec discard watch
  
- 缓存穿透 大量无效key,直接查询数据库
  - 做好key的校验
  - 缓存无效的key 时间短点
  - 布隆过滤器 存在误判(存在情况)
  
- 缓存雪崩 大量key无效
  
  - Redis不可用  集群 限流
  
  - 增加随机时间
  - 缓存不失效 热点数据
  
- **<u>缓存与数据库一致性问题</u>**
  
  - 先改数据库，再删缓存，增加缓存重试机制，放入队列
  
-  [Redis 分布式锁实现](https://blog.csdn.net/weixin_33953322/article/details/112773634)
  
  - nx px 同时
  - 自己主动删除要对比是不是自己加的锁(业务执行时间过长，lua脚本) 对比+删除 原子性

## 多线程

- 线程与进程区别

- 并发与并行区别   --能否同时运行

- 线程生命周期

  - ready状态，获得CPU时间片以后就变成running状态

  - timed_waiting 与waiting区别，增加超时时间限制；过了超时时间，自动转为runnable状态

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210304220223053.png" alt="线程生命周期" style="zoom: 25%;" align=left />

- 上下文切换 Linux系统，切换时间少

- 死锁产生原因 -等待别的资源释放
  - 互斥条件
  - 请求与保持条件
  - 不剥夺条件
  - 循环等待条件

- 如何避免死锁
  
  - 一次性申请所有条件
  - 进一步申请资源时，如果申请不到，释放申请到的资源
  
  - 破坏循环条件
  
- sleep 与 wait 对比 
  - sleep不释放锁  wait释放锁
  - sleep 暂停执行；wait用于线程之间通信
  - wait调用后，需要其他线程执行notify()释放；sleep字段唤醒
  
- start与run区别
  
  - Start 多线程执行；run就是一个普通方法
  
- Synchronized **可重入锁！！**
  - 多线程之间访问资源同步；保证只有一个线程执行
  - 修饰静态方法(锁当前class)，普通方法(锁类实例)，代码块(锁对象或者class)
  - 单例模式实现 uniqueInstance = new Singleton();
    - 先分配内存
    - 初始化 uniqueInstance
    - 将uniqueInstance 指向分配的地址
    - JVM有指令重排，执行顺序变更，使用**volatile**禁止JVM指令重排
  
- 构造方法不能用Synchronized修饰，本身就是线程安全的

- Synchronized底层原理实现
  - 同步代码块
    - javap 查看字节码信息，使用的是 monitorenter 和 monitorexit 在代码的开始和结束位置
    - 获取对象的monitor(监视器)，获取后变1，其他线程进入等待队列,释放后变0
    - wait也是监听器，所以只能在同步的代码块中使用
  - 同步方法
    - ACC_SYNCHRONIZED标志标识是否是一个同步方法
  - <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210306215327068.png" alt="image-20210306215327068" style="zoom:25%;" align=left />
  
- Synchronized优化  **偏向锁 --> 轻量级锁 --> 自旋锁(10次) --> 重量级锁**

  - ⾃适应锁 自适应的自旋锁
  - ⾃旋锁 【用户态和内核态来回切换影响性能，防止用户态转入内核态】**自旋其实就是不断的循环判断，然后处理操作**
  - 锁消除 JVM检测一些代码不存在竞争情景，就不加锁
  - 锁粗化
  - 偏向锁 对象头和栈帧中锁记录里存储偏向锁的线程id
  - 轻量级锁  JVM的对象的对象头中包含有⼀些锁的标志位

- 除Synchronized，其他的锁

  - <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/2c3c8213214b44558576bfb0aaacfd7f~tplv-k3u1fbpfcp-zoom-1.image" alt="image-20210306162316895" style="zoom:40%;" align = left />
  - ReentrantLock 可重入锁，内部定义公平锁和非公平锁;需要finally释放锁
  - ReadWriteLock 读写锁
  - 可重入锁的原理
    - 每个锁关联一个计数器和持有者
    - 一个现场拿到锁，会记录下锁的持有者和计数器+1
    - 持有锁的线程再次获取锁，计数器再+1，退出同步方法，计数器依次-1，到0释放锁

- Synchronized与Lock的区别

  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210308141101390.png" alt="image-20210308141101390" style="zoom:33%;" align = left />

- CPU缓存问题
  
  - 解决内存和CPU速度不匹配问题
  - 直接从CPU缓存中读取数据，然后更新到内存中
  - 需要多线程情况下，保证缓存和内存的数据一致性
  
- JMM模型
  
  - CPU与内存速度差异，在CPU中加入多级缓存
  - 缓存一致性协议
  
  - 每个线程有自己的内存副本，导致不一致问题
  - 使用**volatile**，直接去主内存读取
  
- volatile原理 值更新后能立马被其他线程可见，禁止指令重排
  
  - 内存屏障

- volatile与Synchronized区别
  
  - volatile 保证多线程可见性,不保证原子性，只用于变量；
  
  - Synchronized 注重的是访问资源同步
  
- ThreadLocal原理
  - 每个线程维护着ThreadLocalMap，key是ThreadLocal对象，map是真正存储的对象
  - 线程之间互不干扰
  
  <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210305152114114.png" alt="ThreadLoca" style="zoom:25%;" align=left />
  
- ThreadLocal内存泄露问题
  - key被GC了，value没有被GC
  - key是ThreadLocal的弱引用(声明周期更短)

- **AQS**

    <img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210314173719856.png" alt="image-20210314173719856" style="zoom: 25%;" />

    - 里面维护着一个变量(voliate)以及**双向队列(CLH)**
    - 线程获取同步状态失败后，放入CLH的队尾，同时保持自旋状态
    - 自旋过程，不断获取前节点是否为首节点，如果是，则不断尝试获取同步状态；获取成功，则退出CLH队列，业务执行完毕，释放同步状态，唤醒后续节点
    - 共享式和独占式

- 线程池
  - 饱和策略 4种
  - 为什么使用？
    - 提高线程利用率，减少资源消耗
    - 线程统一管理
    - 提高响应时间(创建销毁时间)
    - 控制最大并发数
  - 线程数的设置
    - 计算型
    - IO型
    - 混合型
  - 常见的线程池 单线程 定长 可缓存 定时
  - 常见的阻塞队列
  
- 默认的有什么弊端
    - fix和single的创建的，队列长度为最大值,导致OOM
    - cache和schedule创建的，线程数量可为最大值,导致OOM
  
- CAS compareAndSet  

  - 乐观锁
  - **比Synchronized优势？**
  - 存在问题
    - ABA问题  加版本号解决
    - 循环时间长开销大 --> 竞争资源严重冲突
    - 只能对一个共享变量进行原子操作
  - AtomicInteger 如何实现
    - value使用volatile修饰
    - getAndIncrement 才用CAS操作，每次拿到值加一，对原数据，新结果进行CAS操作，失败重试，成功返回
    - JNI 本地方法，调用其他语言实现，CPU指令；lock_if_mp  cmpxchg

- 



## 高并发

### [限流算法](https://snailclimb.gitee.io/javaguide/#/docs/system-design/high-availability/limit-request)

1. 固定窗口计数器

   > 规定单位时间处理请求的个数
   >
   > 1. 比如每秒接收请求100个，每秒到100个的话，后续请求全部拒绝
   > 2. 过了这一秒，计数器清零
   > 3. 无法保证限流速率，可能前半秒压力很大，后半秒服务器空闲，一个请求都没有

2. 滑动窗口计数器

   > 1. 上述固定窗口计数器的升级版，更加平滑
   > 2. 加入每分钟限流60个请求，平均分成60个窗口，每隔1s，移动一次，每个窗口处理不超过1个请求
   > 3. 每个窗口超过限制的话，不再处理其他请求
   > 4. 窗口分的越多，越平滑

3. 漏桶算法

   > 1. 跟漏桶一样，容量固定，把请求放到漏桶中，定期从桶里拿来去处理，桶满了，直接丢弃

4. 令牌桶算法

   > 1. 跟漏桶算法差不多，桶里装的是令牌，令牌是以一定的速率往桶里装，满了，令牌丢弃
   > 2. 每个请求过来都需要从桶里拿个令牌，处理完了以后，令牌删除
   > 3. 如果桶空了，请求拿不到令牌，丢弃请求



## 分布式

### 系统为何拆分，如何拆分？不用dubbo是否可以

- 项目一体，代码量太多，维护麻烦，改一部分涉及到很多点
- 拆分要很多轮，先按业务粗略分，业务量大了，继续拆
- 不用dubbo，超时机制，负载均衡，序列化，上下线等问题自己处理特别麻烦；

###分布式锁的实现。zk和redis实现的区别

- Redis上锁失败，需要不断循环尝试，消耗性能；解决主从问题，RedLock算法
- zk获取不到的话，加个监听器即可，性能开销小；临时顺序节点代码比较优雅；
- Redis如果客户端挂了，只能等待超时时间；zk的话是临时节点，客户端宕机，自动释放锁

### 分布式事务  

参考文章

[20 张图搞懂「分布式事务」](https://juejin.cn/post/6874788280378851335#heading-16)

> 一般接口都不做分布式事务，代码量复杂，吞吐量下降，性能太差
>
> 监控(发送邮件、报警)、记录日志(完整的调用日志)、时候快速定位、排查与解决、订正数据
>
> 资金、交易、订单，可以使用；
>
> 积分、消费券这些没必要不用

- **两阶段提交方案/XA方案**    事务管理器 spring+JTA；系统里跨库的不符合规范；基本不用

  - 存在问题：同步阻塞、单点故障、数据不一致

- **TCC方案**(Try/Confirm/Cancel)
  
  - try : 先进行检查是否操作可行；银行资金冻结
  - confirm : 执行实际的转账操作
  - cancel：回滚；执行过的步骤取消
  - 主要问题：自己代码控制这几个流程，实现起来非常复杂；对一致性要求非常高的(比如资金、支付相关)，可以使用，保证数据的正确性，执行时间相对较短的适合；
  
- **本地消息表方案**(国外的 ebay 搞出来的这么一套思想)  --> 大量依赖本地数据库的消息表；高并发不适用
  - A: 先插入业务表，再插入消息表
  - B: 先插入消息表，再插入业务表(会检查幂等性等)
  - B执行成功，通知A更改消息表状态；同时A会定时检查消息表状态，如果长时间未更改，会再次发生消息给B,知道B成功为止

- **可靠消息最终一致性方案**  借助rocketmq 实现(3.2.6版本之前的思路)，目前互联网大厂都是这么玩

  - zk的作用，在A发送完成后，监听一个节点；如果B发送失败，就修改节点值，然后通知A再次发送等
  - B执行失败，1. 不断重试  2. 想办法通知A再次发送消息；3. 发送报警，人工参与回滚等措施
  - B系统保证幂等性

  ![image-20210320151725718](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210320151725718.png)

- 最大努力通知方案    增加一个最大努力通知服务(把mq消息存起来)

  - 允许少数的分布式事务执行失败，一般用于要求不严格的情况，日志或者状态等
  - 最大努力通知服务，一般会重复几次，不行就放弃


### 接口幂等性

- 不是技术问题，考察经验，结合业务
- 幂等性要点：
  - 唯一的id，例如订单id
  - 请求处理完，有一个标识记录已经处理过；mysql 唯一键 redis
  - 每次请求判断是否处理的操作

### 分布式服务接口顺序性保证

<img src="https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/image-20210329145738229.png" alt="image-20210329145738229" style="zoom:50%;" />

- 请求hash分发，保证同一个业务id到一个服务器；服务器在通过hash分发，放到不同的消息队列里
- 服务器多线程从对应队列拿取数据保证顺序；基本不会顺序乱掉
- zk分布式锁100%顺序(不建议，并发问题)；几个请求，结合业务状态，获取锁或者释放锁

### RPC框架如何设计

注册中心、动态代理、负载均衡算法、传输协议、序列化协议、服务器监听端口、服务上下线通知、SPI扩展

### zk使用场景

- 多系统分布式协调  --> 监听
- 分布式锁
- 配置中心、注册中心
- 高可用  主备机制

### 消息的可靠性传输

1. 基于RabbitMQ

- 生产者消息搞丢    -->   网络问题没到MQ；MQ内部保存的时候报错了，保存失败
  - 发送消息之前，开启事务；发送消息；报错，重试或者回滚；没有问题，提交事务； --> 吞吐量下降
  - confirm机制   --> 异步处理，发送完消息就OK了，不阻塞；吞吐量高一些
    - channel 设置为confirm机制；发送消息；MQ接收成功，回调一个接口；MQ接收失败，回调一个接口，你可以重发；
    - 本地可以维护消息的状态，长时间没有接收到回调，可以重发消息
- RabbitMQ消息搞丢
  - 持久化到磁盘   --> 内存的数据还未来得及持久化到磁盘，MQ宕机，会导致内存里的一点点数据丢失
  - 1. 创建queue的时候设置为持久化的，这样只会持久化队列的元数据
    2. 发送消息的时候，deliverMode设置为2，这样消息持久化
- 消费者消息搞丢
  - 拿到消息，autoAck开启的话，自动给MQ发送ack，但是消费者处理过程中宕机了，消息就丢失了
  - 解决办法：关掉autoAck，消息处理完成，再发送autoAck；如果没处理完宕机，MQ没有收到你的ack,就会自动将这条消息发送给其他的消费者; 还有一种可能，消息处理完，还没发送autoAck，宕机了，MQ会重新发送消息给消费者，所以消费者要做好幂等处理

2. 基于Kafka 

- 消费者弄丢消息    --> 跟Rabbit类似，手动提交offset，自己保证幂等性，防止重复消费
- 参数设置，这样设置消息一定不会丢
  - topic端 replication.factor >1 每个partition至少有两个副本
  - 服务端 min.insync.replicas >1   一个leader感知到至少有一个follower与自己保持联系
  - productor端  acks = all, 消息必须写入所有的leader与follower才算成功；这样即使消息到了leader，然后宕机了，follower没有这个消息，生产者会不断重试往新选举的leader发消息
  - productor端  retries=MAX   生产者无限重试

### 消息如何保证不重复消费

- 这个是消费者系统去做的，保证幂等性，重复消息发过来保证数据不错乱
- 大致方案：
  - 结合业务，看看这个消息有没有消费过
  - 数据库唯一键约束
  - 基于redis，消费过的往Redis加，重复消息来了以后判断有没有

### 消息如何按照顺序执行

> 使用场景举例：MySQL binlog，肯定要按照顺序执行，不然数据乱掉了

- RabbitMQ 如何保证

  - 出现场景：一个队列多个消息者，最终执行的操作顺序不保证

  - 每个消费者各自对应一个queue，把需要保证顺序的消息，放到一个队列里去

- Kafka如何保证

  - 生产者写的时候指定key,比如订单id，那么这个订单相关的数据一定会分发到一个partition中，并且有顺序的
  - 消费者从partition拿数据也一定是有顺序的   ps:一个消费者只能连接一个partition，消费者个数比partition多的话，多的消费者拿不到数据的
  - 消费者单线程处理消息，顺序能保证，但是吞吐量较低
  - 消费者多线程处理消息，如何保证顺序：消费者增加内存队列，一个线程消费一个队里的数据；将要保证顺序的消息，分发到一个内存队列里即可
  - ![kafka-order-02](https://gitee.com/OceanJia/images/raw/master/pic/markdown_pic/kafka-order-02.png)

### 大量消息积压问题

> 产生背景：消费者故障(执行时间变长或者数据库挂了等等) 导致消息积压 （百万千万级别，即使消费者OK也要1个小时）

- Kafka这种紧急扩容处理：

1. 假设原来有3个消费者，3个partition；此时创建30个partition，临时增加30个消费者；

2. 修改之前3个消费者的代码，消费到了消息，直接往上面30个partition里丢，临时的30个消费者处理消息

3. 恢复正常后，之前3个消费者代码回滚正常操作；30个临时机器下线

   

- Rabbit消息设置过期时间导致消息丢失  --> 不要设置过期时间；丢的消息只能通过补偿措施(编写代码从源头将丢的消息找到，然后再次发送)

- 消息积压导致磁盘满了  --> 消费到的消息直接丢弃，晚上通过补偿措施；或者直接丢到一个临时MQ，跟第一个方案差不多

### 如何设计一个消息队列

- 分布式  数据落在不同的机器
- 持久化   落地磁盘，顺序读写
- 高可用   副本/leader

### 如何设计一个高并发系统

- 单系统 --> dubbo架构多系统,每个系统都有自己的数据库
- 数据库有压力 --> 1. 增加缓存(处理大量的读请求，并发性高)   2. 分库分表，减少每个数据库的压力，提高sql执行效率  3. 读写分离
- 搜索引擎es，天然支持高并发，随便扩容
- MQ 削峰；请求先到MQ,消费系统慢慢处理



### 分库分表

> 正常数据库读写请求并发数建议不要超过2000/s，不然数据库各方面压力就很大了；

- 分库分表的由来  
  - 并发
  - 磁盘容量
  - SQL执行效率

#### 数据库中间件

sharding-jdbc  --> client层方案(一个jar集成到业务系统的)；系统升级，业务系统都要变动，耦合多；

mycat  --> proxy层方案(独立部署)  有运维成本，独立升级即可

#### 如何拆分

- 垂直拆分  --> 1个表变成2个表，每个表字段变少

- 水平拆分  -->  数据分散到多个数据库，每个库均匀存储数据

  ​				 -->  每个数据库再多个表   user_1   user_2    user_3 等，每个表数据量又变少

- 分库分表的方式

  - 按照range来分  --> 每个库存储连续的数据，一般按照时间范围；
    - 扩容方便，每个月都准备好机器，到了时间，数据自然就进来了；
    - 可能会存在热点问题，比如有些数据访问比较多，有些几乎不怎么访问的
    - 所以使用场景注意下，不仅仅访问最新数据，均匀的访问历史数据和最新数据
  - 某个字段hash均匀分散
    - 每个数据库的压力比较均匀
    - 扩容起来麻烦，设计到数据迁移等问题

#### 分库分表扩容方案



#### 分库分表后，全局id怎么生成

> 本质就是分布式id如何生成，就是上面的那几种方案  --> 主要考察雪花算法



#### 读写分离

- 为什么读写分离

  - 减少数据库并发压力，增加从库，承载更多的多请求

  - 一个主库挂了3~5个从库，不要太多

- 主从复制的原理
- ![mysql-master-slave](https://gitee.com/shishan100/Java-Interview-Advanced/raw/master/images/mysql-master-slave.png)
  - 主库将变更写到binlog日志；从库有一个IO线程，将主库的binlog日志拷贝过来写到自己本地的relay日志中；从库有个SQL线程，将relay日志拿出来执行到数据库中
  - 从库这些操作是串行化的，从库数据要慢一点；5.6.x之后，拉取binlog日志支持多线程，但是SQL线程执行还是单线程的；
  - 写并发1000/s，延迟大概有几ms ; 写并发2000/s，延迟大概几十ms; 4000/s 、6000/s 可能有几s延迟

- 同步机制 

  - semi-sync(半同步)复制   --> 主库写入binlog日志后，强制将数据同步到从库，从库写入到自己relay log中，返回ack给主库，才算操作成功；防止数据刚写到主库，此时主库宕机，从库没有数据，**丢失问题**；
  - 并行复制  -->  多个SQL线程，**每个线程**从relay日志里**读一个库**的数据，重放；**库级别**；**缓解主从延迟问题**

- 主从延迟问题解决

  > 可能场景： 1. 先插入数据   2.查询这条数据(有延迟可能这里读不到)   3.更新这条数据(按照第2步查出来的数据)
  >
  > show status    查看 `Seconds_Behind_Master`，可以看到从库复制主库的数据落后了几 ms

  - 分库    将每个库的写并发降低，主从延迟问题可以忽略不计
  - 打开并行复制     -->   针对数据库级别；写并发特别大的时候，意义不大
  - 代码修改    写代码的时候注意下这个问题，避免这个情况，不执行2，直接指执行1和3
  - 如果一定要这样的话，第2步直接读主库(不建议，读写分离没有意义了，特殊情况可以用这个方案)

## 面试术语准备

- 项目中遇到的难点，如何解决的？
  - 接口访问用户信息校验，将用户信息缓存至Redis中，减少数据库的压力；同时缓存的有效期加个随机时间，防止同一时间大量key失效
  - 使用Active MQ进行解耦，推送课程与发送微信消息进行解耦，防止微信推送消息服务异常导致整个流程异常，同时提高系统的并发量
  - 业务复杂的场景，使用线程池+计数器，提高接口响应速度；线程池监控
  - 多线程使用导致的死锁问题；1.一次将资源全部请求过来；2. 继续请求锁定资源超时，释放现在占用的 3. 禁止循环占用形成链
- [多线程的使用场景以及引出的技术点(比较全)](https://juejin.cn/post/6936457087505399821?utm_source=gold_browser_extension#heading-14)
- [JVM系统总结](https://juejin.cn/post/6936390496122044423?utm_source=gold_browser_extension)
- 系统优化
  - 服务器性能优化；增加服务器，负载均衡
  - 数据库优化
    - 加索引，减少join,逻辑运算等写法优化
    - 读写分离
    - 分库分表 大表拆小表
    - 加缓存
  - JVM、Tomcat 调优；减少垃圾回收的频率
  - 业务逻辑优化
    - 合理利用多线程
    - 循环调用
    - 代码写法问题等
    - 同步变异步
- LRU实现 LinkedHashMap 比hashmap多维护一个双向链表，有序，维护一个头节点
- **Netty 网络粘包等**

dubbo：基本的 spi filter ，跟spring里的spi区别如何

mq 内存优化 微服务抽象

对象头信息

redis 数据结构底层实现  分布式锁实现  自旋锁

**分布式事务**

