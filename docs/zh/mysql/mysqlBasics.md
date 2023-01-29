---
title: mysqlBasics
date: 2023-01-28 05:25:48
permalink: /pages/05c23f/
categories:
  - zh
  - mysql
tags:
  - 
author: 
  name: fengsha
  link: https://github.com/ifengshai/
---
# 数据库基础概念

目录：
* [mysql优化](#mysql优化)

## 1.SQL语句的分类DCL,DDL,DML分别是什么含义，有哪些关键字?

DML（data manipulation language）是数据操纵语言：它们是SELECT、UPDATE、INSERT、DELETE，就象它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言。
DDL（data definition language）是数据定义语言：DDL比DML要多，主要的命令有CREATE、ALTER、DROP等，DDL主要是用在定义或改变表（TABLE）的结构，数据类型，表之间的链接和约束等初始化工作上，他们大多在建立表时使用。
DCL（Data Control Language）是数据库控制语言：是用来设置或更改数据库用户或角色权限的语句，包括（grant,deny,revoke等）语句。

## 2.什么是主表，什么是从表(1:n)

主表：以另一个关系的外键作主键的表被称为主表
从表：具有此外键的表被称为主表的从表

## 3.什么叫主键，什么叫外键

1.主键是能确定一条记录的唯一标识
2.外键用于与另一张表的关联。是能确定另一张表记录的字段用于保持数据的一致性。

## 4.实体间的映射关系有哪些，举例说明！

一对一：像人对应身份证号码，一个人只会有一个身份证号，一个身份证号也只会对应一个人
一对多（多对一）：父亲对孩子，一个父亲可以有多个孩子，但一个孩子只会有一个父亲。一对一是一对多（或多对一）的特例。
多对一：一对多反过来就是了，孩子对父亲。
多对多：课程对学生，一门课可以供多个学生选择，一个学生也可以选择多门课程。

## 5.SQL Server/Oracle的约束有哪些？如何增加和删除这些约束？

约束用于强制行数据满足特定的商业规则（数据类型是强制列的数据满足规则）
约束有六种类型：NOT NULL、UNIQUE、PRIMARY KEY、FOREIGN KEY、CHECK、DEFAULT
---添加主键约束：alter table 表名add constraint 约束名 primary key (主键)
---添加唯一约束：alter table 表名add constraint 约束名 unique (字段)
---添加默认约束：alter table 表名add constraint 约束名 default ('默认内容') for 字段
--添加检查check约束,要求字段只能在1到100之间：alter table 表名add constraint 约束名 check (字段 between 1 and 100 )
 ---添加外键约束：
      alter table 从表add constraint 约束名foreign key(关联字段) references 主表(关联字段)
删除约束的语句是：alter table 表名 drop constraint 约束名sp_helpconstraint 表名 找到数据表中的所有列的约束

## 6.SQL,T-SQL,PL/SQL三者有什么关系？

SQL是结构化查询语言，标准的关系型数据库通用的标准语言
T-SQL是在SQL基础上扩展的sqlserver中使用的语言
PL/SQL是在SQL基础上扩展的oracle中使用的语言

## 7.软件开发的基本步骤是什么

需求分析
系统设计
系统编码
测试运行
升级维护

## 8.请说出数据库的三大范式

第一范式：又称1NF，它指的是在一个应用中的数据都可以组织成由行和列的表格形式，且表格的任意一个行列交叉点即单元格，都不可再划分为行和列的形式，实际上任意一张表格都满足1NF；
第二范式：又称2NF，它指的是在满足1NF的基础上，一张数据表中的任何非主键字段都全部依赖于主键字段，没有任何非主键字段只依赖于主键字段的一部分。即，可以由主键字段来唯一的确定一条记录。比如学号+课程号的联合主键，可以唯一的确定某个成绩是哪个学员的哪门课的成绩，缺少学号或者缺少课程号，都不能确定成绩的意义。
第三范式：又称3NF，它是指在满足2NF的基础上，数据表的任何非主键字段之间都不产生函数依赖，即非主键字段之间没有依赖关系，全部只依赖于主键字段。例如将学员姓名和所属班级名称放在同一张表中是不科学的，因为学员依赖于班级，可将学员信息和班级信息单独存放，以满足3NF。

#  SQL:

## 1.在ORACLE中，一张表有10000条数据，如何取出第11条至第100条数据？

```sql
(select * from test where rownum<=100) minus (select * from test where rownum<=10);
```

## 2.表product和表chance。

product

| productId | name  | price |
| --------- | ----- | ----- |
| 1         | name1 | 110   |
| 2         | name2 | 120   |
| 3         | name3 | 250   |
| 4         | name4 | 150   |

chance

| productId | clientname | count | saleprice |
| --------- | ---------- | ----- | --------- |
| 2         | m          | 2     | 150       |
| 1         | n          | 3     | 110       |
| 3         | n          | 3     | 250       |
| 4         | m          | 2     | 150       |

（1）将产品名称为“name2”的产品在销售机会表中的单价修改为产品表中的单价。

```sql
update chance set saleprice=(select price from product where name='name2') where productId=(select productId from product where
name='name2');
```

（2）统计出各个客户各自的销售总额的sql语句。

```sql
select clientname,sum(saleprice*count) total from chance group by clientname;
```

（3）找出销售总额最大的客户名字的sql语句。

```sql
select * from (select clientname from chance group by clientname order by sum(saleprice*count) desc ) where rownum=1;
```

## 3.我们经常要做数据统计的工作，如下示例统计各部门每月的工作业绩(学习使用CASE-WHEN-THEN-ELSE-END)

table1(月份, 部门, 业绩)

| mon  | dep  | yj   |
| ---- | ---- | ---- |
| 1    | 1    | 10   |
| 1    | 2    | 10   |
| 1    | 3    | 5    |
| 2    | 2    | 8    |
| 2    | 4    | 9    |
| 3    | 3    | 8    |

table2(部门, 部门名称)

| dep  | dname   |
| ---- | ------- |
| 1    | A业务部 |
| 2    | B业务部 |
| 3    | C业务部 |
| 4    | D业务部 |

table3 （结果）

| 部门 | 一月份 | 二月份 | 三月份 |
| ---- | ------ | ------ | ------ |
| 1    | 10     | 0      | 0      |
| 2    | 10     | 8      | 0      |
| 3    | 5      | 0      | 8      |
| 4    | 0      | 0      | 9      |

```sql
select dep 部门,
sum(case when mon=1 then yj else 0 end) 一月份,
sum(case when mon=2 then yj else 0 end) 二月份,
sum(case when mon=3 then yj else 0 end) 三月份
from table1 group by dep;
```

(查询时列名前，加表名或表别名前辍（如果字段在两个表中是唯一的可以不加）.)



## 4.有一张成绩表，里面有3个字段：语文，数学，英语。请用一条sql语句查询出这表里的记录并按以下条件显示出来： 

成绩表score

| 姓名 | chinese | math | english |
| ---- | ------- | ---- | ------- |
| 小明 | 62      | 99   | 25      |
| 小王 | 89      | 65   | 35      |

 大于或等于80表示优秀，大于或等于60表示及格，小于60分表示不及格。 显示格式： 

语文       数学        英语 

及格        优秀        不及格  

```sql
select student,
case
when chinese>=80 then '优秀'
when chinese>=60 and chinese<80 then '及格'
else '不及格' end 语文,
case
when math>=80 then '优秀'
when math>=60 and math<80 then '及格'
else '不及格' end 数学,
case
when english>=80 then '优秀'
when english>=60 and english<80 then '及格'
else '不及格' end 英语 from score;
```

\--------------------------------------------------------------------------

## 5.一张表内容如下：

score

| mydate     | myscore |
| ---------- | ------- |
| 2005-05-09 | 胜      |
| 2005-05-09 | 胜      |
| 2005-05-09 | 负      |
| 2005-05-09 | 负      |
| 2005-05-10 | 胜      |
| 2005-05-10 | 负      |
| 2005-05-10 | 负      |


如果要生成下列结果, 该如何写sql语句?

| 日期       | 胜   | 负   |
| ---------- | ---- | ---- |
| 2005-05-09 | 2    | 2    |
| 2005-05-10 | 1    | 2    |

```sql
select mydate 日期,
sum(case when mysocre='胜' then 1 else 0 end ) 胜,
sum(case when mysocre='负' then 1 else 0 end ) 负
from score group by mydate;
```



## 6.学生表student，字段sID,sName,sAge；

教师表teacher，字段tID,TName,tAge；
教师-学生表ts，字段id,tID,sID。
请统计每位教师有多少学生，
要求教师年龄大于30，学生年龄大于12，并列出教师ID，教师姓名，学生人数
create table student(
   sID int primary key,
   sName varchar(50),
   sAge int
);
insert into student values(1,'zs',12);
insert into student values(2,'ls',13);
insert into student values(3,'ww',14);
insert into student values(4,'zl',15);


create table teacher(
   tID int primary key,
   tName varchar(50),
   tAge int
);
insert into teacher values(1,'t1',30);
insert into teacher values(2,'t2',31);
insert into teacher values(3,'t3',32);
insert into teacher values(4,'t4',33);

create table ts(
   id int primary key,
   sID int,
   tID int
);
insert into ts values(1,1,1);
insert into ts values(2,1,2);
insert into ts values(3,1,3);
insert into ts values(4,1,4);
insert into ts values(5,2,2);
insert into ts values(6,2,3);
insert into ts values(7,2,4);
insert into ts values(8,3,4);
insert into ts values(9,4,4);

select t.tid 老师ID,tname 老师姓名,count(s.sid) 学生人数 from student

s,teacher t,ts where ts.tid=t.tid and s.sid=ts.sid and t.tage>30 and

s.sage>12 group by t.tid,t.tname order by t.tid asc;

## 7.有下列三张表：

CARD   借书卡。 CNO卡号，NAME姓名，CLASS班级

BOOKS  图书。   BNO 书号，BNAME 书名，AUTHOR 作者，PRICE 单价,QUANTITY 库存册数 BORROW  借书记录。 CNO 借书卡号，BNO 书号，RDATE 还书日期

备注：限定每人每种书只能借一本；库存册数随借书、还书而改变。

要求实现如下15个处理：

1).写出建立BORROW表的SQL语句，要求定义主码完整性约束和引用完整性约束。
create table borrow(
   cno int not null,
   bno int not null,
   rdate date
   primary key(cno,bno)
);
create table card(
   cno int primary key,
   name varchar(30),
   class varchar(30)
);

