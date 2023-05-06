
## 1. 表

建空间表语句

```sql
CREATE TABLE webgis_occ1 (
  objectid bigserial NOT NULL,
  geom geometry(POINT, 3857) NULL,    --- LINESTRING/POLYGON
  CONSTRAINT webgis_occ1_pkey PRIMARY KEY (objectid)
);
CREATE INDEX sidx_webgis_occ1_geom ON public.webgis_occ1 USING gist (geom);
```

或者

```sql
ALTER TABLE poi ADD COLUMN geom_p_alter geometry(POINT,4326);
```

## 2. 数据

```sql
-- 插入线
insert into webgis_occ1(objectid, geom) values(1, ST_GeomFromText('LINESTRING(12 23, 26 30)', 3857));
-- 插入面
insert into webgis_occ1(objectid, geom) values(1, ST_GeomFromText('POLYGON((10 20, 20 40, 50 15, 10 20))', 3857));
```

## 3. 常用GEOMETRY操作方法

*   ST\_UNION    合并对象，比如合并点，可以合并为多点
    `ST_Union(geom)`
*   ST\_ConvexHull    计算多点，然后得到其外部最小凸多边形
    `ST_ConvexHull(ST_Union(geom))`
*   ST\_Buffer    计算面的缓冲区，容限为地图单位\
    `ST_BUFFER(ST_ConvexHull(ST_Union(geom)), 100)`
*   ST\_TRANSFROM    投影转换

```sql
ALTER TABLE mytable 
  ALTER COLUMN geom 
  TYPE Geometry(Point, 32644) 
  USING ST_Transform(geom, 32644);
```

## 4. 数据库

*   导出数据库

    *   方式一
        `pg_dump -U postgres -f c:\db.sql postgis`

    *   方式二
        `pg_dump -U postgres postgis > c:\db.sql`

*   导入数据库
    `psql -d postgis -f c:\db.sql postgres`

*   导出具体表
    `pg_dump -Upostgres -t mytable -f dump.sql postgres`

*   导入具体表：
    `psql -d postgis -f c:\ dump.sql postgres`

