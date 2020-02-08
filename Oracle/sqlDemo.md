~~~mysql
--1、创建学生表
create table sdemo.student
(
    sno varchar(32),
    sname varchar(128),
    age int,
    sex varchar(10),
    primary key(sno)
)
insert into sdemo.student(sno,sname,age,sex) values('000001','name1','23','nan'),('000002','name2','23','nv'),('000003','name3','23','nan'),('000004','name4','23','nan'),
('000005','name5','23','nan'),('000006','name6','23','nan'),('000007','name7','23','nv'),('000008','name8','23','nan'),('000009','name9','23','nv'),('000010','name10','23','nan'),
('000011','name11','23','nan'),('000012','name12','23','nv'),('000013','name13','23','nan'),('000014','name14','23','nv'),('000015','name15','23','nan'),('000016','name16','23','nv')

--2、创建教师表
create table sdemo.teacher
(
    tno varchar(20),
    tname varchar(128),
    primary key(tno)
)
insert into sdemo.teacher(tno,tname) values('1001','tname1'),('1002','tname2'),('1003','tname3'),('1004','tname4'),('1005','tname5'),('1006','tname6'),('1007','tname7'),('1008','tname8'),
('1009','tname9'),('1010','tname10'),('1011','tname11')

--3、创建课程表
create table sdemo.course
(
    cno varchar(20),
    cname varchar(128),
    tno varchar(20),
    primary key(cno),
    foreign key(tno) references teacher(tno)
)
insert into sdemo.course(cno,cname,tno) values('101','shuxue','1009'),('102','chinese','1002'),('103','english','1005'),('104','wuli','1010'),('105','huaxue','1006'),
('106','lishan','1001'),('107','computer','1007')

--4、创建成绩表
create table sdemo.sc
(
    sno varchar(20),
    cno varchar(20),
    score float,
    primary key(sno,cno),
    foreign key(sno) references student(sno),
    foreign key(cno) references course(cno)
)
insert into sdemo.sc(sno,cno,score) values('000001','101','84'),('000002','101','65'),('000004','101','93'),('000006','101','67'),('000007','101','46'),
('000008','101','78'),('000009','101','87'),('000010','101','79'),('000011','101','80'),
('000012','101','82'),('000014','101','67'),('000016','101','95'),
('000001','102','74'),('000003','102','65'),('000004','102','80'),('000005','102','73'),('000007','102','76'),
('000008','102','80'),('000009','102','83'),('000010','102','83'),('000011','102','85'),
('000012','102','89'),('000013','102','85'),('000014','102','45'),('000015','102','81'),('000016','102','75')

1、查询“101”课程比“102”课程成绩高的所有学生的学号
select a.sno from 
(select sno,score from sdemo.sc where cno='101') a,
(select sno,score from sdemo.sc where cno='102') b where a.sno=b.sno and a.score>b.score

2、查询平均成绩大于60分的同学的学号和平均成绩
select sno,avg(score) from sdemo.sc group by sno having avg(score)>60

3、查询所有同学的学号、姓名、选课数、总成绩
select a.sno,a.sname,count(b.sno),sum(b.score) from sdemo.student a left join sdemo.sc b on a.sno=b.sno group by b.sno

4、查询姓“yu”的老师的个数
select count(*) from sdemo.teacher where tname like 'yu%'

5、查询没学过“tname9”老师课的同学的学号、姓名
--先查找全部学过“tname9”老师课的同学的学号
select sno,sname from sdemo.student where sno not in 
(select sno from sdemo.sc a,sdemo.course b,sdemo.teacher c where a.cno=b.cno and b.tno=c.tno and c.tname='tname9')

6、查询学过“101”并且也学过编号“102”课程的同学的学号、姓名
select t3.sno,t3.sname from sdemo.sc t1,sdemo.student t3 where t1.cno='101' and exists(select * from sdemo.sc t2 where t2.cno='102') and t1.sno=t3.sno

7、查询学过“tname9”老师所教的所有课的同学的学号、姓名； 
select t4.sno,t4.sname from sdemo.sc t1,sdemo.course t2,sdemo.teacher t3,sdemo.student t4 where t1.cno=t2.cno and t1.sno=t4.sno and t2.tno=t3.tno and t3.tname='tname9'

8、查询课程编号“101”的成绩比课程编号“102”课程低的所有同学的学号、姓名； 
select c.sno,c.sname from 
(select * from sdemo.sc where cno='101') a,
(select * from sdemo.sc where cno='102') b,sdemo.student c where a.sno=b.sno and a.score<b.score and a.sno=c.sno

9、查询所有课程成绩小于60分的同学的学号、姓名； 
select distinct(t2.sno),t2.sname from sdemo.sc t1,sdemo.student t2 where t1.sno=t2.sno and t1.score<60
~~~

