---
layout: post
title: MySQL 数据库的常用操作 
date: 2018-05-12
tags: [MySQL]
---

本文主要介绍 MySQL 数据库的安装、服务配置和一些常用的操作命令，系统环境为 Ubuntu 18.04  

### MySQL 数据库的安装、启动和登录

#### 1. MySQL 数据库的安装  
  
安装 MySQL 数据库的客户端  
> sudo apt install mysql-client-5.7 
  
安装 MySQL 数据库的服务端  
> sudo apt install mysql-server-5.7  
  
安装 MySQL 数据库的 C 语言开发库  
> sudo apt install libmysqlclient-dev  
  
因为服务端中包含了客户端的安装，所以安装客户端也可以省略  

#### 2. MySQL 服务的启动、停止和状态  

启动 MySQL 服务   
> service mysql start  
  
停止 MySQL 服务  
> service mysql stop  
  
重启 MySQL 服务  
> service mysql restart  
  
查看 MySQL 状态  
> service mysql status  

#### 3. MySQL 数据库的登录和退出  
  
登录 MySQL 的两种方式  
> mysql -u用户名 -p密码 // 这种方式会暴露明文密码  
> mysql -u用户名 -p 回车后再输入密码 // 密文的密码  
  
退出 MySQL  
> exit  
  
<br />  
### 对数据库的操作  
  
创建数据库 (默认字符集是拉丁文，不支持中文)  
> create database 数据库名;    
  
