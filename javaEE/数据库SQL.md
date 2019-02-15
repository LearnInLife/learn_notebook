# 数据库SQL

[TOC]

## 数据库概述

* 数据库就是存储数据的仓库，其本质是一个文件系统，数据按照特定的格式将数据存储起来，用户可以通过SQL对数据库中的数据进行增删改查的操作
* 数据库管理系统（DBMS）：操作和管理数据库的大型软件。
* 常见数据库：
  * mysql：开源免费，小型的数据库，被oracle收购后，6.x版本也开始收费
  * oracle：收费的大型数据库
  * db2：IBM公司的产品，收费，常应用在银行系统中
  * SqlServer：微软公司的中型数据库，收费，C#、.net等语言常使用
  * SyBase：已经退出历史舞台，提供了专业数据建模的工具PowerDesigner
  * SQLite：嵌入式小型数据库，应用在手机端



## SQL语句

* 什么是SQL语句
  * 一种结构化查询语言，关系数据库语言的国际标准
  * 各个数据库厂商都支持ISO的SQL标准，相当于官方普通话
  * 各个厂商在标准基础上做了自己的扩展，相当于方言
* SQL的分类
  * 数据定义语言：DDL，用来定义数据库对象：数据库 database、表 table、列 column等。关键字（操作的是结构）：创建：create、修改：alter、删除：drop等
  * 数据操作语言：DML，用来对数据库中表的记录进行增删改。关键字（操作的是数据）：插入：insert、删除：delete、更新：update等
  * 数据查询语言：DQL，用来查询数据库中表中记录。关键字：select、from、where等



## DDL

### 对数据库的操作

#### 数据库创建

```mysql
  create database 库名;
  create database 库名 character set 字符集;
```



#### 数据库的删除

```mysql
drop database 数据库名
```



#### 修改数据库

```mysql
alter database 数据库 character set 字符集
```



#### 查看数据库

```mysql
#查看所有的数据库
show databases;
#查看数据库定义的语句
show create database 数据库名;
```



#### 数据库其它命令

```mysql
#切换数据库
use 数据库名;

#查看当前正在使用的数据库
select database();
```



### 对表的操作

#### 表的创建

```mysql
create table 表名(
    列名 列的类型 列的约束,
    列名 列的类型 列的约束
);

#列的类型
java			sql
int				int
char/string		 char/varchar
				char:固定长度
				varchar:可变长度
double			 double
float			 float
boolean			 boolean
date			 date:  YYYY-MM-DD
				 time: hh:mm:ss
				 datetime: YYYY-MM-DD hh:mm:ss 默认为null
				 timestamp: YYYY-MM-DD hh:mm:ss 默认使用当前时间
				 
				 text:用来存放文本
				 blob:存放的是二进制

#列的约束
	primary key 主键约束 要求被修饰的字段：唯一 和 非空
	unique 唯一约束 
	not null 非空约束
	
#自动增长
	auto_increment
```



#### 删除表

```sql
drop table 表名
```



#### 修改表

``` mysql
alter table 表名 (add,modify,change,drop)

#添加一列
alter table 表名 add 字段名 类型(长度) [约束];

#修改列的类型（长度、约束）
alter table 表名 modify 修改的字段名 类型(长度) [约束];

#修改列的列名
alter table 表名 change 旧列名 新列名 类型(长度) [约束];

#删除列
alter table 表名 drop 列名;

#修改表名
rename table 表名 to 新表名;
```



#### 查看表

```mysql
#查询所有的表
show tables;
#表的创建语句,表的定义
show create table 表名;
#表的结构
desc 表名; 
```



#### 修改表的字符集

```mysql
  alter table 表名 character set 编码;
```



## DML

### 对数据的操作

