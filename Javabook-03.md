> Java道经第3卷

# 第一阶：WEB通信模型

**心法：** web通信就是基于HTTP协议，客户端向服务端发送Http请求 `HttpRequest`，服务端返回给客户端HTTP响应 `HttpResponse` 的过程：
- 。协议：web服务器必须遵守 `HTTP` 协议，其底层是 `TCP/IP` 协议：
  - `TCP` 负责将数据完整的送到目的地，尽管路途中可能会将数据拆成若干小块。
  - `IP` 负责把数据准确地送到目的地
- 客户端 `client`：可以统一指代浏览器，客户端或一个人类用户：
    - BS架构：从浏览器到服务端，使用方便，无需跟服务端同步更新。
    - CS架构：从客户端到服务端，需要跟服务端同步更新。
- 服务端 `server`：指物理主机硬件或WEB服务器应用软件，如Tomcat。
- res: `资源/图片/第2阶-springboot入门$WEB通信模型`

## Tomcat服务器

**心法：** Tomcat是Apache软件基金会的Jakarta项目中的一个核心项目产品，是一个开源的，运行时占用的系统资源很小的轻量级Web容器：
- 服务器其实是web容器和EJB程序的总称，但Tomcat没有EJB，所以只能勉强算一个服务器。
- res: `资源/残卷/残卷-Tomcat.md`

### 安装Tomcat服务器

