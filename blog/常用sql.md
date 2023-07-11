1.  分组,.取组内最新的一条数据

    下面sql就是标识取t表中根据type分类之后，每一组中update_time最大的一条数据

   ```sql
   select * from t a   where a.update_time in ( select max(update_time)  FROM t b  where a.type = b.type group by b.type)
   ```

   未防止组内可能有两条update_time相等且最大,可以查询之后再次分组

   ```
   select * from t a   where a.update_time in ( select max(update_time)  FROM t b  where a.type = b.type group by b.type) group by a.update_time
   ```

   