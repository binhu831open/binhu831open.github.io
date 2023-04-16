---
title: Oracle_DW
date: 2023-02-18 20:48:52
tags:
- [PerformanceTuning]
categories:
- [Oracle]
---

Data Warehouse 大資料匯入的優化手段

<!--more-->

## external table 截入cvs和dmp

```sql
CREATE TABLE emp_ext (
  id NUMBER(4),
  addr VARCHAR2(10),
  ctime DATE
)
ORGANIZATION EXTERNAL (
  TYPE CSV
  DEFAULT DIRECTORY data_dir
  LOCATION ('test.csv')
)
REJECT LIMIT UNLIMITED;
```



```sql
--產生dmp File
CREATE TABLE emptest_ext (
  id NUMBER(4),
  addr VARCHAR2(10),
  ctime DATE
)
ORGANIZATION EXTERNAL (
  TYPE ORACLE_DATAPUMP
  DEFAULT DIRECTORY dump_dir
  LOCATION ('emptest.dmp')
)
PARALLEL 4
REJECT LIMIT UNLIMITED;

INSERT INTO emptest_ext SELECT * FROM emptest;

--戴入 emp_ext
CREATE TABLE emp_ext (
  id NUMBER(4),
  addr VARCHAR2(10),
  ctime DATE
)
ORGANIZATION EXTERNAL (
  TYPE ORACLE_DATAPUMP
  DEFAULT DIRECTORY data_dir
  LOCATION ('emptest.dmp')
)
REJECT LIMIT UNLIMITED;
```

> 適合資料量很大，並且都是Full Table Scan處理的情境，要留意是無法建立index



------



## DBMS_DATAPUMP

```sql
DECLARE
   v_handle     NUMBER;
   v_status     VARCHAR2(20);
BEGIN
   v_handle := DBMS_DATAPUMP.OPEN(
                  operation   => 'EXPORT',
                  job_mode    => 'TABLE',
                  remote_link => 'DBLINK_NAME'
               );

   DBMS_DATAPUMP.ADD_FILE(
      handle  => v_handle,
      filename=> 'EXPORT_DIR:table_export.dmp',
      directory => 'EXPORT_DIR',
      filetype => DBMS_DATAPUMP.KU$_FILE_TYPE_DUMP_FILE);

   DBMS_DATAPUMP.ADD_FILE(
      handle  => v_handle,
      filename=> 'EXPORT_DIR:table_export.log',
      directory => 'EXPORT_DIR',
      filetype => DBMS_DATAPUMP.KU$_FILE_TYPE_LOG_FILE);

   DBMS_DATAPUMP.METADATA_FILTER(
      handle => v_handle,
      name   => 'SCHEMA_EXPR',
      value  => 'IN(''SCHEMA_NAME'')');

   DBMS_DATAPUMP.START_JOB(handle => v_handle);

   DBMS_DATAPUMP.WAIT_FOR_JOB(handle => v_handle, job_state => v_status);

   DBMS_DATAPUMP.DETACH(handle => v_handle);
END;
/
```



```sql
DECLARE
   v_handle     NUMBER;
   v_status     VARCHAR2(20);
BEGIN
   v_handle := DBMS_DATAPUMP.OPEN(
                  operation   => 'IMPORT',
                  job_mode    => 'TABLE',
                  remote_link => 'DBLINK_NAME'
               );

   DBMS_DATAPUMP.ADD_FILE(
      handle  => v_handle,
      filename=> 'IMPORT_DIR:table_export.dmp',
      directory => 'IMPORT_DIR',
      filetype => DBMS_DATAPUMP.KU$_FILE_TYPE_DUMP_FILE);

   DBMS_DATAPUMP.ADD_FILE(
      handle  => v_handle,
      filename=> 'IMPORT_DIR:table_export.log',
      directory => 'IMPORT_DIR',
      filetype => DBMS_DATAPUMP.KU$_FILE_TYPE_LOG_FILE);

   DBMS_DATAPUMP.METADATA_REMAP(
      handle     => v_handle,
      name       => 'REMAP_SCHEMA',
      old_value  => 'OLD_SCHEMA_NAME',
      new_value  => 'NEW_SCHEMA_NAME');

   DBMS_DATAPUMP.START_JOB(handle => v_handle);

   DBMS_DATAPUMP.WAIT_FOR_JOB(handle => v_handle, job_state => v_status);

   DBMS_DATAPUMP.DETACH(handle => v_handle);
END;
/
```

> 使用DBMS的方式不落地用expdp和impdp，一直在db裏完成，不需要進行os操作



------



## Hint -- insert direct path

> INSERT /*+ APPEND */ 避開Buufer，直寫入DataFile，但要留意index會導致失效



------



## Partition Table 

```
CREATE TABLE emp (
  id     NUMBER,
  t_date   DATE,
  amount   NUMBER
)
PARTITION BY RANGE(t_date)
INTERVAL(NUMTOYMINTERVAL(1, 'MONTH'))
(
  PARTITION p1 VALUES LESS THAN (TO_DATE('2022-01-01', 'YYYY-MM-DD'))
);
```

自動增加Partition

```
ALTER TABLE emp ENABLE ROW MOVEMENT;  --一定要
EXEC DBMS_PART_ADMIN.ENABLE_AUTO_PARTITIONING( 
  table_name  => 'emp',
  interval    => 'INTERVAL(NUMTOYMINTERVAL(1, ''MONTH''))',
  start_date  => '2022-01-01',
  retention   => 'INTERVAL ''1'' MONTH' --個為間隔單位
);
```

自動刪除舊的Partition (19c可以用，較舊的版本要測一下)

```
BEGIN
  DBMS_PART_ADMIN.ALTER_RETENTION(
    table_name  => 'emp',
    retention   => 'INTERVAL ''1'' MONTH'
  );
END;
/
```

### 針對Index再優化

Partial Indexes

```sql
CREATE INDEX idx_emp_partial
ON emp(t_date)
WHERE t_date >= '2022-01-01';
-- index的資料只包含 2022-01-01之後的，之前的資料並不包含到index裏，可縮小index size
```

index生效的方式

```sql
SELECT *
FROM t_date
WHERE t_date >= '2022-02-01' and t_date <= '2022-04-01'
```

> 刪除和新增資料佔用I/O的時間錯開



------



## 換別的OLAP DB

> 考量用apache doris、clickhouse，要考量是否能維運



------



## 結論

> 1.首選先考慮升級成SSD
>
> 2.了解AP邏輯運作後再看有多少空檔時間可做維護，再選擇方案

