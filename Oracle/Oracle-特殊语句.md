1. Oracle分页

   select * from
   (
   select a.* from, rownum as rn from
   (
   select * from tablename where 1=1 and… order by createtime desc
   ) a where rownum <= limit
   ) where rn > start

2. 