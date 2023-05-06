## 一、建立表空间

```sql
CREATE TABLESPACE OSS_DATA DATAFILE
'/oradata/eodn/oss_data01.dbf' SIZE 2048M AUTOEXTEND OFF,
'/oradata/eodn/oss_data02.dbf' SIZE 2048M AUTOEXTEND OFF,
'/oradata/eodn/oss_data03.dbf' SIZE 2048M AUTOEXTEND OFF
LOGGING
ONLINE
EXTENT MANAGEMENT LOCAL AUTOALLOCATE
BLOCKSIZE 8K
SEGMENT SPACE MANAGEMENT AUTO
FLASHBACK ON;

CREATE TABLESPACE OSS_INDEX DATAFILE
'/oradata02/eodn/oss_index01.dbf' SIZE 2048M AUTOEXTEND OFF
LOGGING
ONLINE
EXTENT MANAGEMENT LOCAL AUTOALLOCATE
BLOCKSIZE 8K
SEGMENT SPACE MANAGEMENT AUTO
FLASHBACK ON;

CREATE TABLESPACE BASE_DATA DATAFILE
'/oradata02/eodn/base_data01.dbf' SIZE 2048M AUTOEXTEND OFF,
'/oradata02/eodn/base_data02.dbf' SIZE 2048M AUTOEXTEND OFF,
'/oradata02/eodn/base_data03.dbf' SIZE 2048M AUTOEXTEND OFF,
'/oradata02/eodn/base_data04.dbf' SIZE 2048M AUTOEXTEND OFF
LOGGING
ONLINE
EXTENT MANAGEMENT LOCAL AUTOALLOCATE
BLOCKSIZE 8K
SEGMENT SPACE MANAGEMENT AUTO
FLASHBACK ON;
```

## 二、 建立用户

```sql
CREATE USER ossplan
IDENTIFIED BY "123"
DEFAULT TABLESPACE OSS_DATA
TEMPORARY TABLESPACE TEMP
PROFILE DEFAULT
ACCOUNT UNLOCK;

CREATE USER deveodn
IDENTIFIED BY "123"
DEFAULT TABLESPACE BASE_DATA
TEMPORARY TABLESPACE TEMP
PROFILE DEFAULT
ACCOUNT UNLOCK;

grant dba,connect,resource to ossplan;
grant EXECUTE on DBMS_LOB to ossplan;
grant dba,connect,resource to deveodn;
grant EXECUTE on DBMS_LOB to deveodn;
```

## 三、 导入导出

```sql
-- 通用语句
expdp otpoc/smart directory=LIUJUN dumpfile=otpoc_20140903.dmp logfile=otpoc_20140903.log 
impdp otpoc/smart directory=LIUJUN dumpfile=otpoc_20140903.dmp logfile=otpoc_20140903_imp.log 

-- 要在同一个oracle实例里进行导入导出
impdp testplan2/smart directory=LIUJUN dumpfile=singleplan.dmp logfile=singleplan.log remap_schema=singleplan:testplan2

-- 查看目录
select * from dba_directories;

-- 新建目录
create directory LIUJUN as '/opt/oracle/oradata/bak';
grant read, write on directory LIUJUN to inne_pm;

-- 更改密码
alter user eodn_gis identified by 123;

-- 取消大小写限制
ALTER SYSTEM SET SEC_CASE_SENSITIVE_LOGON = FALSE;

-- 查询数据库字符集
select userenv('language') from dual;

-- 查询临时表空间状态
select tablespace_name,file_name,bytes/1024/1024 file_size,autoextensible from dba_temp_files;

-- 扩展临时表空间
-- 增大临时文件大小：
alter database tempfile ‘/u01/app/oracle/oradata/orcl/temp01.dbf’ resize100m;

-- 将临时数据文件设为自动扩展：
alter database tempfile ‘/u01/app/oracle/oradata/orcl/temp01.dbf’ autoextend on next 5m maxsize unlimited;
```

## 四、 游标相关

```SQL
-- 查看数据库当前的游标数配置
SHOW PARAMETER OPEN_CURSORS;

-- 查看游标使用情况
SELECT O.SID, OSUSER, MACHINE, COUNT(*) NUM_CURS
FROM V$OPEN_CURSOR O, V$SESSION S
WHERE USER_NAME = 'DEVEODN' AND O.SID=S.SID
GROUP BY O.SID, OSUSER, MACHINE
ORDER BY NUM_CURS DESC;

-- 查看游标的SQL执行情况
SELECT O.SID, Q.SQL_TEXT
FROM V$OPEN_CURSOR O, V$SQL Q
WHERE Q.HASH_VALUE=O.HASH_VALUE AND O.SID = 123;

-- 增大游标数
ALTER SYSTEM SET OPEN_CURSORS=2000 SCOPE=BOTH;
```

## 五、死锁

```sql
-- 查看死锁
SELECT OBJECT_NAME, OS_USER_NAME ,S.SID,S.SERIAL#,S.STATUS,S.LAST_CALL_ET,
TO_CHAR(SYSDATE - S.LAST_CALL_ET/3600/24,'YYYYMMDD HH24:MI:SS' ) "LAST CALL"
FROM V$LOCKED_OBJECT L , DBA_OBJECTS O , V$SESSION S , V$PROCESS P
WHERE L.OBJECT_ID=O.OBJECT_ID AND L.SESSION_ID=S.SID AND S.PADDR=P.ADDR;

-- 删除死锁的SESSION
ALTER SYSTEM KILL SESSION '21,73'; --SID AND SERIA# ABOVE
```

## 六、其余

```sql
-- 数据库表分析，构建索引
analyze table trm_physical_link compute statistics;

-- 恢复被误操作的表数据
select * from TABLE_NAME as of timestamp(systimestamp -interval'72000'second);

-- 查找所有关联的表
SELECT
a.table_name, a.column_name
FROM
user_cons_columns a,
user_constraints b
WHERE
a.constraint_name = b.constraint_name AND
b.constraint_type = 'R' AND
a.column_name = 'AREA_ID';
```