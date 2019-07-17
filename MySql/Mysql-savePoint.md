##### 业务场景：某转账业务有 A B C D 四个事务。需求：AB（必须），CD（可选）。

**代码实现如下:**

```java
Connection conn = null;
Savepoint savepoint = null;  // 保存点，记录操作的当前位置，之后可以回滚到指定的位置。（可以回滚一部分）
try {
  // 1 获得连接
  conn = ...;
  // 2 开启事务
  conn.setAutoCommit(false);
  A
  B
  savepoint = conn.setSavepoint();   // 保存节点
  C
  D
  // 3 提交事务
  conn.commit();
} catch(Exception e) {
  if(savepoint != null) {   //CD异常
     // 回滚到CD之前
     conn.rollback(savepoint);
     // 提交AB
     conn.commit();
  } else {   // AB异常
     // 回滚AB
     conn.rollback();
  }
}
```

savePoint的应用：

​	（上图AD和BC代表两个事务，1，2，3代表事务执行的三个阶段。图简陋了点，有点像“金箍棒”）

​	使用嵌套事务的场景有两点需求：

1.需要事务BC与事务AD一起commit，即：作为事务AD的子事务，事务BC只有在事务AD成功commit时（阶段3成功）才commit。这个需求简单称之为“联合成功”。这一点PROPAGATION_REQUIRED可以做到。

2.需要事务BC的rollback不（无条件的）影响事务AD的commit。这个需求简单称之为“隔离失败”。这一点PROPAGATION_REQUIRES_NEW可以做到。

​	使用PROPAGATION_REQUIRED满足需求1，但子事务BC的rollback会无条件地使父事务AD也rollback，不能满足需求2。

​	使用PROPAGATION_REQUIRES_NEW满足需求2，但子事务（这时不应该称之为子事务）BC是完全新的事务上下文，父事务（这时也不应该称之为父事务）AD的成功与否完全不影响BC的提交，不能满足需求1。

​	同时满足上述两条需求就要用到PROPAGATION_NESTED了。PROPAGATION_NESTED在事务AD执行到B点时，设置了savePoint（关键）。

​	当BC事务成功commit时，PROPAGATION_NESTED的行为与PROPAGATION_REQUIRED一样。只有当事务AD在D点成功commit时，事务BC才真正commit，如果阶段3执行异常，导致事务AD rollback，事务BC也将一起rollback ，从而满足了“联合成功”。

​	当阶段2执行异常，导致BC事务rollback时，因为设置了savePoint，AD事务可以选择与BC一起rollback或继续阶段3的执行并保留阶段1的执行结果，从而满足了“隔离失败”。

​	强调，补充一点：PROPAGATION_NESTED只是Spring针对JDBC3.0以上版本SavePoint机制的一个事务传播机制的扩展，J2EE体系中是没有的，所以如果应用中使用JTA作为底层的事务管理机制的话，使用Spring也是不可能支持PROPAGATION_NESTED。不过JPA的体系中好像是有SavePoint的机制（还没有细研究过），Spring应该可以在之上做相应的支持。这点有待进一步研究！

