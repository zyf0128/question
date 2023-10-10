> Java道经第2卷

# 第一阶：MySQL

**心法：** mysql是一个关系型数据库，由瑞典mysql-ab公司开发，目前属于oracle旗下产品，在web应用方面是最好的RDBMS应用软件之一：

- Relational Database Management System：关系数据库管理系统的数据都是存放在表中的。
- 关系型数据库VS非关系型数据库：
  - 关系型数据库由1或N个表格组成，格式一致易于维护但灵活度稍欠，高并发时读写性能较差。
  - 非关系型数据库的本质是一种数据结构化存储方法的集合，如文档，键值对，图片等，使用灵活，速度快，成本低，但不支持SQL导致学习和使用成本较高，在复杂查询方面稍欠，无事务处理等问题。
- mysql数据库：
  - mysql软件采用了双授权政策，分为社区版（开源免费，支持定制）和商业版，体积小，速度快。
  - mysql支持5000万条记录的数据仓库，32位系统表文件最大可支持4GB，64位系统支持最大的表文件为8TB。
  - mysql使用标准的SQL数据语言形式，跨平台，支持多种语言，如 `C/C++/Python/Java/Perl/PHP/Ruby` 等。
- 存储引擎：mysql存储引擎在不同的版本下，使用的也不同：
  - `MyISAM` 是mysql5之前的默认数据库引擎，最为常用，拥有较高的插入，查询速度，但不支持事务。
  - `InnoDB` 是mysql5之后的默认数据库引擎，支持ACID事务，支持行级锁定。
- SQL语言：`Structured Query Language` 结构化查询语言，是在关系型数据库上执行数据操作，数据检索以及数据维护的标准语言：
  - 目前数据库厂商实现的都是SQL92标准，但是不是说所有的数据库的SQL都一样，就比如少数民族也得遵循中国的高考标准，但是加分。

**武技：** 卸载mysql数据库：

- 在服务中查看是否存在MySQL服务，若存在右键停止，并卸载该服务：
  - cmd: `sc delete MySQL57`
- 控制面板卸载mysql相关服务。
- 删除 `Program Files/Program Files(x86)/Program Data` 目录中所有与mysql相关的目录。
- 运行 `regedit` 打开注册表，删除以下两个目录中的所有mysql相关目录：
  - `HKEY_LOCAL_MACHINE\SYSTEM\ControlSetXXX\Services\Eventlog\Application`
  - `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application`
- 重启电脑，卸载完成。

**武技：** 安装mysql数据库：

- 双击 `mysql-installer-community-5.7.23.0.msi` 进入安装界面：接受协议，Next。
- `Choosing a Setup Type` 界面：勾选 `Server only`，Next。
- `Installation` 界面：点击 `Execute` 执行安装，进度条结束后，Next。
  - 若提示电脑缺少环境，点击 `Execute` 安装c++环境，Next。
- `Product Configuration` 界面：Next。
- `Group Replication` 界面：Next。
- `Type and Networking` 界面：确认端口号为3306，Next。
- `Accounts and Roles` 界面：填两遍登录密码如 `root`，Next：
  - 若需要额外指定用户，在下方进行添加。
- `Windows Service` 界面：确认服务名为 `MySQL57`，Next。
- `Plugins and Extensions` 界面：Next。
- `Apply Configuration` 界面：点击 `Execute` 开始安装，安装结束后点击 `Finish` 回到 `Product Configuration` 界面。
- `Product Configuration` 界面：Next。
- `Installation Complete` 界面：点击 `Finish` 完成安装。
- 配置mysql环境变量：
  - MYSQL_HOME: `C:\Program Files\MySQL\MySQL Server 5.7`
  - Path：`%MYSQL_HOME%\bin`
- 查看mysql版本：
  - cmd: `mysqladmin --version`
- 自启动管理：
  - 在服务中找到mysql57服务，右键更改启动类型为手动。
- 错误整理：
  - 系统缺少NetFramework4.0：安装插件 `.NetFramework4.5.exe`
  - 2502/2503错误：找到 `C:\Windows\Temp` ，右键属性 -> 安全 -> 添加everyone权限（完全控制）。

**武技：** 使用cmd操作数据库：

- 启动/停止mysql服务：
  - cmd: `net start/stop mysql57`
- 登录mysql服务端：
  - cmd: `mysql [-D库名] [-hlocalhost] -uroot -p`
- 登出mysql服务端：
  - mysql: `exit/quit`
- 查看数据库实例：
  - mysql: `show databases`
- 创建数据库实例：
  - mysql: `create database 库名 [character set utf8mb4];`
- 删除数据库实例：
  - mysql: `drop database 库名`
- 使用数据库实例：
  - mysql: `use 库名`
- 查看数据库实例元数据（和表中的记录无关的数据）：
  - `select version()/database()/user()`：查看当前数据库版本/当前库名/当前用户名。
  - `show status/variables`：查看当前数据库状态/配置变量。
  - `show tables`：查看当前数据库表，无法看到临时表。

**武技：** 在IDEA中连接mysql服务端：

- 创建maven-jar项目 `mysql`，添加jdk8配置。
- 创建mysql连接：
  - `Database选项卡 -> + -> Data Source -> MySQL`
- 添加mysql本地驱动包 `mysql-connector-java-8.0.15.jar`：
  - `Driver:MySQL -> Go to Driver -> + -> Custom Jars`
- 填写mysql连接基本信息：
  - `Name/Comment/Host/Port/User/Password`
- 填写mysql连接的URL连接串：
  - `jdbc:mysql://localhost:3306?serverTimezone=UTC`
- 点击 `Test Connection` 进行测试。
- 添加mysql方言：
  - 找到 `File -> Settings -> Languages & Frameworks -> SQL Dialects`。
  - 选择 `Global SQL Dialect/Project SQL Dialect` 为 `MySQL`。

## DDL数据定义语言

**心法：** 数据定义语言DDL负责操作数据库的元信息，如表，用户，列，索引等。

### DDL用户

**心法：**

- 创建用户：`grant all on 库名.* to '账号'@'localhost' identified by '密码' with grant option`：
  - `with grant option` 表示该用户可以为其他用户授权。
- 刷新权限：`flush privileges`：不刷新可能导致授权失败。
- 查询用户：`select user, host from mysql.user`：IDEA爆红不影响。
- 删除用户：`drop user '账号'@'localhost'`。
- 修改密码(cmd)：`mysqladmin -u账号 -p旧密码 password 新密码`。
- src: `ddl.用户.sql`

### DDL表

> decimal(n, m) 比 double(n, m) 精度高：小数位数最多m位，超出四舍五入，不足补0，且小数位数 + 整数位数 + 小数点占1位，总体位数不能超出n，超出时报错。

**心法：** mysql的表是所有账号通用的：

- 建表：`create table if not exists 表名 (字段列表) engine=引擎 default charset=字符集 comment '注释'`
  - 字段可用 `tinyint(n)/int(n)/decimal(n, m)/char(n)/varchar(n)/datetime` 等类型。
  - 字段可用 `unsigned` 标记为无符号，默认有符号。
  - 字段可用 `not null` 标记为不能插入空数据，列最好不为null。
  - 字段可用 `auto_increment` 标记为该列自增，只能用于数字列。
  - 字段可用 `comment` 指定注释。
  - 字段可用 `default` 指定默认值，字段尽量给默认值。
  - 字段列表末尾可用 `primary key ()` 指定本张表的主键列。
- 当表不存在时建表：`create table if not exists user()`
- 查看建表语句：`show create table 表名`
- 查看表结构：`desc 表名/show columns from 表名`
- 查看表性能：`show table status like '%表名%'`
- 重命名表：`rename table 表名 to 新表名`
- 删除表：`drop table 表名`
- 当表存在时删除表：`drop table if exists 表名`
- 清空表：`truncate table 表名`
- 增加一列：`alter table 表名 add 列名 类型`
- 修改某列的类型：`alter table 表名 modify 列名 新类型`
- 重命名列：`alter table 表名 change 字段名 新字段名 字段属性`
- 删除某列：`alter table 表名 drop column 列名`
- 快速复制表数据：`create table 新表名 (select * from 表名)`
- 创建临时表：用于保存临时数据，支持正常的CRUD，仅在当前连接可见，关闭连接销毁并释放所有空间，当然也可以手动drop：
  - 在建表语句中的 `table` 前额外添加 `temporary` 修饰即可。
- src: `ddl.表.sql`

### DDL约束

**心法：** 约束就是对column添加一些强制校验规则以保证数据的正确性，添加约束前尽量保持表中无数据或者无错误数据：

- 查询指定表的约束：非空约束除外：
  - `select * from information_schema.table_constraints where table_name = '表名'`
- 非空约束 `not null`：该列非空，即不能插入空值：
  - 造表时：`字段 类型 not null`
  - 造表后：`alter table 表名 modify 字段 类型 not null`
  - 删除它：`alter table 表名 modify 字段 类型 null`
