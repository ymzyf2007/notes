##### 1、查询一个表中有重复数据的SQL语句（即是ID重复）

select * from tableName where ID in (select ID from tableName group by ID having count(ID) > 1);

##### 2、分页查询语句

select * from 

(

select a.*, rownum rn from 

(select * from tableName where columnName='' order by columnTime desc) a where rownum <= limit

) where rn > start;

##### 3、查看当前不为空的连接

select * from v$session where username is not null;

##### 4、查看不同用户的连接数

select username, count(username) from v$session where username is not null group by username;

##### 5、连接数

select count(*) from v$session;

##### 6、并发连接数

select count(*) from v$session where status='ACTIVE';

##### 7、最大连接

show parameter processes;

##### 8、查询表有多少个字段

* oracle查询方法

  select count(*) from user_tab_columns where table_name='表名';

* mysql查询方法

  select count(*) from information_schema.COLUMNS where TABLE_SCHEMA='库名' and TABLE_NAME='表名';

