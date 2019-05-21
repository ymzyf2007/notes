cmd连接本地
mysql -u root -p
然后输入密码即可


##创建数据库
create database ym_test;

show databases;

##使用数据库
use ym_test;

##查看数据库表
show tables;

##创建表
create table account(
    id int primary key auto_increment,
    name varchar(40),
    money float
);

##显示表结构
describe|[或者desc] account;

##删除表
drop table account;

##删除数据库
drop database ym_test;

##插入数据
insert into account(name, money) values('A', 1000);
insert into account(name, money) values('B', 1000);
insert into account(name, money) values('C', 1000);

start transaction;    // 开启MySQL数据库的事务
我们首先在数据库中模拟转账失败的场景，首先执行update语句让A用户的money减少100块钱，如下图所示：
update account set money=money-100 where name='A';
然后我们关闭当前操作的dos命令窗口，这样就导致了刚才执行的update语句的数据库的事务还没有提交，
那么我们对A用户的修改就不算是真正的修改了，下次再查询A用户的money时，依然还是之前的1000，如下图所示：

##提交事务
start transaction;
commit;

##回滚事务
rollback;

##MySql数据库查询当前事务隔离级别【默认是REPEATABLE-READ（可重复读）】
select @@tx_isolation;

##设置事务隔离级别为read-uncommitted(读未提交)；会引发脏读、不可重复读和虚读
set tx_isolation='read-uncommitted';
##设置事务隔离级别为read-committed(已提交读)；不会引发脏读、但不可重复读和会出现虚读
set tx_isolation='read-committed';
##设置事务隔离级别为repeatable-read(可重复读)；不会引发脏读，也可重复读，但是会出现虚读
还有个隔离级别是序列化，不会引发脏读、可重复读、不会出现虚读

##设置全局的隔离级别
set global transaction isolation level read committed;
##设置当前会话
set session transaction isolation level read committed;