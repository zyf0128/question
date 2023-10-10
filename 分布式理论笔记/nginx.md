# Nginx 笔记 

重点在于： 

这次实验目的： 验证 nginx 负载均衡的效果。

我们把两个 springboot 项目 放到 nginx 还是里面运行。 然后配置负载均衡为 ： 加权轮询。

修改nginx主配 `%NGINX_HOME%/conf/nginx.conf`：

- 在 `http{}` 中使用 `upstream` 负载均衡器定义集群服务列表配置区，如 `upstream apps{}`，均衡器名随意。   
- 在 `apps{}` 中使用 `server 127.0.0.1:13001 weight=2;` 配置nginx-01服务，weight表示轮询权重。
- 在 `apps{}` 中使用 `server 127.0.0.1:13002 weight=1;` 配置nginx-02服务，weight表示轮询权重。
- 在 `location / {}` 中添加 `proxy_pass http://apps;`：拦截全部请求并代理转发到集群服务列表 `apps` 中。

```java
		upstream apps{
			server 127.0.0.1:13001 weight=2;
			server 127.0.0.1:13002 weight=1;
		}

        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://apps;
        }
```

####  nginx + redis  进行 session ID 的共享

我们来想象一下这样的场景： 

有两台服务器 A B ， 用户在 服务器A 登录， 服务器A 中的session 就有了 用户的登录信息。 染病然后用户第二次 登录的时候，访问的是 服务器B  但是由于 服务器B 中并没有 该用户的登录信息， 所以就会让用户重新登陆， 但是这一次 对于用户来说应该是 不用登陆的， 但是却需要登录。 

这就是 session 在 不同的服务器之间 没有进行共享造成的。

那么怎么解决呢？

目前最好的办法是： 使用 redis 进行 session 的共享。因为 redis 是跑在内存上的，而且是数据库可以存储数据，又可以进行持久化。 这样做还可以解决 session 丢失 造成的很多问题。 

怎么使用呢？



- 添加两个依赖， 然后在主配中进行 配置；然后就可以了。

```
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
```





### 一些概念

**心法：** nginx是一款开源的服务器和反向代理服务器，邮件服务器，支持很多第三方的模块扩展：

#### **正向代理**：

客户端的代理，对客户端负责：

- 代理流程：客户端发送请求到代理服务器，代理服务器转发请求到具体服务器，响应也是原路返回。
- 破解限制：某服务器限制客户端直接访问，则可以找个可访问的代理服务器来访问该服务器，如科学上网。
- 加速访问：电信客户端访问联通服务器太慢，则可以找个电信联通都能访问的代理服务器调节，如游戏加速器。
- 缓存数据：每次请求获取的数据都可以缓存到代理服务器中，后续相同的请求可以直接从缓存中获取。
- 保护客户端隐私：服务端仅知道请求来自于哪个代理服务器，但并不知道请求具体来自于哪个客户端。

#### 反向代理：

服务端的代理，对服务端负责：

- **代理流程**：代理服务器接收到请求之后，按一定规则分发给某个具体的服务器，响应也是原路返回。
- **负载均衡**：代理服务器可以使用轮询，加权轮询，ip_hash，url_hash等策略分发请求，分摊服务器集群的压力。
- **保护服务端隐私**：客户端仅知道请求发送给了哪个代理服务器，但并不知道请求具体由哪台服务器进行处理。

- **nginx服务端工作模式**：nginx采取多进程模式，启动nginx服务时，后台会至少启动2个进程，分为两类：
   - master：守护进程，负责向worker转发外界信号，监控worker状态，当worker异常时，自动重启新的worker进程。
   - worker：工作进程，负责监听端口和处理用户请求，worker之间相互隔离独立，worker数量建议配置与CPU核数一致。
- nginx服务端启动流程：以下对 `%NGINX_HOME%/logs/nginx.pid`，简称PID文件：
   - 使用命令启动`nginx`服务时，自动生成PID文件，该文件用于记录nginx中master进程的ID号。
   - 双击程序启动`nginx`服务时，不会生成PID文件，此时只能在任务管理器中手动结束相关进程，不建议。
   - 使用命令关闭`nginx`服务时，会根据PID文件内容来结束nginx所有后台进程，之后自动删除PID文件。
   - 多次启动`nginx`服务，不会端口占用，但会导致PID文件内容发生覆盖，进而导致之前的nginx进程失联并泄露。

