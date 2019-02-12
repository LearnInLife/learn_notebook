## MySql数据库

[TOC]

### 数据库概述

* 数据库就是存储数据的仓库，其本质是一个文件系统，数据按照特定的格式将数据存储起来，用户可以通过SQL对数据库中的数据进行增删改查的操作
* 数据库管理系统（DBMS）：操作和管理数据库的大型软件。
* 常见数据库：
  * mysql：开源免费，小型的数据库，被oracle收购后，6.x版本也开始收费
  * oracle：收费的大型数据库
  * db2：IBM公司的产品，收费，常应用在银行系统中
  * SqlServer：微软公司的中型数据库，收费，C#、.net等语言常使用
  * SyBase：已经退出历史舞台，提供了专业数据建模的工具PowerDesigner
  * SQLite：嵌入式小型数据库，应用在手机端



### SQL语句

* 什么是SQL语句
  * 一种结构化查询语言，关系数据库语言的国际标准
  * 各个数据库厂商都支持ISO的SQL标准，相当于官方普通话
  * 各个厂商在标准基础上做了自己的扩展，相当于方言
* SQL的分类
  * 数据定义语言：DDL，用来定义数据库对象：数据库 database、表 table、列 column等。关键字（操作的是结构）：创建：create、修改：alter、删除：drop等
  * 数据操作语言：DML，用来对数据库中表的记录进行增删改。关键字（操作的是数据）：插入：insert、删除：delete、更新：update等
  * 数据查询语言：DQL，用来查询数据库中表中记录。关键字：select、from、where等



### DDL

#### 对数据库操作

* 创建库

  ```mysql
  create database 库名;
  create database 库名 character set 字符集;
  ```

* 查看数据库

  ```mysql
  #查看所有数据库
  show databasese;
  #查看某个数据库的定义信息
  show create database 库名;
  ```

* 删除数据库

  ```mysql
  drop database 库名;
  ```

* 切换数据库

  ```mysql
  use 库名;
  ```

* 查看正在使用的数据库

  ```mysql
  select database();
  ```


#### 对表操作

* 创建表

  ```mysql
  create table 表名(
      字段名 类型(长度) [约束],
      字段名 类型(长度) [约束],
      字段名 类型(长度) [约束],
  );
  ```

* 查看表

  ```mysql
  #查看所有表
  show tables;
  #查看表结构
  desc 表名;
  ```

* 删除表

  ```mysql
  drop table 表名;
  ```

* 修改表

  ```mysql
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

* 修改表的字符集

  ```mysql
  alter table 表名 character set 编码;
  ```

* 查看表的创建信息

  ```mysql
  show create table 表名;
  ```



### DML

* 插入数据

  ```mysql
  #表中某些列插入数据
  insert into 表名(列名1,列名2,列名3……) values(值1,值2,值3……)
  #插入所有列
  insert into 表名 values(值1,值2,值3……)
  ```

  * 注意：

    * 列的顺序与插入的值的顺序一致
    * 值如果是字符串或者日期需要加引号''(一般是单引号)

  * 插入数据中文乱码问题解决：

    * 直接修改数据库安装目录里面的my.ini文件的第57行

      ![1549961831109](assets/1549961831109.png)

    * set names gbk;