#### 插入数据

 ```mysql
  #表中某些列插入数据
  insert into 表名(列名1,列名2,列名3……) values(值1,值2,值3……)
  #插入所有列
  insert into 表名 values(值1,值2,值3……)
  #批量插入
  insert into 表名(列名1,列名2,列名3) values(值1,值2,值3),(值1,值2,值3),(值1,值2,值3);
 ```

  * 注意：

    * 列的顺序与插入的值的顺序一致
    * 值如果是字符串或者日期需要加引号''(一般是单引号)
    * 批量插入的效率比单条插入高

  * 插入数据中文乱码问题解决：

    * 直接修改数据库安装目录里面的my.ini文件的第57行

      ![1549961831109](assets/1549961831109.png)

    * set names gbk;



#### 删除数据

```sql
delete from 表名 [where条件]
```

* delete 和 truncate删除数据 有什么差别？

  * delete:属于DML命令，一条一条删除表中的数据，它配合事务，可以将删除的数据找回。

  * truncate:属于DDL命令，它是将整个表摧毁，然后再创建一张一模一样的表。它删除的数据无法找回。

  * delete删除，不请空auto_increment记录数；而使用truncate操作，auto_increment将置为零，从新开始。因为truncate删除了表结构

    

* delete 和 truncate两者执行效率

  * 如果数据比较少,delete比较高效
  * 如果数据比较多,truncate比较高效



#### 更新数据

```mysql
update 表名 set 列名1=列的值,列名2=列的值2 [where 条件]
```



## DQL

### 单表查询数据

```mysql
select [distinct] [*] [列名1,列名2] from 表名 [where 条件]
#distinct: 去除重复的数据
#select: 显示所选列的内容
```



#### 简单查询

```mysql
#查询所有的商品
select * from product;

#查询商品名称和商品价格
select pname,price from product;

#别名查询 as 关键字,可以省略
  #表别名: 主要用于多表查询
  select p.pname,p.price from product as p;
  
  #列别名
  select pname as 商品名称,price as 商品价格 from product;
  #省略 as
  select pname 商品名称,price 商品价格 from product;
  
#去掉重复的值
#查询商品所有的价格
select distinct price from product;
 
#select运算查询 : 仅仅在查询结果上做运算 + * - /
select *,price*0.9 as 折后价 from product;
```



#### 条件查询

关键字：**where**

* 逻辑运算符
  * and ：多个条件同时成立
  * or ： 多个条件任一成立
  * not ： 不成立 。比如：工资不大于100  where not (salary>100);

* 比较运算符：
  * **>， >=， <， <=， =， !=， <>**
    * <>:不等于 标准SQL语法
    * !=:不等于 非标准SQL语法
  *  between...and...  ：在某一区间的值（包含头和尾）
  * in(值1，值2，....) ：在in列表中的值
  * like ：模糊查询
    * _ : 代表的是一个字符
    * % : 代表的零个或多个任意字符
  * is null ：判断是否为空

```mysql
#查询商品价格10到100之间
select * from product where price > 10 and price < 100;
    
#between...and...
select * from product where price between 10 and 100;

#查询价格等于 10 20 99这三个中的所有商品
select * from product where price in (10,20,99);
#等价于
select * from product where price=10 or price=20 or price=99;

#查询商品名中包含 里 的商品
select * from product where pname like '%里%'
       

```



#### 排序查询

关键字：**order by**

* asc : 升序(默认的排序方式)
* desc : 降序

```mysql
#查询所有商品，按价格降序排序
select * from product order by price desc;
```



#### 聚合函数

常用聚合函数：

* sum():求和
* avg():求平均值
* count():统计数量
* max():最大值
*  min():最小值

```mysql
select avg(price) from product;
  
  --注意:where 条件后面不能接聚合函数
  --查出商品价格大于平均价格的所有商品
  --子查询
  select * from product 
  where price > (select avg(price) from product);
  
--分组:group by
  -- 根据cno 字段分组,分组后统计商品的个数
  select cno,count(*) from product group by cno;
  
  --根据cno分组,分组统计每组商品的平均价格,并且平均价格>60
  select cno,avg(price)
  from product group by cno
  having avg(price) > 60;
  
  --having关键字,可以接聚合函数  出现在分组之后
  --where关键字,不可以聚合函数  出现在分组之前
  
  
--填写顺序
select .. from .. where .. group by .. having .. order by

--执行顺序
from .. where .. group by .. having .. select .. order by  
```









