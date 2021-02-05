## 报错解决
### ORA-56935
#### 报错信息
```
连接到: Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
ORA-39006: 内部错误
ORA-39065: DISPATCH 中出现意外的主进程异常错误
ORA-56935: 现有数据泵作业正在使用其他版本的时区数据文件
...
ORA-39097: 数据泵作业出现意外的错误 -56935
```
#### 原因
之前一次的datapump操作没有正常退出，比如强制退出的。没有清除DST_UPGRADE_STATE. 当执行多个job 同时导入时也有可能导致此问题。
#### 解决方案
sqlplus登录，执行以下操作
```sql
SQL> col property_name for a30
SQL> col value for a12
SQL> SELECT PROPERTY_NAME, SUBSTR (property_value, 1, 30) value
  2  FROM   DATABASE_PROPERTIES
  3  WHERE  PROPERTY_NAME ='DST_UPGRADE_STATE'
  4  ORDER  BY PROPERTY_NAME;

-- 返回如下
PROPERTY_NAME                  VALUE
------------------------------ ------------
DST_PRIMARY_TT_VERSION         26
DST_SECONDARY_TT_VERSION       0
DST_UPGRADE_STATE              DATAPUMP(1)
```
从上面查询的结果可以看到，DST_UPGRADE_STATE 的 value 值是DATAPUMP(1), 正常应该是NONE. 下面进行修复:
```sql
SQL> ALTER SESSION SET EVENTS '30090 TRACE NAME CONTEXT FOREVER, LEVEL 32'
会话已更改。
SQL> exec dbms_dst.unload_secondary
PL/SQL 过程已成功完成。
```
再次查看 DST_UPGRADE_STATE的值
```sql
SQL> col property_name for a30
SQL> col value for a12
SQL> SELECT PROPERTY_NAME, SUBSTR (property_value, 1, 30) value
  2  FROM   DATABASE_PROPERTIES
  3  WHERE  PROPERTY_NAME ='DST_UPGRADE_STATE'
  4  ORDER  BY PROPERTY_NAME;
-- 返回如下
PROPERTY_NAME                  VALUE
------------------------------ ------------
DST_PRIMARY_TT_VERSION         26
DST_SECONDARY_TT_VERSION       0
DST_UPGRADE_STATE              NONE
```

### ORA-01654
#### 报错信息
```sql
Resumable error: ORA-01654: 索引 SYS.I_TABPART_BOPART$ 无法通过 8 (在表空间 SYSTEM 中) 扩展
```
#### 解决方案
>[详见参考链接](https://www.cnblogs.com/zzdbullet/p/11130490.html)

查看表空间使用情况
```sql
SELECT
UPPER(F.TABLESPACE_NAME) "TABLESPACE_NAME",
D.TOT_GROOTTE_MB "TABLESPACE_SIZE(M)",
D.TOT_GROOTTE_MB - F.TOTAL_BYTES "TABLESPACE_USED(M)",
TO_CHAR (ROUND((D.TOT_GROOTTE_MB - F.TOTAL_BYTES) / D.TOT_GROOTTE_MB * 100,2),'990.99') "TABLESPACE_USED_BI",
F.TOTAL_BYTES "TABLESPACE_FREE(M)"
FROM
(SELECT TABLESPACE_NAME, ROUND(SUM(BYTES) /(1024 * 1024), 2) TOTAL_BYTES, ROUND(MAX(BYTES) /(1024 * 1024), 2) MAX_BYTES FROM SYS.DBA_FREE_SPACE GROUP BY TABLESPACE_NAME) F,
(SELECT DD.TABLESPACE_NAME, ROUND(SUM(DD.BYTES) / (1024 * 1024), 2) TOT_GROOTTE_MB FROM SYS.DBA_DATA_FILES DD GROUP BY DD.TABLESPACE_NAME) D
WHERE
D.TABLESPACE_NAME = F.TABLESPACE_NAME
ORDER BY 4 DESC;
-- 返回如下
TABLESPACE_NAME                TABLESPACE_SIZE(M) TABLESPACE_USED(M) TABLESP
------------------------------ ------------------ ------------------ -------
TABLESPACE_FREE(M)
------------------
LIWEI_DATA                                  32720           32459.44   99.20
            260.56

SYSTEM                                      23920           22922.19   95.83
            997.81

WYC_TS                                      14784              12801   86.59
              1983
```
可知SYSTEM表空间使用比 已经达到 95.83。若表空间不是自增，则修改为自增模式。先查看表空间是否自增
```sql
SQL > select FILE_NAME,TABLESPACE_NAME,AUTOEXTENSIBLE from dba_data_files;
-- 返回如下
FILE_NAME
--------------------------------------------------------------------------------
TABLESPACE_NAME                AUT
------------------------------ ---
D:\APP\ADMINISTRATOR\VIRTUAL\ORADATA\ORCL\SYSTEM01.DBF
SYSTEM                         NO
```
可知SYSTEM表空间不支持自增，新增表空间物理文件，以增加SYSTEM表空间大小
```sql
ALTER TABLESPACE SYSTEM ADD DATAFILE 'D:\app\Administrator\virtual\oradata\orcl\SYSTEM02.DBF' SIZE 1000M
alter database datafile 'D:\app\Administrator\virtual\oradata\orcl\SYSTEM02.DBF' autoextend on next 200m maxsize unlimited
```
