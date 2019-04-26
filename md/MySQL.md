## MySQL 常用命令

### 启动、重启和关闭

```bash
service mysql start		---启动mysql
service mysql stop		---关闭mysql
service mysql restart		---重启mysql
```



### 连接 MySQL

```bash
mysql -h <主机地址> -u <用户名> -p <密码>
mysql -h<主机地址> -u<用户名> -p<密码>			---可以省略空格
```



### 修改密码

```bash
# 未登录mysql
mysqladmin -u<用户名> -p password <新密码>
# 之后输入原来的密码就更改好了

# 登录mysql
use mysql；
update user set password = password("新密码") where user = 'root';
flush privileges;
```



### 对数据库进行操作

```mysql
# 创建数据库
create database <数据库名>;

# 创建数据库并设置字符编码
create database <数据库名> character set <字符编码>;

# 显示所有的数据库
show databases;

# 显示数据库的定义信息
show create database <数据库名>;

# 修改数据库字符编码
alter database <数据库名> character set <字符编码>;

# 删除数据库
drop database <数据库名>;

# 查看当前使用的数据库
select database();

# 切换数据库
use <数据库名>;
```



### 对表进行操作

```mysql
# 创建表
CREATE TABLE `user` (
	`id` INT (11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR (255) DEFAULT NULL,
	`age` INT (11) DEFAULT NULL,
	PRIMARY KEY (`id`)
) ENGINE = INNODB DEFAULT CHARSET = utf8;

# 命令解释如下 

CREATE TABLE <表名> (
	<字段名> <字段类型>(长度) <约束条件>,
	...
	...
	PRIMARY KEY (字段名)	---设置主键
) ENGINE = <存储引擎> DEFAULT CHARSET = <字符编码>;
```



```mysql
# 查看数据库中所有的表
show tables;

# 查看系统支持的存储引擎
show engines；

# 查看表的字段信息
desc <表名>;

# 显示表的定义信息
show create table <表名>;

# 添加字段和约束
alter table <表名> add <字段名> <数据类型>(长度) <约束条件>;

# 修改字段和约束
alter table <表名> modify <字段名> <数据类型>(长度) <约束条件>;

# 修改表的存储引擎
alter table <表名> engine = <存储引擎>;

# 修改表名
rename table <表名> to <新表名>;

# 修改表的字符编码
alter table <表名> character set <字符编码>;

# 修改字段名
alter table <表名> change <字段名> <新字段名> <数据类型>(长度);

# 删除字段
alter table <表名> drop <字段名>;

# 删除表
drop table <表名>;

# 删除表数据，保留表结构
truncate table <表名>;
```



### 字段数据类型

* int：整型
* double：浮点型
  * 如 double（5，2）表示最多有 5 位，其中必须有 2 位小数

* char：**固定长度字符串类型**
  * 如 char（10）如果不足 10 位则会自动补足 10 位

* varchar：**可变长度字符串类型**
  * 如 varchar（10）如果不足10位不会补足，**性能不如 char 高**

* text：字符串类型
  * 适用于大文本内容

* date：日期类型
  * 格式为 `yyyy-MM-dd`

* time：时间类型
  * 格式为 `hh:mm:ss`

* timestamp：时间戳类型
  * 格式为 `yyyy-MM-dd hh:mm:ss`，**会自动赋值**

* datetime：日期时间类型
  * 格式为 `yyyy-MM-dd hh:mm:ss`

   

### 对字段进行操作

#### 增删改

```mysql
# 插入数据
# 可以使用null插入空值，日期和字符要使用单引号
insert into <表名> (字段名,...) values (值,...);
insert into <表名> values (值,...);

# 删除数据
# 没有where条件会删除所有的数据
delete from <表名> where <字段名> = <值>;

# 修改数据
# 没有where条件会修改所有的数据
update <表名> set <字段名> = <值>,... where <字段名> = <值>;
```



#### 查