create table books(
   bno int primary key,
   bname varchar(30),
   author varchar(30),
   price int,
   quantity int
);
insert into card values(1,'zs','c01');
insert into card values(2,'ls','c02');
insert into card values(3,'ww','c03');

insert into books values(1,'水浒传','sna',120,8);
insert into books values(2,'西游记','wce',150,12);
insert into books values(3,'红楼梦','cxq',100,11);
insert into books values(4,'三国演义','lgz',140,15);
insert into books values(5,'Java宝典','wu',80,7);
insert into books values(6,'C++宝典','wu',60,9);
insert into books values(7,'网络协议详解','wu',59,6);

insert into borrow values(1,1,to_date('2010-10-1','yyyy-mm-dd'));
insert into borrow values(1,2,to_date('2010-11-1','yyyy-mm-dd'));
insert into borrow values(1,3,to_date('2010-12-1','yyyy-mm-dd'));
insert into borrow values(1,4,to_date('2010-8-1','yyyy-mm-dd'));
insert into borrow values(1,5,to_date('2010-7-1','yyyy-mm-dd'));
insert into borrow values(1,6,to_date('2010-12-1','yyyy-mm-dd'));
insert into borrow values(2,1,to_date('2010-9-5','yyyy-mm-dd'));
insert into borrow values(2,2,to_date('2010-9-5','yyyy-mm-dd'));
insert into borrow values(2,3,to_date('2010-12-1','yyyy-mm-dd'));
insert into borrow values(2,4,to_date('2010-3-23','yyyy-mm-dd'));
insert into borrow values(2,5,to_date('2010-5-21','yyyy-mm-dd'));
insert into borrow values(3,1,to_date('2010-12-6','yyyy-mm-dd'));
insert into borrow values(3,2,to_date('2010-11-1','yyyy-mm-dd'));
insert into borrow values(3,3,to_date('2010-11-1','yyyy-mm-dd'));
insert into borrow values(3,4,to_date('2010-11-1','yyyy-mm-dd'));
insert into borrow values(3,5,to_date('2010-11-1','yyyy-mm-dd'));
insert into borrow values(3,6,to_date('2010-11-1','yyyy-mm-dd'));
2).找出借书超过5本的读者,输出借书卡号及所借图书册数。

