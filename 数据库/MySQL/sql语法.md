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

## 

## 多个Order by

```sql
select * from Score order by Cno asc,Degree desc;
```

## 

## 选最高项对应的其他字段 (排序或者子查询)

```sql
select Sno,Cno from Score order by Degree desc limit 0,1;
```

```sql
select Sno,Cno from Score where Degree = (select max(Degree) from Score);
```

## 

## 有where的查询，还有group by的查询 （牢记查询的顺序）

select ...from ...where ... group by ... having ...

```sql
select cno, avg(degree)
from score
where cno like '3-%'
group by cno
having count(sno)>=3;
```

## 

```sql
select distinct Depart from teacher;
```

## 

```sql
select distinct Depart from teacher;
```

## 

```sql
select distinct Depart from teacher;
```

## 

```sql
select distinct Depart from teacher;
```

## 

```sql
select distinct Depart from teacher;
```

## 

```sql
select distinct Depart from teacher;
```

## 

```sql
select distinct Depart from teacher;
```