```mysql
# 查询数据
# 没有where条件会查找所有的数据
# 不建议使用*，因为会先将其编译成字段，然后再去查询，会影响一些性能，而且也不明确
select <字段名> from <表名> where <字段名> = <值>;

# 在mysql中<>表示不等于，!=也表示不等于，两者的含义和使用方式是一样的
select <字段名> from <表名> where <字段名> != <值>;
select <字段名> from <表名> where <字段名> <> <值>;

# between...and...，按范围查找
select <字段名> from <表名> where id between 1 and 3;
select <字段名> from <表名> where id not between 1 and 3;
# 等价于
select <字段名> from <表名> where id >= 1 and id <= 3;
select <字段名> from <表名> where id < 1 or id > 3;

# and的优先级大于or
# 如要查询name为user，id为1或2
select <字段名> from <表名> where name = 'user' and id = 1 or id = 2;
# 错误写法为
select <字段名> from <表名> where (name = 'user' and id = 1) or id = 2;
# 正确写法为
select <字段名> from <表名> where name = 'user' and (id = 1 or id = 2);


# in，表示只要满足其中一项条件即可，也可以采用or来表示，采用in会更简洁一些
select <字段名> from <表名> where id in (1,2);
# not in，则表示不满足其中任何一项，并且结果集不能有null，否则没有返回结果
select <字段名> from <表名> where id not in (1,2);

# like，模糊查询
# _表示匹配一个任意字符，%匹配任意个任意字符
select <字段名> from <表名> where name like '_end';
select <字段名> from <表名> where name like '%end';

# is null，查询为空的数据
select <字段名> from <表名> where name is null;
# is not null，查询不为空的数据
select <字段名> from <表名> where name is not null;

# order by，排序，可排序多个字段
# 默认为升序排列asc，降序排列使用desc
select <字段名> from <表名> order by <字段名> desc;
# 如存在where子句，order by必须放到where语句后面
select <字段名> from <表名> where id = 1 order by <字段名> desc;
```



#### 处理函数

```mysql
# 转换为大写
select lower(字段名) from <表名>;

# 转换为小写
select upper(字段名) from <表名>;

# 截取子串，下标从1开始
select substr(字段名，起始下标，截取长度) from <表名>;

# 获取字段中值的长度
select length(字段名) from <表名>;

# 空值处理
# 有null参与的运算结果都为null，建议先使用ifnull函数处理，将null替换为0或别的值
select ifnull(字段名，替换值) from <表名>;

# 去除首尾空格，MySQL默认去除字段后面的空格
select trim(字段名) from <表名>;

# 四舍五入
select round(字段名，保留的小数位数) from <表名>;

# 生成随机数
select rand();
# 生成0~100随机数
select round(rand()*100) 字段名 from <表名>;

# case when语句，计算条件列表并返回多个可能结果
case <字段名>
when <条件> then <处理>
else <其他>
end
# 如给名为user1薪水加100，user2薪水减200，其他人薪水加50
select id,name,(
	case name
	when 'user1' then sal + 100
	when 'user2' then sal - 200
	else sal + 50
) as sal
from test;

# 格式化日期，转换为字符串
select date_format(日期类型数据,日期格式)；

# 字符串转日期
select str_to_date(日期字符串,日期格式);

# 日期格式
%Y，四位的年份
%y，两位的年份
%m，月份，01,02，...
%c，月份，1,2，...
%d，日
%H，24小时制
%h，12小时制
%i，分
%s，秒
```



#### 聚合函数

聚合函数在计算时会 **自动忽略空值，不能直接写在where语句的后面。聚合函数可以一起使用**

```mysql
# 求和
select sum(字段名) from <表名>;

# 取平均值
select avg(字段名) from <表名>;

# 取最大值，日期也可以进行比较
select max(字段名) from <表名>;

# 取最小值
select min(字段名) from <表名>;

# 计算数据总数，不会统计数据为null的记录
select count(字段名) from <表名>;
```



#### 去重

```mysql
# 去除重复记录，将查询结果中某一字段的重复记录去除掉
# 只能出现在所有字段最前面，后面如果有多个字段即为多字段联合去重
select destinct 字段名 from <表名>;
```