select cno 读者卡号,count(bno) 个数 from borrow group by cno having count(bno)>5;

3).查询借阅了"水浒传"一书的读者，输出姓名及班级。

select name,class from card,borrow where borrow.bno=(select bno from books where bname='水浒传') and card.cno=borrow.cno;

4).查询过期未还图书，输出借阅者（卡号）、书号及还书日期。

select * from borrow where sysdate>rdate;

5).查询书名包括"网络"关键词的图书，输出书号、书名、作者。

select bno,bname,author from books where bname like '%网络%';

6).查询现有图书中价格最高的图书，输出书名及作者。

select bname,author from books where price=(select max(price) from books);

7).查询当前借了"水浒传"但没有借"西游记"的读者，输出其借书卡号，并按卡号降序排序输出。

select c.cno,c.name from card c,books b,borrow bo where c.cno=bo.cno

and b.bno=bo.bno and bname='水浒传' and c.cno not in(select c.cno

from card c,books b,borrow bo where c.cno=bo.cno and b.bno=bo.bno and

bname='西游记') order by c.cno desc;

8).将"C01"班同学所借图书的还期都延长一周。

update borrow set rdate=rdate+7 where cno=(select cno from card where class='c01');

9).从BOOKS表中删除当前无人借阅的图书记录。

