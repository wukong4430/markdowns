# 1、创建表

## 外键

```sql
create table course (

cno varchar(20) NOT NULL Primary key,
cname varchar(20) not null ,
tno varchar(20) not null,
foreign key(tno) references teacher(tno)

)Engine=Innodb Default CHARSET=utf8;
```



## 复合主键

```sql
create table score (

sno varchar(20) NOT NULL,
cno varchar(20) not null ,
degree decimal(4,1) ,
foreign key(sno) references student(sno),
foreign key(cno) references course(cno),
primary key (sno,cno)

)Engine=Innodb Default CHARSET=utf8;
```



## 字段唯一约束

一个表中可以有多个unique，但只能有一个主键。在唯一性约束功能上，两者一致。

```sql
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (Id_P)
)
```



# 2、查询

## 两个顺序



- 写的顺序：select ... from... where.... group by... having... order by..
- 执行顺序：from... where...group by... having.... select ... order by...



## 不重复

distinct

```sql
select distinct Depart from teacher;
```

## 

## 介于区间

```sql
select * from Score where Degree between 60 and 80;
```

```sql
select * from Score where Degree>=60 and Degree<=80;
```



## 多个Order by

```sql
select * from Score order by Cno asc,Degree desc;
```



## 选最高项对应的其他字段 (排序或者子查询)

```sql
select Sno,Cno from Score order by Degree desc limit 0,1;
```

```sql
select Sno,Cno from Score where Degree = (select max(Degree) from Score);
```



## 有where的查询，还有group by的查询 （牢记查询的顺序）

select ...from ...where ... group by ... having ...

查询Score表中至少有5名学生选修的并以3开头的课程的平均分数。

```sql
select cno, avg(degree)
from score
where cno like '3-%'
group by cno
having count(sno)>=3;
```





## 两表查询 & 内连接

查询所有学生的Sname、Cno和Degree列。

```sql
select s.sname, c.cno, c.degree
from score c, student s
where c.sno = s.sno;
```

```sql
select s.sname, c.cno, c.degree
from score c inner join student s
on c.sno = s.sno;
```

注意：

- 多表查询用逗号分隔
- inner join 的关键字用的是on，不是where。



## 多表查询 （原理跟2表一样）

查询所有学生的Sname、Cname和Degree列

```sql
select s.sname, sc.cno, c.cname, sc.degree
from student s, score sc, course c
where s.sno = sc.sno and c.cno = sc.cno;
```





## 多表查询

查询“95033”班学生的平均分。

- 子查询

```sql
select avg(degree)
from score
where sno in
(select sno
from student
where class='95033');
```

- 多表

```sql
select avg(degree) from score,student where student.sno=score.sno and class='95033';
```

- 内连接

```sql
select avg(Degree) from Score inner join Student on Student.Sno=Score.Sno where Class='95033';
```

## 

```sql
select distinct Depart from teacher;
```





# 3、函数的应用



## 3.1 year(date) month(date) day(date)

查询和学号为108、101的同学同年出生的所有学生的Sno、Sname和Sbirthday列

```sql
select 
	sno, sname, sbirthday
from 
	student
where 
	year(sbirthday) in 
(select 
 	year(sbirthday)
from 
 	student
where 
 	sno in ('101', '108')
);
```

year(date) 范围 1000~9999，不支持索引。



## 3.2 Any All

- 查询选修编号为“3-105“课程且成绩**至少高于**选修编号为“3-245”的同学的Cno、Sno和Degree,并按Degree从高到低次序排序。

    至少高于用Any

    ```sql
    select Cno,Sno,Degree from Score where Cno = '3-105' and Degree >
    any(select Degree from Score where Cno = '3-245') order by Degree desc;
    
    select Cno,Sno,Degree from Score where Cno = '3-105' and Degree >
    (select min(Degree) from Score where Cno = '3-245') order by Degree desc;
    #大于任何一个或者大于最小的
    ```

- 查询选修编号为“3-105”且成绩高于选修编号为“3-245”课程的同学的Cno、Sno和Degree。

    ```sql
    select Cno,Sno,Degree from Score where Cno = '3-105' and Degree > all(select Degree from Score where Cno = '3-245');
    ```



## 3.3 union

联合查询，并集。

- 查询所有教师和同学的name、sex和birthday。

    ```sql
    select Sname as '学生',Ssex,Sbirthday from Student
    union
    select Tname as '老师',Tsex,Tbirthday from Teacher;
    ```

- 查询所有“女”教师和“女”同学的name、sex和birthday。

    ```sql
    select Sname,Ssex,Sbirthday from Student where Ssex = '女'
    union
    select Tname,Tsex,Tbirthday from Teacher where Tsex = '女';
    ```

    























# N、引擎

|              | MYISAM       | INNODB          |
| ------------ | ------------ | --------------- |
| 事务         | 不支持       | 支持            |
| 数据行锁定   | 不支持（锁表 | 支持            |
| 外键约束     | 不支持       | 支持            |
| 全文索引     | 支持         | 不支持          |
| 表空间的大小 | 较小         | 较大 约等于 2倍 |

常规使用操作：

- MYISAM 节约空间 速度较快
- INNODB 安全性高，事务处理，多表用户操作



> 在物理空间存在的位置

所有的数据库文件都存在MySQL 的data目录下， 一个文件夹就对应一个数据库。



MySQL引擎在物理文件上的区别

- InnoDB在数据库表中只有一个*.frm文件，以及上级目录下的idbdata1文件
- MyISAM对应文件
    - *.frm	表结构的定义文件
    - *.MYD  数据文件 （data）
    - *.MYI    索引文件 （index）