- 唯一约束 `unique`：该列唯一，即不能插入重复数据：
  - 造表时：`字段 类型 unique` 或字段列表末尾添加 `,unique (字段)`
  - 造表后：`alter table 表名 add [constraint 约束名] unique (字段)`，约束名默认字段名。
  - 删除它：`alter table 表名 drop key/index 约束名`
- 主键约束 `primary key`：该列唯一且非空，一张表只能有一个主键约束列：
  - 造表时：`字段 类型 primary key` 或字段列表末尾添加 `primary key (字段)`
  - 造表后：`alter table 表名 add [constraint 约束名] primary key (字段)`，约束名默认primary。
  - 删除它：`alter table 表名 drop primary key`
- 外键约束 `foreign key`：主表的某字段连接从表的主键或唯一约束字段，二者必须同类型同长度：
  - 造表时：末尾添加 `,foreign key (外键字段) references 从表(从表主键)`
  - 造表后：`alter table 表名 add [constraint 约束名] foreign key (字段) references 从表(主键)`，约束名随机。
  - 删除它：`alter table 表名 drop foreign key 约束名`
- src: `ddl.约束.sql`

### DDL索引

**心法：** 索引底层使用自平衡B-tree结构，可以提高查询效率，但建立索引时需要额外创建一张索引结构表，耗时长，占空间，需要定期维护，且索引会随着表中记录的变化而变化，这意味着每条记录的操作将为此多付出4、5次的磁盘I/O，所以尽量只在经常被查询的字段上建立索引：

- 查看指定表中的索引：`show index from 表名`
- 建立索引：对字段添加唯一/主键约束时，会自动创建索引：
  - `create index 索引名 on 表名 (字段)`，索引名必须指定。
  - 若给a/b/c字段添加一个联合索引，则单独查询a/b/c时都走索引，但注意不要中间断档。
- 删除索引：
  - `drop index 索引名 on 表名`
- src: `ddl.索引.sql`

### DDL视图

**心法：** 视图就是一条查询sql语句的结果集，不占空间，且除了本身外不包含任何数据：

- 创建视图：不建议使用 `*` 作为结果集的查询字段：
  - `create view 视图名 as (结果集)`：创建视图。
  - `create or replace view 视图名 as (结果集)`：创建/修改视图。
- 删除视图：`drop view 视图名`
- src: `ddl.视图.sql`

## DML数据操作语言

**心法：** DML负责操作表中的记录，如 `insert/delete/update` 等语句：

- 增加：`insert into 表名 (字段列表) values (值列表)`
  - 值列表必须和字段列表的个数，顺序，类型一一对应。
  - 按表正确的顺序插入全部字段的值时，可以省略字段列表。
  - 如果想要批量插入，可以在一条语句中，直接用逗号分隔多组值列表。
- 修改：`update 表名 set 字段名=值, 字段名=值... where 条件`
- 删除：`delete from 表名 where 条件`：
  - 如果不指定删除条件，表示全部删除，永远不要这样做。
- src: `dml.数据操作语言.sql`

### 事务机制

**心法：** 事务是一组不可分割的业务关系，在SQL中指的是一组不可拆分的DML语句，这组语句要么都成功，要么都失败：

- 事务主要用于处理操作量大，复杂度高的数据，用来维护数据库的完整性。
- 事务的关键字：`commit/rollback`。
- 事务必须满足原子性 `Atomicity`：一组事务，要么成功提交，要么失败回滚。
- 事务必须满足稳定性 `Consistency`：一个事务执行之前和执行之后数据库都必须处于一致性状态。
- 事务必须满足隔离性 `Isolation`：并发环境中，不同事务同时操作相同数据时，事务之间互不影响。
- 事务必须满足持久性 `Durability`：只要事务成功，它对数据库所做的更新就必须永久的保存下来。

### 事务隔离级别

**心法：** 事务隔离级别指一个事务对数据库的修改与并行的另一个事务的隔离程度，即当我操作数据库的时候，允许你做什么，当两个并发事务同时访问数据库表相同的行时，不同的数据隔离级别会发生不同的并发现象：

- `read uncommitted` 读未提交：允许读取未提交的数据，此时可能出现脏读现象：
  - 案例：公司发工资了，领导写了一张通知发给财务，内容为奖励四哥1000元，此时四哥正好在财务部门并看到了通知单，高兴地回家了，可不幸的是，领导突然发现四哥出勤有问题，于是迅速回滚了事务，补发了第二张通知为奖金扣1000元并提交事务，最终四哥需要扣除1000元。
  - 总结：t1读取了已经被t2更新但还没有被t2提交的字段，之后若t2回滚，t1读取的内容就是临时且无效的脏数据。
- `read committed` 读已提交：只允许读取已提交的数据，避免脏读，但会发生不可重复读的现象：
  - 案例：四哥拿着工资卡去消费，POS机读取到余额1000元，而此时四嫂也正好在网上淘宝，从四哥工资卡花掉999元，并在四哥消费前提交了事务，之后四哥输入密码结账，结果POS机提示余额不足。
  - 总结：t1读取某数据时，允许t2修改该数据，之后t1再次读该数据时，值就有可能变化了。
- `repeatable read` 重复读：我操作某行数据时不允许别人操作，此时可以避免脏读和不可重复读，但是会发生幻读：
  - 案例：某天，四嫂查询到四哥当月工资卡共消费80元，而四哥此时正好在外面胡吃海塞并消费1000元，即新增了一条1000元的消费记录，并提交了事务，随后四嫂将四哥当月消费的明细打印到A4纸上，却发现消费总额为1080元，四嫂很诧异，以为出现了幻觉。
  - 总结：t1读取某数据时，允许t2插入一些新的数据行，之后若t1再次读该表，就有可能会多出几行数据。
- `serializable` 序列化读：一个事务执行完毕再执行下一个：
  - 总结：t1读取某数据时，不允许t2做任何操作，隔离级别最高，可避免脏读/不可重复读/幻读，但性能最差。
- mysql支持全部四种事务隔离级别，默认使用的是 `repeatable read`：
  - `set transaction isolation level read uncommitted/read committed/repeatable read/serializable`

### 数据导入导出

**武技：** 利用idea导出表：

- 在指定的表上右键，选择 `dump data to file(s)`：
  - 勾选 `add table definition(sql)`：用于生成表结构。
  - 勾选 `single file`：用于生成一个单独的sql文件。
- 再次右键，选择 `dump data to file(s) -> SQL Insert`，选择路径，导出文件。

**武技：** 利用idea导入表：

- 在指定的数据库上右键，选择 `Run SQL Script`
- 找到你要导入的sql文件，点击ok。

**武技：** 利用cmd导出表：

- cmd: `mysqldump -hlocalhost -u帐号 -p 库名 [表1 表2...]> 文件绝对路径.sql`：
  - `-p` 后面不能加密码，命令末尾不要添加分号。
  - 在库名前添加 `--default-character-set=utf8mb4` 可以指定编码。
  - 在库名前添加 `-d`，代表删除数据，只导出表结构。

**武技：** 利用cmd导入表：

- cmd: `mysql -uroot -p`
- mysql: `source 文件绝对路径.sql`

## DQL数据查询语言

**心法：** DQL负责查询表中的记录，所有select开头的语句全属于DQL语句：

- 引入部门表和员工表：
  - `资源/SQL/mysql.sql`
- 基本查询：`select 字段列表 from 查询范围`：
  - 全查不建议使用 `*`，更不建议将 `*` 和其它字段一起查询，影响查询效率。
  - 代码语法不区分大小写，建议写成全大写有助于提高效率，且mysql表中的值也不区分大小写。
  - 字段可以起别名，用 `as` 标注，也可省略不写，别名不建议使用中文。
  - 表名可以起别名，但不建议使用 `as` 标注，对表起别名可以提高查询效率。
  - 字符类型数据用单/双引号标注，建议使用单引号。
  - 数字类型数据可以直接进行计算，null值和任何值进行计算结果都为null（可以使用ifnull函数进行处理）。
- 查询排序：`order by 排序字段列表 排序规则` 用于对结果集排序：
  - `asc` 代表升序，默认可省略，`desc` 代表降序。
  - `order by` 是对结果集的排序，所以必须要写在语句最后，也是程序最后一步运行的。
  - `order by` 不参与任何计算，也不会改变任何一个值。
- 条件查询：`where 条件` 用于在查询的过程中，逐条进行条件判断：
  - 即使多个条件，`where` 关键字也只能有一个。
  - 条件允许使用 `and/or/not/<>/!=` 等符号，且 `and` 优先级高于 `or`。
  - 为了提高效率，筛选过程慢，即筛选量大的条件，要尽量往后放。
