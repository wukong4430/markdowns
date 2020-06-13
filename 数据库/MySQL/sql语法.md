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

