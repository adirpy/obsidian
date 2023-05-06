
1. 无ORDER BY排序的写法。(效率最高)(经过测试，此方法成本最低，只嵌套一层，速度最快！即使查询的数据量再大，也几乎不受影响，速度依然！)

```
SELECT * FROM (Select ROWNUM AS ROWNO, T.* from k_task T where Flight_date between to_date('20060501', 'yyyymmdd') and to_date('20060731', 'yyyymmdd') AND ROWNUM <= 20) TABLE_ALIAS WHERE TABLE_ALIAS.ROWNO >= 10;
```

2. 有ORDER BY排序的写法。(效率最高)(经过测试，此方法随着查询范围的扩大，速度也会越来越慢哦！)

```
SELECT * FROM (SELECT TT.*, ROWNUM AS ROWNO FROM (Select t.*from k_task T where flight_date between to_date('20060501', 'yyyymmdd') and to_date('20060531', 'yyyymmdd') ORDER BY FACT_UP_TIME, flight_no) TT WHERE ROWNUM <= 20) TABLE_ALIAS where TABLE_ALIAS.rowno >= 10;
```

3. 无ORDER BY排序的写法。(建议使用方法1代替)(此方法随着查询数据量的扩张，速度会越来越慢哦！)

```
SELECT * FROM (Select ROWNUM AS ROWNO, T.* from k_task T where Flight_date between to_date('20060501', 'yyyymmdd') and to_date('20060731', 'yyyymmdd')) TABLE_ALIAS WHERE TABLE_ALIAS.ROWNO <= 20 AND TABLE_ALIAS.ROWNO >= 10;--TABLE_ALIAS.ROWNO between 10 and 100;
```

4. 有ORDER BY排序的写法.(建议使用方法2代替)(此方法随着查询范围的扩大，速度会越来越慢哦！)

```
SELECT * FROM (SELECT TT.*, ROWNUM AS ROWNO FROM (Select * from k_task T where flight_date between to_date('20060501', 'yyyymmdd') and to_date('20060531', 'yyyymmdd') ORDER BY FACT_UP_TIME, flight_no) TT) TABLE_ALIAS where TABLE_ALIAS.rowno BETWEEN 10 AND 20;
```

5. 另类语法。(有ORDER BY写法）(语法风格与传统的SQL语法不同，不方便阅读与理解，为规范与统一标准，不推荐使用。)

```
With partdata as (SELECT ROWNUM AS ROWNO, TT.* FROM (Select * from k_task T where flight_date between to_date('20060501', 'yyyymmdd') and to_date('20060531', 'yyyymmdd') ORDER BY FACT_UP_TIME, flight_no) TT WHERE ROWNUM <= 20) Select * from partdata where rowno >= 10;
```

6. 另类语法 。(无ORDER BY写法）

```
With partdata as (Select ROWNUM AS ROWNO, T.* From K_task T where Flight_date between to_date('20060501', 'yyyymmdd') and To_date('20060531', 'yyyymmdd') AND ROWNUM <= 20) Select * from partdata where Rowno >= 10;
```