

数据库表 : 

chapter : 一个视频有很多章  : 

episode :  一章有很多集 : 

user : 用户表: 

video : 视频表

video_banner : 视频轮播图表 

video_order : 视频和订单的中间表 : 

- 当下单的时候, 订单微服务通过 openFeign远程调用用户微服务查询用户的信息,然后进行一系列的判断(todo : 业务处理)
- 然后往**订单视频中间表**中插入 一条购买记录: 这条记录包含一些 UserID,  VideoId, 和 VideoState 状态
- 购买完成之后会向MQ投递消息(给用户发一个邮件 这里用的RocketMQ )
- 这些操作都是在 seata 分布式事务保护中进行的,
- 然后用了 sentinel 进行 限流, 熔断,降级 操作

*******

# Lesson在线课堂 

## 项目分成三个子项目

项目简介:

项目名称: 在线课程视频交易平台. 主要用于课程视频的交易,分成三个子项目,包括管理员后台管理,用户后台管理,项目文档,前端.管理员可以通过管理员后台管理所有的表的CRUD,上传图片,视频,进行日志的查看备份等.用户可以购买视频,可以在线观看或下载课程视频.







### Lesson Doc

项目文档

仿造 swagger 做的,用于展示项目的接口Api 和 测试 Api



### lesson-admin

管理员后台

管理员登录之后,就可以进行

在这里使用到的技术主要有 :

  jdbc：连库 statement(运送SQL) resultset（结果集）
  dao：数据层：写SQL 发SQL
  service：业务层：逻辑处理
  servlet：跟前台交互
  html5：布局
  css3：美化
  javascript6：数据传输 

- 用户表的CRUD
- 视频表的增删改查
- video_banner 表的CRUD
- chapter表的CRUD
- episode表的CRUD
- play_Record的CRUD
- 视频的上传下载删除
- 轮播图的上传下载



### lesson-background

lesson-background

- 用户：
  - 登录  
    - jwt token 验证
  - 注册 
    -  头像的上传:
    - 可以点击预览大图
    - 进行了 Hibernate validator 参数的校验(关键信息不能为空(密码,账户,电话))
  - 查询个人信息 
    - 需要进行token 验证, 如果未登录,引导用户转到登录页面
    - 登录之后才能查看个人信息
  - 修改个人信息
    - 也是需要进行登录的
    - 可以修改 头像,电话 等...
  - 删除个人信息
    - mysql 的 触发器,当用户在删除自己的账户的时候,我们先把他的信息存入一个单独的表中,定期的导出.

- 视频：
  - 全查 
    - pageHelper 分页查询 
  - 视频详情 
    - 根据主键查看视频详情,如果视频不存在,返回错误信息(放在了video的 id title 的字段里面)
  - 购买 
    - 微服务的下单流程
  - 在线观看 
    - 登录之后才能看
    - 西瓜视频播放器
    - 可以发弹幕(WebSocket 技术)
  - 购物车 
    - redis 缓存和持久化
  - 搜索
    - ES 搜索引擎用来在视频列表中搜索视频
  - 订单：
    - 查询订单 
      - 还是要验证token ,登录之后才能查看.
    - 添加订单 
    - 删除订单

使用到的技术亮点:

 - 日志框架: logback 将日志使用定时任务在每周三晚上通过 IO流 读取日志文件，通过 logback框架拆分日志存储到ES中
 - Aop 前置（校验参数） 后置（日志） 返回后（脱敏，加密） 异常（环绕通知）
 - 本地缓存: 使用的时谷歌的 Guava(古哇)来 存储  轮播图
 - 使用 redis 实现了 购物车

>   mysql: 七张表CRUD
>   mybatis: annotation 使用注解开发
>   spring: IOC DI AOP  spring 三大特性
>   springmvc：接调存转 
>   定时：每周三晚上IO流读取日志文件，拆分logback日志框架，ES
>   异步：不占用主线程资源
>   异常：全局异常
>   AOP：前置（校验参数） 后置（日志） 返回后（脱敏，加密） 异常（环绕通知）
>   缓存：Guava（轮播图） redis（购物车）
>   ES：搜索视频
>   JWT：登录时校验
>   WS：弹幕
>   pageHelper：分页
>   拦截器：token校验 cors跨域
>   JSON格式化：Jackson
>   参数校验：Hibernate validator
>   redis：数据类型 持久化 哨兵 主从 集群
>   nignx：负载均衡

### lesson-foreground

 



lesson-doc

 **mysql:  联表 子查询 事务机制 隔离级别 触发器** 存储过程 优化**
  jdbc：连库 statement(运送SQL) resultset（结果集）
  dao：数据层：写SQL 发SQL
  service：业务层：逻辑处理
  servlet：跟前台交互
  html5：布局
  css3：美化
  javascript6：数据传输 

lesson-admin

  7张表的CRUD
  mybatis：xml
  service:
  springmvc：xml版本
  jquery：代码更少 
  bootstrap：响应式
  spring: IOC DI AOP

  上传下载

lesson-background

​	用户：登录 注册 查询个人信息 修改个人信息
​	视频：全查 视频详情 购买 在线观看 购物车 搜索 
​	订单：查询订单 添加订单 删除订单

  mysql:
  mybatis: annotation
  spring: IOC DI AOP
  springmvc：接调存转
  定时：每周三晚上IO流读取日志文件，拆分logback日志框架，ES
  异步：不占用主线程资源
  异常：全局异常
  AOP：前置（校验参数） 后置（日志） 返回后（脱敏，加密） 异常（环绕通知）
  缓存：Guava（轮播图） redis（购物车）
  ES：搜索视频
  JWT：登录时校验
  WS：弹幕
  pageHelper：分页
  拦截器：token校验 cors跨域
  JSON格式化：Jackson
  参数校验：Hibernate validator
  redis：数据类型 持久化 哨兵 主从 集群
  nignx：负载均衡

lesson-foreground

 vue3 + elementPlus

mall-alibaba

​	下单模块：分布式事务保护

  mybatis-plus
  jmeter： 压测 
  gataway 网关  限流 路由 断言
  nacos 注册发现  配置
  sleuth zipkin 链路追踪
  *************seata 分布式事务
  sentinel 保护 高可用 
  openfeign：远程调用
  rocketmq：下订单之后，发短信