创建数据库 (指定字符集）  
> create database 数据库名 character set utf8;  

删除数据库  
> drop database 数据库名;  

修改数据库的字符集  
> alter database 数据库名 character set utf8;  

修改数据库名称  
> mysql 不支持直接修改数据库名，可以创建一个新数据库，把旧数据库中的内容导入的新数据库中，再删除旧数据库;导入导出数据的方法在本文档最后  

查看当前使用的数据库  
> select database();  

查看所有数据库  
> show databases;  
  
切换数据库  
> use 数据库名;  

查看创建数据库时的语句  
> show create database 数据库名;  
  
<br />
### 对用户的操作  
  
创建用户的两种方式  
> 第一种：  
> create user '用户名'@'允许连接的主机IP' identified by '密码';  
> 第二种：  
> insert into mysql.user(Host,User,Authentication_string) values('允许连接的主机IP','用户名','密码');  
> Authentication_string 仅在 5.7 版本后有效，以前的版本使用 password 替代  
  
创建用户的示例
> ``` sql  
> -- 创建 test 用户，密码为 test，限定该用户名只能在本地访问数据库
> create user 'test'@'127.0.0.1' identified by 'test';
> create user 'test'@'localhost' identified by 'test';  
> insert into mysql.user(host,user,authentication_string) values('localhost','test','test');  
> -- 创建 test 用户，密码为 test，% 代表可在任意 IP 地址使用该用户名访问数据库
> create user 'test'@'%' identified by 'test';
> ```  
  
删除用户的两种方式  
> 第一种：  
> drop user '用户名'@'允许连接的主机IP';  
> 第二种：  
> delete from mysql.user where user = '用户名' and host = '允许连接的主机IP';  
  
修改用户密码  
> update mysql.user set authentication_string = password('新密码') where user = '用户名' and host = '允许连接的主机IP';  
  
设置用户权限的两种方式  
> 第一种：(需要先创建用户，确保用户已存在)  
> grant 权限 on 数据库名.表名 to '用户名'@'允许连接的主机IP';  
> 第二种：(不需要事先创建用户，执行此语句时自动创建)  
> grant 权限 on 数据库名.表名 to '用户名'@'允许连接的主机IP' identified by '密码';  
  
设置用户权限的示例  
> ``` sql  
> -- 把 test 数据库下所有表的增删改查权限赋予给 test 用户    
> grant insert,delete,update,select on test.* to 'test'@'localhost';  
> -- 把所有数据库下所有表的查询权限赋予给 test 用户  
> grant select on '.' to 'test'@'localhost';    
> -- 把 test 数据库下所有表的所有权限赋予给 test 用户  
> grant all privileges on test.* to 'test'@'localhost';  
> ```  
    
刷新权限表 (每次修改权限后必须刷新权限表使之生效)  
> flush privileges; 
  
查看所有用户  
> select host,user from mysql.user;  
  
<br /> 
### 对数据表的操作  
  
创建数据表  
> create table 表名(字段名 字段类型，字段名 字段类型，...);  
> create table 表名(字段名 字段类型，字段名 字段类型，...) character set utf8;  
  
删除数据表  
> drop table 表名;  
  
修改数据表  
1. 修改表名  
> rename table 原表名 to 新表名;  

2. 增加字段  
> alter table 表名 add column 字段名 字段类型; //column 可省略  

3. 修改字段类型  
> alter table 表名 modify column 字段名 新字段类型;  

4. 修改字段名和字段类型  
> alter table 表名 change column 原字段名 新字段名 新字段类型;  

5. 删除字段  
> alter table 表名 drop column 字段名;  

6. 修改表的字符集  
> alter table 表名 character set utf8;  
  
显示数据库中的表  
> show tables;  
  
显示数据表的结构  
> desc 表名;  
> describe 表名;  
  
查看创建表时的语句  
> show create table 表名;  

<br />
### 数据类型  
  
整数类型  
> tinyint    1字节  
> smallint   2字节  
> int或integer    4字节  
> bigint     8字节  
> float(m,n)      4字节  //不指定m,n有可能无法通过where条件查询  
> double     8字节  
> decimal(m,d)   m>d?m+2:d+2     m代表数据的宽度，d代表小数宽度  
  
字符串类型  
> char   定长字符串  
> varchar(n)    变长字符串，n为最大长度  
> text   长文本数据  
> blob   二进制长文本数据  
  
日期和时间类型  
> date   4字节 yyyy-mm-dd  日期  
> time   3字节 hh:mm:ss    时间  
> year   1字节 yyyy        年份  
> datetime   8字节 yyyy-mm-dd hh:mm:ss 混合日期时间  
> timestamp  4字节 yyyy-mm-dd hh:mm:ss 混合日期时间 默认值为当前系统时间，可以指定该字段值是否随着记录更新而更新  
  
复合类型  
> enum   只能使用集合中的一个值或null  
> set    只能使用集合中的多个值或null  

<br />
### 对数据的操作  
  
插入数据  
> insert into 表名(字段名，字段名，...) values(字段值，字段值，...); //如果为所有字段插入数据可以不指定字段名  
  
删除数据  
> delete from 表名; //删除表中所有数据  
> delete from 表名 where 字段名 = ?; //根据条件删除  
> truncate 表名; //删除表中所有数据，这种方式效率高  
  
修改数据  
> update 表名 set 字段名 = ?; //修改单个字段  
> update 表名 set 字段名 = ?,字段名=?; //修改多个字段  
> update 表名 set 字段名 = ? where 字段名 = ?; //根据条件修改  
  
查询数据  
  
1. 查询所有数据  
> select * from 表名;   
  
2. 查询指定字段  
> select 字段名,字段名 from 表名;  
  
3. 根据条件查询  
> select 字段名 from 表名 where 字段名 = ?;  
  
4. 字段别名  
> select 字段名 as 别名 from 表名; //as 可以省略  
  
5. 去掉重复数据  
> select distinct 字段名 from 表名; //当指定多个字段时，多个字段必须全部匹配才会成功  
    
6. 滤空修正  
> select ifnull(字段名,替换值) from 表名;  
  
7. 使用算术表达式  
> select 字段名 + 字段名 as '别名' from 表名;  
  
8. 字符串拼接  
> select concat(str1,str2,...) as '别名' from 表名;  
  
9. 聚合函数 min,max,avg,count  
> select min(字段名) from 表名;  
  
10. 条件查询运算符  
> select * from 表名 where 字段名 > 3; // =,>,>=,<,<=  
> select * from 表名 where 字段名 <> 3; // <> 不等于  
> select * from 表名 where 字段名 between 3 and 5; //区间  
> select * from 表名 where 字段名 is null; //空值  
> select * from 表名 where 字段名 is not null; //非空值  
> select * from 表名 where 字段名 = ? and 字段名 = ?; //与  
> select * from 表名 where 字段名 = ? or 字段名 = ?; //或  
  
11. 模糊查询  
> select * from 表名 where 字段名 like '%xxx'; // % 表示任意个数的任意字符  
> select * from 表名 where 字段名 like '_xxx'; // _ 表示单个的任意字符  
> select * from 表名 where 字段名 like '[0-9]abc'; // [] 表示单个字符的取值范围  
> select * from 表名 where 字段名 like '[^0-9]abc';// [^] 表示单个字符的反向取值范围  
> select * from 表名 where 字段名 like '%\%%'; // \ 表示转义,查询字段中包含%的数据  
  
12. 分组查询  
> select * from 表名 where 查询条件 group by 字段名 having 查询条件;  
  
13. 排序查询  
> select * from 表名 where 查询条件 order by 字段名 asc; // asc 表示升序排列,可省略  
> select * from 表名 where 查询条件 order by 字段名 desc; // desc 表示降序排列  
  
14. 嵌套查询  
> select * from 表名 where 字段名 = (select 字段名 from 表名 where 字段名=?);  

    示例：  
    > select * from 表名 where 字段名 > all (select 字段名 from 表名);  

    * 当子查询返回的值只有一个时，才可以直接使用 =,>,< 等运算符;  
    * 当子查询返回的值是一个结果集时，需要使用 in,not in,any,some,all 等操作符;  
    * all : 匹配结果集中的所有结果, 例如 > all 表示比所有结果都大;  
    * any : 匹配结果信中任意一个结果, 例如 < any 表示只要比其中任意一个小就符合条件;  
    * in : 查询的数据在子查询的结果集中,相当于 = any;  
    * not in : 查询的数据不在子查询结果集中,相当于 <> all;  
    * some : 匹配结果集中的部分结果，基本等同于 any,不常用;  
    * 子查询通常只用于返回单个字段的结果集，如果需要匹配多个字段，可以使用多表联查;   
  
15. 多表联查  
  
    数据表的别名  
    > select * from 表名 as 别名; // as 可省略  

    交叉连接(笛卡尔积，显示两张表的乘积)    
    > select * from 表1 cross join 表2 on 连接条件 where 查询条件; //cross join on 可省略  
    > select * from 表1,表2 where 查询条件; //默认就是交叉连接  

    内连接(显示两张表的交集)  
    > select * from 表1 inner join 表2 on 连接条件 where 查询条件; //inner 可省略  

    左外连接(左表显示全集,右表显示交集)  
    > select * from 表1 left outer join 表2 on 连接条件 where 查询条件; //outer 可省略  

    右外连接(左表显示交集,右表显示全集)  
    > select * from 表1 right outer join 表2 on 连接条件 where 查询条件; //outer 可省略  

    联合查询(显示两张表的并集,要求两张表的查询字段名必须保持一致)  
    > select id,name from 表1 union select id,name from 表2;  

    全外连接(MySQL不支持，显示两张表的并集)  
    > select * from 表1 full outer join 表2 on 连接条件 where 查询条件; //outer 可省略  
    //可以通过左外连接和右外连接的联合查询来实现  
    > select * from 表1 left join 表2 on 连接条件 where 查询条件 union select * from 表1 right join 表2 on 连接条件 where 查询条件;  
   
16. 常用字符串函数    
    把所有字符转换为大写字母 
    > upper 和 ucase  
    > select upper(name) from 表名;  

    把所有字符转换为小写字母  
    > lower 和 lcase  
    > select lcase(name) from 表名;  

    把str中的from_str替换为to_str  
    > replace(str, from_str, to_str)  
    > select replace(字段名,替换前的值,替换后的值) from 表名;  

    返回str重复count次的字符串  
    > repeat(str,count)  
    > select repeat('abc',2) from 表名; // abcabc  

    逆序字符串  
    > reverse(str)  
    > select reverse('abc') from 表名; // cba  

    把str中pos位置开始长度为len的字符串替换为newstr  
    > insert(str,pos,len,newstr)  
    > select insert('abcdef',2,3,'hhh'); // ahhhef  

    从str中的pos位置开始返回一个新字符串
    > substring(str from pos)  
    > select substring('abcdef',3); // cdef  

    返回str中第count次出现的delim之前的所有字符,如果count为负数,则从右向左  
    > substring_index(str,delim,count)  
    > select substring_index('abacadae','a',3); // abac  

    去除字符串左边的空格  
    > ltrim(str)  
    > select ltrim('  abc');  

    去除字符串右边的空格  
    > rtrim(str)  
    > select rtrim('abc  ');  

    去除字符串左右两边的空格  
    > trim(str)  
    > select trim('   abc   ');  

    从str中的pos位置开始返回len个长度的字符串  
    > mid(str,pos,len)  
    > select mid('abcdef',2,3); // bcd  

    在str左边填充padstr直到str的长度为len  
    > lpad(str,len,padstr)  
    > select lpad('abc',8,'de'); // dededabc  

    在str右边填充padstr直到str的长度为len  
    > rpad(str,len,padstr)  
    > select rpad('abc',8,'de'); // abcdeded  

    返回str左边的len个字符  
    > left(str,len)  
    > select left('abcd',2); // ab  

    返回str右边的len个字符  
    > right(str,len)  
    > select right('abcd',2); // cd  

    返回substr在str中第一次出现的位置  
    > position(substr in str)  
    > select position('c' in 'abcdc'); // 3

    返回字符串的长度  
    > length(str)  
    > select length('abcd'); // 4

    合并字符串  
    > concat(str1,str2,...)  
    > select concat('abc','def','gh'); // abcdefgh  

17. 日期时间函数  
    返回date是星期几,1代表星期日,2代表星期一...  
    > dayofweek(date)  
    > select dayofweek('2017-04-09');  

    返回date是星期几,0代表星期一,1代表星期二...  
    > weekday(date)  
    > select weekday('2017-04-09');  

    返回date是星期几(按英文名返回)  
    > dayname(date)  
    > select dayname('2017-04-09');  

    返回date是一个月中的第几天(范围1-31)  
    > dayofmonth(date)  
    > select dayofmonth('2017-04-09');  

    返回date是一年中的第几天(范围1-366)  
    > dayofyear(date)  
    > select dayofyear('2017-04-09');  

    返回date中的月份数值  
    > month(date)  
    > select month('2017-04-09');  

    返回date是几月(按英文名返回)  
    > monthname(date)  
    > select monthname('2017-04-09');  

    返回date是一年中的第几个季度  
    > quarter(date)  
    > select quarter('2017-04-09');  

    返回date是一年中的第几周(first默认值是0,表示周日是一周的开始,取值为1表示周一是一周的开始)  
    > week(date,first)  
    > select week('2017-04-09');  
    > select week('2017-04-09',1);  

    返回date的年份  
    > year(date)  
    > select year('2017-04-09');  

    返回time的小时数  
    > hour(time)  
    > select hour('18:06:53');  

    返回time的分钟数
    > minute(time)  
    > select minute('18:06:53');  

    返回time的秒数  
    > second(time)  
    > select second('18:06:53');  

    增加n个月到时期p并返回(p的格式为yymm或yyyymm)  
    > period_add(p,n)  
    > select period_add(201702,2);  

    返回在时期p1和p2之间的月数(p1,p2的格式为yymm或yyyymm)  
    > period_diff(p1,p2)  
    > select period_diff(201605,201704);  

    根据format字符串格式化date  
    > date_format(date,format)  
    > select date_format('2017-04-09','%d-%m-%y');  

    根据format字符串格式化time  
    > time_format(time,format)  
    > select time_format('12:22:33','%s-%i-%h');  

    以'yyyy-mm-dd'或yyyymmdd的格式返回当前日期值  
    > curdate() 和 current_date()  
    > select curdate();  
    > select current_date();  

    以'hh:mm:ss'或hhmmss格式返回当前时间值  
    > curtime() 和 current_time()  
    > select curtime();  
    > select current_time();  

    以'yyyy-mm-dd hh:mm:ss'或yyyymmddhhmmss格式返回当前日期时间 
    > now(),sysdate(),current_timestamp()  
    > select now();  
    > select sysdate();  
    > select current_timestamp();  

    返回一个unix时间戳(从'1970-01-01 00:00:00'开始到当前时间的毫秒数)  
    > unix_timestamp()  
    > select unix_timestamp();  

    把秒数seconds转化为时间time  
    > sec_to_time(seconds)  
    > select sec_to_time(3666);  

    把时间time转化为秒数seconds  
    > time_to_sec(time)  
    > select time_to_sec('01:01:06');  

18. top N 问题  
> 从第 M 行开始取 N 行数据，索引 M 从 0 开始  
> select * from 表名 limit M,N;  

### 索引  
* 普通索引   
> create index 索引名 on 表名(字段名(索引长度));  
> alter table 表名 add index 索引名 on (字段名(索引长度));  
> create table 表名(字段名 字段类型,字段名 字段类型,index 索引名 (字段名(索引长度));  

* 唯一索引  
> create unique index 索引名 on 表名(字段名(索引长度));  
> alter table 表名 add unique 索引名 on (字段名(索引长度));  
> create table 表名(字段名 字段类型,字段名 字段类型,unique 索引名 (字段名(索引长度));  

* 全文索引  
//只支持 MyISAM 引擎  
> create fulltext index 索引名 on 表名(字段名);  
> alter table 表名 add fulltext 索引名(字段名);  
> create table 表名(字段名 字段类型,字段名 字段类型,fulltext (字段名);  

* 组合索引  
> create index 索引名 on 表名(字段名(索引长度),字段名(索引长度),...);  
> alter table 表名 add index 索引名 on (字段名(索引长度),字段名(索引长度),...;  
> create table 表名(字段名 字段类型,字段名 字段类型,index 索引名 (字段名(索引长度),字段名(索引长度));  

### 约束  
* 主键约束  
> create table 表名(字段名 字段类型 primary key,字段 字段类型,...);  
> //一个表只能有一个主键,这个主键可以由一列或多列组成  
> create table 表名(字段1 字段类型,字段2 字段类型,primary key(字段1,字段2);  

* 唯一键约束  
> create table 表名(字段名 字段类型 unique,字段名 字段类型,...);  

* 外键约束  
> create table 表1(字段名 字段类型,字段名 字段类型 references 表2(字段名),...);  
> create table 表1(字段1 字段类型,字段2 字段类型,foreign key(字段1) references 表2(字段名),...);  

* 非空约束  
> create table 表名(字段名 字段类型 not null,字段名 字段类型,...);  

* 默认值约束  
> create table 表名(字段名 字段类型 default 默认值,字段名 字段类型,...);  

* check约束(MySQL 不支持)  
> create table 表名(字段1 字段类型,字段2 字段类型,check(字段1 > 30),...);  

## 11. 事务  
`begin;`  
`要执行的操作;`  
`commit;` 或 `rollback;`  
## 12. 触发器  
`create trigger 触发器名`  
`{before|after}`  
`insert|update|delete}`  
`on 表名`  
`for each row`  //行触发器,MySQL不支持语句触发器  
`begin`  
`触发器执行的操作`  
`end;`
## 13. 存储过程  
* 创建存储过程  
`create procedure 存储过程名()`  
`{参数}`  
`begin`  
`要执行的操作`  
`end;`  
* 调用存储过程  
`call 存储过程名();`  
* 删除存储过程  
`drop procedure 存储过程名;`
## 14. 视图  
`create view 视图名`  
`as`  
`查询的SQL语句`  
## 15. 数据库的导入导出  
* 导入  
`source xxx.sql;`  
* 导出  
`mysqldump -u root -p 数据库名[.表名] > xxx.sql;`
