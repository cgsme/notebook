# Postgresql

参考[常用操作](https://www.cnblogs.com/kaituorensheng/p/4667160.html)

- 连接pg
  
        psql -h hostname -p port -U username -d dbname

- 退出pg

        \q

- 退出操作

        q

- 列出所有数据库

        \l

- 切换数据库

        \c dbname

- 查看指定表里的所有字段

        \c tablename

- 查看当前数据库中所有的表

        \d

- 查看指定表的基本情况

        \d+ tablename

- 建立索引

1、单字段索引

    CREATE INDEX index_name ON table_name (field1);

2、多字段索引

    CREATE INDEX index_name ON table_name (field1,field2);

- 查看sql执行时间
  
        select * from pg_stat_statements order by total_time desc limit 5;
