
```sql
-- 查询当前连接数:
select count(1) from pg_stat_activity;

-- 查询最大连接数
show max_connections;

-- 查询版本
select version();

-- 查询postgis版本
SELECT PostGIS_full_version();

-- 设置Postgres 用户默认的Schema
ALTER USER postgres SET search_path to "$user", public, sde;

--导出数据库：
---方式一
pg_dump -U postgres -f c:\db.sql postgis
---方式二：
pg_dump -U postgres postgis > c:\db.sql

--导入数据库：
psql -d postgis -f c:\db.sql postgres

--导出具体表：
pg_dump -Upostgres -t mytable -f dump.sql postgres

--导入具体表：
psql -d postgis -f c:\ dump.sql postgres
```

