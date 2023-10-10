> Java道经第5卷

# 第一阶：MessageQueue

**心法：** MessageQueue 消息队列，简称MQ，即存放消息的FIFO队列，主要用于对进程/线程间通信进行解耦，对高并发任务进行削峰填谷，提高程序性能：
- JMS规范：`Java Message Service`，简称JMS，是一个Java平台中关于面向消息中间件的api，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。
- MQ使用原则：尽可能提高消息入队速度，灵活调整消息出队速度。
- MQ技术特点：
    - 消息不丢失：MQ采取put-get-delete模式，仅在消息被完整处理后才会将其删除。
    - 进程无关联：MQ下游进程崩溃，上游进程仍可继续put，等待下游恢复。
    - 处理不重复：MQ中的一个消息仅被处理一次，被某个下游进程获取时会锁定。
    - 处理可延时：MQ中的消息可以被延时处理，更加灵活。

**心法：** [RocketMQ](http://rocketmq.apache.org/) 是阿里巴巴开源的一款高性能，高吞吐量的分布式MQ，源于jms规范但不遵守jms规范：
- NameSrv：邮局，用于管理Broker，生产者和消费者都需要通过它来寻找发现Broker。
- Broker：快递员，用于接收，存储和投递消息，需要提前向NameSrv中注册自己，一个broker中包含多个Topic。
- Topic：地区，用来区分不同类型的消息，发送和接收消息之前都需要先创建Topic，一个Topic中包含1或N个MQ。
- Tag：更详细的地区，Topic中可以按照Tag进行更详细的分类。
- MessageQueue：邮件，用于提高吞吐量，支持并行地向多个MQ发送消息，也支持并行的从多个MQ中获取消息。
- Message：快递本身，用于存放具体消息的对象。
- Producer：寄件人，即消息生产者，需要从NameSrv中获取Broker，然后生产消息并投递给Broker。
- Consumer：收件人，即消息接收者，需要从NameSrv中获取Broker，然后从Broker中消费消息。
- ProducerGroup：生产者组，多个发送同一消息的生产者统称为一个生产者组。
- ConsumerGroup：消费者组，多个消费同一消息的消费者统称为一个消费者组。
- res：`资源/图片/第1阶-MessageQueue.jpg$RMQ结构图`

## RMQ环境搭建

**武技：** 搭建RocketMQ环境，需要提前安装JDK8版本的JDK：

- 安装RMQ服务端：
    - res: `rocketmq-all-4.9.0-bin-release.zip`：解压缩即可。
    - 配置环境变量 `ROCKETMQ_HOME` 和 `path`。
- 在RMQ家目录的 `bin` 目录中启动 `namesrv.cmd` 邮局服务：
    - cmd：`mqnamesrv.cmd -n localhost:9876`：端口默认9876。
- 在RMQ家目录的 `bin` 目录中启动 `broker.cmd` 快递员服务：
    - cmd：`mqbroker.cmd -n localhost:9876 autoCreateTopicEnable=true`：自动创建topic和tag。
- RMQ管控台是一个springboot项目：
    - res: `rocketmq-console.zip`：解压缩即可。
- 修改RMQ管控台项目的主配 `/src/main/resources/application.properties`：
    - `server.port=12581`：RMQ管控台端口，默认8080，容易冲突，建议修改。
    - `rocketmq.config.namesrvAddrs=localhost:9876`：NameSrv地址，默认 `localhost:9876`。
- 将RMQ管控台项目打包，然后启动，最后使用浏览器访问：
    - cmd：`cd D:\rocketmq\rocketmq-console`：项目打包操作需要在项目的根目录进行。
    - cmd：`mvn clean package -Dmaven.test.skip=true`：打包RMQ管控台项目。
    - cmd：`cd target`：RMQ管控台项目的jar包在target目录下生成。
    - cmd：`java -jar rocketmq-console-ng-1.0.0.jar`：启动RMQ管控台项目。
    - cli：`localhost:12581`：第一次访问会有点慢，耐心等待。
- 设计bat启动：bat文件可以双击启动：
    - 创建 `namesrv-9876.bat`：编写namesrv服务的启动命令，注意要添加命令所在的绝对路径。
    - 创建 `broker.bat`：编写broker服务的启动命令，注意要添加命令所在的绝对路径。
    - 创建 `console-12581.bat`：编写 `D:`，`cd ..` 和 `java -jar ..` 3条命令，注意两条命令不要使用分号分隔。

## RMQ生产消费模型

**心法：** 生产者生产消息并投递给broker中的指定队列，消费者轮流获取并消费该消息：
- 生产者 `Producer`：向broker中的指定队列投递消息。
- 消费者 `Consumer`：监听broker中的指定队列，一旦生产者向该队列投递了消息，就立刻触发消费动作。
- 测试准备：创建springboot项目 `rocketmq`，并添加依赖：
    - `org.apache.rocketmq.rocketmq-spring-boot-starter(2.0.3)`

**武技：** 使用junit模拟生产者生产消息到broker中：
- 创建生产者实例，并纳入指定的生产者组中，生产者组自动创建：
    - `new DefaultMQProducer("test-producer-group")`
- 为生产者指定RMQ服务地址并启动：
    - `producer.setNamesrvAddr("localhost:9876")`：集群以分号分隔。
    - `producer.start()`：启动生产者。
- 创建一个消息实例：
    - `new Message("主题", "标签", "消息".getBytes(StandardCharsets.UTF_8))`
- 生产者将消息同步发送到broker中：
    - `producer.send(消息实例, 超时毫秒数)`：返回一个SendResult实例。
- 从SendResult实例中获取消息ID和发送状态：
    - `sendResult.getMsgId()`：每个消息都拥有一个唯一标识ID。
    - `sendResult.getSendStatus()`：发送成功会返回 `SEND_OK` 结果。
- tst：`rmq.ProducerTest.testProducer()`：
    - 在管控台查看是否已经接收到了生产者生产的消息。

**武技：** 使用junit模拟2个消费者轮流消费broker中的消息：
- 创建消费者实例，并纳入指定的消费者组中，消费者组自动创建：
    - `new DefaultMQPushConsumer("test-consumer-group")`
- 为消费者指定RMQ服务地址：
    - `consumer.setNamesrvAddr("localhost:9876")`：集群以分号分隔。
- 为消费者订阅主题和标签：
    - `consumer.subscribe("主题", "标签")`：多标签用 `||` 分隔，`*` 表示订阅所有标签。
- 为消费者设置监听：参数可直接使用Lambda表达式：
    - `consumer.registerMessageListener((MessageListenerConcurrently)(msgs, context)->{})`
    - param1：`List<MessageExt>` 类型的消息列表。
    - param2：`ConsumeConcurrentlyContext` 类型的上下文环境对象。
    - return：`ConsumeConcurrentlyStatus.RECONSUME_LATER`：表示稍后再试。
    - return：`ConsumeConcurrentlyStatus.CONSUME_SUCCESS`：表示稍后再试。
    - 无论返回什么，都需要强转为 `MessageListenerConcurrently` 类型。
- 在监听中获取消息体：
    - `new String(messageExt.getBody())`：获取 `byte[]` 类型的消息体内容，并转为字符串形式。
- 启动消费者实例：
    - `consumer.start()`
- tst：`rmq.ConsumerATest/ConsumerBTest`：
    - 消费者需要一只保持监听状态，所以需要使用main方法进行测试。

## RMQ客户端模板

**武技：** 使用 `o.a.r.s.c.RocketMQTemplate` 作为RMQ的java客户端，测试生产消费模型：
- 添加依赖：
    - `org.apache.rocketmq.rocketmq-spring-boot-starter(2.0.3)`
- 主配添加：
    - `rocketmq.name-server=localhost:9876`：RMQ的NameServer的地址。
    - `rocketmq.producer.group=test-producer-group`：配置生产者组名。
- 开发监听类 `listener.RmqListener`：用于接收RMQ的Broker投递的消息：
    - 标记 `@Component` 以加入spring管理。
    - 实现 `o.a.r.s.c.RocketMQListener<String>`，类泛型为具体消息的类型。
    - 重写 `onMessage()`：该方法在Broker投递消息时触发并执行。
- 为监听类标记 `@RocketMQMessageListener()` 以声明为一个RMQ消费监听类：
    - `consumerGroup="test-consumer-group"`：消费者组名。
    - `topic="test-topic"`：监听来自 `test-topic` 主题的消息。
    - `selectorExpression="*"`：监听来自所有标签的消息，默认为 `*`。
    - `consumeMode=ConsumeMode.CONCURRENTLY`：同组的消费者并发接收消息，默认值。
    - `consumeMode=ConsumeMode.ORDERLY`：同组的消费者按顺序接收消息。
    - `messageModel=MessageModel.CLUSTERING`：同组的消费者平均分摊消费该消息，默认值。
    - `messageModel=MessageModel.BROADCASTING`：同组的消费者全部消费该消息。
- 开发测试类：模拟生产者向broker发送消息：
    - 标记 `@RunWith(SpringRunner.class)`：指定启动方式。
    - 标记 `@SpringBootTest(classes = RocketMqApp.class)`：指定启动类。
    - 使用 `@Autowired` 注入RocketMQTemplate类。
    - 调用 `rocketMQTemplate.convertAndSend("test-topic:test-tag", "hello2")` 同步发送消息。
    - tst：`rmq.RocketMqTemplateTest.testConvertAndSend()`

## RMQ消息类型

**心法：** RMQ的消息分为普通消息和顺序消息两种：
- 普通消息：
    - 同步发送：发送方发出数据后，会在收到响应后才发下一个消息，用于重要通知邮件，报名短信，营销短息等。
    - 异步发送：发送方发出数据后，无需等待响应，直接发送下个消息，用于链路耗时长，对RT时间敏感的业务。
    - 单项发送：发送方只负责发送消息，不需要接收响应，适用于耗时短，不是特别重要的业务，速度最快。
- 顺序消息：一种严格按照顺序来发布和消费的消息类型：
    - 针对API的最后一个字符串参数进行hash取余，以决定该消息落入哪个MessageQueue中。

**武技：** 使用RocketMQTemplate测试三种普通消息和一种顺序消息：
- 开发测试方法 `testSyncSend()`，测试同步发送消息：
    - `rmqTemplate.syncSend("主题:标签", "消息", 超时时间)`
- 开发测试方法 `testAsyncSend()`，测试异步发送消息：
    - `rmqTemplate.asyncSend("主题:标签", "消息", new SendCallback())`
    - 当消息发送成功时触发回调函数中的 `onSuccess()`。
    - 当消息发送异常时触发回调函数中的 `onException()`。
- 开发测试方法 `testSendOneWay()`，测试单项发送消息：
    - `rmqTemplate.sendOneWay("主题:标签", "消息")`
- 开发测试方法 `testSendOneWayOrderly()`，测试顺序发送消息：
    - `rmqTemplate.sendOneWayOrderly("主题:标签", "消息", "xx")`：最后一个参数随意起名。
- tst：`rmq.MessageTypeTest`

# 第二阶：Redis

**心法：** `Not Only SQL` 简称NoSQL，指的是一种泛指非关系型数据库的全新理念，
如键值对结构的redis，文档结构的mongodb，图形结构的neo4j等：
- 高扩展：NoSQL数据之间无关联，故非常容易进行拓展。
- 高性能：NoSQL数据结构简单，在海量数据量场景下具有高读写性能。
- 更灵活：NoSQL无需事前建立表结构，更灵活的操作数据，避免繁琐的表，字段的关系操作。
- 高可用：NoSQL在不太影响性能的情况下，可以方便的实现高可用的架构，如集群等。

**心法：** redis是基于C语言开发的一款遵守BSD协议的高性能开源数据库，
github，twitter，stackOverFlow，阿里巴巴，百度，美团，搜狐，新浪微博等都在使用：
- 高性能：官方测试50个并发执行10W个请求时，redis读速为11W次/s，写速为8.1W次/s。
- 单线程：一次仅运行一条命令，更简单且无并发问题，避免了多线程切换开销和竞态消耗。
- 持久化：默认所有数据保存在内存中以提高性能，亦可将数据更新异步地保存到硬盘。
- 多结构：支持字符串，哈希，列表，集合，有序集合五种数据结构以更灵活的操作数据。
- 多场景：用于缓存数据为DB减压，用于高效计数如播放量和转发数等，用于消息队列系统，排行榜，实时系统如秒杀，限购等。
a 
**武技** 安装redis并搭建环境：
- 下载安装：官网提供 [redis for linux](http://redis.io/download)，
    github提供 [redis for windows](https://github.com/MSOpenTech/redis/tags)：
    - 偶数版本号如3.0为稳定版，奇数版本号如3.1为不稳定版。
    - redis2.8+支持redis-sentinel哨兵，redis3.0+支持redis-cluster集群。
- 目录结构：windows版本的redis下载后直接解压缩即可： 
    - `redis-server.exe`：服务端程序，提供redis服务
    - `redis-cli.exe`：客户端程序，通过它连接redis服务和操作数据
    - `redis-check-aof.exe`：更新日志检查工具，用于对AOF文件进行修复。
    - `redis-benchmark.exe`：性能测试工具，用于模拟客户端向服务端发送操作请求。
    - `redis.windows.conf`：redis主配文件，在将redis作为第三方软件使用时生效。
    - `redis.windows-service.conf`：redis主配文件，在将redis作为系统服务使用时生效。
  
- 新建 `start` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下。
- 开发配置文件 `start/6379.conf`，内容可以从 `redis.windows.conf` 中复制：
    - `bind 127.0.0.1`：绑定本机某个网卡的IP，若不指定，表示接收来自任意一个网卡的redis请求。
    - `port 6379`：redis服务器端口号，默认端口6379
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 6379.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。 
    - `requirepass 123`：设置客户端连接密码为123，提高安全性。
    - `protected-mode no`：关闭保护模式，允许外网远程连接，前提是注释掉 `bind` 配置并不设置密码。
- 启动服务端：在 `start` 目录中开启cmd窗口，服务器运行需要一直保持窗口：
    - cmd：`redis-server 6379.conf`：以指定配置文件启动redis服务端，通过日志文件查看启动日志。
    - cmd：`tasklist | findstr redis*`：查看redis服务端进程。
    - cmd：`netstat -ano | findstr 6370`：查看6379端口号当前被哪个进程占用。
    - cmd：`taskkill /pid 11820 /f`：强制杀死11820进程。
- 启动客户端：在start目录中开启cmd窗口：
    - cmd：`redis-cli -h 127.0.0.1 -p 6379 -a 123`：连接服务端。
    - `-h 127.0.0.1`：服务器IP，默认127.0.0.1，可省略。
    - `-p 6379`：服务器端口，默认6379，可省略。
    - `-a` 表示密码，不填写密码也可以登录，但无操作数据权限。

## 数据类型

**心法：** redis所有键都是字符串类型，而值则可使用 `string/hash/list/set/sorted_set` 五种自带本地方法的数据类型：
- 返回值 `nil` 表示无值，任何变量在没有被赋值之前的值都为 `nil`。
- 除string外的容器型数据结构，都可以在添加元素时自动创建。
- 除string外的容器型数据结构，都可以在容器中无元素时自动删除以释放内存。
- 当容器型数据结构中的元素很多时，建议按时间或hash等维度进行拆分以避免数据倾斜。

**武技：** 测试redis五种数据类型：
- 新建 `type` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下。
- 开发配置文件 `type/6380.conf`：
    - `port 6380`：redis服务器端口号，默认端口6379。
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 6380.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。 
- 启动服务端：在 `type` 目录中开启cmd窗口，服务器运行需要一直保持窗口：
    - cmd：`redis-server 6380.conf`：以指定配置文件启动redis服务端。
- 通用命令：
    - 6380：`debug populate 100`：生成100条测试数据，key值默认为 `key:0` 到 `key:999`。
    - 6380：`dbsize`：返回总键数。
  - 6380：`keys *:?9`：返回符合RE表达式的所有键，重量级命令。
    - 6380：`del key:0`：删除键 `key:0`，删除成功返回1，键不存在返回0。
    - 6380：`exists key:0 key:1`：判断 `key:0` 和 `key:1` 是否存在，返回存在的总个数。
    - 6380：`type key:0`：返回 `key:0` 的数据类型，键不存在返回none。
    - 6380：`expire key:0 10`：设置 `key:0` 在10秒后过期，成功返回1，键不存在返回0。
    - 6380：`ttl key:0`：返回 `key:0` 的当前寿命，单位秒，键不存在返回-2，键永生返回-1。
    - 6380：`persist key:0`：移除 `key:0` 的过期时间，使其永生。
    - 6380：`flushdb`：删除库中的全部key，慎用。
    - 6380：`debug sleep 5`：模拟阻塞5秒钟。
    - 6380：`exit`：优雅退出redis客户端，等效于 `quit` 命令。
    - 6380：`shutdown`：优雅关闭redis服务端，客户端显示 `NotConnect`，但部署在window的服务无法关闭。
- 字符串string：包括字符串，整数和二进制等，底层最终都是用字节数组存储数据，一个string最多存储512MB数据，常用于缓存，计数器，分布式锁，全局ID，限流，位统计在线用户等场景：
    - 6380：`help @string`：查看string类型的全部本地方法。
    - 6380：`set name lucky`：永久存储 `name=lucky`，同名覆盖，无论键是否存在都成功，总是返回OK。
    - 6380：`setnx name lucky`：永久存储 `name=lucky`，键不存在时才成功并返回1，否则返回0。
    - 6380：`setex name 9 lucky`：存储 `name=lucky`，并设置9秒后过期，返回OK。
    - 6380：`get name`：获取name值，键不存在返回nil。
    - 6380：`strlen name`：获取name值的字节数组长度，UTF8编码下中文占3字节，GBK编码下中文占2字节。
    - 6380：`incr age`：对age值自增1，键不存在时视为a=0，自增后返回1，age不为数字报错。
    - 6380：`decr age`：对age值自减1，键不存在时视为a=0，自减后返回-1，age不为数字报错。
- 哈希hash：可视为HashMap容器，每个hash最多存储42亿个键值对，常用于bean数据存储，购物车等场景：
    - 6380：`help @hash`：查看hash类型的全部本地方法。
    - 6380：`hset user name lucky`：为user对象设置 `name=lucky` 属性，成功返回1。
    - 6380：`hsetnx user name lucky`：为user对象设置 `name=lucky` 属性，若name已存在则失败返回0。
    - 6380：`hexists user name`：判断user中是否存在age属性，存在返回1，不存在返回0。
    - 6380：`hget user name`：返回user中的name值，键不存在返回nil。
    - 6380：`hincrby user age 5`：为user中的lucky值自增5，键不存在时视为a=0，返回5，age不为数字报错。
    - 6380：`hlen user`：返回user中所有的键值对数。
    - 6380：`hdel user age gender`：同时删除user中的name属性和gender属性，返回成功总数。
    - 6380：`hkeys user`：返回user中所有属性列表。
    - 6380：`hvals user`：返回user中所有值列表。
- 字符列表list：类似有序双向LinkedList，允许重复元素，最多包含42亿个元素：
    - `6380> help @list`：查看list类型的全部本地方法。
    - `6380> lpush words b a`：在words队列头依次插入b和a，返回插入后队列长度。
    - `6380> rpush words c d`：在words队列尾依次插入c和d，返回插入后队列长度。
    - `6380> llen words`：返回队列长度。
    - `6380> lrange words 0 3`：取出words队列中，索引0-3的全部元素，两端包括，负数视为倒数。
    - `6380> lpop words`：弹出words队列的头元素，空列表返回nil。
    - `6380> rpop words`：弹出words队列的尾元素，空列表返回nil。
- 字符集合set：可视为无序的HashSet，元素不允许重复，最多包含42亿个元素，常用于用户关注，共同好友，抽奖，点赞，签到，打卡等场景：
    - 6380：`help @set`：查看set类型的全部本地方法。
    - 6380：`sadd likes a b`：向likes中添加a元素和b元素，重复添加返回0，返回总影响数。
    - 6380：`scard likes`：返回likes中的元素个数总数。
    - 6380：`smembers likes`：返回likes中的所有元素，重量级命令。
    - 6380：`srem likes a b`：从likes中删除a元素和b元素，返回总成功影响数。
    - 6380：`sismember likes a`：判断likes中是否存在a元素，存在返回1，不存在返回0。 
    - 6380：`srandmember likes 5`：随机从likes中返回5个元素，结果中的元素不重复。
    - 6380：`srandmember likes -5`：随机从likes中返回5个元素，结果中的同个元素有可能重复出现。
    - 6380：`spop likes 5`：随机从likes中返回5个不重复的元素，不支持负数。
    - 6380：`sdiffstore r s1 s2`：返回s1和s2的差集并存入r，重量级命令。
    - 6380：`sinterstore r s1 s2`：返回s1和s2的交集并存入r，重量级命令。
    - 6380：`sunionstore r s1 s2`：返回s1和s2的并集并存入r，重量级命令。
- 有序字符集合sorted_set：每个元素都会关联一个double类型的分数，元素不允许重复但是分数可以重复，常用于排行榜，GEO等场景：
    - 6380：`help @sorted_set`：查看zset类型的全部本地方法。
    - 6380：`zadd age 1 a 2.5 b`：向age中添加1分的a元素和2.5分的b元素，同名覆盖，返回总影响数。
    - 6380：`zcard age`：返回age中全部元素总数。
    - 6380：`zcount age 0 9`：返回age中分数在0到9之间的元素数量。
    - 6380：`zscore age a`：返回age中a元素的分数，元素不存在返回nil。
    - 6380：`zrange age 0 3 withscores`：升序返回age中索引0到3的元素，带分数。
    - 6380：`zrevrange age 0 3 withscores`：降序返回age中索引0到3的元素，带分数。
    - 6380：`zrangebyscore age 3 7 withscores`：升序返回age中分数在3到7之间的元素，带分数。
    - 6380：`zrevrangebyscore age 7 3 withscores`：降序返回age中分数在7到3之间的元素，带分数。
    - 6380：`zrank age a`：返回元素a在age中的升序排名，从0开始。
    - 6380：`zrevrank age a`：返回元素a在age中的降序排名，从0开始。
    - 6380：`zrem age a b`：从age中删除a元素和b元素，返回总影响数。
    - 6380：`zincrby age 5 a`：将age中a元素的分数自增5并返回，负数表示自减。

## 慢查询

**心法：** 慢查询只记录redis命令的执行时间，而不记录redis服务端和客户端之间的网络IO时间，
慢查询记录在内存中的一个FIFO的定长队列中，该队列随redis服务关闭而销毁，建议定期将慢查询记录持久化到数据库中以便日后分析和维护：

- 每一条慢查询日志都由四部分组成：
    - 慢查询日志的唯一标识，自动递增。
    - 命令发生的时间，是一个时间戳。
    - 命令执行的总耗时，单位微秒。
    - 组成该命令的参数数组。
- 临时配置慢查询：在redis服务器运行期使用命令配置，当redis服务器关闭时销毁：
    - 6380：`config set slowlog-log-slower-than 1`：耗时超过1微妙的命令都会进入队列，-1表示关闭慢查询。
    - 6380：`config set slowlog-max-len 1024`：慢查询队列最大长度，超过此值后，先进的记录会被丢弃。
    - 6380：`config get slowlog-log-slower-than`：查看当前的慢查询设置的时间阈值，单位微秒。
    - 6380：`config get slowlog-max-len`：查看当前的慢查询设置的队列最大长度。
- 永久配置慢查询：在配置文件中配置，修改配置文件需要重启redis服务才能生效：
    - `slowlog-log-slower-than 1`：作用同上，默认10000微妙，实际开发中建议配置为1000微秒。
    - `slowlog-max-len 1024`：作用同上，默认128，实际开发中建议配置为1000以上。
- 常用慢查询相关API命令：
    - 6380：`slowlog get 3`：获取慢查询队列中的3条数据（倒查）。
    - 6380：`slowlog len`：获取慢查询队列中有多少条数据。
    - 6380：`slowlog reset`：重置整个慢查询队列。

## 事务

**心法：** redis事务的本质是一次性顺序执行多条命令，批量命令先被放入队列缓存，
当收到 `exec` 命令时开始依次顺序执行，任意命令执行失败都不会影响其他命令，
即不保证原子性也没有回滚操作，且整个过程也不会被其他客户端打断，集群模式下事务无法跨节点：

- 事务流程：执行事务时，正常的命令返回OK，异常的命令返回error：
    - 6380：`multi`：返回OK，表示正式进入一个事务中。
    - 6380：`set name lukcy`：命令正常进入缓存队列，返回QUEUED。
    - 6380：`set age 18`：命令正常进入缓存队列，返回QUEUED。
    - 6380：`incr name`：命令有问题，但仍进入缓存队列，返回QUEUED。
    - 6380：`exec/discard`：依次执行事务队列中的全部命令/丢弃事务。
- watch跟踪：被跟踪的变量一旦在事务内发生改变，则事务提交的结果为 `nil`：
    - 6380-A：`set num 1`：设置num值为1，返回OK。
    - 6380-A：`watch num`：跟踪num，初始值为1，返回OK。
    - 6380-A：`multi`：开启事务。
    - 6380-B：`set num 2`：将num值修改为2，返回OK。
    - 6380-A：`set num 3`：将num值修改为3，命令进入缓存队列，返回QUEUED。
    - 6380-A：`exec`：执行事务，但因为num值被客户端B改动，事务提交失败，返回nil。

## 持久化

**心法：** 持久化指的是将内存中的数据备份到硬盘的操作，主要用于避免意外宕机时丢失数据的情况：
- redis可以同时使用RDB和AOF两种持久化方式，此时redis服务重启时会优先读取AOF文件来恢复数据。
- 若不想持久化，将所有RDB和AOF相关配置删除，并执行 `save ""` 命令，把之前持久化的本地文件全部干掉即可。

### RDB持久化

**心法：** 
RDB是半持久化，即将内存数据备份到硬盘的RDB快照文件中，重读RDB快照可以恢复数据，RDB快照内部采用二进制压缩，
体积比较小，数据恢复速度快，但系统一旦在某次持久化操作之前宕机，则有可能丢失一部分数据，所以RDB快照方式适用恢复数据的规模比较大，
且对数据的完整性不敏感的情况：

- 同步触发：客户端执行 `save` 命令时：
    - 生成临时RDB快照并向其dump数据，dump成功后替换旧RDB快照。
    - 过程不会额外消耗内存但会阻塞其他命令，重量级命令，不建议使用。
- 异步触发：客户端执行 `bgsave` 命令时：
    - 主进程fork出一个子进程来异步执行 `save`，该命令不阻塞，会立刻返回 `background saving start` 响应。
    - 过程需要额外消耗内存进行fork操作，fork的过程会阻塞其它命令，但一般用时很短。
    - redis默认在60s内进行1W次写，300s内进行10次写，或900s内进行1次写时自动触发 `bgsave`，视情况修改。
    - redis客户端执行 `flushall` 命令时会自动生成一个空的RDB快照，意义不大。
    - redis客户端执行 `shutdown` 命令时会先自动触发 `bgsave` 然后再关闭redis服务。
    - redis客户端执行 `debug reload` 命令时，会自动生成一个空的RDB快照，意义不大。
    - redis默认在发生主从全量复制时会自动生成RDB快照。

**武技：** 配置RDB持久化功能：
- 新建 `rdb` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下。
- 开发配置文件 `rdb/6378.conf`：端口，超时时间，工作目录，日志级别和日志文件：
    - `port 6378`：redis服务器端口号，默认端口6379。
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 6378.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。
    - `save 300 10`：300秒内有10个以上的key值发生改变，则触发 `bgsave`，`save ""` 表示禁用，可写多条。
    - `dbfilename dump-6378.rdb`：配置RDB快照，默认为 `dump.rdb`，生成在工作目录中。
    - `stop-writes-on-bgsave-error yes`：最后一次 `bgsave` 失败时，主进程将停止接收数据加以警示，默认yes。
    - `rdbcompression no`：用LZF算法压缩RDB快照，节省硬盘空间，但会额外消耗CPU资源进行压缩操作，默认yes。
    - `rdbchecksum no`：用CRC64算法检查RDB快照，提升安全性，但会额外增加10%左右的CPU消耗，默认yes。 
- 启动服务端：在 `rdb` 目录中开启cmd窗口，服务器运行需要一直保持窗口：
    - cmd：`redis-server 6378.conf`：以指定配置文件启动redis服务端。
- 连接redis服务测试RDB持久化效果：
    - 6378：`set name lucky`：存储数据，这里可以多存储几条用于测试。
    - 6378：`bgsave`：将数据异步存储到RDB快照中，此时观察RDB快照是否生成。
    - 重启redis服务并重新连接redis服务，此时redis会从RDB快照中恢复数据，该过程会阻塞其它命令。
    - 6378：`get name`：发现数据仍在，表示RDB持久化功能配置成功。

### AOF持久化

**心法：** AOF是全持久化，将每一次的写命令都追加到磁盘的aof_buffer缓冲区，然后再fsync（刷盘）到硬盘的AOF文件中，当redis服务重启时，会重新执行AOF文件中的命令以恢复数据，该方式的数据完整性和安全性更高，但相对的AOF日志文件更大，可能会导致数据恢复速度比RDB方式慢很多：
- AOF刷盘策略：
    - `always`：redis每次写操作都立刻刷盘，数据完整度高，但是磁盘IO次数多，效率低。
    - `everysec`：redis每秒刷盘，磁盘IO次数少，但有可能丢失最后1秒的数据，是redis默认配置。
    - `no`：由OS决定如何刷盘，大多数linux系统是30秒刷盘一次，不用管，不可控，不建议。
- AOF文件重写：将一些被覆盖的，过期的命令进行优化，以压缩AOF文件大小，加速恢复速度：
    - 主进程fork一个子进程去异步执行rewrite命令，fork过程是阻塞的。
    - 此时主进程接收到的任何写命令都会被暂时追加到aof_buffer缓冲区中（需要配置）。
    - 子进程执行完rewrite操作后，会返回给主线程一个新的AOF文件。
    - 主进程将aof_buffer缓冲区中新接收到的写命令追加到新的AOF文件中。
    - 主进程使用新的AOF文件替换掉旧的AOF文件，至此完成rewrite全过程。
    - 当客户端发送 `bgrewriteaof` 命令会立刻执行AOF文件重写。
    - 当AOF文件当前大小超过了64M，并且文件大小增长率超过了100%时，自动触发AOF文件重写。

**武技：** 配置AOF持久化功能：
- 新建 `aof` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下。
- 开发配置文件 `rdb/6377.conf`：配置IP，端口，超时时间，工作目录，日志级别和日志文件：
    - `port 6377`：redis服务器端口号，默认端口6379。
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 6377.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。
    - `appendonly yes`：开启AOF持久化，默认no，表示不开启AOF持久化。
    - `appendfilename appendonly-6377.aof`：配置AOF文件名，默认 `appendonly.aof`，生成在工作目录中。
    - `appendfsync everysec`：配置刷盘策略为每秒刷盘。
    - `no-appendfsync-on-rewrite yes`：rewrite期间禁止刷盘，仅将数据暂存缓冲区，不阻塞但可能丢数据，默认no。
    - `auto-aof-rewrite-min-size 64mb`：AOF文件当前大小超过64M时满足rewrite条件之一，默认64M。
    - `auto-aof-rewrite-percentage 100`：AOF文件当前大小比上一次增大了100%时满足rewrite条件之一，默认100%。
- 启动服务端：在 `aof` 目录中开启cmd窗口，服务器运行需要一直保持窗口：
    - cmd：`redis-server 6377.conf`：以指定配置文件启动redis服务端。
- 连接redis服务测试AOF持久化效果：
    - 6377：`set age 58`：存储数据，这里可以多存储几条用于测试。
    - 重启redis服务并重新连接redis服务，此时redis会将AOF文件中的命令全部都执行一遍以恢复数据。
    - 6377：`get age`：发现数据仍在，表示aof配置成功。

## Jedis客户端

**心法：** 官方推荐使用 [Jedis](http://xetorthio.github.io/jedis/) 作为java语言的客户端，
且提供了 `r.c.j.JedisPool` 连接池以避免频繁创建和销毁jedis连接产生的性能消耗：
- 首先创建一个连接池的配置实例 `jedisPoolConfig`，通过该实例配置连接池大小，阻塞时间，最大最小空闲数等参数。
- 然后通过连接池的配置实例 `jedisPoolConfig` 来创建一个连接池实例 `jedisPool`。
- 然后通过连接池实例 `jedisPool` 来获取一个redis连接 `jedis`。
- 然后通过redis连接 `jedis` 来操作redis数据库。
- 最后关闭jedis连接，释放资源。

**武技：** 使用Jedis客户端连接redis(6379)服务器：
- 启动服务端：在 `start` 目录中开启cmd窗口，服务器运行需要一直保持窗口：
    - cmd：`redis-server 6379.conf`：以指定配置文件启动redis服务端。
- 新建springboot项目 `redis` 并添加依赖：
    - `redis.clients.jedis(2.9.0)`：Jedis依赖。
- 创建一个连接池的配置实例 `jedisPoolConfig`，并配置相关参数，建议在静态块中开发：
    - `new JedisPoolConfig()`：创建连接池配置实例。
    - `jedisPoolConfig.setMaxTotal()`：设置连接池最多容纳连接数，负值表示无限制。
    - `jedisPoolConfig.setMaxWaitMillis()`：设置连接池最大阻塞等待时间，单位毫秒，负值表示无限制。
    - `jedisPoolConfig.setMaxIdle()/setMinIdle()`：设置连接池中的最大/最小空闲连接数。
    - `jedisPoolConfig.setBlockWhenExhausted()`：默认true，表示连接耗尽时是否阻塞直到超时，false则抛异常。
- 创建一个连接池实例 `jedisPool` 并获取一个连接 `jedis`，建议在静态块中开发：
    - `new JedisPool(连接池配置实例, "IP", port, 超时时间, "密码")`：创建连接池实例。
    - `jedisPool.getResource()`：从连接池中获取一个Jedis连接。 
- 测试Jedis常用API命令：
    - `jedis.ping()`：返回是否连通redis服务端，成功连通返回 `PONG`。
    - `jedis.set(K, V)`：调用redis的 `set` 命令，其余命令同理。
    - `jedis.close()`：关闭redis连接，释放资源。
- tst：`jedis.JedisPoolTest.*`

### PipeLine流水线

**心法：** pipeline是流水线功能，以节省网络IO消耗，但并不能节省命令执行时间，
- 将一批相同或不同的少量命令打包。
- 将打包的命令通过一次网络IO到达服务端。
- 在服务端解包并依次执行命令，此过程不是原子性的。
- 集群模式下，pipeline仅能作用于一个节点。
  

**武技：** 测试pipeline耗时：
- 通过Jedis连接获取一个Pipeline实例：
    - `jedis.pipelined()`：获取一个Pipeline实例。
- 通过Pipeline实例操作redis数据库：    
    - `pipeline.hset()`：使用Pipeline调用 `hset` 命令。
    - `pipeline.syncAndReturnAll()`：异步执行并返回全部结果。
- tst：`jedis.PipeLineTest.testPipeline()`

### BitMap位图

**心法：** redis字符串的底层使用bitmap位图，即二进制形式存储，支持直接对位图进行按位操作：
- 统计登录用户：使用bitmap，在用户id对应的索引上设置1或0以表示登录或未登录：
    - 假设网站有1亿注册用户，则使用bitmap最多需要存储1亿个用户id作为索引，共需 `1bit*1亿=12.5M` 内存。
    - 若日平均活跃用户5000W，用set集合存储int类型id，共需 `32bit*5000W=200M` 内存，差于bitmap。
    - 若日平均活跃用户10W，用set集合存储int类型id，共需 `32bit*10W=4M` 内存，优于bitmap。
- 布隆过滤器：底层使用一个bitmap和N个函数，用于过滤大量无效请求以解决部分缓存穿透问题，函数越多失误率越低：
    - 对数据库中的每个元素依次执行布隆过滤器的N个函数，然后在结果对应的bitmap位图索引处标1。
    - 当客户端请求查询某元素时，先经过布隆过滤器，分别执行N个函数，然后在bitmap中比对，全为1才允许通过。
    - 布隆过滤器说有，不一定是真的，但布隆过滤器说没有，一定是真的。
- bitmap可以字节（8bit）为单位自动扩容：
    - 字符串 `"a"` 的ascii值为 `97`，对应位图为 `01100001`：无需扩容。
    - 字符串 `"aa"` 的ascii值为 `97 97`，对应位图为 `01100001 01100001`：自动扩容1字节。
    - 字符串 `"a"` 的位图仅8位，若强行操作1000号位，则会将其9-999位全补0，严重耗时。

**武技：** 测试bitmap常用API方法，以下 `name位图` 指的是name对应的值，如字符串 `"a"` 的位图：
- `jedis.getbit("name", 0)`：返回name位图0号位上的布尔值，同 `getbit name 0` 命令。 
- `jedis.setbit("name", 0, true)`：设置name位图0号位上的值为1，同 `setbit name 0 1` 命令：
- `jedis.bitcount("name", 0, 9)`：返回name位图中，0-9号位中有多少个1，同 `bitcount name 0 9` 命令。
- `jedis.bitpos("name", false/true)`：返回name位图，第一个0/1的位置，同 `bitpos name 0/1` 命令。
- `jedis.bitop(BitOP.AND, "r", "a", "b")`：将a和b位图的位与结果存入r，同 `bitop and r a b` 命令。
- `jedis.bitop(BitOP.OR, "r", "a", "b")`：将a和b位图的位或结果存入r，同 `bitop or r a b` 命令。
- `jedis.bitop(BitOP.XOR, "r", "a", "b")`：将a和b位图的位异或结果存入r，同 `bitop xor r a b` 命令。
- `jedis.bitop(BitOP.NOT, "r", "a")`：将a位图按位取反的结果存入r，同 `bitop not r a` 命令。
- tst：`jedis.BitMapTest.testApi()`

### GEO地理定位

**心法：** geo表示地理信息定位，存储经纬度，计算两地直线距离，范围距离等，是redis3.2+添加的一个特性，底层使用sorted_set结构。

**武技：** 测试geo常用API方法：
- `jedis.geoadd("cities", 1.1, 2.2, "北京")`：将北京的经纬度 (1.1, 2.2) 添加到cities中：
    - 同 `geoadd cities 1.1 2.2 北京` 命令。
- `jedis.geopos("cities", "北京")`：获取cities中北京的经纬度：
    - 同 `geopos cities 北京` 命令。
- `jedis.geodist("cities", "北京", "天津", GeoUnit.KM)`：获取cities中北京和天津的直线距离，单位千米：
    - 同 `geodist cities 北京 天津 km` 命令。
- `jedis.georadius("cities", 1.0, 2.0, 5, GeoUnit.KM)`：获取cities中以指定经纬度为中心，5km范围内城市列表：
    - 同 `georadius cities 1.0 2.0 5 km` 命令。
    - 最后一个参数可选，用于配置额外参数，可通过 `GeoRadiusParam.geoRadiusParam()` 获取。
    - `config.withCoord()`：设置返回结果中，包含经纬度，对应命令添加 `... withcoord` 即可。
    - `config.withDist()`：设置返回结果中，包含相对距离，单位自动获取，对应命令添加 `... withdist` 即可。
    - `config.count(10)`：设置最多返回10条元素，对应命令添加 `... count 10` 即可。
    - `config.sortAscending()`：设置返回结果以升序排列，对应命令添加 `... asc 10` 即可。
- `jedis.georadiusByMember("cities", "北京", 5, GeoUnit.KM)`：获取cities中以北京为中心，5km范围内城市列表：
    - 同 `georadiusbymember cities 北京 5 km` 命令。
- tst `jedis.GeoTest.testGeoAddAndPos()`

### 发布订阅模型

**心法：** redis支持发布订阅模型，但不支持消息堆积，即订阅者无法查看历史消息，集群中的发布订阅会影响所有节点，消耗带宽，此时建议单独设置一套哨兵以专门维护发布订阅功能：
- 发布订阅模型：频道不存在时会在服务端自动创建：
    - 订阅者subscriber订阅频道channel。
    - 发布者publisher将信息发布到频道channel。
    - 订阅者subscriber就会收到该信息。
- 开发订阅者监听 `jedis.SubscriberListener`：建议使用构造接收订阅者名称：
    - 继承 `r.c.j.JedisPubSub` 并重写 `onMessage()/onSubscribe()/onUnsubscribe()`。
- 开发订阅者类 `jedis.SubscriberA/SubscriberB`：
    - `jedis.pubsubChannels("*")`：列出所有频道，对应 `pubsub channels` 命令。
    - `jedis.pubsubNumSub("sina")`：列出sina频道有多少订阅者，对应 `pubsub numsub sina` 命令。
    - `jedis.subscribe()`：订阅频道并阻塞，对应 `subscribe sina` 命令，参数为订阅者监听实例和频道列表。
    - `jedisPubSub.onUnsubscribe()`：退订频道，对应 `unsubscribe sina` 命令。
- 开发发布者类 `jedis.Publisher`： 
    - `jedis.publish()`：向指定频道发送消息，对应 `publish sina hi` 命令，参数为频道和消息。
- 使用main方法启动两个订阅者类，在启动发布者类，观察控制台变化。

## 主从模型

**心法：** redis支持1主N从模型以提高数据容灾性，主从节点不建议部署在同一台机器上：
- 主从模型中，数据是单向的，只能从master流向slave，故主写从读，读写分离。
- 主从模型中，若master挂了，仍可以在slave中读取数据，只不过暂时无法写数据而已。
- 主从模型可以在配置文件中直接配置，但修改配置文件需要重启redis服务：
    - `slaveof 127.0.0.1 7007`：指定自己的master节点的IP和端口号。 
    - `masterauth 123`：配置master节点的连接密码，master节点无密码时可省略。

**武技：** 搭建1主2从模型，其中7007为master节点，7008和7009为slave节点：
- 新建 `master-slave` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下。
- 开发master节点配置文件 `master-slave/7007.conf`：
    - `port 7007`：redis服务器端口号，默认端口6379。
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 7007.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。
    - `save 300 10`：300秒内有10个以上的key值发生改变，则触发 `bgsave`，`save ""` 表示禁用，可写多条。
    - `dbfilename dump-7007.rdb`：配置RDB快照，默认为 `dump.rdb`，生成在工作目录中。
    - `stop-writes-on-bgsave-error yes`：最后一次 `bgsave` 失败时，主进程将停止接收数据加以警示，默认yes。
    - `rdbcompression no`：用LZF算法压缩RDB快照，节省硬盘空间，但会额外消耗CPU资源进行压缩操作，默认yes。
    - `rdbchecksum no`：用CRC64算法检查RDB快照，提升安全性，但会额外增加10%左右的CPU消耗，默认yes。
- 开发两个slave节点配置文件：`master-slave/7008.conf` 和 `master-slave/7009.conf`：
    - 配置尽量和上面的7007节点的保持一致，只需要将7007全部替换7008或7009即可。
- 服务过多时，可以将3个节点配置为windows服务，启动后可以一直保持后台运行，以7007为例：
    - cmd：`redis-server --service-install 7007.conf --service-name redis7007`：部署服务。
    - cmd：`redis-server --service-uninstall 7007.conf --service-name redis7007`：卸载服务。
    - cmd：`redis-server --service-start 7007.conf --service-name redis7007`：启动服务。
    - cmd：`redis-server --service-stop 7007.conf --service-name redis7007`：停止服务。
    - cmd：`sc delete redis6379`：在任意位置删除redis6379服务，需要管理员权限。
    - cmd：`net start redis6379`：在任意位置启动redis6379服务，需要管理员权限。
    - cmd：`net stop redis6379`：在任意位置停止redis6379服务，需要管理员权限。
- 使用命令指定7008和7009为7007的slave节点：该命令异步执行，立即返回OK：
    - 7008：`slaveof 127.0.0.1 7007`：先清空slave节点全部数据，然后再建立主从关系。
    - 7009：`slaveof 127.0.0.1 7007`：先清空slave节点全部数据，然后再建立主从关系。
    - 建立主从关系后，master节点使用 `bgsave` 命令将数据备份到RDB文件，再恢复到slave节点中完成数据同步。
- 使7008脱离主从关系，此后不再接收master同步的数据，但之前同步的数据仍然保留：
    - 7008：`slaveof no one`：使7008脱离主从关系。
- 分别在3个节点中查看replication主从信息，以7007为例：
    - 7007：`info replication`：通过role识别自己是master主节点还是slave从节点。
- 主从模式下的测试操作：
    - 7007：`set name lucky`：在master节点中写数据，返回OK。
    - 7007：`get name`：在master节点中读数据，返回100。
    - 7008：`set name joezhou`：在slave节点中写数据，报错。
    - 7008：`get name`：在slave节点中读数据，返回100。
    - 7009：`get name`：在slave节点中读数据，返回100。

## 哨兵模型

**心法：** sentinel哨兵无法操作和存取数据，它仅用于解决主从模型的高可用问题：
- 每个哨兵都可以监视（monitor）多套主从结构：
    - 哨兵只需要监控某个主从模型中的master，就可以发现该master下的所有从节点。
    - 哨兵之间可以互相发现，消息也可以共享。
    - 建议对哨兵本身搭建集群，以提升哨兵体系的高可用性。
- 每个哨兵都会不断地对自己监视的主从结构进行心跳检测，当master节点失联或报错时，对其添加一枚主观下线标记：
    - 主观下线 `subjective down`，简称 `sdown`：我觉得这个节点好像挂了。
- 当超过一定数量（需要配置）的哨兵都对master节点添加了主观下线标记，则对其添加客观下线标记：
    - 客观下线 `objective down`，简称 `odown`：这个节点确实挂了。
- 当master节点客观下线时，所有监视该master节点的哨兵开始投票选举哨兵队长，负责进行故障转移 `failover`：
    - 故障转移：将发生故障的节点转移出去，从它的slave节点中选出一个，晋升为新的master节点。
- 故障转移具体流程：sentinel队长操作，其它sentinel队员等待故障转移结果：
    - 先从发生故障的master节点的slave节点中选择一个候选人节点。
    - 然后对候选人节点执行 `slaveof no one` 命令，取消其slave身份，恢复其自由身。
    - 然后再依次修改master节点和候选人节点的配置信息，完成身份互换。
    - 最后通知其它slave节点和其它sentinel哨兵，这次 "人事变更" 的结果。
    - 故障转移结束后，即使原master节点复活了，也只能称为新master节点的slave节点。

**武技：** 搭建哨兵集群，成员分别为27007，27008和27009节点：
- 新建 `sentinel` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下。
- 开发一个sentinel节点的配置文件，如 `sentinel/27007.conf`：
    - `port 27007`：redis服务器端口号，默认端口6379。
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 27007.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。
    - `protected-mode no`：关闭保护模式，否则Jedis无法访问。
    - `sentinel monitor x 127.0.0.1 7007 2`：监视7007的主从结构并起名为x，超过2枚主观下线标记时故障转移。
    - `sentinel down-after-milliseconds x 5000`：哨兵在5s内ping不通x或x直接报错时，添加1枚主观下线标记。
    - `sentinel failover-timeout x 15000`：若某哨兵在15s内仍未能对x完成故障转移，则直接报错。
- 开发其余两个sentinel节点配置文件：`sentinel/27008.conf` 和 `sentinel/27009.conf`：
    - 配置尽量和上面的27007节点的保持一致，只需要将27007全部替换27008或27009即可。 
- 将3个sentinel部署为window服务并启动，哨兵服务需要额外添加 `--sentinel` 参数，以27007为例：
    - cmd：`redis-server --service-install 27007.conf --sentinel --service-name sentinel27007`
    - cmd：`redis-server --service-start 27007.conf --sentinel --service-name sentinel27007`
    - 若直接使用cmd命令启动哨兵服务，也需要在命令末尾额外添加 `--sentinel` 参数。
- 分别查看3个sentinel日志：生成哨兵ID，发现了全部slave，哨兵之间互相发现以保证高可用。
- 分别查看3个sentinel的监视信息，以27007为例：
    - 27007：`info sentinel`：哨兵之间可以互相发现，也可以发现其监视的master下的所有slave节点。

### 故障转移failover

**武技：** 模拟节点故障，触发故障转移：
- 手动下线master节点7007，模拟节点故障：
    - cmd：`redis-server --service-stop 7007.conf --service-name redis7007`：下线7007。
- 查看任一sentinel的监视信息，以27008为例：
    - 27008：`info sentinel`：会发现master发生变更，已经完成故障转移。
- 查看sentinel队长的日志，3个sentinel中随机选举队长：
    - `+sdown ..7007`：每个sentinel都对7007添加1枚主观下线标记。
    - `+odown ..7007`：7007的主观标记超过2枚，对7007添加客观下线标记。
    - `try-failover .. 7007`：尝对7007进行故障转移。
    - `+vote-for-leader xxx 1`：投1票给xxx作为sentinel队长。
    - `+elected-leader ..7007`：本人当选sentinel队长。
    - `+failover-state-select-slave ..7007`：准备为7007选择一个主节点的候选人。
    - `+select-slave ..7009`：选择7009作为新master的候选人。
    - `+failover-state-send-slaveof-noone ..7009`：取消7009的slave状态。
    - `+failover-state-wait-promotion ..7009`：通知7009等待晋升。
    - `+promoted-slave ..7009`：7009成功晋升为master。
    - `+failover-state-reconf-slaves`：重写7007的配置信息，对其降级。
    - `+slave-reconf-sent/inprog/done ..7009`：重写7009的配置信息，对其晋升。
    - `-odown ..7007`：取消对7007添加的客观下线标记，准备下一次重新标记。
    - `+failover-end ..7007`：对7007的故障转移过程圆满结束。
    - `+switch-master ..7007 ..7009`：完成master从7007到7009的切换。
    - `+slave ..7008 ..7009`：将7008设置为7009的slave节点。
    - `+slave ..7007 ..7009`：将7007设置为7009的slave节点。
    - `+sdown ..7007`：7007还没恢复，再添加1枚主观下线标记。
- 查看sentinel任一队员的日志：
    - `+sdown ..7007`：每个sentinel都对7007添加1枚主观(subjective)下线标记，即我觉得7007好像挂了。
    - `+odown ..7007`：7007的主观标记超过2枚，对7007添加客观(objective)下线标记，即7007确实挂了。
    - `+vote-for-leader xxx 1`：投1票给xxx作为sentinel队长。
    - 等待sentinel队长完成故障转移操作。
    - `+config-update-from sentinel ..7007`：同步接收故障转移的结果。
- 重新上线7007，再次查看全部sentinel日志：
    - 发现三个sentinel都删除了对7007的主观下线。
    - 其中一个sentinel执行了 `+convert-to-slave` 将7007变更为当前master的slave。

### Jedis哨兵模型

**武技：** 使用Jedis访问哨兵模式：
- 如果哨兵配置文件中配置了 `bind 127.0.0.1` 则有可能无法访问哨兵，请注释掉。
- 创建一个哨兵连接池：
    - `new JedisSentinelPool(主从结构名，哨兵set集合，连接池配置)`：构建sentinel连接池。
- tst：`jedis.JedisSentinelTest.testJedisSentinel()`

## 分区集群

**心法：** redis默认采取虚拟槽slot方式进行数据分区以分担redis服务器压力，集群仅能使用一个库db0，集群中每个节点的配置内容保持统一且定期检查一致性，
且cluster集群内部实现了故障转移机制，无需使用sentinel 即可完成主从切换：
- 总共设置16384个（0-16383）虚拟槽用于存储数据，尽量平均分配给每个master以避免数据倾斜。
- 存储数据时，先用key值对16383取余，根据结果存入对应的master中，获取数据同理。
- master间可以互知槽范围，slave不带槽。
- 在客户端提前计算key值的所在槽，并直接访问到对应的master中可以提高操作效率。

### ruby集群搭建

**心法：** ruby是一种简单快捷的面向对象的脚本语言，可用于对redis进行快速分区，可以省略meet和分配槽的工作：
- 手动创建6个集群节点：ruby方式需要至少6个节点。
    - 节点需要开启分区集群模式，指定集群配置文件，设置集群超时时间等。
- 使用ruby完成集群快速分区：
    - 用任一集群节点去meet其他集群节点：完成集群节点之间的互通。
    - 为主节点尽量平均地分配16384个槽，不分配槽的主节点无法使用且从节点不带槽。
    - 指定集群节点之间的主从关系。

**武技：** 使用ruby快速搭建redis分区集群：
- 安装ruby应用：
   - `rubyinstaller-3.0.2-1-x64.exe` 傻瓜式安装，勾选自动配置环境变量。
- 安装redis驱动：
    - cmd：`cd D:\redis\Ruby30-x64`：进入ruby家目录。
    - cmd：`gem install redis`：在ruby家目录中安装redis驱动。
- 新建 `ruby` 目录，将redis的客户端和服务端exe程序都拷贝到该目录下：
    - 将集群脚本 `redis-trib.rb` 分别拷贝到6个redis实例目录中：
    - res：`资源/附件/redis-trib.rb`：该脚本在redis源码的src目录中可以找到。
- 开发配置文件 `ruby/redis-7011.conf`：
    - `port 7011`：redis服务器端口号，默认端口6379。
    - `timeout 10`：若10s内客户端和服务端都没有进行数据交互则断开连接，默认0，表示永不断开。
    - `dir ./`：配置数据目录，RDB快照和AOF文件都会存储在这里。
    - `loglevel notice`：日志级别，可选 `debug/verbose/notice/warning`，建议默认。
    - `logfile 7011.log`：日志文件，默认空串，表示在控制台打印日志，支持使用绝对路径。
    - `save 300 10`：300秒内有10个以上的key值发生改变，则触发 `bgsave`，`save ""` 表示禁用，可写多条。
    - `dbfilename dump-7011.rdb`：配置RDB快照，默认为 `dump.rdb`，生成在工作目录中。
    - `stop-writes-on-bgsave-error yes`：最后一次 `bgsave` 失败时，主进程将停止接收数据加以警示，默认yes。
    - `rdbcompression no`：用LZF算法压缩RDB快照，节省硬盘空间，但会额外消耗CPU资源进行压缩操作，默认yes。
    - `rdbchecksum no`：用CRC64算法检查RDB快照，提升安全性，但会额外增加10%左右的CPU消耗，默认yes。 
    - `cluster-enabled yes`：开启分区集群模式。
    - `cluster-config-file nodes-7011.conf`：集群相关信息的配置文件。
    - `cluster-node-timeout 15000`：集群创建超时时间，单位毫秒。
    - `cluster-require-full-coverage no`：默认yes，表示当集群中有一个节点故障则整体不对外服务，建议关闭。   
- 开发其它5个集群节点的配置文件：`7012~7016.conf`：
    - 配置尽量和上面的7011节点的保持一致，只需要将7011全部替换成7012~7016即可。
- 分别将6个节点配置为windows服务并启动，以7011为例：
    - `redis-server --service-install 7011.conf --service-name redis7011`：部署服务。
    - `redis-server --service-start 7011.conf --service-name redis7011`：启动服务。
- 在 `ruby` 目录中搭建集群：
    - cmd：`ruby redis-trib.rb create --replicas 1 127.0.0.1:7011 127.0.0.1:7012 127.0.0.1:7013 127.0.0.1:7014 127.0.0.1:7015 127.0.0.1:7016`：数字1表示1主1从模型，0表示没有从节点。
    - 提示 `Can I set the above configuration? (type 'yes' to accept)` 时输入yes回车即可。
- 在任一节点查看集群信息，槽信息和主从关系，以7011为例：
    - 7011：`cluster nodes`：查看集群节点。
    - 7011：`cluster info`：查看集群信息。
- 在 `ruby` 目录中查看任一节点的集群信息：
    - cmd：`ruby redis-trib.rb info 127.0.0.1:7011`
- 连接集群中任意一个master节点并操作数据，以7011为例：
    - cmd：`redis-cli -c -p 7011`：`-c` 表示集群方式连接以自动重定向到key对应槽位所在的节点并执行命令。
    - 7011：`set name zhaosi`：多测几组数据，观察重定向现象。

### ruby集群伸缩

**心法：** 集群伸缩指得就是槽和数据在节点之间的迁移，数据量越大迁移过程越慢，但不会影响程序正常运行。

**武技：** 对集群进行扩容2个节点，假设要添加7017/7018节点：
- 将redis根目录整体拷贝2份 `redis-7017~7018`，预设1主1从：
    - 配置同其他节点，并分别拷贝 `redis-trib.rb` 脚本文件。
- 将2个新节点部署成windows服务并启动，以7017为例：
    - cmd：`redis-server --service-install 7017.conf --service-name redis7017`
    - cmd：`redis-server --service-start 7017.conf --service-name redis7017`
- 将2个新节点加入到集群中，以7017为例：
    - 7011：`cluster meet 127.0.0.1 7017`
- 对2个新节点配置主从关系：将7018配置为7017的从节点：
    - 7018：`cluster replicate 7017的nodeId`
- 对新的主节点如7017进行分配槽：
    - cmd：`ruby redis-trib.rb reshard 127.0.0.1:7017`
    - 提示 `how many slots do you want to move?`：给新的主节点总共分配16384/4=4096个槽。
    - 提示 `what is the receiving node ID`：输入7017节点ID，该节点用于接收槽数据。
    - 提示 `please enter all the source node IDs`：输入all，表示所有节点都为新节点分配槽数据，并生成计划。
    - 提示 `do you want to proceed with the porposed reshard plan`：输入yes执行计划。
- 查看集群节点信息：7017的三段槽分别来自于其他所有节点：
    - 7011：`cluster nodes`

**武技：** 对集群进行收缩2个节点，假设要删除7017/7018节点：
- 将7017的1366+1365+1365个槽尽量平均迁移到7011/7012/7013，以迁移到7011为例：
    - cmd：`ruby redis-trib.rb reshard --from 7017的id --to 7011的id --slots 1366 127.0.0.1:7011`
- 从集群中删除7017/7018节点，为了避免触发故障转移，先删除从节点，后删除主节点，以7018为例：
    - cmd：`ruby redis-trib.rb del-node 127.0.0.1:7018 7018的id`
- 查看集群节点信息：7018和7017已被移除集群：
    - 7011：`cluster nodes`

### Jedis分区集群

**武技：** 使用Jedis访问分区集群：
- 如果集群配置文件中配置了 `bind 127.0.0.1` 则有可能无法访问哨兵，请注释掉。
- 创建一个集群连接池：
    - `new HostAndPort("127.0.0.1", 7011)`：创建地址实例。
    - `new JedisCluster(地址实例set集合, 超时毫秒数, 池配置)`：创建一个集群实例。
    - `jedisCluster.getClusterNodes()`：获取集群中的全部节点。
    - `jedis.info("replication")`：返回节点 `replication` 信息。
- tst：`jedis.JedisClusterTest.testJedisCluster()`

## Redis客户端模板

**心法：** `o.s.d.r.c.StringRedisTemplate` 类高度封装了Jedis的API且可以自动管理连接池，使用比Jedis简单但效率比原生Jedis低：
- `template.hasKey(KEY)`：返回指定key是否存在，全局命令。
- `template.delete(KEY)`：删除指定key，全局命令。
- `template.expire(KEY, 过期时间, 时间单位)`：设置指定key的过期时间，全局命令。
- `template.getExpire(KEY)`：返回指定key的过期时间，全局命令。
- `template.boundValueOps(KEY)`：绑定指定string类型的key，后续可进行 `set/get/increment` 等操作。
- `template.boundHashOps(KEY)`：绑定指定hash类型的key，后续可进行 `put/increment/expire/delete` 等操作。
- `template.boundListOps(KEY)`：绑定指定list类型的key，后续可进行 `rightPush/range/index/size` 等操作。
- `template.boundSetOps(KEY)`：绑定指定set类型的key，后续可进行 `add/isMember/members` 等操作。
- `template.boundZSetOps(KEY)`：绑定指定sorted_set类型的key，后续可进行 `add/rangeWithScores` 等操作。

**武技：** 使用StringRedisTemplate连接redis服务端并操作数据：
- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-data-redis`
- 在主配中配置单机redis：均以 `spring.redis` 为前缀：    
    - `timeout=6379`：连接超时时间，单位毫秒。
	- `host=127.0.0.1`：单机redis-server的主机地址，默认127.0.0.1。
	- `port=6379`：单机redis-server的端口，默认6379。
	- `password=123`：单机redis-server的认证密码，默认为空字符串。   
    - `jedis.pool.max-active=1024`：连接池最大连接数，默认8，-1表示无限制。
    - `jedis.pool.max-idle=200`：连接池最大空闲连接数，默认8，-1表示无限制。
    - `jedis.pool.min-idle=0`：连接池最小空闲连接数，默认0，-1表示无限制。
    - `jedis.pool.max-wait=10000`：连接池最大阻塞等待时间，单位毫秒，默认-1表示永不超时。
- 在主配中配置集群redis：均以 `spring.redis` 为前缀：    
    - 将单机配置中的 `host/port/password` 删除，其余配置项不变。
    - 添加 `cluster.nodes=127.0.0.1:7011,127.0.0.1:7012,127.0.0.1:7013 ...` 即可，主从都配。
- tst: `jedis.StringRedisTemplateTest.*`：
    - 测试类标记 `@SpringBootTest(classes = RedisApp.class)` 以开启SpringBootTest模式。
    - 类中使用 `@Autowired` 直接注入 `StringRedisTemplate` 即可：
- 重连日志问题：lettuce的 "心跳检测" 方案是将连接池中的空闲客户端每隔一段时间就主动断开重连，所以在启动项目后，控制台会不断打印重连redis的INFO级别日志 `reconnect`，若不想看到的话将这个日志级别调高一点就行了：
    - 在logback.xml中添加 `<logger name="io.lettuce.core.protocol" level="ERROR"/>` 即可。


# 第三阶：ElasticSearch

**心法：** [ElasticSearch](https://www.elastic.co) 是基于 lucene 核心并由java开发的一款开源的检索引擎，支持全文检索，分布式检索，数据统计分析，
内容纠错和提示等功能，可扩展到上百台服务器，可处理PB级数据，且检索延迟通常在1秒以内，常用于日志搜索，数据聚合，数据监控，报表统计分析等业务：

- `node` 节点：1个启动的ES服务即为一个ES节点，默认名随机生成，用于存储，索引，搜索数据。
- `cluster` 集群：1或N个ES节点可构成1个ES集群，默认名elasticsearch，集群间共享数据，均可操作数据。
- `index` 索引：对应DB实例database，索引名建议全小写，包含1或N个type。
- `type` 文档类别：对应DB表table，默认名 `_doc`，ES7+中已被废弃，包含1或N个document。
- `document` 文档：对应DB表记录record，采用JSON格式存储数据。
- `field` 字段：对应DB表列column，是构成document文档的单元。
- `mapping` 映射：index的配置项，如field类型约束，某些数据的使用规则等。
- `shard` 分片：将index拆成多个shard可以水平切分数据以提升整体查询性能：
    - shard数量必须在index创建时指定，默认为1个，index创建后不可动态修改。
    - shard也是一个功能完整的index，最多存放int最大值减去128个文档。
- `replica` 分片副本：每个shard可以配置1或N个replica副本以提升高可用性和并发性能：
    - replica数量可以在index创建时指定，默认为1个，index创建后也仍可动态修改。

## ES服务端搭建

**武技：** 安装ES服务端，需要提前检查JDK环境：

- 下载[ES服务端](https://www.elastic.co/cn/downloads/elasticsearch)：
    - res：`elasticsearch-7.11.2-windows-x86_64.zip`：解压缩到硬盘。
- 启动ES服务端：
    - 双击 `%ES_HOME%\bin\elasticsearch.bat`：提示 `started` 表示启动成功。
- 访问ES服务端：
    - cli：`localhost:9200`：返回一个JSON格式的ES服务的基本信息。

## Kibana客户端搭建

**武技：** 安装ES客户端kibana，需要提前检查JDK环境：

- 安装node环境：傻瓜式安装：
    - res：`node-v16.14.0-x64.msi`：双击启动。
    - cmd：`node -v`：查看node是否安装成功。
- 下载[kibana](https://www.elastic.co/cn/downloads/past-releases#kibana)：必须和ES服务端版本一致，下载后傻瓜式安装：
    - res：`kibana-7.11.2-windows-x86_64.zip`：解压缩到硬盘，速度很慢，耐心等待。
- 配置Kibana：修改 `%KIBANA_HOME%\config\kibana.yml` 文件：
    - `i18n.locale："zh-CN"`：kibana界面使用中文。
    - `elasticsearch.requestTimeout: 100000`：连接ES服务端的超时时间。
- 启动kibana：
    - 双击 `%KIBANA_HOME%\bin\kibana.bat`：控制台展示 `Server running..` 表示启动成功。
    - 如果提示超时，重启ES服务端，然后重连即可。
- 访问kibana管控台：
    - cli：`localhost:5601`

## REST基本命令

**心法：** REST请求类型中，GET查询，POST添加，PUT修改，DELETE删除：

- 索引操作：
    - `PUT i_user`：创建 `i_user` 索引，重复创建报错，该命令不支持POST。 
    - `GET i_user`：查看 `i_user` 索引。
    - `GET _cat/indices?v`：查看节点中全部索引。
    - `DELETE i_user2`：删除 `i_user2` 索引，索引不存在报错。
- 文档操作：索引中的文档类型type和文档id需要同时指定，重复创建视为修改：
    - `POST i_user/_doc/1 {"age"：18}`：添加索引 `i_user` 中的1号文档内容，body需要另起一行编写。
    - `GET i_user/_doc/1`：查看索引 `i_user` 中的1号文档。
    - `DELETE i_user/_doc/1`：删除索引 `i_user` 中的1号文档。

**武技：** 向索引i_user中添加6条数据：

- 启动es9200节点和kibana客户端：
    - 点击kibana左侧 `Dev Tool` 进入kibana控制台。
- `POST i_user/_doc/1~6`：向索引i_user中添加6条数据：
    - `{"id":1, "name":"赵四", "gender":"male", "age":58, "info":"亚洲舞王"}`
    - `{"id":2, "name":"刘能", "gender":"male", "age":59, "info":"村副主任"}`
    - `{"id":3, "name":"王云", "gender":"female", "age":18, "info":"刘夫人"}`
    - `{"id":4, "name":"谢广坤", "gender":"male", "age":60, "info":"广坤山货"}`
    - `{"id":5, "name":"谢兰", "gender":"female", "age":60, "info":"校长老婆"}`
    - `{"id":6, "name":"李大国", "gender":"male", "age":32, "info":"卡车司机"}`
- 在kibana左侧导航中点击 `Home` 进入主页，然后点击左上角 `管理` 进入管理界面。
- 在管理界面左侧导航中点击 `索引模式` 以创建索引规则：
    - 随意输入索引名称，创建好之后可以看到索引中的全部属性。
- 在kibana左侧导航中点击 `Discover`，选择刚才创建的索引规则，即可查看该索引下的全部数据。

## REST全文检索

**心法：** 全文检索支持配合query增加检索条件，支持配合aggs进行聚合查询：

- `GET i_user/_search`：检索i_user索引中的全部文档：
    - `took`：本次检索耗时，单位毫秒。
    - `timed_out`：本次检索是否超时。
    - `_shards`：本次检索的分片情况，包括总计/成功/跳过/失败检索了少分片。 
    - `hits`：本次检索命中的数据情况，包括命中数量/命中关系/命中匹配度分数/具体命数数据等。
- `{"from":0,"size":3}`：对结果集分页，从第0条文档开始，检索并显示3条文档，默认10条。
- `{"sort":[{"_score":"asc"},{"age":"desc"}]}`：结果集按匹配分数升序，分数相同按年龄降序：
    - ES默认使用_score降序，使用其他字段排序时分数排序失效，需要显示设置_score排序规则。
    - 尽量只使用数字进行排序，若非要对字符串排序，用 `name.keyword` 替换 `name`。
- `{"query":{"match":{"name":"谢广"}}}"`：name中必须包含 `谢` 或 `广`。
- `{"query":{"match_phrase":{"name":"谢广"}}}`：name中必须包含 `谢广` 短语。
- `{"query":{"bool":{"must":{"match":{"name":"谢"}}}}}`：name中必须包含 `谢`。
- `{"query":{"bool":{"must_not":{"match":{"id"：1}}}}}`：文档id必须不为1。
- `{"query":{"bool":{"filter":{"range":{"age":{"gte":1,"lte":9}}}}}}`：age必须在1-9之间。
- `{"aggs":{"key值":{"sum":{"field":"age"}}}}`：返回全部文档中的所有age之总和，key值命名随意。
- `{"aggs":{"key值":{"min":{"field":"age"}}}}`：返回全部文档中的所有age之最小值，key值命名随意。
- `{"aggs":{"key值":{"max":{"field":"age"}}}}`：返回全部文档中的所有age之最大值，key值命名随意。
- `{"aggs":{"key值":{"avg":{"field":"age"}}}}`：返回全部文档中的所有age之平均值，key值命名随意。
- `{"aggs":{"key值":{"value_count":{"field":"age"}}}}`：返回全部文档中有多少个age，key值命名随意。    
- `{"aggs":{"key值":{"cardinality":{"field":"age"}}}}`：返回全部文档中有多少种age，key值命名随意。    
- `{"aggs":{"key值":{"stats":{"field":"age"}}}}`：返回全部文档中的所有聚合属性，key值命名随意。
- `{"aggs":{"key值":{"terms":{"field":"gender.keyword"}}}}`：按性别分组，返回每组数量，key值命名随意：
    - 支持在terms同级再次用aggs来指定单聚合函数，如返回按性别分组后每组的最小年龄等。

## IK中文分词器

**心法：** ES自带的标准分词器仅对英文分词合理，对中文采取逐字拆分法，此时建议安装中文的IK分词器插件已解决中文分词问题：

- `ik_smart`：IK最小拆分法，检索时建议使用。
- `ik_max_word`：IK最大拆分法，存储时建议使用。

**武技：** 在ES中搭建IK分词器插件：
- 下载IK分词器并解压到 `%ES_HOME%\plugins` 中：
    - `elasticsearch-analysis-ik-7.11.2.zip`：版本建议和ES服务端一致。
- 重新启动ES服务和kibana服务。
- `GET i_user/_analyze`：必须添加body才能看到效果：
    - `{"text":"我是一个中国人"}`：逐字拆分法。
    - `{"analyzer":"ik_smart","text":"我是一个中国人"}`：IK最小拆分。
    - `{"analyzer":"ik_max_word","text"："我是一个中国人"}`：IK最大拆分。

## ES集群搭建

**武技：** 创建 `D:\elasticsearch\elasticsearch-cluster` 集群目录：

- 拷贝两个ES服务端根目录到集群目录内，分别命名 `es-node-a/es-node-b`，视为两个ES节点：
    - 将两个ES节点中的 `data` 目录全部删除以保证节点数据一致性。
- 修改两个ES节点的配置 `config/elasticsearch.yml`：不要在yml中添加注释：
    - `cluster.name:es-cluster`：集群名，默认 `elasticsearch`，必须保证两个ES节点的集群名一致。
    - `node.name:es-node-a`：节点名，必须保证两个ES节点的名称不一致。
    - `node.master:true`：该节点拥有成为master的资格。
    - `node.data:true`：该节点允许存储数据。
    - `network.host:127.0.0.1`：该ES节点所在的服务器IP，`0.0.0.0` 表示允许所有IP对我访问。
    - `http.port:9201`：该ES节点的端口号，默认9200。
    - `transport.tcp.port:9301`：集群间通信端口号，必须保证与ES节点端口号不同。
    - `discovery.zen.ping.unicast.hosts:["127.0.0.1:9301", "127.0.0.1:9302"]`：集群节点列表。
    - `cluster.initial_master_nodes:es-node-a`：配置初始主节点名。
    - `http.cors.enabled:true`：允许被跨域访问，默认false。
    - `http.cors.allow-origin:"*"`：允许所有域名对我跨域访问。
- 同时启动两个ES节点：通过任一节点测试：
    - cli：`localhost:9201/9202`：分别查看两个节点信息。
    - cli：`localhost:9201/_cat/health?v`：查看ES集群的健康状态。
    - cli：`localhost:9202/_cat/nodes?v`：查看ES集群的节点列表。 

## ES客户端模板

**心法：** `o.s.d.e.c.ElasticsearchRestTemplate` 是ES常用的一个Java客户端模板：

- `elasticsearchRestTemplate.save(new Book() ...)`：用于添加1或N个文档，返回添加的文档集合。
- `elasticsearchRestTemplate.get(id, Book.class)`：按主键查询文档数据。
- `elasticsearchRestTemplate.exists(id, Book.class)`：查询指定主键的文档是否存在。
- `elasticsearchRestTemplate.delete(id, Book.class)`：按主键删除文档数据。          

**武技：** springboot整合ES并测试ElasticsearchRestTemplate模板的常用API方法：

- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-data-elasticsearch`
- 在主配中指定ES服务器或集群，以及忽略CustomConversions的日志警告：
    - `spring.elasticsearch.rest.uris：http://127.0.0.1:9200`
    - `logging.level.org.springframework.data.convert.CustomConversions=ERROR`
- 开发实体类 `entity.Book`： 
    - 实体类上标记 `@Document(indexName="索引名")`：一个实体类对应一条文档，索引自动创建。
    - 主键属性标记 `@Id`：按主键查询时会使用该字段。
    - 普通属性标记 `@Field(type=FieldType.Text)`：type属性用于指定字段类型。
- 开发测试类：
    - 标记 `@RunWith(SpringRunner.class)`：指定启动方式。
    - 标记 `@SpringBootTest(classes = RocketMqApp.class)`：指定启动类。
    - 使用 `@Autowired` 注入ElasticsearchRestTemplate类。
    - 测试 `ElasticsearchRestTemplate` 模板的常用API方法。
    - tst: `app.ElasticsearchRestTemplateTest.testBaseApi()`

### ES条件计数

**武技：** 测试ElasticsearchRestTemplate的条件计数：

- 构建条件查询器：
    - `QueryBuilders.matchPhraseQuery("info", "word")`：将word当成短语模糊匹配info值。
- 构建本地搜索器：
    - `new NativeSearchQueryBuilder().withQuery(条件查询器).build()`：绑定条件查询器。
- 根据本地搜索器进行条件计数：
    - `elasticsearchRestTemplate.count(本地搜索器, Book.class)`
- tst: `app.ElasticsearchRestTemplateTest.testCountByInfo()`

### ES条件删除

**武技：** 测试ElasticsearchRestTemplate的条件删除：

- 构建条件查询器：
    - `QueryBuilders.matchQuery("name", "word")`：将word逐字拆开并依次模糊匹配name值。 
- 构建本地搜索器：
    - `new NativeSearchQueryBuilder().withQuery(条件查询器).build()`：绑定条件查询器。
- 根据本地搜索器进行条件删除：
    - `elasticsearchRestTemplate.delete(本地搜索器, Book.class)`
- tst: `app.ElasticsearchRestTemplateTest.testDeleteByName()`：通过Kibana查看是否删除成功。

### ES条件搜索

**武技：** 测试ElasticsearchRestTemplate 的条件搜索：

- 构建条件查询器：
    - `QueryBuilders.matchQuery("info", "word")`：将word逐字拆开并依次模糊匹配name值。
    - `QueryBuilders.queryStringQuery(word).field("info")`：将word逐字拆开并依次模糊匹配info值。
- 构建本地搜索器：
    - `new NativeSearchQueryBuilder().withQuery(条件查询器).build()`：绑定条件查询器。
- 本地搜索器额外配置（均可选），需要在 `.build()` 之前进行配置：
    - `.withPageable(PageRequest.of(0, 2))`：仅显示从第1条开始，满足条件的2条数据。
    - `.withSort(SortBuilders.fieldSort("_score").order(SortOrder.DESC))`：结果集按分数倒序（默认）。
    - `.withHighlightFields(new HighlightBuilder.Field("info"))`：info中符和条件的词语高亮（默认倾斜）。
- 根据本地搜索器进行搜索：
    - `elasticsearchRestTemplate.search(本地搜索器, Book.class)`：返回值可用Object类型接收并直接响应。
    - `books.forEach(System.out::println)`：可以对返回值调用 `forEach()` 遍历查看。 
- tst: `app.ElasticsearchRestTemplateTest.testSearch()`：未使用IK分词，逐字拆分后进行搜索。

### ES分词搜索

**武技：** 测试ElasticsearchRestTemplate的分词搜索：
- 在kibana配置IK分词：需要手动更改索引的mapping内容：
    - `GET i_book`：查看并复制全部mapping内容，其他的不要复制。
    - `DELETE i_book`：删除索引。
    - `PUT i_book`：重新创建该索引，在body中粘贴mapping内容：
        - 在 `info` 字段中额外添加 `"analyzer"："ik_smart"` 配置。
    - `GET i_book`：再次查看，发现分词器已经作用在该字段上。
- 对实例类的info属性的 `@Field` 额外添加分词规则属性：
    - `analyzer = "ik_max_word"`：设置存储该字段时候采用IK最大拆分。
    - `searchAnalyzer = "ik_smart"`：设置查询该字段采用IK最小拆分。
- tst: `app.ElasticsearchRestTemplateTest.testSearch()`：使用IK分词，以词为单位进行搜索。

# 第四阶：Nginx

**心法：** nginx是一款开源的服务器和反向代理服务器，邮件服务器，支持很多第三方的模块扩展：
- 正向代理：客户端的代理，对客户端负责：
	- 代理流程：客户端发送请求到代理服务器，代理服务器转发请求到具体服务器，响应也是原路返回。
	- 破解限制：某服务器限制客户端直接访问，则可以找个可访问的代理服务器来访问该服务器，如科学上网。
	- 加速访问：电信客户端访问联通服务器太慢，则可以找个电信联通都能访问的代理服务器调节，如游戏加速器。
	- 缓存数据：每次请求获取的数据都可以缓存到代理服务器中，后续相同的请求可以直接从缓存中获取。
	- 保护客户端隐私：服务端仅知道请求来自于哪个代理服务器，但并不知道请求具体来自于哪个客户端。
- 反向代理：服务端的代理，对服务端负责：
	- 代理流程：代理服务器接收到请求之后，按一定规则分发给某个具体的服务器，响应也是原路返回。
	- 负载均衡：代理服务器可以使用轮询，加权轮询，ip_hash，url_hash等策略分发请求，分摊服务器集群的压力。
	- 保护服务端隐私：客户端仅知道请求发送给了哪个代理服务器，但并不知道请求具体由哪台服务器进行处理。
- nginx服务端工作模式：nginx采取多进程模式，启动nginx服务时，后台会至少启动2个进程，分为两类：
	- master：守护进程，负责向worker转发外界信号，监控worker状态，当worker异常时，自动重启新的worker进程。
	- worker：工作进程，负责监听端口和处理用户请求，worker之间相互隔离独立，worker数量建议配置与CPU核数一致。
- nginx服务端启动流程：以下对 `%NGINX_HOME%/logs/nginx.pid`，简称PID文件：
	- 使用命令启动nginx服务时，自动生成PID文件，该文件用于记录nginx中master进程的ID号。
	- 双击程序启动nginx服务时，不会生成PID文件，此时只能在任务管理器中手动结束相关进程，不建议。
	- 使用命令关闭nginx服务时，会根据PID文件内容来结束nginx所有后台进程，之后自动删除PID文件。
	- 多次启动nginx服务，不会端口占用，但会导致PID文件内容发生覆盖，进而导致之前的nginx进程失联并泄露。

## 安装Nginx服务器

**武技：** 安装nginx服务器：
- 下载nginx for windows：
	- res：`nginx-1.20.2.zip`：解压缩即可。
- 启动nginx服务：以下对 `%NGINX_HOME%/logs/nginx.pid`，简称PID文件：
	- cmd：`cd D:\nginx\nginx-1.20.2`：切换到nginx家目录。
	- cmd：`start nginx`：启动nginx服务并生成PID文件。
	- cmd：`tasklist | findstr "nginx*"`：查看nginx服务启动的两个 `nginx.exe` 后台进程。
	- cmd：`netstat -ano | findstr "80"`：确认nginx默认的80端口确实正在被worker进程监听着。
	- cmd：`taskkill /pid 进程号 /f`：强制结束某个后台进程。
	- cmd：`nginx -s stop`：关闭nginx服务，结束所有相关进程，并删除PID文件，若PID文件不存在则报错。
- 访问nginx服务器，查看环境界面：
	- cli：`localhost:80`：默认端口号80，80可省略。
- 修改nginx端口号：若windows上安装了IIS服务器，则会和80端口冲突：
	- 在nginx主配文件 `%NGINX_HOME%/conf/nginx.conf` 中修改 `server -> listen` 的值为 `81`。
	- cmd：`nginx -s reload`：重新加载配置文件，此时无需重启nginx进程，但若PID文件不存在则报错。
	- cli：`localhost:81`：出现nginx欢迎界面，表示配置成功。 

## 使用nginx代理项目

**武技：** 创建springboot项目 `nginx-01` 和 `nginx-02`：
- 分别添加依赖：添加springboot父项目：
	- `org.springframework.boot.spring-boot-starter-web`：web组件。
	- `org.springframework.boot.spring-boot-devtools(runtime)(true)`：热部署组件。
	- `org.springframework.boot.spring-boot-starter-test(test)`：测试组件。
	- `org.projectlombok.lombok(true)`：lombok组件。
  
- 分别主配添加：
	- `spring.application.name=nginx-01`：项目名分别设置为 `nginx-01` 和 `nginx-02`。
	- `server.port=13001`：端口号分别设置为 `13001` 和 `13002`。
  
- 开发控制方法 `app.controller.TestController.getSession()`：
	- param1：直接获取 `HttpSession` 实例，对其调用 `getId()` 可获取其ID号。
	- return：返回当前项目端口号信息以及本次sessionID号。
  
- 开发启动类并测试接口：
	- cli：`localhost:13001/api/test/get-session`
	- cli：`localhost:13002/api/test/get-session`
- 修改nginx主配 `%NGINX_HOME%/conf/nginx.conf`：
	- 在 `http{}` 中使用 `upstream` 负载均衡器定义集群服务列表配置区，如 `upstream apps{}`，均衡器名随意。	
	- 在 `apps{}` 中使用 `server 127.0.0.1:13001 weight=1;` 配置nginx-01服务，weight表示轮询权重。
	- 在 `apps{}` 中使用 `server 127.0.0.1:13002 weight=1;` 配置nginx-02服务，weight表示轮询权重。
	- 在 `location / {}` 中添加 `proxy_pass http://apps;`：拦截全部请求并代理转发到集群服务列表 `apps` 中。
- 启动nginx服务并测试负载均衡效果：
	- cmd：`nginx -s reload`：重新加载配置文件，若是重新启动的nginx服务则可忽略。
	- cli：`localhost:81/api/test/get-session`：使用nginx多次代理访问控制方法，发现默认负载均衡策略为轮询。

## 配合redis完成session共享

**武技：** 设置nginx代理下，使用redis完成session共享：
- 分别对两个项目添加依赖：
	- `org.springframework.boot.spring-boot-starter-data-redis`：redis开发组件。
	- `org.springframework.session.spring-session-data-redis`：redis-session存储组件。
- 分别对两个项目主配添加：
	- `spring.redis.host=localhost`：redis服务器IP地址。
	- `spring.redis.host=6380`：redis服务器端口号。
	- `spring.session.store-type=redis`：设置session的存储方式设置为redis，默认redis，关闭使用none。
- 重启两个项目并使用nginx多次代理访问控制方法：
	- `localhost:81/api/test/get-session`：发现完成session共享。
	- `6380> keys *session*`：在redis数据库中查询是否自动存入了session信息。

## 详解nginx配置文件

**心法：** nginx配置文件默认可分为5部分，其中每个指令都必须用分号结束：
- 全局区：配置影响nginx全局的指令：第1项是linux配置：
    - `user nobody`：启动worker进程的用户和组，windows上不能写该配置。
    - `worker_processes 1`：生成的worker进程数，设置为auto时可以自动设置为机器的CPU核数。
    - `error_log logs/error.log`：错误日志文件，默认 `logs/error.log`，支持额外指定日志级别。
    - `pid logs/nginx.pid`：PID文件，默认 `logs/nginx.pid`，用于记录当前master进程的id号。
- events区：配置影响nginx服务器或与用户的网络连接，第234项是linux配置：
    - `worker_connections 1024`：每一个worker进程能并发处理的最大连接数，包括前后端所有连接。
    - `use epoll`：设置处理连接请求的事件驱动模型，可选 `select/poll/kqueue/epoll` 等，默认 `epoll` 模型。
    - `accept_mutex on`：规避惊群，即一个请求会同时唤醒全部睡眠进程（但只有一个进程能获得连接），以提高性能。
    - `multi_accept on`：设置一个进程可以同时接受多个请求，默认为off，一个进程一次只能接收一个请求。
- http区：可以嵌套多个server区，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置：
    - `include mime.types`：在本配置中导入 `mime.types` 文件，该文件是扩展名与文件类型的映射表。
    - `default_type application/octet-stream`：nginx不识别的文件类型会默认进行下载操作，默认 `text/plain`。
    - `access_log logs/access.log`：访问日志文件，默认 `logs/access.log`，采用 `log_format` 配置的格式。
    - `log_format xxx`：访问日志格式，默认 `客户端IP 用户 访问时间 请求信息/状态 内容大小 请求源头 浏览器`。
    - `sendfile on`：开启高效文件传输模式，默认off，普通应用建议开启，下载相关的应用建议关闭。
    - `tcp_nopush on`：开启sendfile的情况下，会将请求合并，然后统⼀发送给客⼾端。
    - `keepalive_timeout 65`：http连接超时时间，默认65秒，上传文件时设置大一些，避免超时断开连接而上传失败。
    - `gzip no`：对网络传输的数据内容进行压缩以提升传输效率。
- upstream区：用户自定义配置服务集群，集群名不能重复，集群中每个server都可以额外添加以下设置：
    - `weight=1`：设置该服务的权重，权重越大，访问的机率越大，压力也就越大。
    - `down`：设置该服务不参与负载均衡。
    - `max_fails=2`：设置该服务允许请求失败的最大次数，超出次数后返回 `proxy_next_upstream` 中定义的错误。
    - `fail_timeout=30s`：设置该服务请求失败后暂停访问的超时时间。
    - `backup`：设置服务为备用服务，即当其它服务全忙或宕机时，该服务才会被启用。
- server区：配置一个虚拟代理服务器的相关参数，一个http区中可以配置多个server区：
    - `listen 80`：配置服务器监听的端口号，默认监听80端口。
    - `server_name localhost`：配置服务器名，该名需要在机器的 `hosts` 文件中映射到具体IP地址才能使用。
    - `charset utf-8`：设置编码格式，默认是俄语编码，建议改为 `utf-8` 以避免参数中文乱码。
    - `error_page 500 502 503 504 /50x.html`：爆发500/502/503/504错误时，返回 `html/50x.html` 页面。
- location区：配置请求的访问规则，匹配的是域名之后的uri部分，同一个server区中可以配置多个location区：
    - `root html`：配置请求资源在服务器上的真实路径，支持使用相对nginx家目录的相对路径或绝对路径。
    - `index index.html index.htm`：配置首页列表，会在root指定的目录下寻找。
    - `proxy_pass http://apps`：请求转发到apps服务集群列表，并使用负载均衡的方式进行调用。
    - `deny 192.168.0.1`：可以配置拒绝的IP地址。
    - `allow 192.168.0.2`：可以允许的拒绝的IP地址。

## 详解location匹配规则

**心法：** location匹配顺序：内容匹配度越高，优先级就越高：
- 首先精准匹配：匹配成功立刻停止继续匹配。
- 然后正则匹配：按照编写顺序依次匹配，匹配成功立刻停止继续匹配。
- 最后前缀匹配：只保留匹配得分最高的那一个。

**武技：** 测试location匹配规则：windows下的nginx服务器对大小写不敏感，均可匹配成功：
- `location = /abc {}`：使用 `=` 精准匹配URI：
    - cli：`http://joezhou.com/abc`：精准匹配，成功。
    - cli：`http://joezhou.com/abc?name=jack`：忽略查询串，成功。
    - cli：`http://joezhou.com/abc/`：对末尾 `/` 敏感，失败。
    - cli：`http://joezhou.com/abcde`：不匹配，失败。
- `location /abc {}`：省略匹配修饰符，表示前缀匹配URI：
    - cli：`http://joezhou.com/abc`：前缀匹配，成功。
    - cli：`http://joezhou.com/abc?name=jack`：忽略查询串，成功。
    - cli：`http://joezhou.com/abc/`：前缀匹配，成功。
    - cli：`http://joezhou.com/abcde`：前缀匹配，成功。
- `location ^~ /abc {}`：效果和省略匹配修饰符一致，不同的是匹配成功后立即停止匹配。
- `location ~ ^/abc$ {}`：使用 `~` 区分大小写正则匹配URI，取反使用 `!~`：
    - cli：`http://joezhou.com/abc`：正则匹配，成功。
    - cli：`http://joezhou.com/ABC`：linux下匹配失败，windows下匹配成功。
    - cli：`http://joezhou.com/abc?name=jack`：忽略查询串，成功。
    - cli：`http://joezhou.com/abc/`：正则不匹配，失败。
    - cli：`http://joezhou.com/abcde`：正则不匹配，失败。
- `location ~* ^/abc$ {}`：使用 `~*` 忽略大小写正则匹配URI，取反使用 `!~*`：
    - cli：`http://joezhou.com/abc`：正则匹配，成功。
    - cli：`http://joezhou.com/ABC`：linux和windows下均匹配成功。
    - cli：`http://joezhou.com/abc?name=jack`：忽略查询串，成功。
    - cli：`http://joezhou.com/abc/`：正则不匹配，失败。
    - cli：`http://joezhou.com/abcde`：正则不匹配，失败。

