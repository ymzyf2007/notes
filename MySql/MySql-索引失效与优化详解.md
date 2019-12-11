---
typora-root-url: assets
---

##### MySql-索引失效与优化详解

###### 案例所用的表结构、索引与数据如下：

~~~mysql
CREATE TABLE `staffs` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(16) NOT NULL COMMENT '用户名',
  `age` int(4) NOT NULL COMMENT '年龄',
	`pos` varchar(32) NOT NULL COMMENT '岗位',
  `add_time` datetime DEFAULT NULL COMMENT '加入时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='员工表';

alter table staffs add index idx_name_age_addtime (name, age, pos);
show index from staffs;

staffs	0	PRIMARY	1	id	A	3				BTREE		
staffs	1	idx_name_age_addtime	1	name	A	3				BTREE		
staffs	1	idx_name_age_addtime	2	age	A	3				BTREE		
staffs	1	idx_name_age_addtime	3	pos	A	3				BTREE		

insert into staffs(name, age, pos, add_time) values('zhangsan', 32, 'manager', now());
insert into staffs(name, age, pos, add_time) values('lisi', 25, 'dev', now());
insert into staffs(name, age, pos, add_time) values('wangwu', 24, 'dev', now());

select * from staffs;
1	zhangsan	32	manager	2019-12-10 10:10:46
2	lisi	25	dev	2019-12-10 10:10:46
3	wangwu	24	dev	2019-12-10 10:10:46
~~~

###### 索引失效与优化

**1、全值匹配我最爱**

~~~mysql
explain select * from staffs where name='zhangsan';
~~~

![](/全值匹配我最爱a.jpg)

~~~mysql
explain select * from staffs where name='zhangsan' and age=32;
~~~

![](/全值匹配我最爱ab.jpg)

~~~mysql
explain select * from staffs where name='zhangsan' and age=32 and pos='manager';
~~~

![](/全值匹配我最爱abc.jpg)

**2、最左前缀要遵守（带头索引[兄弟]不能死，中间索引[兄弟]不能断）**

如果索引多个列，要遵守最佳左前缀法则。指的是查询从索引的最左列开始并且不跳过中间的列，正确的示例参考上图。

错误的示例：

带头索引[大哥]死：

~~~mysql
explain select * from staffs where age=32 and pos='manager';
~~~

![](/带头大哥死.jpg)

中间索引[兄弟]断(带头索引生效，其它索引失效):

~~~mysql
explain select * from staffs where name='zhangsan' and pos='manager';
~~~

![](/中间兄弟断了.jpg)

**3、索引列上少计算（不要在索引列上做任何操作，包括计算、函数、自动/手动类型转换，不然会导致索引失效而转向全表扫描）**

~~~mysql
explain select * from staffs where left(name,4)='zhangsan';
~~~

![](/索引列上少运算.jpg)

**4、范围之后全失效（MySql存储引擎不能继续使用索引中范围条件between、<、>、in等）右边的列**

~~~mysql
explain select * from staffs where name='zhangsan' and age>30 and pos='manager';
~~~

![](/范围之后全失效.jpg)

**5、LIKE百分写最右，索引覆盖不写星（索引字段使用like以通配符['%字符串']开头时，会导致索引失效而转向全表扫描）**

~~~mysql
explain select * from staffs where name like '%zhangsan';
~~~

![](/LIKE百分写最右.jpg)

注：like百分写在最左边，索引失效导致全表扫描。

~~~mysql
explain select * from staffs where name like 'zhangsan%';
~~~

![](/LIKE百分写最右情况-走索引.jpg)

**问题：解决like ‘%字符串%’时，索引失效问题的方法？ 答案：使用索引覆盖可以解决**

~~~mysql
explain select id,name,age from staffs where name like '%zhangsan%';
~~~

![](/索引覆盖不写星.jpg)

**6、空值不等还有or，索引失效要少用（索引字段上使用!=、is null/is not null、还有or判断时，会导致索引失效而转向全表扫描）**

索引列上用不等于：

~~~mysql
explain select * from staffs where name !='zhangsan';
~~~

![](/索引列上使用不等于运算符.jpg)

索引列上用is not null：

~~~mysql
explain select * from staffs where name is not null;
~~~

![](/索引列上使用is_not_null.jpg)

索引列上用or：

~~~mysql
explain select * from staffs where name ='zhangsan' or name='lisi';
~~~

![](/索引列上使用or.jpg)

###### 小总结

假设index(a,b,c)

| Where语句                                               | 索引是否被使用                     |
| ------------------------------------------------------- | ---------------------------------- |
| where a = 3                                             | Y，使用到a                         |
| where a = 3 and b = 5                                   | Y，使用到a，b                      |
| where a = 3 and b = 5 and c = 4                         | Y，使用到a，b，c                   |
| where b = 3 或者 where b = 3 and c = 4 或者 where c = 4 | N                                  |
| where a = 3 and c = 4                                   | 使用到a，但是c不可以，b中间断了    |
| where a = 3 and b > 5 and c = 4                         | 使用到a和b，c不能在范围之后，b断了 |
| where a = 3 and b like 'kk%' and c = 4                  | Y，使用到a，b，c                   |
| where a = 3 and b like '%kk' and c = 4                  | Y，只用到a                         |
| where a = 3 and b like '%kk%' and c = 4                 | Y，只用到a                         |
| where a = 3 and b like 'k%kk%' and c = 4                | Y，使用到a，b，c                   |

**优化总结**

全值匹配我最爱，最左前缀要遵守；

带头大哥不能死，中间兄弟不能断；

索引列上少计算，范围之后全失效；

LIKE百分写最右，覆盖索引不写星；

不等空值还有or，索引失效要少用；



