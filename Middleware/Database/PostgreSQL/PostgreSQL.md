# Postgresql

- 查看sql执行时间
  
        select * from pg_stat_statements order by total_time desc limit 5;