#### 分组

```mysql
# 在有group by的语句中，select语句后面只能跟聚合函数和参与分组的字段
# order by语句只能放在group by语句后面
# 如果想对分组的数据进行过滤，需要使用having子句。
# 能够在where后过滤的数据不要放到having中进行过滤，否则影响SQL询句的执行效率
select <字段名> from <表名> group by id having id != 1;
```

**where和having区别**

* where 和 having 都是为了完成数据的过滤，它们后面都是添加条件

* where 是在 group by 之前完成过滤
* having 是在 group by 之后完成过滤



#### 返回的记录的数目

```mysql
# limit子句可以获取前几条或中间某几行数据，下标从0开始
# 主要用来分页处理，limit关键字只在MySQL中起作用
select <字段名> from <表名> limit <起始下标>,<截取长度>;
# 如果只给定一个参数，表示返回最大的记录行数目
select <字段名> from <表名> limit <截取长度>;
```



#### select 语句执行顺序

1. from：将硬盘上的表文件加载到内存
2. where：将符合条件的数据筛选出来，生成一张新的临时表
3. group by：根据列中的数据种类，将当前临时表划分成若干个新的临时表
4. having：可以过滤掉 group by 生成的不符合条件的临时表
5. select：对当前临时表进行整列读取
6. order by：对 select 生成的临时表，进行重新排序，生成新的临时表
7. limit：对最终生成的临时表的数据行，进行截取



### 连接查询

在实际开发中，数据通常是存储在多张表中，这些表与表之间存在着关系，在检索数据的时候需要多张表联合起来检索，这种多表联合检索被称为连接查询。如果多表进行连接查询时没有任何条件，最终的结果会是多表结果数量的乘积

```mysql
# 内连接，inner可省略
select a.name,b.name from atable a inner join btable b on a.id = b.id

# 右连接，outer可省略
select a.name,b.name from atable a right outer join btable b on a.id = b.id

# 左连接
select a.name,b.name from atable a left join btable b on a.id = b.id

# 也可以直接使用where语句进行查询
select a.name,b.name from atable a,btable b where a.id = b.id
```

**内连接与外连接的区别**

* 内连接：指连接结果仅包含符合连接条件的行，参与连接的两个表都应该符合连接条件
* 外连接：连接结果不仅包含符合连接条件的行同时也包含自身不符合条件的行。包括左外连接、右外连接和全外连接（MySQL不支持）
  * 左外连接：左边表数据行全部保留，右边表保留符合连接条件的行
  * 右外连接：右边表数据行全部保留，左边表保留符合连接条件的行



### 子查询

select 语句嵌套 select 语句被称为子查询，select 子句可出现在 select（使用较少）、from、where 关键字后面

```mysql
# 以下命令只做参考，没有实际用途
# select后
select
  id,
  (
	select name
 	from test
 	where id = 1
  ) name
from test

# from后
select
  a.name,b.name
from
  test a,
  (select name from test) b

# where后
select
  id,name
from
  test
where
  id in (select id from bank)
```



### 合并结果集

```mysql
# UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

# union，将查询的结果集合并
# 合并结果集时查询字段的个数必须一致，多个select语句会删除重复的数据
select id from test
union
select name from test
```



### 循环

#### while...do...end while循环

```mysql
create procedure ww()
begin
declare i int;
set i = 0;
while i < 10 do
	insert into test values(i);
	set i = i + 1;
end while;
end;

call ww();
```



#### repeat...until...end repeat

```mysql
create procedure pp()
begin
declare i int;
set i = 0;
repeat
	insert into test values(i);
	set i = i + 1;
	until i < 10;
end repeat;
end;

call pp();
```



#### loop...end loop

```mysql
create procedure ll()
begin
declare i int;
set i = 0;
loop_lable:loop
	insert into test values(i);
	set i = i + 1;
	if i >= 10 then
	leave loop_lable;
	end if;
end loop;
end;

call ll();
```