**武技：** 下载并安装Tomcat服务器：
- 下载[Tomcat](https://tomcat.apache.org/tomcat-9.0-doc/index.html)：
    - 点击tomcat9版本的 `64-bit Windows zip (pgp, sha512)` 进行下载，直接解压缩即可。
- Tomcat主要目录结构：
    - `bin`：存放windows/Linux平台上Tomcat相关的可执行文件。
    - `conf`：存放Tomcat容器的各种全局配置文件，其中最重要的是 `server.xml` 和 `web.xml`。
    - `lib`：存放Tomcat执行时的一些jar包文件。
    - `logs`：存放Tomcat执行时的日志文件。
    - `temp`：存放Tomcat的临时文件。
    - `webapps`：Tomcat的Web项目发布目录。
    - `work`：存放JSP编译后产生的class文件。 
- 解决Tomcat9命令行日志中文乱码问题：
    - 打开 `%TOMCAT_HOME%\conf\logging.properties` 文件。
    - 将 `java.util.logging.ConsoleHandler.encoding` 修改为 `GBK`，兼容windows命令行。
- 启动Tomcat：点击 `%TOMCAT_HOME%\bin\startup.bat` 文件。
- 关闭Tomcat：点击 `%TOMCAT_HOME%\bin\shutdown.bat` 文件。

### 解决端口占用问题

**武技：** 模拟8080端口被占用并使用命令行解决：
- 查看占用8080端口的进程的PID：
    - cmd: `netstat -ano | findstr 8080`：结果集的最后一列为进程PID值。
- 通过PID查询是该进程的具体信息：
    - cmd: `tasklist | findstr PID`
- 通过PID强制关闭该进程（`/f`），及其子进程（`/t`）。
    - cmd: `taskkill /f /t /pid PID`

## 客户端请求

**心法：** 所谓请求，就是客户端想要从服务端获取某个资源，而响应就是服务端找到了这个资源并返回给客户端：
- 浏览器请求：一般浏览器仅支持GET和POST这两种请求：
    - `GET`：请求URL上的某资源，常用于查询。
    - `POST`：请求URL上的某资源，且要求服务器额外接收请求体中的数据，相同请求不覆盖，常用于添加。
    - `PUT`：请求URL上的某资源，且要求服务器额外接收请求体中的数据，相同请求会覆盖，常用于修改。
    - `DELETE`：请求删除URL上的某资源，常用于删除。
    - `HEAD`：请求URL上的某资源，但要求只返回响应头信息，不返回响应体。
    - `TRACE`：请求URL上的某资源，要求回送请求消息，以便客户端测试或者排错。
- 请求URL格式：`通信协议://服务器IP或名称:服务器端口/资源所在目录/资源?查询串`：
    - `通信协议://`：只要是web通信，就必须满足HTTP协议。
    - `服务器IP或名称:服务器端口`：指向某台物理服务器主机上的tomcat服务器软件。
    - `资源所在目录/资源`：指向tomcat服务器中的具体资源，若不存在会响应404。
    - `?查询串`：提供给服务器的一些KV格式的参数，可选。

### 浏览器GET请求

**心法：** GET请求没有请求体，速度更快，但不安全，且数据携带量小。

**武技：** 使用chrome查看一次GET请求：
- 开发一个文档 `index.html` 并编写一些内容。
- 将文档部署到 `%TOMCAT_HONE%\webapps\ROOT\` 下并启动tomcat。
- 打开chrome浏览器开发者模式，切换到 `Network` 选项卡。
- 直接在地址栏键入 `http://localhost:8080/index.html?a=1` 并回车以发送一次GET请求。
- 点击 `Network` 中出现的本次请求路径，可以观察到以下常用信息：
    - `Headers/General/RequestURL`：请求的URL，包括查询串。
    - `Headers/General/RequestMethod`：请求的方式。
    - `Headers/General/Status Code`：请求状态码和文字描述。
    - `Headers/Response Header/Content-Length`：响应数据的字节大小。
    - `Headers/Response Header/Content-Type`：响应的数据MIME类型。
    - `Headers/Response Header/Date`：响应发生的时间。 
    - `Headers/Request Header/Accept`：设定请求可接收的MIME数据类型列表，响应数据类型不匹配时拒绝。
    - `Headers/Request Header/User-Agent`：请求的浏览器信息。
    - `Headers/Query String Parameters`：查询串信息。
    - `Response`：响应体中的响应数据。

### 浏览器POST请求

**心法：** POST就是使用请求体携带额外数据的GET请求，牺牲了速度，但安全，且数据携带量更大。

**武技：** 使用chrome查看一次POST请求：
- 开发一个文档 `index.html` 并编写一些内容：
    - 开发一个表单：`<form action="#" method="POST">`
- 将文档部署到 `%TOMCAT_HONE%\webapps\ROOT\` 下并启动tomcat。
- 打开chrome浏览器开发者模式，切换到 `Network` 选项卡。
- 直接在地址栏键入 `http://localhost:8080/index.html`：
    - 在账号框中随意输入一个账号，并点击send按钮以发送一次POST请求。
- 点击 `Network` 中出现的本次请求路径，可以观察到以下常用信息：   
    - 额外查看 `Headers` 选项卡中的 `Form Data` 请求体表单数据。

# 第二阶：SpringBoot入门

**心法：** [springboot](https://spring.io/) 是由Pivotal团队提供的全新框架，用于简化项目搭建过程，开发过程和部署过程等：
- springboot最大的特点在于可以自动装配各种组件。
- springboot项目独立运行，不依赖外部web容器。
- springboot对很多主流开发框架的无配置集成，如原生JDBC，JPA，Hibernate，MyBatis等。

## SpringBoot基础配通

**心法** 使用springboot提供的工具网站可以快速自动搭建springboot项目：
- [springboot项目快速创建页面](https://start.spring.io/)

### IDEA手动搭建

**武技：** 使用IDEA手动搭建springboot项目：
- 新建项目：`File -> New Project/Module -> Maven`：
    - 确定JDK版本，填写横纵坐标和项目位置，点击 `Finish` 创建maven-jar项目。
- 在pom文件中使用 `<parent>` 将springboot配置为本项目的父项目：
    - `<groupId>org.springframework.boot</groupId>`：父项目横坐标。
    - `<artifactId>spring-boot-starter-parent</artifactId>`：父项目纵坐标。
    - `<version>2.3.1.RELEASE</version>`：父项目版本，后续某些依赖会自动选择适合的版本，无需手配。
    - `<relativePath/>`：始终从父项目中获取该依赖，而不从本地仓库或远程仓库中获取。
- 配置第三方依赖：spring家族的依赖会自动匹配最佳版本：
    - `org.springframework.boot.spring-boot-starter-web`：sb前端starter。
    - `org.springframework.boot.spring-boot-starter-test(test)`：sb测试starter。
    - `org.springframework.spring-boot-devtools(runtime)`：sb热部署。
    - `org.projectlombok.lombok`：也可以自动匹配最佳版本。
- 在pom文件中使用 `<properties>` 调整JDK版本和编码：
    - `<java.version>1.8</..>`：设置JDK版本。
    - `<project.build.sourceEncoding>UTF-8</..>`：设置JDK源码编码。
    - `<project.reporting.outputEncoding>UTF-8</..>`：设置JDK编译编码。
- 配置pom插件：若不打算用maven插件来运行应用则忽略：
    - `org.springframework.boot.spring-boot-maven-plugin`
- 在 `resources` 下开发配置包和主配文件：
    - `static`：存放静态文件，如css/js/image等，浏览器可直接访问。
    - `templates`：存放页面文件，如jsp/html等。
    - `application.properties`：springboot主配，名称和位置暂时固定。
- 主配选配：
    - `server.port`：项目端口号，默认8080。
    - `spring.application.name`：项目部署名，仅在独立部署并访问时使用。
- 开发启动类 `boot.App`：一个项目仅有一个启动类可生效：
    - `@SpringBootApplication`：声明为配置类并开启自动装配，@Component扫描等功能。
    - `SpringApplication.run(类.class, args)`：启动类代码。
- 开发控制类 `start.StartController`：：
    - 标记 `@Controller` 加入spring管理，该注解必须被启动类注解扫描到才能生效。
- 开发一个无参无返回值的控制方法 `execute()`：
    - 标记 `@RequestMapping` 以配置方法URL。
- 启动项目并使用 `http://服务器IP:服务器端口/方法URL地址` 格式访问控制方法：
    - cli：`api/start/execute`
    - 一个tomcat只跑一个项目，访问路径中勿加项目名。
- res: `资源/图片/第2阶-springboot入门$Springboot手动配通流程图`

### IDEA自动搭建

**武技：** 使用IDEA自动搭建springboot项目：
- 新建项目：`File -> New Project/Module -> Spring Initializr`：
- 填写项目横纵坐标，Maven管理，JDK版本，项目名，项目描述，启动类包名等。
- 勾选依赖：注意修改右上角的springboot版本：
    - `Spring Boot DevTools`：热部署。
    - `Lombok`：简化工具。
    - `Spring Web`：springboot前端依赖。
- 确定项目名和项目工作空间地址，点击Finish完成创建。

## SpringBoot常用注解

### @ResponseBody

**心法：** `@ResponseBody` 标记的控制方法，会将方法返回值通过springboot默认的转换器进行转换，然后附加在响应体中，最后IO回写给调用方：
- `@ResponseBody` 若标记在控制类上，则相当于对该类中所有的控制方法全部标记该注解。
- `@ResponseBody` 标记的控制方法不使用视图解析器，即不进行页面的跳转。

**心法：** 测试 `@ResponseBody` 效果：
- 开发控制器 `start.RestControllerController`：
    - 标记 `@Controller`：加入spring容器管理。
- 开发无参有返回值的控制方法 `execute()`：
    - 标记 `@RequestMapping` 以配置方法URL。
    - 标记 `@ResponseBody` 将返回值转换为响应体数据。

### @RestController

**心法：** `@RestController` 整合了 `@Controller` 和 `@ResponseBody` 两个注解。

**心法：** 测试 `@RestController` 效果：
- 开发控制器 `start.RestControllerController`：
    - 标记 `@RestController`：加入spring容器管理并对该类中的全部方法标记 `@ResponseBody`。
- 开发无参有返回值的控制方法 `execute()`：
    - 标记 `@RequestMapping` 以配置方法URL。

### @RequestMapping

**心法：** `@RequestMapping` 用于映射一个控制方法的URL值和访问规则，该值可用浏览器直接访问：
- 注解位置：
    - 控制类上的 `@RequestMapping` 用于设置该类中全部控制方法的URL前缀，可选。
    - 控制方法上的 `@RequestMapping` 用于设置所在控制方法的URL具体值，必选。
- 注解属性 `value`：String数组类型，用于设置1或N个方法URL，支持模糊匹配：
    - `?`：匹配一个字符，如 `/?/execute`。
    - `*`：匹配一层路径中的任意个字符，如 `/*/execute`，个数不能为零。
    - `**`：匹配多层路径中的任意个字符，如 `/**/execute`，层数可以为零。
    - src: `start.RequestMappingController.testValue()`
- 注解属性 `method`：RequestMethod枚举类型数组，用于限定1或N种请求方式：
    - `@GetMapping`：相当于 `@RequestMapping(method = RequestMethod.GET)`
    - `@PostMapping`：相当于 `@RequestMapping(method = RequestMethod.POST)`
    - src: `start.RequestMappingController.testMethod()`
- 注解属性 `params`：String数组类型，限定1或N种请求参数：
    - `params={"a"}`：查询串中必须有a属性，值无所谓。
    - `params={"!b"}`：查询串中必须没有b属性。
    - `params={"c=3"}`：查询串中必须有c属性，且其值必须为3。
    - `params={"d!=4"}`：查询串中若有d属性，则其值必须不能为4，若没有则无所谓。
    - src: `start.RequestMappingController.testParams()`

### @RequestParam

**心法：** 控制方法允许直接使用 `@RequestParam` 标记的形参来接收请求参数：
- 格式：`@RequestParam(value="age", required=false, defaultValue="18") Integer age`：
    - `value="age"`：表示客户端必须以 `age` 作为参数的key值，默认为其所标记的形参变量名。
    - `required=false`：表示允许客户端不传递该参数，默认为true。
    - `defaultValue="18"`：表示该参数未传递时的默认值为18，默认为null。
    - src: `start.RequestParamController.testSimpleParam()`
- 允许直接使用数组类型的形参接收同名的URL请求参数：
    - 要求URL参数的key值必须和数组变量名一致。
    - src: `start.RequestParamController.testArrayParam()`
- 允许直接使用实体类直接接收请求参数：
    - 要求URL参数的key值必须和实体类中的属性名一致，支持连调。
    - 若存在多个相同实体类形参，则全部进行接收。
    - 实体类形参前无法添加 `@RequestParam` 注解。
    - src: `start.RequestParamController.testEntityParam()`

### @PathVariable

**心法：** 控制方法允许直接使用 `@PathVariable` 标记的形参来接收请求头中的路径参数。

**武技：** 测试 `@PathVariable` 效果：
- 开发控制方法 `start.PathVariableController.selectById()`：
    - 类上标记 `@RestController`：加入spring容器管理。
- 使用 `@RequestMapping("select-by-id/{id}")` 标记控制方法以配置URL映射：
    - `{id}` 表示对路径进行占位，用于接收对应位置上的路径值，命名随意，支持多层。
- 使用 `@PathVariable("id") Integer id` 标记控制方法形参：
    - 该注解也支持 `required` 属性，但不支持 `defaultValue` 属性。
- 若真实URL为 `localhost:8080/api/path-variable/select-by-id/5`：
    - 则 `{id}` 接收到 `5`，并赋值给形参变量 `id`。

### @RequestHeader

**心法：** 控制方法允许直接使用 `@RequestHeader` 标记的形参来接收请求头中的指定参数。

**武技：** 测试 `@RequestHeader` 效果：
- 开发控制方法 `start.RequestHeaderController.testHost()`：
    - 类上标记 `@RestController`：加入spring容器管理。
- 使用 `@RequestHeader("Host") String host` 标记控制方法形参：
    - 表示取出请求头中的Host值并赋值给host变量：
    - 该注解也拥有 `required` 和 `defaultValue` 属性。

### @CookieValue

**心法：** 控制方法允许直接使用 `@CookieValue` 标记的形参来接收请求中的Cookie参数。

**武技：** 测试 `@CookieValue` 效果：
- 开发控制方法 `start.CookieValueController.testToken()`：
    - 类上标记 `@RestController`：加入spring容器管理。
- 使用 `@CookieValue("token") String token` 标记控制方法形参：
    - 表示获取请求Cookie中的token值并赋值给token变量。
    - 该注解也拥有 `required` 和 `defaultValue` 属性。
- 在chrome控制台输入 `document.cookie="token=admin";` 设置cookie后再发送测试请求。

### @SpringBootApplication

**心法：** `@SpringBootApplication` 启动类注解是一个组合注解，其中包括：
- `@SpringBootConfiguration`：标记该类为springboot项目的配置类，相当于一个XML配置文件。
- `@EnableAutoConfiguration`：启用自动装配，即自动将某些类交给spring管理：
    - 自动装配类的信息都在 `spring-boot-autoconfigure-xxx.jar\META-INF\spring.factories` 文件中记录。
    - exclude属性用来排除某些类的自动装配以节省资源，该属性已同步给 `@SpringBootApplication` 注解。
- `@ComponentScan`：用于扫描所在包及子包的@Component注解，可用value属性修改扫描范围。

### @SpringBootTest

**心法：** `@SpringBootTest` 是springboot1.4开始引入的一个junit测试注解，是对spring-test的二次封装：
- `@RunWith(SpringRunner.class)`：表示使用Junit4整合spring。
- `@SpringBootTest`：代替spring-test中 `@ContextConfiguration`，用于加载和启动spring容器：
    - 单元测试：若测试类上没标记该注解，则直接执行单元测试方法，不加载和启动spring容器。
    - 功能测试：若测试类上标记了该注解，则先加载和启动spring容器，然后再执行单元测试方法。
- `@SpringBootTest` 注解中可以配置功能测试的MOCK环境：
    - `webEnvironment = SpringBootTest.WebEnvironment.MOCK`：默认值，服务器并未真正启动。
    - `webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT`：服务启动，监听一个随机端口。
    - `webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT`：服务启动，监听主配中的端口。
    - `webEnvironment = SpringBootTest.WebEnvironment.NONE`：不提供MOCK环境，等效于单元测试。

**武技：** 使用SpringBootTest的功能测试访问业务方法：
- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-test(test)`
- 开发业务类 `start.SpringBootTestService`：
    - 仅为测试，省略业务接口。
    - 开发一个模拟添加的业务方法 `insert()`
- 开发测试类 `start.SpringbootJunitTest`：
    - 标记 `@RunWith(SpringRunner.class)`：表示使用Junit4整合spring。
    - 标记 `@SpringBootTest`：表示加载和启动spring容器，配置DEFINED_PORT的MOCK环境。
    - 使用 `@Autowired` 注入业务类，并开发测试方法调用业务方法，查看服务是否启动。

## SpringBoot常用功能

### 全局异常处理

**武技：** 全局异常处理指的是将指定范围的异常进行提取并进行统一处理：
- 开发全局异常处理类 `exception.GlobalExceptionHandler`：
    - 标记 `@ControllerAdvice` 以声明为异常处理类并使用 `value` 属性指定包范围。
- 开发异常处理方法 `runtimeException()`：
    - 标记 `@ExceptionHandler` 并使用 `value` 指定对哪些异常进行捕获，缺省表示捕获全部异常。
    - 标记 `@ResponseBody` 以返回JSON格式的响应信息。
    - 该方法仅允许额外使用一个 `Exception` 类型的形参，用于接收具体异常实例。
- 开发控制方法 `exception.ExceptionHandlerController.execute()`：
    - 埋一个对应的异常，访问测试。
- res: `资源/图片/第2阶-springboot入门$全局异常处理流程图`

### 定时执行任务

**心法：** springboot支持使用 `@Scheduled` 标记方法，使其成为一个周期定时或延迟定时的任务方法：
- res: `资源/图片/第2阶-springboot入门$Scheduled延迟和周期`

**武技：** 使用定时任务每隔3s在控制台打印当前时间：
- 启动类标记 `@EnableScheduling` 以开启定时功能。
- 开发任务类 `schedule.ScheduleTask`：
    - 标记 `@Component` 以被spring管理。
- 开发任务方法并标记 `@Scheduled`：
    -  `cron`：表达式的格式为 `秒 分 时 日 月 星期 年（可省略）`，具体模板百度。
    - `fixedDelay=3000`：立即执行任务，每次任务完成后计时，每隔3秒执行一次任务。
    - `fixedRate=3000`：立即执行任务，每次任务开始前计时，每隔3秒执行一次任务。
- 启动入口类，控制台观测任务执行情况。

### 异步执行任务

**心法：** 异步执行方法不占用主线程资源，可以提高项目执行效率：
- `j.u.c.Future`：异步方法无论是否拥有返回值，都需要返回一个Future类型的值：
    - 无返回值的方法返回 `Future<Void>` 类型。
    - 有返回值的方法返回 `Future<类型>` 类型。
- `o.s.s.a.AsyncResult`：Future的一个实现类，用于构建异步方法的返回值：
    - 无返回值的方法使用 `new AsyncResult<>(null)` 构建返回值。
    - 无返回值的方法返回 `new AsyncResult<>(数据)` 构建返回值。
- Future常用API方法：
    - `future.isDone()`：判断Future任务是否已完成，该方法不阻塞。 
    - `future.get()`：获取Future中封装的内容。
- res: `资源/图片/第2阶-springboot入门$Async异步任务流程图`
  

**武技：** 测试异步方法的执行效率：
- 启动类上添加 `@EnableAsync` 开启异步功能。
- 开发异步任务类 `async.AsyncTask`：
    - 标记 `@Component` 以被spring管理。
    - 标记 `@Async` 表示该类中的所有方法均为异步调用，该注解可单独作用在某个方法上。
- 开发无返回值的任务方法 `Future<Void> taskA()`:模拟耗时3s。
- 开发有返回值的任务方法 `Future<String> taskB()`:模拟耗时4s。
- 开发控制器 `async.AsyncController`：
    - 使用 `@Autowried` 注入异步任务类。
- 开发控制方法 `testAysncTask()`：
    - 依次调用 `taskA()` 和 `taskB()` 这两个任务方法。
    - 使用while轮询两个任务方法是否全部执行完毕。
    - 计算整个控制方法的总耗时并响应到客户端。

### 自定义拦截器

**心法：** Interceptor拦截器作用于请求和响应的途中，并执行拦截代码，支持开发多个拦截器以组成一个拦截器链：
- res: `资源/图片/第2阶-springboot入门$Interceptor拦截器原理图`

**武技：** 使用拦截器模拟门卫大爷，美女和携带身份证的人直接放行，没携带身份证的帅哥直接阻止：
- 开发拦截器 `interceptor.CardInterceptor`：
    - 标记 `@Component` 以加入spring管理。
    - 实现 `o.s.w.s.HandlerInterceptor` 接口以成为拦截器类。
- 重写前置拦截方法 `preHandle()`：
    - 前置拦截方法在控制方法调用前执行，执行器链中正序依次执行。
    - 方法返回true表示放行请求，返回false表示阻止请求，即整个请求就结束了。
- 重写后置拦截方法 `postHandle()`：
    - 后置拦截方法在控制方法调用后，渲染视图前执行，执行器链中倒序依次执行。
    - 后置拦截多用于对ModelAndView实例进行操作。
- 重写渲染拦截方法 `afterCompletion()`：
    - 渲染拦截方法在渲染视图后执行，多用于清理资源，执行器链中倒序依次执行。
- 开发拦截器配置类 `interceptor.CardInterceptorConfig`：
    - 标记 `@Configuration` 以声明为配置类。
    - 使用 `@Autowried` 注入自定义拦截器 `CardInterceptor` 备用。
    - 重写 `o.s.w.s.c.a.WebMvcConfigurer.addInterceptors()` 以添加拦截器。
    - 方法内使用 `registry.addInterceptor(拦截器实例)` 将自定义拦截器注册到全局拦截器链中。
    - 方法内使用 `registry.addPathPatterns(拦截规则)` 配置自定义拦截器的拦截规则。
    - 方法内使用 `registry.excludePathPatterns(排除规则)` 配置自定义拦截器的排除规则。
- 开发控制器 `interceptor.CardInterceptorController`：
    - 分别测试拦截规则和排除规则。

### 客户端MOCK工具

**心法：** `o.s.w.c.RestTemplate` 是一个用来模拟客户端向服务端发送请求的工具类：

**武技：** 测试向Controller发送GET/POST请求：
- 开发控制器 `rest.RestTemplateController`：开发三个控制方法：
    - `getMapping()`：仅接受GET请求，方法最终返回一个String类型的数据。
    - `postMapping()`：仅接受POST请求，方法最终返回一个String类型的数据。
    - `redirect()`：接受一切请求，方法最终重定向到 `view/index.html` 页面。
- 开发测试类 `rest.RestTemplateTest`：
    - 标记 `RunWith(SpringRunner.class)` 以使用SpringBootTest功能。
    - 标记 `@SpringBootTest` 并使用 `webEnvironment` 属性指定DEFINED_PORT的Mock环境。
    - 使用 `new RestTemplate()` 创建一个RestTemplate成员属性，用于向Controller发送请求。
- 使用 `rest.getForEntity()` 向Controller发送GET请求并返回ResponseEntity实例：
    - param1：请求完整的URL路径，支持KV和 `{}` 占位符两种方式携带查询串数据。
    - param2：响应数据的类型，对应控制方法的返回值类型。
    - param3：可选，当请求使用占位符方式携带查询串数据时可使用数组或Map结构进行赋值。
    - return：返回ResponseEntity实例，其中包括响应头，响应体和状态码等内容。
- 使用 `rest.postForEntity()` 向Controller发送POST请求并返回ResponseEntity实例：   
    - 比 `getForEntity()` 多一个请求体参数，本质是LinkedMultiValueMap结构，不需要则填null。
- 使用 `rest.postForLocation()` 向Controller发送POST请求并返回重定向路径字符串：
    - 和 `postForEntity()` 的形参一致，但返回值是一个URI实例，表示控制方法的重定向路径。
    - 不存在 `rest.getForLocation()` 这个API方法。

### 原生Servlet对象

> JSP和Thymeleaf默认情况下无法共存，且springboot内嵌的tomcat默认不支持JSP页面，所以我和springBoot都并不建议使用JSP作为前端页面。

**心法：** springboot默认支持原生的Servlet，可以友好共存。

**武技：** 整合原生Servlet：
- 开发原生Servlet类 `servlet.TestServlet`，标记 `@WebServlet`。
- 开发原生过滤器类 `servlet.filter.ScanFilter`，标记 `@WebFilter`。
- 开发原生监听器类 `servlet.listener.ScanListener`，标记 `@WebListener`。
- 在启动类中使用 `@ServletComponentScan` 扫描servlet类，过滤器类和监听器类所在包。

## SpringBoot常用整合

### 整合LogBack日志框架

**心法：** springboot默认使用logback作为日志框架，它是log4j的改进版日志框架，推荐配合slf4j一起使用：
- logback日志格式解析：规则越复杂，效率越低：
    - `%d{yyyy-MM-dd HH:mm:ss.SSS}`：日志记录的日期，对应 `年-月-日 时:分:秒.毫秒`。
    - `%-5p`：日志级别单词，不够5字符时右补空格至5，正数为左补。
    - `%4.15(%t)`：线程名，当长度小于4时左补空格至4，负数为右补，超出15的部分截掉。
    - `%c{50}.%M:%L`：对应 `类全名.方法名:代码行号`，类全名长度大于50时从头开始折叠包名。
    - `%msg`：对应日志输出内容。
    - `%n`：换行符。
- logback日志配置文件：
    - springboot默认寻找classpath下的 `logback.xml/logback-spring.xml` 作为日志配置文件。
    - 自定义的日志配置文件需要在主配中使用 `logging.config` 指定。
- res: `资源/图片/第2阶-springboot入门$logback日志框架搭建流程图`

**武技：** 完成日志配置准备工作：
- 添加依赖：
    - `org.apache.logging.log4j.log4j-core(2.3)`
    - `log4j.log4j(1.2.17)`
    - `org.slf4j.slf4j-api(1.7.25)`
    - `org.slf4j.slf4j-log4j12(1.7.25)(test)`
- 在classpath下开发日志配置文件 `logback.xml`：
    - 使用默认位置和文件名时不需要在主配中指定。
    - 编写XML头和 `<configuration>` 根标签。

**武技：** 使用logback将INFO及以上级别的日志打印在控制台：
- 在根标签中配置控制台追加器 `<appender>`：
    - 属性 `name="STDOUT"`：控制台追加器名称，命名随意。
    - 属性 `class="c.q.l.c.ConsoleAppender"`：控制台追加器类。
- 在控制台追加器中配置编码器 `<encoder>`：
    - 子标签 `<pattern>`：配置日志格式。
    - 子标签 `<charset>`：配置日志编码。
- 在根标签中配置根记录器 `<root>` 对全局/运行时日志进行记录，建议放在最后：
    - 属性 `level="INFO"`：日志最低级别，不区分大小写。
    - 子标签 `<appender-ref ref="">`：关联某追加器的name值，可配置0或N个。    
- 开发控制方法 `logback.LogBackController.consoleLog()`：
    - `LoggerFactory.getLogger(getClass())`：获取当前类的Logger实例，可用lombok简化。
    - `logger.debug()/info()/warn()/error()`：记录一条debug/info/warn/error级别的日志信息。

**武技：** 使用logback将WARN及以上级别的日志打印在日志文件：
- 在根标签中配置文件追加器 `<appender>`：
    - 属性 `name="FILEOUT"`：文件追加器名称，命名随意。
    - 属性 `class="c.q.l.c.r.RollingFileAppender"`：文件追加器类。
- 在文件追加器中配置编码器 `<encoder>`：
    - 子标签 `<pattern>`：配置日志格式。
    - 子标签 `<charset>`：配置日志编码。
- 在文件追加器中配置日志文件 `<file>`：    
    - 支持相对所在项目的相对路径或绝对路径。
- 在文件追加器中配置过滤器 `<filter>`（可选）： 
    - 属性 `class="c.q.l.c.f.LevelFilter"`：过滤器类。
    - 子标签 `<level>`：参考级别。
    - 子标签 `<onMatch>`：对所有参考级别的日志DENY不记录/ACCEPT记录，单词大写。
    - 子标签 `<onMismatch>`：对所有参考级别之外的日志DENY不记录/ACCEPT记录，单词大写。
- 在文件追加器中配置滚动策略 `<rollingPolicy>`：
    - 属性 `class="c.q.l.c.r.SizeAndTimeBasedRollingPolicy"`：滚动策略实现类。
- 在滚动策略中配置日志文件拆分名模板 `<fileNamePattern>`：
    - 当日期变化，或当天的单个日志文件大小超限时，按此文件名模板进行拆分和滚动，如 `log/my.%d.%i.log`。
- 在滚动策略中配置日志文件最长时效 `<maxHistory>`：
    - 当日志文件保存时间超出此值时立刻过期删除，单位根据 `<fileNamePattern>` 自动识别。
- 在滚动策略中配置全部日志文件最大值 `<totalSizeCap>`：
    - 全部日志文件总大小超出此值时，立刻删除早期日志，需要添加单位如 `MB`。
- 在滚动策略中配置单个日志文件最大值 `<maxFileSize>`：
    - 单个日志文件大小超出此值时，立刻按照 `<fileNamePattern>` 进行拆分，默认10MB。
- 在根标签中配置运行时记录器 `<logger>` 对运行时日志进行记录：
    - 属性 `name="com.joezhou.boot"`：仅对指定包中的运行时日志进行记录。
    - 属性 `level="INFO"`：日志最低级别，不区分大小写。
    - 使用子标签 `<appender-ref ref="">` 关联某追加器的name值，可配置0或N个。    
- 开发控制方法 `logback.LogBackController.fileLog()`：
    - `LoggerFactory.getLogger(getClass())`：获取当前类的Logger实例，可用lombok简化。
    - `logger.debug()/info()/warn()/error()`：记录一条debug/info/warn/error级别的日志信息。

### 整合Jackson序列化框架

**心法：** jackson是spring默认的JSON序列化开源框架，简单易用，速度快，占用内存低，性能好，支持扩展和定制：
- JSON语言：`JavaScript Object Notation` 是目前最理想的前后台数据交换语言：
    - JSON格式：`{"k01": v01, "k02": v02 ...}`：key值均建议使用双引号包裹。
    - JSON内外均支持组合数组：`[{"k01:" [1, 2]}, {"k02:" [3, 4]}]`。
- jackson功能：
    - 序列化：Jackson可以将List/Map/Set/Entity等数据结构序列化成JSON格式的字符串。
    - 反序列化：Jackson可以将JSON格式的字符串反序列化成List/Map/Set/Entity等数据结构。
- jackson依赖：springboot默认支持jackson框架，无需引入依赖：
    - `com.fasterxml.jackson.core.jackson-core(2.9.7)`
    - `com.fasterxml.jackson.core.jackson-annotations(2.9.7)`
    - `com.fasterxml.jackson.core.jackson-databind(2.9.7)`
- res: `资源/图片/第2阶-springboot入门$Jackson序列化与反序列化流程图`

**心法：** jackson常用API：
- 构造jackson框架的核心ObjectMapper实例：
    - `ObjectMapper mapper = new ObjectMapper()`
- 配置ObjectMapper实例：null值不参与序列化：
    - `mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL)`
- 配置ObjectMapper实例：Date型数据禁止被序列化为时间戳：
    - `mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)`
- 配置ObjectMapper实例：Date型数据按指定模板进行序列化：
    - `mapper.setDateFormat(new SimpleDateFormat("yyyy/MM/dd hh:mm:ss"))`
- 将Map/List/Set/Entity等数据结构序列化为JSON字符串：
    - `mapper.writeValueAsString(Object obj)`
- 将JSON字符串反序列化为Map/List/Set/Entity等数据结构：
    - `mapper.readValue(jsonStr, Map.class)`：将jsonStr反序列化为Map结构。

**武技：** chrome浏览器安装JsonView插件：
- 下载 [JsonView](https://github.com/gildas-lormeau/JSONView-for-Chrome) 插件：
    - 选择 `Clone or download -> Download Zip`：直接解压缩到某目录中。
    - res: `资源\附件\JSONView-for-Chrome-master.zip`
- 打开chrome浏览器并直接访问 `chrome://extensions/` 进入拓展程序管理界面：
    - 选中开发者模式，点击 `加载正在开发的扩展程序…`，选择插件目录中的 `WebContent` 目录。
    - 点击确定后，就安装上了，使用 `ctrl+ r ` 重新加载即可使用。

**武技：** 测试jackson的序列化和反序列化过程：
- 开发控制器 `jackson.JacksonController`：
    - 构造jackson框架的核心ObjectMapper属性并进行序列化相关的配置。
- 开发控制方法 `testSerialization()`：
    - 准备Map/Entity/List/Set结构的四个测试数据并分别序列化成JSON字符串。
- 开发控制方法 `testDeSerialization()`：
    - 准备一个单条数据的JSON字符串并将其反序列化为Map/Entity结构的数据。
    - 准备一个多条数据的JSON字符串并将其反序列化为Set/List结构的数据。

**武技：** 封装Jackson工具类：
- 开发工具类 `jackson.JacksonUtil`：
    - 构造jackson框架的核心ObjectMapper属性并进行序列化相关的配置。
- 开发工具方法：
    - `build(int code, String msg, Object data)`：通过Jackson将数据转换为前端需要的JSON响应格式字符串。
    - `build(Object data)`：仅传递响应数据时，响应状态码视为1，响应消息视为success字符串。
    - `build(int code, String msg)`：仅传递响应状态码和响应消息时，响应数据视为null。
    - `parseCode(String str)`：将JSON字符串中的code值解析为int并返回。
    - `parseMsg(String str)`：将JSON字符串中的msg值解析为String并返回。
    - `parseData(String str)`：将JSON字符串中的data值解析为Map并返回。
- 开发控制方法 `jackson.JacksonController.selectById()`：测试Jackson工具类。

### 整合Thymeleaf模板引擎

**心法：** thymeleaf是一种模版引擎，能处理HTML，XML，TEXT，JAVASCRIPT和CSS模板内容:
- res: `资源/图片/第2阶-springboot入门$Thymeleaf使用流程图`

**武技：** 整合thymeleaf模板并测试跳页效果：
- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-thymeleaf`
- 主配选配：
    - `spring.resources.static-locations`：默认静态资源加载位置。
    - `spring.thymeleaf.enabled`：启用thymeleaf，默认true。
    - `spring.thymeleaf.cache`：启用thymeleaf缓存，默认true
    - `spring.thymeleaf.prefix`：配置thymeleaf响应路径前缀，默认classpath:/templates/。
    - `spring.thymeleaf.suffix`：配置thymeleaf响应路径后缀，默认.html。
    - `spring.thymeleaf.encoding`：配置thymeleaf编码，默认UTF-8。
- 开发页面 `classpath:/templates/view/index.html`：用于测试路径跳转。
- 开发控制器 `thymeleaf.ThymeleafController`：
    - 标记 `@Controller` 而非 `@RestController` 否则无跳页效果。
- 开发返回值为String类型的控制方法 `execute()`：
    - 标记 `@RequestMapping` 以配置方法的URL。
    - 使用 `return` 响应跳转页面，该值会自动拼接主配中的相应路径前后缀。 

**武技：** 测试请求转发是一次请求且地址栏不发生改变：
- 请求转发流程：
    - 客户端将请求发给Controller，Controller将请求转发到目标资源，然后服务端响应。
- 请求转发格式：`return "forward:/资源地址"`：
    - 该格式无法自动拼接主配中的前后缀，需要自己手动补全。
    - 该格式中的资源地址必须使用 `/` 开头，且需要忽略 `templates` 目录。
- src: `thymeleaf.ThymeleafController.testForward()`

**武技：** 测试重定向是两次请求，地址栏会改变为最终资源的位置：
- 重定向流程：
    - 客户端将请求发给Controller，Controller将资源的真实位置返回给客户端。
    - 客户端再次发送请求，直接请求目标资源，然后服务端响应。
- 重定向格式：`return "redirect:/资源地址"`：
    - 该格式无法自动拼接主配中的前后缀，需要自己手动补全。
    - 该格式中的资源地址必须使用 `/` 开头，且需要忽略 `templates` 目录。
- src: `thymeleaf.ThymeleafController.redirectForward()`

**武技：** springboot支持自定义4xx和5xx状态的反馈页面以替代默认的错误页面：
- 开发4xx和5xx状态的反馈页面，仅浏览器生效，名称和位置固定：
    - `classpath:templates/error/4xx.html`
    - `classpath:templates/error/5xx.html`
- 开发控制方法 `thymeleaf.FriendlyPageController.execute()`：埋一个空指针异常。
- 测试友好页面：
    - 直接在浏览器地址栏键入一个错误的URL路径即可测试404效果。
    - 访问控制方法，触发空指针异常，即可测试500效果，注意需要关闭全局异常处理后再测。

### 整合HibernateValidator校验器

**心法：** HibernateValidator校验器对 JSR303校验器进行了拓展，用于对实体类型的请求参数进行校验：
- 校验原则：
    - HibernateValidator仅支持对实体类型的形参进行规范校验。
    - 实体类型的形参必须标记了 `@Validated` 才表示开启校验功能，支持实体类连用。
- 校验注解：每个属性都可以标注以下1或N个校验注解，均可使用 `message` 重写报错时的信息：
    - res: `资源/图片/第2阶-springboot入门$HibernateValidator参数校验流程图`
- 校验结果：
    - 若校验成功，放行请求到后续的业务。
    - 若校验失败，会将错误信息存储到一个紧随该形参之后的 `ResultBinding` 实例中，需要手动处理。

**武技：** 使用HibernateValidator校验请求参数：
- 添加依赖：
    - `org.hibernate.validator.hibernate-validator(6.1.5.Final)`
- 开发实体类 `validator.Teacher`：
    - 为属性标记一些HibernateValidator的校验注解。
- 开发控制方法 `valid.HibernateValidatorController.execute()`：
    - 使用一个Teacher类型的形参并对其标记 `@Validated` 以开启HibernateValidator校验。
    - 在该形参紧随其后的位置手动补充一个ResultBinding类型的形参用于存放校验失败的信息。
    - `bindingResult.hasErrors()`：判断ResultBinding是否存在校验失败的信息。
    - `bindingResult.getFieldErrors()`：获取ResultBinding中所有属性错误信息集合。
    - `error.getObjectName()`：返回是哪个实例爆发了校验错误。
    - `error.getField()`：返回是哪个属性爆发了校验错误。
    - `error.getDefaultMessage()`：返回具体的校验错误信息。

**武技：** 使用HibernateValidator分组校验请求参数：
- 开发两个空接口 `InsertGroup/UpdateGroup`，仅用于分组。
- 开发实体类 `validator.Student`：开发id属性并添加两组校验注解：
    - `@Null(message = "添加业务中，主键id必须为空", groups = InsertGroup.class)`
    - `@NotNull(message = "修改业务中，主键id必须不为空", groups = UpdateGroup.class)`
- 开发添加的控制方法 `insert()` 并对参数开启HibernateValidator校验：
    - `@Validated(InsertGroup.class)`
- 开发添加的控制方法 `updateById()` 并对参数开启HibernateValidator校验：
    - `@Validated(InsertGroup.class)`
- 启动服务，并测试分组验证的效果。

### 整合AOP面向切面编程

**武技：** springboot整合AOP面向切面编程：
- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-aop`
- 开发切面配置类 `aop.AspectConfig`：
    - 标记 `@Aspect` 以声明为切面类。
    - 标记 `@Configuration` 以声明为配置类。
- 在切面类中开发环绕通知方法：
    - 标记 `@Around("execution()")`：直接配合PE表达式锁定AOP范围。
    - 异常通知优先级高于全局异常处理类，建议将处理异常的工作交给全局异常处理类。
- 开发控制类 `aop.AopController` 并开发一个会爆发异常的控制方法。
- res: `资源/图片/第2阶-springboot入门$AOP搭建流程图`

### 整合Actuator健康检查

**心法：** [Actuator](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/production-ready-features.html#production-ready) 
监控组件负责使用HTTP端点对线上项目进行健康检查，审计，收集指标等，目的是保持项目的高可用：
- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-actuator`
- 主配选配：启用指得是开启功能，暴露指得是对外开放，前缀均为 `management`：
    - `.server.port=8000`：actuator服务端口，默认8080。
    - `.endpoint.health.show-details=always`：展示 `info` 端点的项目健康信息。
    - `.endpoint.health.cache.time-to-live`：健康信息缓存时间，默认0ms，即不缓存。
    - `.endpoints.web.exposure.include`：暴露端点列表，逗号分隔，`*` 为全部暴露。
    - `.endpoints.web.exposure.exclude`：禁用端点列表，逗号分隔，`*` 为全部禁用，优先级高于include。
- 启动项目：控制台可观察到，项目和监控会分别启动:
    - cli: `localhost:8000/actuator`：查看actuator对外暴露的所有端点，默认仅暴露health和info端点。
    - res: `资源\图片\第2阶-springboot入门$Actuator常用端点`：查看其他常用端点。

**武技：** 自定义健康检查器：
- 开发健康检查器 `actuator.MyHealthIndicator`：
    - 标记 `@Component` 以被spring管理，可使用value值设置health端点的key值。
    - 实现 `o.s.b.a.h.HealthIndicator` 接口。
- 重写 `health()` 接口方法：
    - `File.listRoots()`：返回所有系统中所有盘符文件数组，如 `[C:\, D:\, F:\]`。
    - `file.getTotalSpace()`：返回某个盘符文件的总容量。
    - `file.getUsableSpace()`：返回某个盘符文件的可用容量。
    - `Health.up().withDetail(KV).bulid()`：创建一个UP状态的Health对象。
    - `Health.down().bulid()`：创建一个DOWN状态的Health对象。
- cli: `localhost:8000/actuator/health`

**武技：** 自定义端点：

- 开发端点类 `actuator.MyEndPoint`：
    - 标记 `@Configuration` 以声明为配置类。
    - 标记 `@Endpoint` 以声明为端点类，建议使用 `-` 分割或简单的id值设置端点名。
    - 开发端点方法并标记 `@ReadOperation`，方法返回List/Map数据，该数据即为端点内容。
- cli: `localhost:8000/actuator/my-end-point`：需要提前在主配中暴露该端点。

### 整合JWT鉴权工具

**心法：** [JSON Web Token](https://jwt.io/) 
是一种基于JSON的高效，简洁，安全的开放标准，使用一个token字符串进行通信，常用于登录鉴权，单点登录等分布式场景的技术：

- jwt校验流程：
    - 客户端首次登录成功时，服务端将部分用户信息和密钥组合加密成一个token字符串返回客户端。
    - 客户端将该token字符串保存在cookie，sessionStorage或localStorage中。
    - 后续客户端请求均携带此token字符串，服务端解密成功时执行请求，否则阻止。
- token结构：`header.payload.signature`：
    - header头：存放token使用的加密算法，如HS256，ES256，RS256等。
    - payload负载：可携带部分不敏感的用户信息以避免多次查库。    
    - signature签名：根据服务端秘钥secretKey生成一个防伪签名标识。
- res: `资源/图片/第2阶-springboot入门$JWT的token校验流程图`

**武技：** 开发Token工具类用于生成和验证token字符串：

- 添加依赖：`com.auth0.java-jwt(3.4.0)`
- 开发实体类 `jwt.entity.Vip`：
    - 埋 `id/username/password/avatar` 属性并配置setter和getter。
- 开发工具类 `jwt.util.TokenUtil`：
    - `JWT.create()`：返回一个可配置的 `Builder` 对象。
    - `builder.withClaim("K", V)`：设置payload中的自定义键值对。
    - `builder.sign(Algorithm.HMAC256(密钥))`：通过HS256算法和秘钥设置签名。
    - `JWT.require(Algorithm.HMAC256(密钥)).build()`：通过HS256算法和秘钥创建token验证器。
    - `JWTVerifier.verify(token)`：验证token并返回一个 `DecodedJWT` 实例。
- tst: `jwt.TokenUtilTest`：
    - `decodedJwt.getSubject()`: token主题内容。
    - `decodedJwt.getIssuedAt()`: token发行时间。
    - `decodedJwt.getExpiresAt()`: token过期时间。
    - `decodedJwt.getIssuer()`: token发行作者。
    - `decodedJwt.getHeader()`: 加密后的token头。
    - `decodedJwt.getPayload()`: 加密后的token负载。
    - `decodedJwt.getSignature()`: 加密后的token签名。
    - `decodedJwt.getClaims()`: token负载中的全部Entry。
    - `decodedJwt.getClaim("id").asInt()()`: token负载中的id值。
    - `decodedJwt.getClaim("avatar").asString()()`: token负载中的avatar值。
    - `decodedJwt.getAlgorithm()`: token算法名。

**武技：** 使用Token工具类完成登录鉴权业务：

- 开发标记注解 `jwt.annotation.TokenVerify`：
    - 仅对标记此注解的方法进行token验证。
- 开发业务接口 `jwt.service.VipService`：
    - 开发接口方法：`Vip login(String username, String password)`
- 开发业务类 `jwt.service.impl.VipServiceImpl`：
    - 标记 `@Service` 以加入spring管理。
    - 重写业务方法 `login()`：登陆成功返回一个模拟的Vip实例，失败返回id为-1的Vip实例。
- 开发控制器 `jwt.controller.VipController`：
    - 标记 `@Controller` 以加入spring管理。
    - 使用 `@Autowired` 注入业务接口。
- 开发控制方法 `login()`：调用业务方法进行登录。
    - 登录成功时使用Token工具 `TokenUitl` 生成当前登录用户的token并响应。
    - 登录失败时返回失败提示信息。
- 开发控制方法 `execute()`：模拟需要登录鉴权的一个控制方法：
    - 标记 `@TokenVerify` 表示该方法在访问之前需要登录鉴权。  
- 开发拦截器 `jwt.interceptor.JwtInterceptor` 并重写 `proHandle()`：
    - 若请求不指向某个控制方法，直接放行。
    - 若请求指向的控制方法没有标记 `@TokenVerify`，直接放行。
    - 分别尝试从请求头和请求参数中获取token，若均不存在，直接抛出异常。
    - 通过Token工具类验证token并返回一个DecodedJWT实例。
    - 从DecodedJWT中获取指定的负载信息并存入请求域中。   
- 开发拦截器配置类 `jwt.config.JwtInterceptorConfig` 并重写 `addInterceptors()`：
    - 类上标记 `@Configuration` 以声明为配置类。
    - 在重写方法中实例化自定义拦截器，并通过 `addInterceptor()` 注册到全局拦截器链中。
    - 拦截所有 `/api/vip/**` 请求，但放行其中的 `/api/vip/login` 登录请求。

### 整合FileUpload文件上传

**心法：** fileupload是由apache的commons组件提供的上传组件，用于上传文件：
- 添加依赖：
    - `commons-fileupload.commons-fileupload(1.3.3)`
- 在 `static` 中引入JQuery库：
    - `jquery/jquery-3.6.0.js`
- 调整IDEA中ES版本为6：
    - `File -> Settings -> Languages & Frameworks -> JavaScript` 
- res: `资源/图片/第2阶-springboot入门$FileUpload文件上传流程图`

**武技：** 使用springboot完成文件上传：
- 主配添加：均以 `spring.servlet.multipart` 为前缀：
    - `max-request-size=30MB`：上传文件总的最大值为30M，默认为-1表示无限制。
    - `max-file-size=10MB`：单个文件的最大值为10M。
- 开发控制器 `upload.FileController`：
    - 标记 `@Controller`：文件下载的控制方法不能返回JSON字符串。
- 开发控制方法 `upload()`：
    - 标记 `@ResponseBody` 以返回JSON字符串。
    - 形参使用 `@RequestParam MultipartFile/MultipartFile[]` 接收流文件。
    - `file.getOriginalFilename()` 获取文件客户端真实名字。
    - `file.transferTo(File desc)` 将文件对象上传到指定位置。
- 开发页面：`view/upload-ajax.html`：使用ajax完成头像预览和上传：
    - AJAX必须设置 `"type": "post"` 以POST方式提交。
    - AJAX必须设置 `"processData": false` 以禁止JQ将参数转为查询串。
    - AJAX必须设置 `"contentType": false` 以禁止JQ对其进行操作使得无法正常解析文件。
    - cli: `localhost:8082/view/upload-ajax.html`
- 开发页面：`view/upload-form.html`：使用form完成头像预览和上传：
    - 必须对表单添加 `enctype="multipart/form-data"` 属性。
    - 必须设置表单以POST方式提交。
    - cli: `localhost:8082/view/upload-form.html`

**武技：** 使用springboot完成文件下载：

- 在控制器 `upload.FileController` 中开发控制方法 `download()`：
    - 该方法不标记 `@ResponseBody`，因其必须返回一个 `ResponseEntity<byte[]>` 实例。 
- 方法内使用 `FileUtils.readFileToByteArray(File file)`：获取文件字节数组。
- 方法内使用 `new HttpHeaders()`：获取响应头对象：
    - `.setContentDispositionFormData("attachment", file)`：设置以附件形式下载文件及附件名。
    - `.setContentType(MediaType.APPLICATION_OCTET_STREAM)`：设置响应类型为八位字节流。
- 方法内使用 `new ResponseEntity<byte[]>(字节数组，响应头，http状态码)`：
    - `HttpStatus.CREATED`：http状态码201表示请求已成功，且创建了一个新资源并响应回客户端。

### 整合WebSocket通信技术

**心法：** WebSocket是HTML5提供的一种在单个TCP连接上进行全双工通讯的协议，B端和S端只需完成一次握手即可建立一条快速的，持久的双向数据通道，更节省服务器资源和带宽：

- 添加依赖：
    - `org.springframework.boot.spring-boot-starter-websocket`
- 开发配置类 `websocket.WebSocketConfig`：
    - 标记 `@Configration` 以声明为配置类。
    - 管理 `o.s.w.s.s.s.ServerEndpointExporter` 类。
- 开发S端服务类 `websocket.WebSocketServer`：用于接收B端 `ws://` 开头的连接请求：
    - 标记 `@ServerEndpoint` 以指定端点地址，地址中可使用路径参数，并配合 `@PathParam` 取出B端标识。
    - 准备一个安全且共享的Map容器属性，存放每个B端的标识及其独享的 `javax.websocket.Session` 会话对象。
- 开发B端页面 `websocket/cli-jack.html`：使用JS判断WebSocket兼容性：
    - `new WebSocket("ws://端点地址/jack")`：连通websocket服务端，同时传递jack标识。
- 开发B端页面 `websocket/cli-rose.html`：使用JS判断WebSocket兼容性：
    - `new WebSocket("ws://端点地址/rose")`：连通websocket服务端，同时传递rose标识。

**武技：** 使用WebSocket完成jack和rose的上线下线功能：
- 开发S端 `@OnOpen` 方法：该方法在每个B端上线时触发：
    - 将B端标识及其session实例存入Map，session可直接在形参中定义并使用。
- 开发S端 `@OnClose` 方法：该方法在每个B端下线时触发：
    - 从Map中通过B端标识，移除该B端和及其session实例。
- 开发S端 `@OnError` 方法：当连接或通信异常时触发：
    - 形参中可直接获取 `Throwable` 对象。
- 在B端页面 `websocket/cli-*.html` 中：
    - `socket.onopen()`：B端上线时触发。
    - `socket.onclose()`：B端下线时触发。
    - `socket.onerror()`：当连接或通信异常时触发。
    - `socket.close()`：手动下线。
- 启动项目，访问 `websocket/cli-*.html` 并测试上线下线操作。

**武技：** 使用WebSocket完成群发消息功能：
- 开发S端 `@OnMessage` 方法：该方法在当B端发消息时触发：
    - 形参中可直接获取此字符串消息。
    - 遍历Map容器并向所有在线的B端推送该B端的消息（群推送）。
    - 使用 `session.getAsyncRemote().sendText()` 向B端异步推送消息。
    - 使用 `session.getBasicRemote().sendText()` 向B端同步推送消息。
- 在B端页面 `websocket/cli-*.html` 中：
    - `socket.onmessage()`：B端收到S端推送的消息时触发，回调函数中可用 `resp["data"]` 取出消息内容。
    - `socket.send()` 可向S端发送消息。
- 启动项目，访问 `websocket/cli-*.html` 并测试群发消息操作。

# 第三阶：SSM与SSMP整合

## SSM整合框架

**心法：** SSM是三个框架的整合框架，包括springboot框架，spring框架以及mybatis框架：
- res: `资源/图片/第3阶-SSM与SSMP整合$SSM手动配通流程图`

**武技：** SSM整合准备工作：

- 开发数据库：
    - res: `资源/SQL/ssm.sql`
- 创建项目 `ssm` 并配置编码，JDK版本，父项目，然后引入第三方依赖：
    - `org.springframework.boot.spring-boot-starter-web`
    - `org.springframework.boot.spring-boot-devtools(runtime)`
    - `org.springframework.boot.spring-boot-starter-test(test)`
    - `org.springframework.boot.spring-boot-starter-jdbc`
    - `org.mybatis.spring.boot.mybatis-spring-boot-starter(2.0.0)`
    - `mysql.mysql-connector-java(runtime)`
    - `com.alibaba.druid(1.1.6)`
- 引入 `classpath: logback.xml` 日志配置文件。
- 引入 `util.JacksonUtil` JSON转换工具类。
- 主配选配：
    - `server.port`：项目端口号。
    - `spring.application.name`：项目发布名。
    - `spring.datasource.driver-class-name/url/username/password`：数据源四项。
    - `spring.datasource.type`：连接池实现类，默认 `c.z.h.HikariDataSource`。 
    - `mybatis.configuration.map-underscore-to-camel-case=true`：下划线自动转驼峰。
    - `mybatis.configuration.cache-enabled=true`：开启二级缓存。
    - `mybatis.configuration.lazy-loading-enabled=true`：开启全局懒加载。
    - `mybatis.configuration.aggressive-lazy-loading=false`：关闭全局积极加载。
    - `mybatis.mapper-locations`：Mapper配置文件位置，注解SQL时可忽略。
    - `mybatis.type-aliases-package`：实体类别名包扫描。
    - `mybatis.configuration.log-impl=o.a.i.l.s.StdOutImpl`：控制台SQL。
- 开发ORM实体类 `entity.Hero`：完成ORM对应并设置Setter/Getter等。
- 开发Mapper配置 `mapper/HeroMapper.xml`，使用注解SQL时可跳过此步骤。
- 开发Mapper接口 `mapper.HeroMapper`：标记 `@Repository` 以被spring管理。
- 开发业务接口 `service.HeroService`。
- 开发业务类 `service.impl.HeroServiceImpl`：标记 `@Service` 以被spring管理。
- 开发控制器 `controller.HeroController`：标记 `@RestController` 以被spring管理。
- 开发启动类并标记 `@MapperScan` 以扫描Mapper接口的直接所在包：
    - 若对每个Mapper接口单独标记了 `@Mapper` 则可以省略此项扫描。

### SSM增删改查

**武技：** 使用SSM框架完成hero表的CRUD：
- 在Mapper配置 `mapper/HeroMapper.xml` 中编写CRUD语句块，使用注解SQL时可跳过此步骤。
- 在Mapper接口 `mapper.HeroMapper` 中开发CRUD接口方法。
- 在业务接口 `service.HeroService` 中开发CRUD接口方法。
- 在业务类 `service.impl.HeroServiceImpl` 中：
    - 使用 `@Autowired` 注入Mapper接口并重写CRUD接口方法。
- 在控制器 `controller.HeroController` 中开发CRUD控制方法：
    - 使用 `@Autowired` 注入业务接口并调用业务方法。

### PageHelper分页工具

**心法：** [PageHelper](https://pagehelper.github.io/)是一款用适用于mybatis框架中的分页插件，支持任何复杂的单表、多表分页。

**武技：** 使用SSM框架配合PageHelper完成hero表的分页查询：
- 添加依赖：
    - `com.github.pagehelper.pagehelper-spring-boot-starter(1.2.5)`
- 主配选配：
    - `pagehelper.helper-dialect=mysql`：标识数据库方言。
    - `pagehelper.reasonable=true`：pageNum<1时视为查询首页，pageNum>pages视为查询尾页。
    - `pagehelper.page-size-zero=true`：pageSize为0时视为全查。
- 开发控制方法 `controller.HeroController.paging
    - `PageHelper.startPage(当前第一页, 每页多少条)`：设置分页信息，写在第一行。
    - `heroService.list()`：直接调用业务层全查方法即可。
    - `new PageInfo<>(users)`：构建分页查询对象并响应，其中包含了数据和分页数据。

## SSMP整合框架

**心法：** [MyBatisPlus](https://mp.baomidou.com/) 是一个MyBatis的增强工具，在MyBatis的基础上只做增强不做改变，为简化开发、提高效率而生，可以代替MyBatis框架组成新的SSMP整合框架：
- res: `资源/图片/第3阶-SSM与SSMP整合$SSMP手动配通流程图`

**武技：** SSMP整合准备工作：
- 开发数据库：
    - res: `资源/SQL/ssmp.sql`
- 创建项目 `ssmp` 并配置编码，JDK版本，父项目，然后引入第三方依赖：
    - `org.springframework.boot.spring-boot-starter-web`
    - `org.springframework.boot.spring-boot-devtools(runtime)`
    - `org.springframework.boot.spring-boot-starter-test(test)`
    - `org.springframework.boot.spring-boot-starter-jdbc`
    - `mysql.mysql-connector-java(runtime)`
    - `com.alibaba.druid(1.1.6)`
    - `com.baomidou.mybatis-plus-boot-starter(3.4.1)`：MP核心依赖。
    - `com.baomidou.mybatis-plus-generator(3.4.1)`：用于MP生成代码功能。
    - `org.apache.velocity.velocity-engine-core(2.0)`：velocity模板引擎。
    - 注释掉 `mybatis-spring-boot-starter`：mybatis核心依赖与MP核心依赖容易版本冲突。
    - 注释掉 `pagehelper-spring-boot-starter`：pagehelper依赖与MP核心包容易版本冲突。
- 引入 `classpath: logback.xml` 日志配置文件。
- 引入 `util.JacksonUtil` JSON转换工具类。
- 主配选配：mybatis-plus配置无法与mybatis原生配置共存：
    - `server.port`：项目端口号。
    - `spring.application.name`：项目发布名。
    - `spring.datasource.driver-class-name/url/username/password`：数据源四项。
    - `spring.datasource.type`：连接池实现类，默认 `c.z.h.HikariDataSource`。 
    - `mybatis-plus.configuration.map-underscore-to-camel-case=true`：下划线自动转驼峰。
    - `mybatis-plus.configuration.cache-enabled=true`：开启二级缓存。
    - `mybatis-plus.configuration.lazy-loading-enabled=true`：开启全局懒加载。
    - `mybatis-plus.configuration.aggressive-lazy-loading=false`：关闭全局积极加载。
    - `mybatis-plus.mapper-locations`：Mapper配置文件位置，注解SQL时可忽略。
    - `mybatis-plus.type-aliases-package`：实体类别名包扫描。
    - `mybatis-plus.configuration.log-impl=o.a.i.l.s.StdOutImpl`：控制台SQL。
- 开发MP工具类 `util.MpUtil` 并生成MP代码：
    - `new AutoGenerator()`：创建MP生成器实例，调用 `execute()` 可执行生成。
    - `new GlobalConfig()`：创建并设置MP全局配置。
    - `new DataSourceConfig()`：创建并设置MP数据源。
    - `new PackageConfig()`：创建并设置MP的java包配置。
    - `new StrategyConfig()`：创建并设置MP的ORM映射策略。
- 在启动类上使用 `@MapperScan` 扫描MP生成的mapper包。

### SSMP增删改查

**武技：** 使用SSMP框架完成snake_info表的CRUD：
- 在控制器 `controller.SnakeInfoController` 中：
    - 使用 `@Autowried` 注入业务接口并开发CRUD控制方法。
    - 使用 `service.save(entity)` 添加一条记录。
    - 使用 `service.getById(id)` 按主键查询一条记录。
    - 使用 `service.updateById(entity)` 按主键修改一条记录。
    - 使用 `service.removeById(id)` 按主键删除一条记录。
    - 使用 `service.list()` 全查记录。
- 启动项目并访问控制方法。

### SSMP条件操作

**武技：** 使用SSMP框架完成snake_info表的条件操作：
- 在业务接口 `service.SnakeInfoService` 中开发两个业务方法：
    - 开发 `List<SnakeInfo> testQueryWrapper()` 以测试MP的条件查询。
    - 开发 `void testUpdateWrapper()` 以测试MP的条件修改。
- 在业务类 `service.impl.SnakeInfoServiceImpl` 中重写两个业务方法：
    - 使用 `@Autowried` 注入Mapper接口：需提前手动在Mapper接口上标记 `@Repository`。
    - 使用 `new QueryWrapper<>()` 构建条件查询包装器实例，无法使用set方法设置实体类的值。
    - 使用 `new UpdateWrapper<>()` 构建条件修改包装器实例，支持使用set方法设置实体类的值。
    - 使用 `mapper.selectList(queryWrapper)` 发送条件查询语句。
    - res: `资源\图片\第三阶-SSM与SSMP整合$MP的WapperAPI方法`
- 在控制器 `controller.SnakeInfoController` 中开发两个控制方法：
    - 使用 `@Autowried` 注入业务接口并调用业务方法。
- res: `资源/图片/第3阶-SSM与SSMP整合$MP的WapperAPI方法`

### SSMP分页查询

**武技：** 测试MP的分页查询：
- 开发MP分页配置类 `config.MpPagingConfig`：
    - 标记 `@Configuration` 以声明为配置类。
    - 使用 `@Bean` 管理 `c.b.m.e.p.MybatisPlusInterceptor` 类。
    - 使用 `new MybatisPlusInterceptor()` 创建MP拦截器实例。
    - 使用 `new PaginationInnerInterceptor(DbType.MYSQL)` 创建MySQL分页内置拦截器实例。
    - 使用 `interceptor.addInnerInterceptor(内置拦截器实例)` 对MP拦截器添加指定内置拦截器实例。
- 在业务接口 `service.SnakeInfoService` 中开发分页的业务方法：
    - `Page<SnakeInfo> paging(long current, long size)`
- 在业务类 `service.impl.SnakeInfoServiceImpl` 中重写分页的业务方法：
    - `new Page<>()`：构建一个分页实例，参数为当前第几页和每页显示多少条。
    - `mapper.selectPage()`：调用分页方法，参数为分页实例和queryWrapper实例。
    - `page.getCurrent()`：返回当前是第几页。
    - `page.getSize()`：返回每页显示多少条。
    - `page.getTotal()`：返回一共有多少条。
    - `page.getPages()`：返回一共有多少页。
    - `page.getRecords()`：返回分页数据。
    - `page.hasNext()`：返回是否存在下一页。
    - `page.hasPrevious()`：返回是否存在上一页。
- 在控制器 `controller.SnakeInfoController` 中开发分页控制方法并测试。

# 第四阶：SpringSecurity

**心法：** spring-security是spring用于提供声明式安全访问控制解决方案的安全框架：

- 认证：核心是比对用户的账号密码身份等信息，而验证指的是某些规则的匹配。
- 授权：为用户赋权或赋角色。

**武技：**
- 添加第三方依赖：
    - `org.springframework.boot.spring-boot-starter-web`：spring-web核心包。
    - `org.springframework.boot.spring-boot-devtools(runtime)`：热部署包。
    - `org.springframework.boot.spring-boot-starter-test(test)`：springboot测试包。
    - `org.springframework.boot.spring-boot-starter-thymeleaf`：thymeleaf核心包。
    - `org.springframework.boot.spring-boot-starter-security`：security核心包。
    - `org.thymeleaf.extras.thymeleaf-extras-springsecurity5`：thymeleaf整合security。
    - `org.springframework.security.spring-security-test(test)`：security测试包。
    - `org.projectlombok.lombok`：lombok工具包。
- 主配添加thymeleaf相关配置即可。
- 开发测试页面 `templates/view/security/start.html`：内容随意。
- cli: `localhost:8081/view/security/start.html`：发现整个项目资源都被security保护起来：
    - 所有请求重定向到security默认的登录路由 `/login` 并展示security内置的登录页面。
    - 输入账号 `user` 和在日志中生成的随机密码，点击登录。
    - 登陆成功后，security默认访问登陆前一次的URL，404时默认寻找 `error.html`。
    - 测试时注意浏览器缓存问题，浏览器完全关闭时视为注销登陆。
- 自定义账密：在主配中设置自定义账密后，日志将不再生成随机登录密码：
    - `spring.security.user.name=joezhou`：自定义security账号。
    - `spring.security.user.password=123`：自定义security密码。
    - `spring.security.user.roles=TEST`：自定义账号角色，逗号分隔，不要使用 `ROLE_` 前缀。
- cli: `localhost:8081/view/security/start.html`：输入自定义账号密码，登录成功。

## 资源放行

**心法：** 对于首页，广告页等请求不需要security保护，需要放行：
- 开发配置类 `security.config.SecurityConfig`：
    - 标记 `@Configuration` 声明为配置类。
    - 标记 `@EnableWebSecurity` 以启用security功能。
    - 继承 `o.s.s.c.a.w.c.WebSecurityConfigurerAdapter` 类并重写 `configure()`。
- 重写方法中：
    - `http.authorizeRequests()`：权限请求设置。
    - `.antMatchers(URL).permitAll()`：表示放行某些指定规则的请求URL。
    - `.anyRequest().authenticated()`：表示其余的任何请求都需要认证。
    - `.and()`：用于连接并列关系的配置。
    - `.formLogin()`：用于恢复security表单登陆功能。
- cli: 除了以下URL全部被security保护：
    - `localhost:8081/index.html`
    - `localhost:8081/ad.html`
    - `localhost:8081/`

## 重写登陆页面

**武技：** 
- 开发动作类 `controller.UserController`：
    - `loginRouting()`：用于提供登录页面 `templates/login.html` 的路由。
    - `mainRouting()`：用于提供登录成功页面 `templates/main.html` 的路由。
    - `SecurityContextHolder.getContext().getAuthentication()`：登陆后可随时获取security认证信息。
- 在配置类 `.formLogin()` 后额外配置，所有URL都建议使用动作方法的路由：
    - `.loginPage(URL)`：设置自定义登录页面路由。
    - `.loginProcessingUrl(URL)`：设置登录处理路径，命名随意。
    - `.successForwardUrl(URL)`：设置登录成功页面的路由，缺省跳转至上次的请求路径。
    - `.failureForwardUrl(URL)`：设置登录失败页面的路由，缺省跳转至 `loginPage()` 指定的路由。
    - `.permitAll()`：表示放行以上全部路径或路由。
    - `.csrf().disable()`：禁止跨域防护，即允许跨域请求，缺省时自定义表单登录功能失效。
- 开发登录页面 `templates\login.html`：
    - 引入thymeleaf标签：`xmlns:th="https://www.thymeleaf.org"`。
    - 设计表单，`action` 的值必须和 `loginProcessingUrl()` 配置的路径一致。
    - 账号和密码控件建议命名为 `username` 和 `password`。
    - `${param.error}`：获取账密错误时，security跳转至 `loginPage()` 指定的路由时附加的 `?error`。
    - `${param.logout}`：获取注销登录时，security跳转至 `loginPage()` 指定的路由时附加的 `?logout`。
- 开发登录成功页面 `templates\main.html`：
    - 引入security标签：`xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"`。
    - `sec:authorize="isAuthenticated()"`：仅在认证成功时才会展示标签及标签中的内容。
    - `sec:authentication="name"`：获取认证信息中的账号信息，其余同理。
    - `sec:authentication="principal"`：获取认证信息中的主要信息，如密码（受保护），过期时间等。

## 配置用户数据

**武技：** 自定义用户数据，如账密，角色和权限等，此时主配中配置的账密即使保留也会失效：
- 开发实体类 `entity.Permission/Role/User`：集合属性需要给初始值，否则空指针。
- 开发工具类 `util.DataUtil` 模拟数据库数据：
    - `admin/123` 为 `ROLE_ADMIN` 角色，拥有 `insert/select/update/delete` 权限。
    - `zhaosi/123` 为 `ROLE_COMM` 角色，拥有 `select` 权限。
- 开发配置类 `config.PasswordEncoderConfig`：security规定密码必须进行加密才可使用：
    - security默认的 `{noop}` 前缀加密方式不安全，建议更换为BCRYPT加密方式：
    - IOC `o.s.s.c.p.PasswordEncoder`：对其实现类 `o.s.s.c.b.BCryptPasswordEncoder` 进行实例化。
- 开发业务类 `service.UserDetailsServiceImpl`：通过构造注入密码加密类。
- 重写业务方法 `o.s.s.c.u.UserDetailsService.loadUserByUsername()`：
    - 根据前端传递过来的账号，从数据库中获取对应的 `User` 实体。
    - `passwordEncoder.encode(password)`：获取密码并对其进行BCRYPT加密。
    - 构建 `UserDetails` 接口子类 `o.s.s.c.u.User` 对象并返回，构造需要账密和角色/权限集合。
- 开发配置类 `config.SecurityConfig`：用自己的业务类覆盖security默认的业务类：
    - 通过属性注入业务类，构造器注入会产生A构造注入B，B构造注入A的环形注入问题，导致服务器启动崩溃。
    - 通过属性或构造注入密码加密类。
    - 使用 `configureGlobal()` 注入 `AuthenticationManagerBuilder` 对象。
    - `builder.userDetailsService()`：使用自定义业务类以覆盖security默认使用的业务类。
    - `service.passwordEncoder(passwordEncoder)`：设置密码加密方式。
- 分别使用 `admin/123` 和 `zhaosi/123` 登陆并查看角色/权限信息，以及观察两个密码123加密后的值是否一样。

## 路由访问权限

**心法：** 动作方法可以针对不同的角色/权限进行选择性开放：
- 开发测试页面 `templates/auth.html`：添加一些测试超链接。
- 开发动作类 `controller.AuthController`：动作方法标记 `@PreAuthorize` 以设置访问规则：
    - `@PreAuthorize("hasRole(A)")`：仅A角色可以访问我。
    - `@PreAuthorize("hasAnyRole(A, B)")`：仅A或B角色可以访问我。
    - `@PreAuthorize("hasAuthority(A)")`：仅A权限可以访问我。
    - `@PreAuthorize("hasAnyAuthority(A, B)")`：仅A或B权限可以访问我。
- 配置类中开启 `@PreAuthorize` 注解支持：`@EnableGlobalMethodSecurity(prePostEnabled=true)`。
- 开发页面 `auth-success.html`，测试成功时跳转，失败时默认自动寻找 `error.html`，缺省报错。
- cli: `auth.html` 测试不同用户的动作方法访问权限。

## 友好注销用户

**武技：** 浏览器完全退出时表示注销当前登录的用户，但不够友好：
- 开发页面 `logout.html` 页面：布局注销按钮。
- 在动作类 `controller.UserController` 中添加一个对应注销请求的路由方法，空方法并返回void即可。
- 在配置类 `.formLogin()` 后额外配置：
    - `.and().logout()`：定义注销的相关配置。
    - `.logoutUrl()`：必须一致和注销请求URL，以及动作方法路由保持一致。
    - `.logoutSuccessUrl()`：注销成功后的跳转页面，建议使用动作方法路由，缺省调用 `loginPage()` 指定的页面。
    - `.deleteCookies("JSESSIONID")`：删除掉cookie中的JSESSIONID，否则连接一直维持，不算注销。
    - `.invalidateHttpSession(true)`：使session失效。
- 登录再注销，跟踪 `F12/Application/Cookie/JSESSIONID` 值并测试注销后的账户是否仍可访问项目资源。

## 重写登录业务

**武技：** 若希望登陆成功或失败时返回JSON数据而不跳页，则需重写登录成功或失败的处理器类：
- 开发登陆成功处理器 `handler.CustomLoginSuccessHandler`：
    - 重写 `o.s.s.w.a.SavedRequestAwareAuthenticationSuccessHandler.onAuthenticationSuccess()`。
    - `new DefaultRedirectStrategy().sendRedirect(req, resp, URL)`：可使用重定向跳页。
    - `resp.getWriter().write()`：可使用 `repsonse` 回写数据。
- 开发登陆失败处理器 `c.j.s.handler.CustomLoginFailureHandler`：
    - 重写 `o.s.s.w.a.SimpleUrlAuthenticationFailureHandler.onAuthenticationFailure()`。
    - `new DefaultRedirectStrategy().sendRedirect(req, resp, URL)`：可使用重定向跳页。
    - `resp.getWriter().write()`：可使用 `repsonse` 回写数据。
- 在配置类中注入这两个处理器类。
- 将配置类 `configure()` 中的 `successForwardUrl()` 替换为：
    - `.successHandler(customLoginSuccessHandler)`：设置自定义登录成功处理器。
    - `.failureHandler(customLoginFailureHandler)`：设置自定义登录失败处理器