- 模糊查询：`like` 用于在查询的过程中，进行模糊匹配：
  - `%` 表示随意个数占位，`_` 表示只占一位。
  - mysql使用 `\` 作为转义字符，也可使用 `escape` 单独指定。
  - mysql可以直接转义单引号，也可以使用两个单引号表示一个单引号。
- 范围查询：
  - `in(字段列表)`：字段的值只要出现在这个字段列表中就返回true。
  - `not in(字段列表)`：字段的值只要没出现在这个字段列表中就返回true。
  - `between x and y`：字段的值只要在x与y之间返回true，包括x也包括y。
  - `not between x and y`：字段的值只要不在x与y之间返回true，不包括x也不包括y。
- 条件函数：
  - `case when 条件 then 返回值 end`：相当于java中的switch结构。
  - `ifnull(字段, a)`：若字段为null，返回a，否则返回原字段。
  - `if(字段, a, b)`：若字段为null，则返回b，否则返回a，可嵌套。
  - `if(条件, a, b)`：若条件成立，返回a，否则返回b，可嵌套。
- src: `dql.基础查询.sql`

### 父子查询

**心法：** 利用a查询提供一个临时的结果集供b使用，则可以称a为b的子查询：

- 单行子查询：子查询先独立执行且只执行一次，结果返回一行数据，供父查询使用。
- 多行子查询：子查询先独立执行且只执行一次，结果返回多行数据，供父查询使用：
  - `any(子查询)`：某些信息符合，结果集中是or的关系，不等式专用。
  - `some(子查询)`：某些信息符合，结果集中是or的关系，等式专用。
  - `all(子查询)`：全部信息符合，结果集中是and的关系。
- 关联子查询：父查询每查询一条数据都需要先进行一次子查询，子查询无法独立运行：
  - 子查询只要有返回值（不在乎返回值是什么），则父查询输出，否则跳过。
  - 关联子查询的关键字是：`exists(子查询)/not exists(子查询)`。
- src: `dql.父子查询.sql`

### 分页查询

**心法：** `limit m, n` 用于进行分页查询，表示从第m条开始（包括m），显示n条：

- 若省略m，则表示从第1条开始。
- mysql需要从第一条开始进行分页操作，若m值过大，则建议先使用where将数据限定到m后再 `limit n`，以提升分页效率。
- src: `dql.分页查询.sql`

### 联表查询

**心法：** `select * from 表A join 表B on 联合条件` 用于联表查询，两表成功相连需要一个联合条件：

- 内联 `inner join`：仅仅显示满足连接条件的信息，`inner` 可省略。
- 左外联 `left join`：完整显示左侧信息，即使不满足连接条件。
- 右外联 `right join`：完整显示右侧信息，即使不满足连接条件。
- 自联：将自己的表分成多个部分进行联合。
- 联表后如果出现相同列名，建议使用表名或表名的别名作为前缀，加以区分。
- src: `dql.联表查询.sql`

### 分组查询

**心法：** `group by` 可以对1或N个字段进行分组，最终仅返回一列数据，且不会忽略null值：

- 分组查询 `from` 前的字段只建议是 `group by` 后出现的字段。
- 分组查询一般配合聚合函数如 `count(字段)/max(字段)/min(字段)/sum(字段)/avg(字段)` 等一起使用：
  - mysql的聚合函数不会忽略null值，需要自行处理。
- 分组查询在分组前进行条件筛选时使用 `where`，在分组后进行条件筛选时使用 `having`。
- src: `dql.分组查询.sql`

### 并集查询

**心法：** `union/union all` 用于返回两个结果集的并集，并去重/不去重：

- 参加运算的集合必须有相同的列，若不相同用null补，而且列的类型必须对应。
- 并集运算中，null值不进行忽略。
- src: `dql.并集查询.sql`

### 正则查询

**心法：** `regexp` 关键字可以进行正则查询：

- 格式：`select * from 表名 where 字段 regexp '正则表达式模板'`
- src: `dql.正则查询.sql`

# 第二阶：JDBC

**心法：** `Java Data Base Connectivity` 是java和数据库连接中间件，
是sun公司面对各个数据库提供的一组接口，
用于使用java代码连接和操作数据库，但前提是java程序中必须添加了对应数据库的驱动包，该jar包由对应的数据库厂商提供：

- 手动添加jar包：将驱动包拷贝到项目的lib目录下，点击 `Structure -> Libraries -> 加号 -> Java`，选择项目中的驱动包，OK至完成。

**武技：** 搭建JDBC，测试连接：

- 创建数据库实例，用户和表：
  - `资源/SQL/jdbc.sql`
- 添加依赖：
  - `mysql.mysql-connector-java(8.0.15)(runtime)`
  - `junit.junit(4.12)(test)`
- 驱动mysql数据库：
  - `Class.forName("com.mysql.cj.jdbc.Driver")`
- 定义连接串：`jdbc:mysql://IP地址:端口号/数据库名?参数列表`：
  - `username/password`：数据库用户名/密码，可不在此指定。
  - `serverTimezone=UTC`：指定时区为东8区，mysql8+版本的驱动必须填写。
  - `useUnicode=true&characterEncoding=utf8`：使用Unicode字符集的utf8编码实现。
  - `autoReconnect=false`：当数据库连接异常中断时，是否自动重新连接，默认false。
  - `maxReconnects=5`：当autoReconnect设置为true时，重试连接的次数，默认3次。
  - `initialTimeout=3`：当autoReconnect设置为true时，两次重连之间的时间间隔，单位是秒，默认2秒。
  - `autoReconnectForPools=true`：是否使用针对数据库连接池的重连策略，默认false。
  - `failOverReadOnly=true`：自动重连成功后，连接是否设置为只读，默认true。
  - `useSSL=false`：消除控制台的一个红色警告，使用SSL漏洞修复。
  - `connectTimeout`：和数据库服务器建立socket连接时的超时，单位是毫秒，0表示永不超时，默认0。
  - `socketTimeout`：socket操作（读写）超时，单位是毫秒，0表示永不超时，默认0。
- 获取连接：
  - `DriverManager.getConnection(连接串, 账号, 密码)`
- 测试连接是否关闭：
  - `connection.isClosed()`
- 无论测试是否成功，都需要将连接关闭以节省内存资源：
  - `connection.close()`
- tst: `jdbc.ConnectTest`

## DataSource

**武技：** 将驱动数据库，获取连接和关闭连接的代码封装到 `jdbc.v1.DataSource` 类中：

- 引入静态块：静态块唯一：
  - 驱动mysql数据库，需要驱动串。
- 开发一个获取连接的方法：
  - 通过DriverManager获取一个连接并返回，需要连接串，账号和密码作为参数。
- 开发一个关闭连接的方法：
  - 需要设置非空保护。
- tst: `jdbc.DataSourceTest`：
  - 实例化数据源类，调用获取连接的方法。
  - 测试获取到的连接是否关闭。
  - 调用关闭连接的方法。

### 使用连接池

**武技：** 开发 `jdbc.v2.DataSource` 并添加连接池功能：

- 在数据源类中开发一个连接池属性，建议 `List` 类型。
- 在静态块中创建10个连接并且放入连接池。
- 改造获取连接的方法：
  - 当前连接池中还有连接，从连接池中弹出一个连接并返回。
  - 当前连接池中没有连接，创建一个新的连接并返回。
  - 方法添加 `synchronized` 保护，以免多人获取到同一个实例。
- 改造关闭连接的方法：
  - 如果当前连接池中连接数量达到最大，则直接关闭新来的连接。
  - 如果当前连接池中仍有空余位置，则回收新来的连接。
- tst: `jdbc.DataSourceTest`：使用 `jdbc.v2.DataSource` 测试。

### 使用属性文件

**武技：** 开发 `jdbc.v3.DataSource` 并将数据源提取到属性文件中，以解耦代码：

- 创建资源目录：
  - 右键项目新建 `Directory`，命名 `resources`。
  - 右键资源目录，`Mark Directory as -> Resources Root`。
  - 在资源目录中创建属性文件 `jdbc/db.properties`。
  - 资源目录不能嵌套资源目录，但可以包含 `package/folder`。
- 将驱动串，连接串，账号密码等信息以 `K=V` 的形式提取到属性文件中。
- 在数据源类的静态块中，读取属性文件中的信息：
  - `PropertyResourceBundle.getBundle(属性文件地址)`：不要后缀。
  - `bundle.getString(String key)`：通过key来获取属性文件中的value值。
- tst: `jdbc.DataSourceTest`：使用 `jdbc.v3.DataSource` 测试。

### 使用工厂模式

**武技：** 开发 `jdbc.v4.DataSource` 并对数据源使用工厂模式以分离生产者和使用者：

- 开发一个数据源的工厂类：
  - 开发一个静态方法返回一个数据源实例。
- 修改JDBC类，直接调用数据源工厂的静态方法获取数据源实例。
- tst: `jdbc.DataSourceTest`：使用 `jdbc.v4.DataSource` 测试。

## JdbcTemplate

**心法：** JdbcTemplate类用于发送各种类型的SQL语句，需要注入据源类。

- src: `jdbc.v4.JdbcTemplate`

### DDL-execute()

**武技：** 开发执行DDL语句方法并测试：

- 获取一个 `Connection` 连接实例：
  - `dataSource.getConnection()`
- 获取一个 `Statement` 媒介实例，用于发送SQL，执行SQL并带回结果：
  - `connection.createStatement()`
- 将SQL发送到数据库，执行，并带回boolean结果：
  - `statement.execute(String sql)`：仅当返回结果的类型是 `ResultSet` 时为true。
- 顺序关闭媒介实例和连接实例，以节省资源。
- tst: `jdbc.JdbcTemplateTest.execute()`。