### 多表查询

```sql
create table category(
    cid int primary key auto_increment,
    cname varchar(10),
    cdesc varchar(31)
);


create table product(
    pid int primary key auto_increment,
    pname varchar(10),
    price double,
    pdate timestamp,
    cno int
);


--交叉链接查询
	--笛卡尔积,查出来的是两张表的乘积
	select * from product,category;
	--过滤
	select * from product,category where cno=cid;
	
--内连接查询
	--隐式内链接
	select * from product p,category c where p.cno=c.cid;
	--显式内链接
	select * from product p inner join category c on p.cno=c.cid;
	--区别:
	隐式内链接:在查询出结果的基础上去做where条件的过滤
	显式内链接:带着条件去查询结果,执行效率要高

--左外链接
左外链接会将左表中的所有数据都查询出来,如果右表中没有对应的数据,用null代替
select * from product p left outer join category c on p.cno=c.cid;

--右外连接
右外链接会将右表中的所有数据都查询出来,如果左表中没有对应的数据,用null代替
select * from product p right outer join category c on p.cno=c.cid;
```

* 多表之间的关系如何维护

   外键约束: foreign key

  * 给product表中cno添加外键约束

    > alter table product add foreign key(cno) references category(cid);

    

* 多表之间的建表原则

  * 一对多 : 在多的一方添加一个外键指向一的一方的主键
  * 多对多 : 
    * 多建一张中间表,将多对多的关系拆成一对多的关系
    * 中间表至少要有两个外键,分别指向原来的表
  * 一对一 :  不常用  拆表操作
    * 将一对一的情况当作是一对多情况处理,在任意一张表添加一个外键
    * 直接将两张表合并成一张表

* 主键约束: 唯一且不能为空

  * 外键都是指向另外一张表的主键
  * 主键一张表只能有一个

* 唯一约束: 唯一但可以为空

  * 唯一约束不可以作为其他表的外键
  * 一张表可以有多个唯一约束



### 分页查询

* 第一个参数是索引
* 第二个参数是显示的个数
* 起始索引:index代表显示第几页,页数从1开始
* 每页显示3条数据,则起始索引为:startindex = (index-1)*3

```sql
select * from product limit 0,3;
```



### 子查询

```sql
--查询分类名称为手机数码的所有商品
select * from product where cno=(select cid from category where cname='手机数码');

--查询出商品名称,商品分类名称信息
	--左链接
	select p.pname,c.cname from product p left outer join category c on p.cno=c.cid;

	--子查询
	select pname,(select cname from category c where p.cno=c.cid) as '商品分类名称' from product p;
```





## 事务

1. 什么是事务

   一件事情有n个组成单元, 要不这n个组成单元同时成功, 要不n个单元同时失败

2. mysql的事务

   * 默认的事务: 一条sql语句就是一个事务, 默认就开启事务并提交事务
   * 手动事务:
     * 显示的开启一个事务: start transaction
     * 事务提交: commit代表从开启事务到事务提交  中间的所有的sql都认为有效真正的更新数据库
     * 事务的回滚: rollback 代表事务的回滚 从开启事务到事务回滚, 中间的所有的sql操作都认为无效, 数据库没有被更新
   * Oracle默认手动事务



#### JDBC事务操作

* 默认是自动事务:

  执行sql语句: executeUpdate()  -----每执行一次executeUpdate方法 代表事务自动提交

* 通过jdbc的API手动事务:

  * 开启事务: conn.setAutoCommit(false)
  * 提交事务: conn.commit()
  * 回滚事务: conn.rollback()

  

#### DBUtils事务操作





#### 使用ThreadLocal绑定连接操作

实现的是通过线程绑定的方式传递参数