#### 负载均衡的五种策略

1.**轮询**（默认）每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

2、**指定权重**，指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。（**常用）**

3、**IP绑定** ip_hash，每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

4、**fair**（第三方）按后端服务器的响应时间来分配请求，响应时间短的优先分配。

5、**url_hash**（第三方）按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

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

## 详解nginx配置文件

**心法：** **nginx配置文件默认可分为5部分，其中每个指令都必须用分号结束：**

| 全局区       | 配置影响nginx全局的指令                                      |
| ------------ | ------------------------------------------------------------ |
| events区     | 配置影响nginx服务器或与用户的网络连接                        |
| http区：     | 可以嵌套多个server区，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置 |
| upstream区： | 用户自定义配置服务集群，集群名不能重复                       |
| location区   | 配置请求的访问规则，匹配的是域名之后的uri部分，同一个server区中可以配置多个location区 |



##### 1.全局区：

配置影响nginx全局的指令：第1项是linux配置：

- `user nobody`：启动worker进程的用户和组，windows上不能写该配置。
- `worker_processes 1`：生成的worker进程数，设置为auto时可以自动设置为机器的CPU核数。
- `error_log logs/error.log`：错误日志文件，默认 `logs/error.log`，支持额外指定日志级别。
- `pid logs/nginx.pid`：PID文件，默认 `logs/nginx.pid`，用于记录当前master进程的id号。

##### 2.events区：

配置影响nginx服务器或与用户的网络连接，第234项是linux配置：

- `worker_connections 1024`：每一个worker进程能并发处理的最大连接数，包括前后端所有连接。
- `use epoll`：设置处理连接请求的事件驱动模型，可选 `select/poll/kqueue/epoll` 等，默认 `epoll` 模型。
- `accept_mutex on`：规避惊群，即一个请求会同时唤醒全部睡眠进程（但只有一个进程能获得连接），以提高性能。
- `multi_accept on`：设置一个进程可以同时接受多个请求，默认为off，一个进程一次只能接收一个请求。

##### 3.http区：

可以嵌套多个server区，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置：

- `include mime.types`：在本配置中导入 `mime.types` 文件，该文件是扩展名与文件类型的映射表。
- `default_type application/octet-stream`：nginx不识别的文件类型会默认进行下载操作，默认 `text/plain`。
- `access_log logs/access.log`：访问日志文件，默认 `logs/access.log`，采用 `log_format` 配置的格式。
- `log_format xxx`：访问日志格式，默认 `客户端IP 用户 访问时间 请求信息/状态 内容大小 请求源头 浏览器`。
- `sendfile on`：开启高效文件传输模式，默认off，普通应用建议开启，下载相关的应用建议关闭。
- `tcp_nopush on`：开启sendfile的情况下，会将请求合并，然后统⼀发送给客⼾端。
- `keepalive_timeout 65`：http连接超时时间，默认65秒，上传文件时设置大一些，避免超时断开连接而上传失败。
- `gzip no`：对网络传输的数据内容进行压缩以提升传输效率。

##### 4.upstream区：

用户自定义配置服务集群，集群名不能重复，集群中每个server都可以额外添加以下设置：

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

#####     5.location区：

配置请求的访问规则，匹配的是域名之后的uri部分，同一个server区中可以配置多个location区

- `root html`：配置请求资源在服务器上的真实路径，支持使用相对nginx家目录的相对路径或绝对路径。
- `index index.html index.htm`：配置首页列表，会在root指定的目录下寻找。
- `proxy_pass http://apps`：请求转发到apps服务集群列表，并使用负载均衡的方式进行调用。
- `deny 192.168.0.1`：可以配置拒绝的IP地址。
- `allow 192.168.0.2`：可以允许的拒绝的IP地址。

详解location匹配规则

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

