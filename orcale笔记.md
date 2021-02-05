## 新建&初始化
### sqlpus命令行登录
```shell
sqlplus username/password@//host:port/sid
# 例如
sqlplus c##liwe/liwei@//localhost:1521/orcl as sysdba
```

### 创建临时表空间
```sql
create temporary tablespace temp_tablespace_name --表空间名字my_temp
tempfile '物理文件名.dbf' --表空间文件物理路径
size 2G --表空间大小
extent management local; --本地管理表空间
```
### 创建数据表空间
```sql
create  tablespace tablespace_name
datafile '物理文件名.dbf'
size 2G --初始大小
autoextend on  --自动扩展
next 50m maxsize 20480m  --自动扩展每次增加50M，最大可到20480M 也可以写 unlimited 不限制
extent management local; --本地管理表空间
```
### 创建用户并指定表空间
>oracle12c 有一个很大的变动就是引入了pdb可插入数据库，而且在cdb中只能创建c##或者C##开头的用户，只有在pdb数据库中才能创建我们习惯性命名的用户，oracle称之为Local User，前者称之为Common User。
```sql
create user c##username identified by password
default tablespace tablespace_name
temporary tablespace temp_tablespace_name;
```
### 根据需要设置权限
```sql
grant connect to username; --授予连接db的权限
grant dba to username; --DBA 权限
grant unlimited tablespace to username; --全局表空间权限
```
### 查看表空间物理文件地址
```sql
SELECT tablespace_name,
file_id,
file_name,
round(bytes / (1024 * 1024), 0) total_space
FROM dba_data_files
ORDER BY tablespace_name;
```
### 查看表空间路径
```sql
select * from dba_data_files;
```
### 查询建表语句
```sql
select dbms_metadata.get_ddl('TABLE','table_name') from dual;
```
### orcale重启命令
```sql
sql> shutdown immdiate; --关闭
sql> startup; --启动
```
### 删除用户及表空间
```sql
drop user c##username cascade;
DROP TABLESPACE liwei_data INCLUDING CONTENTS AND DATAFILES;
--删除空的表空间，但是不包含物理文件
drop tablespace tablespace_name;
--删除非空表空间，但是不包含物理文件
drop tablespace tablespace_name including contents;
--删除空表空间，包含物理文件
drop tablespace tablespace_name including datafiles;
--删除非空表空间，包含物理文件
drop tablespace tablespace_name including contents and datafiles;
--如果其他表空间中的表有外键等约束关联到了本表空间中的表的字段，就要加上CASCADE CONSTRAINTS
drop tablespace tablespace_name including contents and datafiles CASCADE CONSTRAINTS;
```
## 数据导入导出
### 创建directory
```sql
create or replace directory dump_dir as 'D:\gps'
```
### directory赋权
```sql
grant read,write on directory dump_dir to user
```
### expdp/impdp导出导入数据
>Oracle Data Pump(以下简称数据泵)是Oracle 10g开始提供的一种数据迁移工具，同时也被广大DBA用来作为数据库的逻辑备份工具和体量较小的数据迁移工具。与传统的数据导出/导入工具，即exp/imp工具相比，数据泵更为高效和安全，数据泵主要包含以下三个部分：
1. 操作系统命令行客户端，expdp和impdp；
2. DBMS_DATAPUMP PL/SQL包(也被认为是Data Pump API)；
3. DBMS_METADATA PL/SQL包(也被认为是Metadata API)。

>DBMS_DATAPUMP包主要执行实际数据的导出和导入工作，expdp和impdp命令也是通过命令行调用该包当中的存储过程实现数据导出导入功能，这个包是数据泵当中最核心的部分；
DBMS_METADATA包主要提供当数据导出导入用于元数据移动时，对元数据内容的提取、修改和重新创建的功能。

>当使用impdp完成数据库导入时，如遇到表已存在时，Oracle提供给我们如下四种处理方式：
1. 忽略（SKIP，默认行为）；
2. 在原有数据基础上继续增加（APPEND）；
3. 先DROP表，然后创建表，最后完成数据插入（REPLACE）；
4. 先TRUNCATE，再完成数据插入（TRUNCATE）。

```shell
# 示例1（重映射用户、表空间）
impdp c##liwei/liwei directory=nt_bus_dir dumpfile=tm_bus_site_his202012.dmp tables=tmgpshis.tm_bus_site_his remap_schema=tmgpshis:c##liwei remap_tablespace=%:liwei_data
# 示例2（重映射用户、表、表空间）
impdp c##liwei/liwei directory=dump_dir dumpfile=TKGPS_NEPACK_GPS_20201201.dmp tables=TKGPS.NETPACK_GPS remap_schema=TKGPS:c##liwei remap_tablespace=%:liwei_data remap_table=TKGPS.NETPACK_GPS:NETPACK_GPS_20211201
# 示例2（重映射用户、表空间、追加数据至已存在表）
impdp c##liwei/liwei directory=dump_dir dumpfile=TKGPS_NEPACK_GPS_20201201.dmp tables=TKGPS.NETPACK_GPS remap_schema=TKGPS:c##liwei remap_tablespace=%:liwei_data table_exists_action=append
```
### exp/imp导出导入数据
```shell
imp c##liwei/liwei@orcl file=F:\南通网约车GPS数据\jtj.dmp full=y ignore=y
```
### sqoop导入导出
```shell
sqoop import -m 50 --username c##liwei  --password liwei    --connect jdbc:oracle:thin:@192.168.0.14:1521:ORCL --table TM_BUS_SITE_HIS --hive-import --hive-database nantong --hive-overwrite --create-hive-table --hive-table TM_BUS_SITE_HIS --delete-target-dir
```
