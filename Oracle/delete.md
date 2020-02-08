##### 1、删除一个表中重复的数据（即是ID重复的数据）

delete from tableName A where rowid != (select max(rowid) from tableName B where A.ID=B.ID)