### DML-update()

**武技：** 开发执行DML语句方法并测试：

- 获取一个 `Connection` 连接实例：
  - `dataSource.getConnection()`
- 获取一个 `PrepareStatement` 媒介实例，该媒介类型支持动态绑定问号：
  - `connection.prepareStatement(String sql)`
- 为SQL中的第i个问号赋值o，i从1开始：
  - `statement.setObject(int i, Object o)`
- 将SQL发送到数据库，执行，并带回int结果：
  - `statement.executeUpdate()`
- 顺序关闭媒介实例和连接实例，以节省资源。
- tst: `jdbc.JdbcTemplateTest.insert()/update()/delete()`

### DQL-selectForList()

**武技：** 开发执行DQL语句中的查询多条结果集的方法并测试：

- 获取一个 `Connection` 连接实例：
  - `dataSource.getConnection()`
- 获取一个 `PrepareStatement` 媒介实例，该媒介类型支持动态绑定问号：
  - `connection.prepareStatement(String sql)`
- 为SQL中的第i个问号赋值o，i从1开始：
  - `statement.setObject(int i, Object o)`
- 将SQL发送到数据库，执行，并带回ResultSet结果集：
  - `connection.executeQuery()`
- 将ResultSet结果集转化为 `List<Map>` 结构：
  - `resultSet.getMetaData()`：获取元数据。
  - `metaData.getColumnCount()`：获取结果集的总列数。
  - `metaData.getColumnName(int i)`：获取第i个列的列名，i从1开始。
  - `resultSet.getObject(String columnLabel)`：通过列名获取对应的值。
- 顺序关闭结果集实例，媒介实例和连接实例，以节省资源。
- tst: `jdbc.JdbcTemplateTest.selectForList()`

### DQL-selectForMap()

**武技：** 开发执行DQL语句中的查询单条结果集的方法并测试：

- 直接调用JDBC中查询多条的方法，并进行空值保护。
- 取出List类型的结果集中唯一的一条记录，并返回。
- tst: `jdbc.JdbcTemplateTest.queryForMap()`

### DQL-selectForInt()

**武技：** 开发执行DQL语句中的查询结果集数量的方法并测试：

- 直接调用JDBC中查询单条的方法，得到Map型结果集。
- 通过Map型结果集中唯一的key值获取对应的值，强转为int型并返回。
- tst: `jdbc.JdbcTemplateTest.queryForInt()`

### DML-batchUpdate同类型

**心法：** `int[] batchUpdate(String sql, Object[]... params)` 负责批量处理多条相同类型的SQL，需要设置事务保护，返回批量DML操作的影响条目数数组。

**武技：** 开发执行批处理同类型SQL的方法并测试：

- 获取一个 `Connection` 连接实例：
  - `dataSource.getConnection()`
- 关闭连接实例 `connection` 的自动提交属性：
  - `connection.setAutoCommit(false)`：默认true。
- 获取一个 `PrepareStatement` 媒介实例，该媒介类型支持动态绑定问号：
  - `connection.prepareStatement(String sql)`
- 为SQL中的第i个问号赋值o，i从1开始：
  - `statement.setObject(int i, Object o)`
- 将拼装完整的SQL语句加入到预执行区域，等待批处理：
  - `pstmt.addBatch(sql)`
- 将预执行区域中的所有SQL语句发送到数据库，执行并返回int[]类型的结果：
  - `pstmt.executeBatch()`
- 通过连接实例 `connection`，手动提交或者回滚本次操作：
  - `connection.commit()`：提交操作。
  - `connection.rollback()`：在异常捕捉中进行处理。
- 在异常处理的 `finally` 块中，将自动提交属性还原：
  - `connection.setAutoCommit(true)`：默认为true。
- 顺序关闭媒介实例和连接实例，以节省资源。
- tst: `jdbc.JdbcTemplateTest.batchUpdateWithSameSql()`

### DML-batchUpdate不同类型

**心法：** `int[] batchUpdate(String... sqls)` 负责批量处理多条不同类型的SQL，需要设置事务保护，该方法需要传入一个完整SQL的数组，返回批量DML操作的影响条目数数组。

**武技：** 开发执行批处理不同类型SQL的方法并测试：

- 获取一个 `Connection` 连接实例：
  - `dataSource.getConnection()`
- 关闭连接实例 `connection` 的自动提交属性：
  - `connection.setAutoCommit(false)`：默认true。
- 获取一个 `Statement` 媒介实例：
  - `connection.createStatement()`
- 遍历SQL语句并分别加入到预执行区域，等待批处理：
  - `stmt.addBatch(sql)`
- 将预执行区域中的所有SQL语句发送到数据库，执行并返回int[]类型的结果：
  - `stmt.executeBatch()`
- 通过连接实例 `connection`，手动提交或者回滚本次操作：
  - `connection.commit()`：提交操作。
  - `connection.rollback()`：在异常捕捉中进行处理。
- 在异常处理的 `finally` 块中，将自动提交属性还原：
  - `connection.setAutoCommit(true)`：默认为true。
- 顺序关闭媒介实例和连接实例，以节省资源。
- tst: `jdbc.JdbcTemplateTest.batchUpdateWithDifferentSql()`

## 将JDBC导出jar包

**武技：** 将JDBC项目打包成到Java存档，即JAR文件：

- 创建一个新项目/模块，并开发JDBC代码：
  - 若需编写README.txt等附属文件，需要在classpath下创建。
- 配置jar包输出位置：
  - `Project Structure -> Artifacts -> 加号 -> JAR -> From Modules with dependicies`
  - 指定jar包输出的位置，建议默认在当前项目的out目录中，应用OK完成配置。
- 回到主界面构建jar包：
  - `Build -> Build Artifacts`
  - 选择你的jar包，点击 `build`，完成jar包导出。

# 第三阶：MyBatis

> Object Relation Mapping 对象关系映射旨在通过操作java对象的方式来操作数据库，如将类名映射表名，成员属性名映射表字段名，
> 成员属性类型映射表字段类型，类实例映射表记录等。