delete from books where bno not in(select distinct bno from borrow);

10).如果经常按书名查询图书信息，请建立合适的索引。

create index books_index_name on books(bname);

11).在BORROW表上建立一个触发器，完成如下功能：如果读者借阅的书名是"数

据库技术及应用"，就将该读者的借阅记录保存在BORROW_SAVE表中（注

ORROW_SAVE表结构同BORROW表）。

create table borrow_test as select * from borrow where 1=2;
create or replace trigger borrowsj
after insert on borrow
for each row
begin
if :new.bno=6 then
insert into borrow_save values (:new.cno,:new.bno,:new.rdate);
end if;
end;
 /

12).建立一个视图，显示"力01"班学生的借书信息（只要求显示姓名和书名）。

create or replace view show_view as select name 姓名,bname 书名 from

card,books,borrow where card.class='c01' and borrow.cno=card.cno and

borrow.bno=books.bno order by 书名;

13).查询当前同时借有"水浒传"和"西游记"两本书的读者，输出其借书卡号，

并按卡号升序排序输出。

select * from (select card.cno from card,books,borrow where

card.cno=borrow.cno and borrow.bno=books.bno and bname='西游记'

intersect select card.cno from card,books,borrow where

borrow.bno=books.bno and card.cno=borrow.cno and bname='水浒传')

order by cno asc;

14).假定在建BOOKS表时没有定义主码，写出为BOOKS表追加定义主码的语句。

alter table books add constraint bno primary key;

15).对CARD表做如下修改：

?    a.将NAME最大列宽增加到10个字符（假定原为6个字符）。

   alter table card modify name varchar(10);

?    b.为该表增加1列system（系名），可变长，最大20个字符。

   alter table card add(system nvarchar2(20));



# sql常见问题

## 分组取前几

t_test表结构
![](/images/mysql/1637812142609-117.png)

```sql
# 每科分数取前2名学生姓名
SELECT *
 FROM t_test t1
WHERE
(
SELECT count(id) FROM t_test t2
where t2.stuid=t1.stuid -- 分组的字段
and t2.score<t1.score  -- 分组内部排序的字段大于是desc，小于是asc
)<2                -- 每组取几个数据
ORDER BY stuid asc,score DESC -- 结果排序的字段
```

## mysql密码重置

跳过密码验证
vi /etc/my.cnf
[mysqld]
skip-grant-tables

重启mysqld
systemctl restart mysqld

无密码进入mysql命令框
mysql -uroot

进入mysql数据库
use mysql;

修改root密码为root(*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B)为设置密码root
delete from user where user = 'root';
INSERT INTO `user` VALUES ('localhost', 'root', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', 'Y', '', '', '', '', 0, 0, 0, 0, 'mysql_native_password', '*81F5E21E35407D884A6CD4A731AEBFB6AF209E1B', 'N', '2018-12-10 13:21:57', NULL, 'N');
update user set host = '%' where user = 'root';
update user set account_locked="N" where user="root" or Host="localhost" or host="127.0.0.1";

关闭跳过密码验证
vi /etc/my.cnf
[mysqld]
#skip-grant-tables

重启mysqld
systemctl restart mysqld
mysql -u root -p
root

设置整个表和字段的编码格式

```sql
ALTER TABLE read_logs_views CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 死锁解决：

```sql
#查看正在进行中的事务
SELECT * FROM information_schema.INNODB_TRX;
#查看正在锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
#查看等待锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
#查询是否锁表
SHOW OPEN TABLES where In_use > 0;
#杀死进程
kill id

#查看最近死锁的日志
show engine innodb status;
#查看当前进程
show full PROCESSLIST;

```

查询数据库字段和备注

```sql
select COLUMN_NAME,COLUMN_COMMENT from information_schema.COLUMNS where table_name = 'customers' and table_schema = 'vooglam'
```

## mysql优化：

[整理的文档](https://kdocs.cn/l/ctGX99e37KRA)

[MySQL索引原理及慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)

[Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)





















