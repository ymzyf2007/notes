1.mysql查找重复的字段,并删除记录只保留一条

实例：user表中含字段id, name, age, address,请用sql语句删除掉name、age、address字段重复的数据（仅保留一条）

```mysql
insert into user(id, name, age, address) values(1, 'ym', 28, 'testaddress');
insert into user(id, name, age, address) values(2, 'yf', 28, 'testaddress');
insert into user(id, name, age, address) values(3, 'yf', 28, 'testaddress');
insert into user(id, name, age, address) values(4, 'yf', 28, 'testaddress');
insert into user(id, name, age, address) values(5, 'yf', 28, 'testaddress');
```

using用法：

```mysql
select * from user t1 left join user t2 on t1.id=t2.id;
```

<==>

```mysql
select * from user t1 left join user t2 using(id);
```

```mysql
delete from a using user a, user b where a.id<b.id and a.name = b.name and a.age=b.age and a.address=b.address;
```

2.更新某个表字段，值需要另一个表里面的字段

```mysql
UPDATE guide_order_tracking, guide_order_sendinfo   
SET guide_order_tracking.request_id=guide_order_sendinfo.request_id  
WHERE guide_order_tracking.order_no=guide_order_sendinfo.order_no and guide_order_sendinfo.order_no in
('6323404357914963968');
```

3.MySql分页语句

```mysql
select * from table limit 5,10;  // 检索记录行 6-15
```

4.MySql5添加外键约束错误解决方法

​		当添加MySQL表之间外键约束关系的时候，常常会发生这样的错误：Error Code : 1005

​		问题还是不能得到解决，经过一番探索，终于找到了问题所在，当发生此类的错误的时候，从三个角度入手：

1）、确保主表有主键

2）、确保主从表数据引擎为InnoDB类型

3）、确定从表外键字段类型与主表一致。(尤其检查下Integer有无符号是否一致)

ps：MyISAM是不支持外键的。