**心法：** [mybatis](https://github.com/mybatis/mybatis-3/releases) 是一个底层半封装了JDBC的持久层的开源ORM框架，
封装了驱动，连接，statement等业务代码，但不封装SQL语句，支持注解或XML方式单独开发SQL语句，
其前身是apache的ibatis项目，2010年迁移到了google-code并改名为mybatis，2013年11月迁移到github，其核心对象如下：

- `SqlSessionFactoryBuilder`：用于创建session工厂，且创建完session工厂后即可销毁：
  - 创建session工厂时需读取mybatis的主配文件。
  - 主配文件用于配置mybatis的环境信息和Mapper配置文件路径等。
  - Mapper配置文件用于配置CRUD语句块和SQL语句相关配置信息。
- `SqlSessionFactory`：session工厂用于获取一个session会话，可以重复使用，但不该被重复创建，建议以单例模式来管理。
- `SqlSession`：session会话用于获取数据库连接的接口，每个线程都该独享一个session会话，建议设置为局部变量。
- `Executor`：session会话中的SQL执行者，有基本和缓存两种执行器实现。
- `MappedStatement`：SQL执行者中，专门负责SQL组装和结果转换的的SQL管理者：
  - 每个SQL块都通过id属性对应一个SQL管理者者。
  - 输入映射：在执行SQL前，将输入的java实体类中的属性映射到SQL代码中。
  - 输出映射：在执行SQL后，将返回的SQL结果映射到java实体类中保存。

**武技：** mybatis基本配通：

- 开发数据库：创建数据库，用户，和数据库表：
  - `资源/SQL/mybatis.sql`
- 添加依赖：
  - `org.mybatis.mybatis(3.5.1)`：mybatis核心依赖。
  - `mysql.mysql-connector-java(8.0.15)(runtime)`：mysql驱动包，oracle需要ojdbc6.jar依赖。
  - `junit.junit(4.12)(test)`：junit包。
  - `org.projectlombok.lombok(1.18.12)(provided)`：lombok包。
  - `log4j.log4j(1.2.17)`：log for java日志接口包。
  - `org.apache.logging.log4j.log4j-core(2.3)`：log for java日志接口包。
  - `org.slf4j.slf4j-api(1.7.25)`：simple logging facade for java日志接口包。
  - `org.slf4j.slf4j-log4j12(1.7.25)`：slf4j和log4j的整合包，是一个实现。
- 开发数据源属性文件 `classpath:jdbc/mb-db.properties`：配置驱动，连接串，账号和密码。
- 开发日志文件 `classpath:log4j.properties`：名称和位置固定。
- 开发主配文件 `classpath:mybatis/mybatis-core.xml`：
  - `<properties resource=""/>`：从classpath下引入属性文件。
  - `<environments>/<environment>`：环境标签，支持配置多环境。
  - `<transactionManager type="JDBC">`：配置JDBC事务管理类，此事务支持提交和回滚。
  - `<dataSource type="POOLED">`：配置连接池，拥有 `driver/url/username/password` 等属性。
- tst: `MyBatisTest.testBuild()`：
  - `Resources.getResourceAsStream("主配路径")`：返回主配的字节流对象。
  - `new SqlSessionFactoryBuilder().build(主配字节流对象)`：创建session工厂。
  - `factory.openSession()/openSession(true)`：从session工厂中打开一个具有/没有事务保护的session。
  - `session.getConnection().isClosed()`：通过session获取一个连接并测试连接状态。
  - `session.close()`：关闭session，可使用 `try-with-resource` 结构。
**武技：** 封装MyBatisUtil工具类，使用双重检查锁来返回一个单例的session工厂：

- src: `util.MyBatisUtil`
- tst: `MyBatisUtilTest`

## Mapper配置

**心法：** Mapper配置文件用于配置SQL语句块以及SQL相关配置：

- 开发实体类ORM库表 `entity.Student`：
  - 数据库主键无符号使用Integer对应，有符号使用Long对应。
  - 实体类属性必须有setter/getter方法，建议有toString()/全参构造/无参构造。
- 在主配文件中使用 `<typeAliases>/<package name="">` 配置单个/整包别名，别名不区分大小写：
  - `资源/图片/mybatis内置别名.png`
- 开发Mapper接口 `mapper.StudentMapper`：相当于MVC结构中的DAO层：
  - 数据层接口的方法只负责SQL的发送和结果的回收。
- 开发Mapper配置 `mapper/StudentMapper.xml`：须与Mapper接口同名同包（classpath下都算同包）：
  - `<mapper namespace="">`：命名空间必须对应填写Mapper接口全名。
- 在主配文件中使用 `<mappers>/<package name="">` 整包扫描接口：
  - 整包扫描的前提：Mapper配置和Mapper接口必须同名同包。
- 取消IDEA中SQL语句黄绿色背景：`File -> Settings -> Editor`：
  - `Inspections -> SQL`：取勾 `No data sources configured/SQL dialect detection`
  - `Color Scheme -> General -> Code -> Injected language fragment`：取勾 `Background`

**心法：** maven项目在打包时仅会将 `src/main/java` 中的.java文件进行打包，若希望将该包下的.xml文件一并打包，则需要在pom.xml中的 `<build>` 中进行如下配置：

```xml
<!--设置maven打包时将*Mapper文件一并打包-->
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*Mapper.xml</include>
        </includes>
    </resource>
</resources>
```

### CRUD语句块

**心法：** CRUD语句块必须使用id/parameterType/resultType属性分别对应接口方法的方法名/参数类型/返回值或其泛型类型：

- 在Mapper接口中开发CRUD方法：添加/按主键修改/按主键删除/按主键查询/按姓名模糊查询：
  - 简单参数建议使用 `@Param()` 起别名，实体参数随意。
- 在Mapper配置中开发DML语句块：可省略parameterType，必须省略resultType：
  - `#{}` SQL占位符：用于自动获取实体参数中的指定属性，支持连调，并自动补充单引号：
    - 若入参为简单参数，占位符中使用@Param别名获取该参数，若无别名，则内容随意，但不能不写。
  - `useGeneratedKeys="true" keyProperty="id"` 用于配置添加语句块的主键回注功能。
- 在Mapper配置中开发DQL语句块：可省略parameterType，必须使用resultType：
  - `${}` SQL拼接符：用于自动获取实体参数中的指定属性，支持连调，但不补充单引号且存在注入漏洞：
    - 若入参为简单参数，拼接符中使用@Param别名获取该参数，若无别名，则必须写成 `${value}`。
  - `flushCache`：每次调用该语句块前是否清空一级缓存以保证结果最新，默认false。
  - `useCache`：是否使用二级缓存，默认true。
  - `timeout`：语句块超时时间，默认数值由驱动器决定。
  - `fetchSize`：结果集条目数达到此阈值时立刻返回一批结果，默认数值由驱动器决定。
  - `statementType`：设置SQL媒介类型，可选PREPARED/STATEMENT/CALLABLE，默认PREPARED。
- 配置SQL重用块：
  - `<sql id="">` 用来封装一个SQL重用代码段。
  - `<include refid="">` 可以在当前文件中任意位置引入某个SQL重用代码段。
- tst: `StudentMapperTest`：获取Mapper接口实例并调用对应方法：
  - 创建一个session工厂：可以使用封装好的MyBatisUtil工具类获取一个单例的session工厂。
  - 从session工厂中获取一个session会话：`factory.openSession()`
  - 通过session会话来获取指定的Mapper接口：`session.getMapper(StudentMapper.class)`
  - 调用接口方法并手动提交会话（查询操作除外）：`session.commit()`
  - 手动关闭session会话以释放资源：`session.close()`

**心法：** o 注：

```xml
<insert id="insert">
    <!--该语句块在主SQL之后执行-->
    <!--查询最后一条插入记录的主键并回注到参数实体的int类型的id属性中-->
    <selectKey order="AFTER" resultType="int" keyProperty="id">
        <!--`select uuid()`：可以返回一个随机字符串-->
        SELECT last_insert_id()
    </selectKey>

    insert into student (id, name, gender, age, info)
    values (#{id}, #{name}, #{gender}, #{age}, #{info})
</insert>
```

### DQL高级映射

**心法：** DQL语句块支持使用resultMap替代resultType，以指向一个独立的 `<resultMap>`：

- `<resultMap id="" type="">`：使用id唯一标识，使用type设置最终返回值类型。
- `<result column="" property="" javaType="" jdbcType=""/>`：配置普通字段：
  - column：映射数据库表的字段名。
  - jdbcType：映射数据库表字段的枚举类型，必须大写，可省略但不建议。
  - property：映射实体类的属性名。
  - javaType：映射实体类的属性类型，可省略。
- `<id>` 一般用于配置主键字段，无论什么情况下都建议配置以提高效率，全部属性和 `<result>` 一样。
- tst: `StudentMapperTest.list()`

### 二级缓存

**心法：** 二级缓存需要在主配 `<settings>` 中使用 `<setting name="cacheEnabled" value="true"/>` 开启：

- `<cache>`：用于在SQL配置文件中开启二级缓存，此时所有DQL都会被缓存，所有DML都会更新缓存：
  - `eviction`：设置缓存逐出机制，如默认的LRU（逐出最长时间不被使用的），FIFO（先进先出），SOFT（按GC规则逐出）等。
  - `flushInterval`：设置缓存刷新间隔的毫秒数，默认不设置，表示缓存仅仅调用语句时刷新。
  - `readOnly`：缓存默认对每个调用者仅返回一个拷贝，效率低但安全，只读缓存对每个调用者都返回缓存本体，效率高但不安全。
  - `size`：设置缓存大小，默认1024个，表示缓存最多存储1024个结果集或对象的引用。
  - `type`：设置第三方/自定义缓存类全名，以覆盖mybatis的默认缓存机制。
- `<cache-ref namespace="">`：用于引入其他Mapper配置文件的缓存配置。

### 注解SQL

**心法：** 接口开发时，可使用注解替代SQL语句块以简化代码：

- 开发实体类ORM库表 `entity.Teacher`。
- 开发Mapper接口 `mapper.TeacherMapper`：开发CRUD方法：
  - `@Select/@Insert/@Update/@Delete` 用于编写SQL。
  - `@Options` 用于配置缓存，超时时间，主键回注等功能。
  - `@Results/@Result` 用于配置SQL高级映射。
- 开发Mapper配置 `mapper/TeacherMapper`：命名空间仍要对应接口类全名，但不再写SQL语句块。
- 在主配中的 `<mappers>` 中整包扫描接口。
- tst: `TeacherMapperTest`

## 动态SQL

**心法：** 动态SQL指的是在SQL块中使用动态标签来完成选择判断，循环等操作：

- 注解中使用动态SQL时需要为在SQL语句两端加上 `<script></script>`，其中内容和XML版本一致。
- 学习动态SQL的准备工作：
  - src: `entity.User`
  - src: `mapper.UserMapper`
  - res: `mapper/UserMapper.xml`

### 动态条件标签

**心法：** 条件标签配合OGNL表达式用于判断是否允许拼接部分指定的SQL语句：

- `<where>`：WHERE语句不为空时，生成 `WHERE` 关键字并删除WHERE语句中第一个 `AND/OR`：
  - 等效于 `<trim prefix="WHERE" prefixOverrides="AND|OR">`
  - tst: `UserMapperTest.testSelectByWhere()`
- `<if test='OGNL表达式条件'>`：用于单独条件判断，仅在条件满足时拼接指定SQL：
  - tst: `UserMapperTest.testSelectByIf()`
- `<choose>`：用于多选一条件判断：
  - 子标签 `<when test='OGNL表达式条件'>`：仅在条件满足时拼接指定SQL。
  - 子标签 `<otherwise>`：所有条件均不满足时拼接指定SQL。
  - tst: `UserMapperTest.testSelectByChoose()`

### 动态循环标签

**心法：** `<foreach>` 用于循环拼接部分指定的SQL语句：

- `collection`：被遍历的元素，其值建议直接使用入参的@Param别名：
  - 若参数没有设置@Param别名，则数组入参固定使用 `array`，列表入参固定使用 `list`。
  - collection 的值可以使用一个实体类或Map的key，但前提是该key对应的值必须为数组或列表。
- `item`：设置循环中间变量，如 `e`，之后在循环内部可以配合使用 `${e}` 将其取出。
- `open/close`：设置循环开始/结束时拼接的字符串SQL代码。
- `separator`：设置拼接符，自动忽略SQL语句中多余的拼接符。
- tst: `UserMapperTest.testSelectByForeach()`

### 动态修改标签

**心法：** 部分字段的修改操作，建议使用动态SQL以增加灵活度：

- `<set>` 用于控制修改操作时自动生成 `SET` 关键字并删除最后一个逗号：
  - 等效于 `<trim prefix="SET" suffixOverrides=",">`
- tst: `UserMapperTest.testUpdateBySet()`

### 内置参数

**心法：** `_parameter` 是SQL语句块中的内置参数，可以在OGNL表达式中直接使用：

- OGNL表达式中：
  - `_parameter`：可以表示接口方法中的唯一入参，多参时表示参数集合。
  - `_parameter.get("param1")`：可以表示接口方法中第1个入参，其余类推。
  - `_parameter.get("a")`：可以表示接口方法中标记了 `@Param("a")` 的入参。
- SQL语句中：
  - `#{param1}`：可以获取第1个入参，其余类推。
  - `#{a}`：可以获取接口方法中标记了 `@Param("a")` 的入参。
- tst: `UserMapperTest.testSelectByParameter()`

## 多表联查

**心法：** mybatis多表间支持一对一和一对多两种关系：

- 开发员工和部门的实体类 `entity.Dept/Emp`：
  - 从员工看部门，是1对1关系，需要埋一个 `Dept dept` 属性。
  - 从部门看员工，是1对N关系，需要埋一个 `List<Emp> emps` 属性。
- 在主配 `<settings>` 中配置懒加载模式以解决一次主查询附带N次子查询的N+1问题：
  - `<setting name="lazyLoadingEnabled" value="true"/>`
  - `<setting name="aggressiveLazyLoading" value="false"/>`
  - 延迟加载需要mybatis版本在3.5.1以上。
- 准备员工和部门的Mapper接口和Mapper配置文件。

### 一对一级联

**武技：** 查询全部员工信息，级联查询其部门全部信息：

- 在员工Mapper接口中开发级联查询接口方法：
  - `List<Emp> selectWithDeptByJoin()`
- 在员工Mapper配置中开发级联查询语句块：
  - SQL语句中使用 `JOIN` 进行联表，注意消除联表结果中的重复的列。
  - 返回值使用 `<resultMap>` 对Emp表的全部字段进行映射，包括ORM同名字段。
  - 使用 `<association>` 配置一对一关联字段，其内部需要将关联表的全部字段也映射一遍。
- tst: `EmpMapperTest.testSelectWithDeptByJoin()`

### 一对一分步

**武技：** 查询全部员工信息，分步查询其部门全部信息：

- 在员工Mapper接口中开发分步查询接口方法：
  - `List<Emp> selectWithDeptBySelect()`
- 在员工Mapper配置中开发分步查询语句块：
  - 1步SQL块：全查Emp表，返回值使用 `<resultMap>` 对Emp表的主键和ORM异名字段进行映射。
  - 1步SQL块额外使用 `<association select="工作空间.SQL块id">` 来调用2步SQL块，自动传递关联字段。
  - 2步SQL块：按主键查询Dept表，可使用 `_parameter` 来直接表示1步SQL块的关联字段。
  - 2步SQL块建议在部门表的Mapper配置文件中开发，且无需配置对应的接口方法。
- tst: `EmpMapperTest.testSelectWithDeptBySelect()`：使用debug测试延迟加载效果。

### 一对一注解

**武技：** 查询全部员工信息，注解查询其部门全部信息：

- 在员工Mapper接口中开发注解查询接口方法：
  - `List<Emp> selectWithDeptByOne()`
  - `@Result(one=@One(""))`：可以替代 `<association select="">` 配置。
- tst: `EmpMapperTest.testSelectWithDeptByOne()`

### 一对多级联

**武技：** 查询全部部门信息，级联查询其部门下的所有员工信息：

- 在部门Mapper接口中开发级联查询接口方法：
  - `List<Dept> selectWithEmpsByJoin()`
- 在部门Mapper配置中开发级联查询语句块：
  - SQL语句中使用 `JOIN` 进行联表，注意消除联表结果中的重复的列。
  - 返回值使用 `<resultMap>` 对Dept表的全部字段进行映射，包括ORM同名字段。
  - 使用 `<collection>` 配置一对多关联字段，其内部需要将关联表的全部字段也映射一遍。
  - 使用 `<collection>` 的 `ofType` 属性替代 `javaType`属性，配置关联属性的泛型类型。
- tst: `DeptMapperTest.testSelectWithEmpsByJoin()`

### 一对多分步

**武技：** 查询全部部门信息，级联查询其部门下的所有员工信息：

- 在部门Mapper接口中开发分步查询接口方法：
  - `List<Dept> selectWithEmpsBySelect()`
- 在部门Mapper配置中开发分步查询语句块：
  - 1步SQL块：全查Dept表，返回值使用 `<resultMap>` 对Dept表的主键和ORM异名字段进行映射。
  - 1步SQL块额外使用 `<collection select="工作空间.SQL块id">` 来调用2步SQL块，自动传递关联字段。
  - 2步SQL块：按部门编号查询Emp表，可使用 `_parameter` 来直接表示1步SQL块的关联字段。
  - 2步SQL块建议在部门表的Mapper配置文件中开发，且无需配置对应的接口方法。
- tst: `DeptMapperTest.testSelectWithEmpsBySelect()`：使用debug测试延迟加载效果。

### 一对多注解

**武技：** 查询全部部门信息，级联查询其部门下的所有员工信息：

- 在部门Mapper接口中开发分步查询接口方法：
  - `List<Dept> selectWithEmpsByMany()`
  - `@Result(many=@Many(""))`：可以替代 `<collection select="">` 配置。
  - `@Result(javaType="List.class")`：可以替代 `<collection ofType="Emp">` 配置。
- tst: `DeptMapperTest.testSelectWithDeptByOne()`

# 第四阶：Spring

**心法：** [Spring](https://repo.spring.io/release/org/springframework/spring) 
是2003年兴起的一个轻量级的JAVA开源框架， 核心特性是控制反转和面向切面编程：

- spring支持XML配置：基本配置如数据源建议使用，可读性高，耦合度低。
- spring支持注解配置：开发相关如Service注入Dao等建议使用，开发效率更高。

**武技：** spring基本配通：

- 引入依赖：
  - `org.springframework.spring-context-support(4.3.14.RELEASE)`：spring核心包。
  - `junit.junit(4.12)(test)`：junit包。
  - `org.projectlombok.lombok(1.18.12)(provided)`：lombok包。
  - `log4j.log4j(1.2.17)`：log for java日志接口包。
  - `org.apache.logging.log4j.log4j-core(2.3)`：log for java日志接口包。
  - `org.slf4j.slf4j-api(1.7.25)`：simple logging facade for java日志接口包。
  - `org.slf4j.slf4j-log4j12(1.7.25)`：slf4j和log4j的整合包，是一个实现。
  
- 开发日志文件 `classpath:log4j.properties`：名称和位置固定。
- 开发实体类：`spring.entity.Bird`：
  - 埋id/name/age/gender/info属性并分别配置setter/getter方法。
- 开发主配文件：`classpath:spring/app-start.xml`：创建spring容器时需要加载1或N个主配：
  
  - 右键 `New -> Xml Configuration File -> Spring Config` 可以直接创建spring主配文件。
- 在主配中管理实体类：id/name可省略，三者均不建议重名：
  - `<bean id="唯一标识" name="别名" class="实体类全名">`
- tst: `spring.StartTest`：
  - `new ClassPathXmlApplicationContext("")`：创建spring容器，主配路径从classpath出发。
  - `new FileSystemXmlApplicationContext("")`：创建spring容器，主配路径从项目出发。
  - `context.getBean()`：通过id/name/类.class的方式获取spring容器中管理的某实例。
  - `context.close()`：销毁spring容器以释放资源。

## 控制反转IOC

**心法：** `Inversion of Control` 
控制反转：就是将实例化过程和实例生命周期管理都交给spring容器，
以解决手动new实例时，侵入性和耦合度双高的问题：

- 开发实体类：`spring.entity.Cat`：
  - 埋id/name/age/gender/info属性并分别配置setter/getter方法。

- bean的创建之无参构造方案：
  - 优点：速度最快，配置简单，是spring容器创建bean的默认方式。
  - 缺点：无法调用有参构造器。
  - 配置：`<bean class="实体类全名">`

- bean的创建之静态工厂方案：
  - 优点：速度第二快，spring容器不会帮我们new工厂，可以在工厂方法中自行选择调用有参或无参构造器。
  - 缺点：线程不安全。
  - 工厂: `spring.factory.CatFactory`：开发静态工厂方法 `getInstanceByStatic()`。
  - 配置：`<bean class="工厂类全名" factory-method="静态方法名"/>`

- bean的创建之实例工厂方案：
  - 优点：线程安全，可以在工厂方法中自行选择调用有参或无参构造器。
  - 缺点：速度最慢，spring容器需要帮我们new工厂。
  - 工厂: `spring.factory.CatFactory`：开发实例工厂方法 `getInstanceByDynamic()`。
  - 配置：`<bean id="工厂ID" class="工厂类全名"/>`
  - 配置：`<bean factory-bean="工厂ID" factory-method="实例方法名"/>`
  
- bean的加载机制：
  - `<bean class="实体类全名" lazy-init="false">`：该bean在spring容器初始化时创建，默认。
  - `<bean class="实体类全名" lazy-init="true">`：该bean在被 `getBean()` 时才会创建。
- bean的作用范围：
  - `<bean class="实体类全名" scope="singleton">`：每次返回的都是同一个实例，默认。
  - `<bean class="实体类全名" scope="prototype">`：每次都返回一个新new出来的实例，自动开启懒加载。
  
- bean的生命周期：
  - `<bean class="实体类全名" init-method="初始化方法名" destroy-method="销毁方法名"/>`
- tst: `spring.IocTest`

## 依赖注入DI

**心法：** `Depend Injection` 依赖注入是指在IOC产生实例的同时向实例的成员属性，构造器参数等进行注值的过程。

**武技：** spring可以在IOC的过程中向bean的成员属性或构造器参数注值：

- 开发实体类：`spring.entity.Dog`：
  - 埋name/age/gender/info/array/list/set/map/properties属性。
  - 埋一个Cat类型的属性并配置setter/getter方法。
  - 开发一个构造方法，仅涉及name和age两个属性即可。
  - 依赖注入的本质是寻找属性对应的setter方法，故必须配置每个属性的setter/getter方法。
- 开发主配文件 `spring/app-di.xml`：管理实体类并在 `<bean>` 内部向成员属性和构造器参数注值：
  - `<constructor-arg index="构造器形参角标" value="参数值" type="参数类型全名">`
  - `<constructor-arg>` 中仍可以使用 `<array>/<list>/<set>` 等子标签。
  - `<property name="" value=""/>`：注入如String，Integer等简单类型属性值。
  - `<property name=""> + <array>/<list>/<set> + <value>`：注入数组类/List/Set类型属性值。
  - `<property name=""> + <map> + <entry>`：注入Map类型属性值。
  - `<property name=""> + <props> + <prop>`：注入Properties类型属性值。
  - `<property ref="Cat的bean的id">`：外部注入另一个实体类：
    - 若需内部注入，直接在 `<property>` 内部包裹Cat的 `<bean>` 即可。
- tst: `spring.DiTest`

### @Autowired注解

**心法：** @Autowired用于自动进行依赖注入操作，可以设置在属性，属性的setter方法或构造器上：

- @Autowired注入流程：Spring容器初始化时：
  - 使用@Autowired所在属性的类全名到spring容器中匹配全部bean的class值。
  - 若匹配成功直接实例化该bean并注入到属性，注意匹配到其实现类也算匹配成功。
  - 若匹配失败再使用@Autowired所在属性的变量名到spring容器中匹配全部bean的id值。
  - 若匹配成功直接实例化该bean并注入到属性，若仍匹配失败，则直接抛出异常。
- @Qualifier精准注入：
  - @Qualifier注解可以配合@Autowired来精准指定id值，此时优先匹配id，id匹配失败，直接抛异常。

**武技：** 测试@Autowired的自动注入特性：

- 开发实体类：`spring.entity.Fox`：
  - 埋id/name/age/gender/info属性并分别配置setter/getter方法。
- 开发数据接口 `spring.autowired.mapper.FoxMapper`：
  - 开发数据方法。
- 开发数据类 `spring.autowired.mapper.impl.FoxMapperImpl`：
  - 因为未整合mybatis框架，所以这里暂时先手写Mapper接口实现类。
  - 实现数据接口并重写数据方法。
- 开发业务接口 `spring.autowired.service.FoxService`：
  - 开发业务方法。
- 开发业务类 `spring.autowired.service.impl.FoxServiceImpl`：
  - 使用 `@Autowired` 注入数据类。
  - 实现数据接口并重写业务方法：调用数据方法。
- 开发控制类 `spring.autowired.controller.FoxController`：
  - 使用 `@Autowired` 注入业务类。
  - 开发控制方法：调用业务方法。
- 在主配中开启@Autowired注解驱动：
  - `<context:annotation-config />`
- 在主配中管理数据类，业务实现类和控制类，注意不要配置成接口全名：
  - `<bean class="数据类全名"/>`
  - `<bean class="业务类全名"/>`
  - `<bean class="控制类全名"/>`
- tst: `spring.AutowiredTest`：从spring容器中获取控制类实例并调用控制方法。

### @Component注解

**心法：** @Component用于快速配置其所在类的 `<bean>`，拥有三个子注解，
均支持使用value属性绑定bean的id值，默认为 该类名的首字母小写单词：

- `@Repository`：一般用于标记数据类，如 `BatDao` 或 `BatMappr`，注意不要标记接口。
- `@Service`：一般用于标记业务类，如 `BatServiceImpl`，注意不要标记接口。
- `@Controller`：一般用于标记控制类，如 `BatController`。
- `@Component`：一般用于标记不属于以上三种的其他类。

**武技：** 使用@Component的三个子注解替换所有 `<bean>`：

- 开发实体类：`spring.entity.Bat`：
  - 埋id/name/age/gender/info属性并分别配置setter/getter方法。
- 开发数据接口 `spring.component.mapper.BatMapper`：
  - 开发数据方法。
- 开发数据类 `spring.component.mapper.impl.BatMapperImpl`：
  - 因为未整合mybatis框架，所以这里暂时先手写Mapper接口实现类。
  - 标记 `@Repository` 加入spring管理。
  - 实现数据接口并重写数据方法。
- 开发业务接口 `spring.component.service.BatService`：
  - 开发业务方法。
- 开发业务类 `spring.component.service.impl.BatServiceImpl`：
  - 标记 `@Service` 加入spring管理。
  - 使用 `@Autowired` 注入数据类。
  - 实现业务接口并重写业务方法：调用数据方法。
- 开发控制类 `spring.component.controller.BatController`：
  - 标记 `@Controller` 加入spring管理。
  - 使用 `@Autowired` 注入业务类。
  - 开发控制方法：调用业务方法。
- 在主配中扫描指定包及其子包中的所有@Component及其子注解（包扫描自带注解驱动）：
  - `<context:component-scan base-package="包路径"/>`
- tst: `spring.ComponentTest`

### @Configuration注解

**心法：** @Configuration用来标记一个java类为spring配置类，以替代XML版本的spring配置：

- java配置可以使用 `@Bean` 标记的方法来替代XML配置中的 `<bean>`：
  - 方法名对应 `<bean>` 的 `id` 值。
  - 方法返回值类型全名对应 `<bean>` 的 `class` 值。
  - 方法的返回值就是该bean的最终实例。
- java配置可以使用 `@ComponentScan("")` 来替代XML配置中的包扫描。

**武技：** 使用纯java代码开发spring容器配置，并管理一个数据源的bean：

- 添加依赖：
  - `com.jolbox.bonecp-spring(0.8.0.RELEASE)`：第三方bonecp连接池包。
  - `mysql.mysql-connector-java(8.0.15)(runtime)`：mysql数据库驱动包。
- 开发数据源属性文件 `classpath:jdbc/mb-db.properties`。
- 开发配置类 `spring.config.DataSourceConfig`
  - 类上标记 `@Configuration` 表示该类是一个配置类，即java编写的鱼缸。
  - 类上标记 `@PropertySource("classpath:")` 以读取数据源属性文件。
  - 开发属性并标记 `@Value("${xxx}")` 以读取数据源属性文件中的属性值。
- 开发配置方法：`public DataSource dataSource() {}`
  - 方法标记 `@Bean` 表示这个方法相当于一个 `<bean>` 配置。
  - 方法体中创建一个 `BoneCPDataSource` 实例，并手动注入相关属性。
  - 方法返回 `BoneCPDataSource` 实例，或其接口 `DataSource`。
- tst: `spring.DataSourceTest`:
  - `new AnnotationConfigApplicationContext(xxx.class)`：实例化纯java开发的spring容器。
  - `context.getBean(DataSource.class)`：获取DataSource并测试是否可以成功连接到数据库。

## 动态代理Proxy

**武技：**

- 开发客户接口 `spring.proxy.UserService` 并开发CRUD四个接口方法。
- 开发客户实现类 `spring.proxy.UserServiceImpl` 并重写四个接口方法。

### JDK动态代理

**武技：** jdk动态代理的客户（聘请代理的对象）必须是某接口的实现类：

- 开发jdk代理公司类 `spring.proxy.JdkProxyCompany`：
  - 重写 `j.l.r.InvocationHandler.invoke()` 工作清单。
- 开发聘用代理方法 `hireProxy()`：
  - 调用 `Proxy.newProxyInstance(客户的类加载器, 客户的接口们, 任务清单所在类)` 以得到一个代理对象。
- 设计工作清单内容：
  - 模拟鉴权，反射执行业务方法，模拟打印日志。
- tst：`spring.ProxyTest.testJdkProxy()`
  - 创建符和条件的客户，实例类型可以是实现类，也可以是接口。
  - 创建JDK动态代理公司，并调用聘用代理的方法，将客户入参，返回一个对应该客户的代理。
  - 客户和其代理的类型是一样的，需要强转。
  - 使用代理来调用业务方法，查看是否执行了任务清单。

### CGLIB动态代理

**武技：** cglib动态代理的客户（聘请代理的对象）必须是某类的子类：

- 开发cglib代理公司类 `spring.proxy.CglibProxyCompany`：
  - 重写 `o.s.c.p.MethodInterceptor.intercept()` 工作清单。
- 开发聘用代理方法：
  - `new Enhancer().create()`：得到一个代理对象。
  - `enhancer.setSuperclass(customer.getClass())`：设置客户的父类。
  - `enhancer.setCallback(this)`：设置任务清单所在类。
- 设计工作清单内容：
  - 模拟鉴权，反射执行业务方法，模拟打印日志。
- tst：`spring.ProxyTest.testCglibProxy()`
  - 创建符和条件的客户，实例类型可以是实现类，也可以是接口。
  - 创建JDK动态代理公司，并调用聘用代理的方法，将客户入参，返回一个对应该客户的代理。
  - 客户和其代理的类型是一样的，需要强转。
  - 使用代理来调用业务方法，查看是否执行了任务清单。

## 面向切面AOP

**心法：** Aspect Oriented Programming，面向切面编程，是OOP的延续，它通过预编译的方式在运行期，
使用动态代理来实现业务代码和切面代码的逻辑分离，降低耦合度，从而提高程序的重用性，主要作用于控制事务的传播。

- 添加依赖：`org.springframework.spring-aspects(4.3.14.RELEASE)`
- 开发客户类 `spring.aspect.UserServiceImpl`：
  - 标记 `@Component` 以被spring容器扫描和管理。
  - 开发业务方法 `int[] selectById(int id)`

**武技：** 使用注解配置AOP：

- 开发切面类：`spring.aspect.UserAspectByAnn`：
  - 标记 `@Aspect` 表示这是一个切面类。
  - 标记 `@Component` 以被spring容器扫描和管理。
- 开发切点方法：方法名随意仅用于区分，但方法体必须为空：
  - 标记 `@Pointcut("execution([修饰符] 返回类型 [包名].[类名].方法名(形参) [异常])")` 指定切点范围。
  - PE表达式中，`*` 表示占1位，`..` 表示任意占位。
- 在切面类中开发环绕通知方法：
  - 标记 `@Around("切点方法名()")` 指向切点方法名。
  - 方法参数必须是 `ProceedingJoinPoint`：
    - `pjp.getArgs()`：获取业务方法的参数列表。
    - `pjp.proceed()`：通过反射执行业务方法，其返回值就是业务方法的原返回值。
  - 方法返回必须是 `Object` 类型。
- 开发主配 `spring/app-aop.xml`：
  - `<context:component-scan base-package=""/>`：包扫描 `@Component`。
  - `<aop:aspectj-autoproxy />`：驱动 `@Aspect`。
- tst: `spring.AopTest`

**武技：** 使用XML配置AOP：

- 开发切面类：`spring.aspect.UserAspectByXml`：
  - 不标记 `@Aspect`，不开发切点方法，不为环绕通知方法标记 `@Around`，其余同上。
- 在主配中使用 `<context:component-scan base-package=""/>` 包扫描 `@Component`。
- 在主配中配置AOP区域：
  - `<aop:config ></aop:config>`
- 在主配AOP区域中配置切点和切面类：
  - `<aop:pointcut id="" expression="execution(PE表达式)"/>`
  - `<aop:aspect ref="切面类的<bean>的id">`
- 在切面类标签中配置通知方法：
  - `<aop:around method="通知方法名" pointcut-ref="切点id（不带括号）"/>`
- tst: `spring.AopTest.testAop()`

## 整合Junit4

**武技：** 将junit4.12+整合到spring中，可以在单元测试时初始化容器：

- 添加依赖：
  - `org.springframework.spring-test(4.3.14.RELEASE)`
- 开发测试类 `spring.SpringJunit4Test`
  - 标记 `@RunWith(SpringJUnit4ClassRunner.class)` 以指定使用整合方案。
  - 标记 `@ContextConfiguration("classpath:")` 以指定XML配置文件路径：
    - 纯java配置使用 `classes=XXXConfig.class` 即可。
- 在测试类直接使用 `@Autowired`：
  - 此时建议标记在成员属性上，而不要标记在构造器上。

## 整合MyBatis

**武技：** 将mybatis与spring整合练习小猪表的CRUD：

- 开发数据库：创建数据库，用户，和数据库表：
  - `资源/SQL/sm.sql`
- 添加依赖：
  - `org.mybatis.mybatis(3.5.1)`：mybatis核心包。
  - `mysql.mysql-connector-java(8.0.15)(runtime)`：mysql驱动包。
  - `com.jolbox.bonecp-spring(0.8.0.RELEASE)`：bonecp连接池包。
  - `org.apache.logging.log4j(2.3)`：日志包。
  - `log4j.log4j(1.2.17)`：日志包。
  - `org.slf4j.slf4j-api(1.7.25)`：日志包。
  - `org.slf4j.slf4j-log4j12(1.7.25)(test)`：日志包。
  - `org.projectlombok.lombok(1.18.12)(provided)`：lombok工具包。
  - `junit.junit(4.12)(test)`：junit测试包。
  - `org.springframework.spring-context-support(4.3.14.RELEASE)`：spring核心包。
  - `org.springframework.spring-test(4.3.14.RELEASE)`：spring测试包。
  - `org.springframework.spring-aspects(4.3.14.RELEASE)`：spring切面包。
  - `org.springframework.spring-jdbc(4.3.14.RELEASE)`：spring事务包。
  - `org.mybatis.mybatis-spring(1.3.0)`：SM整合包。
  - `com.github.pagehelper.pagehelper(5.2.0)`：PageHelper分页工具。
- 开发数据源属性文件：
  - res: `jdbc/sm-db.properties`：配置驱动，连接串，账号和密码。
- 开发日志文件 `log4j.properties`：名称固定，位置固定在classpath下。
- 开发ORM小猪实体类 `sm.entity.Pig`：
  - 埋 `id/name/age/gender/info` 属性并配置setter/getter等。
- 开发Mapper接口 `sm.mapper.PigMapper`：
  - `void insert()`：添加方法。
  - `void updateById(Pig pig)`：按主键修改方法。
  - `void deleteById(@Param("id") Integer id)`：按主键删除方法。
  - `Pig selectById(@Param("id") Integer id)`：按主键查询方法。
  - `List<Pig> list()`：全查方法。
  - `List<Pig> paging()`：分页查询方法。
- 开发Mapper配置 `sm/mapper/PigMapper.xml`：
  - 建议和Mapper接口同名同包，可以省略mapperLocations的配置。
  - 对应编写CRUD语句块，其中分页SQL语句块的内容为全查。
- 开发spring主配文件 `spring/app-sm.xml`：
  - 使用 `<context:property-placeholder location=""/>` 引入属性文件。
  - 管理Mapper接口扫描类 `o.m.s.m.MapperScannerConfigurer` 并注入 `basePackage` 以配置接口包。
  - 管理BoneCP连接池类 `c.j.b.BoneCPDataSource` 并注入相关数据源信息。
  - 管理会话工厂 `o.m.s.SqlSessionFactoryBean`：
  - 对会话工厂注入 `typeAliasesPackage` 以整包设置别名。
  - 对会话工厂注入 `dataSource` 以引入数据源。
  - 对会话工厂注入 `mapperLocations` 以配置Mapper配置文件，同名同包时可省略此项配置。
  - 对会话工厂注入 `plugins` 以PageHelper插件并进行配置，具体配置如下。
- tst: `sm.PigMapperTest`：使用spring-test测试CRUD和分页方法：
  - `PageHelper.startPage(2, 5)`：查询第2页，每页5条，仅对紧随其后的第一个查询生效。
  - `new PageInfo<>(全查数据)`：可以将全查结果封装到一个PageInfo实例中。
  - `pageInfo.getList()`：从PageInfo中可获取数据信息。
  - `pageInfo.getPageNum()`：从PageInfo中可获取分页信息。

**配置：** PageHelper在会话工厂中的配置：

```
<property name="plugins">
    <array>
        <bean class="com.github.pagehelper.PageInterceptor">
            <property name="properties">
                <props>
                    <!--设置数据库方言，默认自动检测-->
                    <prop key="helperDialect">mysql</prop>
                    <!--当pageSize设置为0时执行全查，默认false-->
                    <prop key="pageSizeZero">true</prop>
                    <!--当pageNum小于0或大于总页数时查询首页或尾页，默认false-->
                    <prop key="reasonable">true</prop>
                </props>
            </property>
        </bean>
    </set>
</property>
```
