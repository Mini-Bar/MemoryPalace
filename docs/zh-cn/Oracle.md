# oracle

####  数据文件（dbf）

数据文件是数据库的物理存储单位。数据库的数据是存储在表空间中的，真正是在某一个或者多个数据文件中，而一个表空间可以由一个或多个数据文件组成，一个数据文件只能属于一个表空间。一旦数据文件被加入到某个表空间后，就不能删除这个文件，如果要删除某个数据文件，只能删除其所属于的表空间才行。

#### 表空间

表空间是Oracle对物理数据库上相关数据文件（ORA 或者DBF文件）的逻辑映射。一个数据库在逻辑上被划分成一到若干个表空间，每个表空包含了在逻辑上相关联的一组结构。每个数据库至少有一个表空间（称之为system表空间）。每个表空间由同一磁盘上的一个或多个文件组成，这些文件叫数据库文件（datafile）。一个数据文件只能属于一个表空间。

![](https://raw.githubusercontent.com/Mini-Bar/MyPicture/main/pic/image-20210111094259718.png)

#### 数据库和实例 

Oracle数据库服务器由一个数据库和至少一个数据库实例组成。 数据库是一组存储数据的文件，而数据库实例则是管理数据库文件的内存结构。此外，数据库是由后台进程组成。

数据库和实例是紧密相连的，所以**我们一般说的Oracle数据库，通常指的就是实例和数据库。**

![](https://raw.githubusercontent.com/Mini-Bar/MyPicture/main/pic/image-20210111095339710.png)

在这种体系结构中，Oracle 数据库服务器包括两个主要部分：文件(Oracle数据库)和内存(Oracle实例)。

![](https://raw.githubusercontent.com/Mini-Bar/MyPicture/main/pic/image-20210111100031217.png)

![](https://raw.githubusercontent.com/Mini-Bar/MyPicture/main/pic/image-20210111100926497.png)

#### 由于oracle没有库名，只有表空间，所以oracle没有提供数据库名称查询的支持，之提供了表空空间名称查询

##### 1：查询所有数据库

> ```sql
> select * from v$tablespace; --查询表空间（需要一定权限）
> ```

##### 2：查询当前数据库中所有表名

> ```sql
> select * from user_tables;
> ```



##### 3：查询指定表中的所有字段

> ```sql
> select column_name from user_tab_columns 
> 	where table_name = 'table_name'; --表名要全部大写
> ```

##### 4：查询指定表中的所有字段名和字段类型

> ```sql
> select column_name,data_type from user_tab_columns 
> 	where table_name = 'table_name'; --表名 要全部大写
> ```



### 语法：

##### 创建表：

> ```sql
> --创建有个person表
> create table person(
>     pid number(20),
>     pname varchar2(10)
> );
> ```



##### 修改表：

> ```sql
> --修改表结构  （添加了一列）
> alter table person add(gender number(1));
> ```



##### 修改列类型：

> ```sql
> --将name 设置为number为不可变长度
> alter table person modify gender char(1);
> ```



##### 删除一列：

> ```sql
> --删除一列
> alter table person drop column sex;
> ```



##### 添加一条记录：

> ```sql
> --添加一条纪录（记得加commit，不加commit提交一下没有真正添加进去）
> insert into person (pid,pname) values (1,'小明');
> 	commit;
> ```



##### 修改列名称：

> ```sql
> --修改列名称
> alter table person rename column gender to sex;
> ```



##### 查看表数据：

> ```sql
> --查看表数据
> select * from person;
> ```



##### 修改一条记录：

> ```sql
> --修改一条记录
> update person set pname = '小马' where pid = 1;
> commit;
> ```



##### 三个删除：

> ```sql
> --删除表中全部纪录
> delete from person;
> --删除表结构
> drop table person;
> --删除表，再创建表，效果等同于删除表全部记录
> --在数据量大的情况下，尤其在表中带有索引的情况下，该操作的效率很高
> --索引可以提高查询效率，但是会影响增删改的效率
> truncate table person;
> ```



```sql
   --序列不真的属于任何一张表，但是可以逻辑和表做绑定
    --序列：默认从1开始，依次递增，主要用来给主键赋值使用
    --dual:虚表，只是为了补全语法，没有任何意义
    create sequence s_person;
    select s_person.nextval from dual;
```

```sql
--添加一条纪录
insert into person (pid,pname) values (s_person.nextval,'小明');
commit;
```



# 数据库基础

##### 数据库基本概念：

> 1. 数据库（Database，DB）
>
> 数据库管理员 （Database Management System,DBMS）
>
> 数据库管理员（Database Administrator,DBA）
>
> 数据库系统（Database System,DBS）

##### Oracle概述

> 

##### 安装oracle数据库

###### 注意：


!> 1：安装的时候，一定要关掉防火墙，否则可能安装不成功

!> 2：全局数据库名SID,类似与Mysql中常用的localhost

!> 3：字符集一定要选择正确，一旦选错，除非更改该字符集的父类，否则只有重新安装

!> 4：安装后主要的用户为：

!> 普通用户： Scott/tiger(联系常用)、

!> 普通管理员：System/system
	
!> 超级管理员：Sys/sys
	
!> 安装完后的服务配置（打开服务的快捷方式：运行中输入services.msc)
