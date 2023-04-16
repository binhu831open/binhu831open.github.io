---
title: oracle19c_auto_index
date: 2023-02-12 18:39:02
tags:
- [PerformanceTuning]
categories:
- [Oracle]
- [Oracle19c]
---

Oracle 19c 新功能 : index 自動建立 (SQL Server早就有.....)

<!--more-->

## Exadate的功能，一般Oracle要自行開起

![image-20230212184248797](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200123.png)





![image-20230212184342490](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200363.png)





## 設定存放Index的Tablespace

![image-20230212184445846](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200130.png)





## 指定那個schema生效，預設是全帳的schema

![image-20230212184520235](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200141.png)





## 可設定index自動維護是否關畢、產出建議、立即生效

![image-20230212184551105](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200146.png)

> auto_index_retention_for_manual : 設定手動建立的index是否也要納入自動管理，也就手動建立的index可能因為很久沒有使用到，將會被自動刪除。預設是不納入自動管理





## 相關的view

![image-20230212184641019](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200151.png)





## 查看report

![image-20230212184706042](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200155.png)



> SET SERVEROUTPUT ON SIZE 1000000
> declare 
>   num number := 1;
>   v_object_id number;
> begin
>   while num <= 500
>   loop    
>     select object_id into v_object_id from tts_binhu.auto_idx_test where object_id=trunc(dbms_random.value(0,10000));
>     dbms_output.put_line('v_object_id:'||nvl(v_object_id,0));
>     num := num + 1;
>   end loop;
> end;
>
> select * from dba_indexes where owner ='TTS_BINHU' and table_name='AUTO_IDX_TEST'



------



## 測試

![image-20230212184812670](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200161.png)



## SELECT DBMS_AUTO_INDEX.report_activity() FROM dual;

```
GENERAL INFORMATION
-------------------------------------------------------------------------------
 Activity start               : 29-10月-2019 14:29:44 
 Activity end                 : 30-10月-2019 14:29:44 
 Executions completed         : 3                    
 Executions interrupted       : 0                    
 Executions with fatal error  : 0                    
-------------------------------------------------------------------------------

SUMMARY (AUTO INDEXES)
-------------------------------------------------------------------------------
 Index candidates                              : 2                       
 Indexes created (visible / invisible)         : 2 (2 / 0)               
 Space used (visible / invisible)              : 7.34 MB (7.34 MB / 0 B) 
 Indexes dropped                               : 0                       
 SQL statements verified                       : 6                       
 SQL statements improved (improvement factor)  : 5 (1419.1x)             
 SQL plan baselines created                    : 0                       
 Overall improvement factor                    : 1183x                   
-------------------------------------------------------------------------------

SUMMARY (MANUAL INDEXES)
-------------------------------------------------------------------------------
 Unused indexes    : 0   
 Space used        : 0 B 
 Unusable indexes  : 0   
-------------------------------------------------------------------------------

INDEX DETAILS
-------------------------------------------------------------------------------
1. The following indexes were created:
-------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------------
| Owner     | Table         | Index                | Key                     | Type   | Properties |
----------------------------------------------------------------------------------------------------
| TTS_BINHU | AUTO_IDX_TEST | SYS_AI_0nfqz048chrwz | OBJECT_ID               | B-TREE | NONE       |
| TTS_BINHU | AUTO_IDX_TEST | SYS_AI_5afppmx4u3fjp | OBJECT_NAME,OBJECT_TYPE | B-TREE | NONE       |
----------------------------------------------------------------------------------------------------
-------------------------------------------------------------------------------

VERIFICATION DETAILS
-------------------------------------------------------------------------------
1. The performance of the following statements improved:
-------------------------------------------------------------------------------
 Parsing Schema Name  : SYS                                                     
 SQL ID               : 6h02v1c3jm99z                                           
 SQL Text             : select * from tts_binhu.auto_idx_test where object_name 
                      = 'AUTO_IDX_TEST' and object_type='TABLE'                 
 Improvement Factor   : 1423.7x                                                 

Execution Statistics:
-----------------------------
                    Original Plan                 Auto Index Plan              
                    ----------------------------  ---------------------------- 
 Elapsed Time (s):  715640                        36386                        
 CPU Time (s):      492167                        127                          
 Buffer Gets:       59872                         3                            
 Optimizer Cost:    395                           4                            
 Disk Reads:        1422                          2                            
 Direct Writes:     0                             0                            
 Rows Processed:    0                             0                            
 Executions:        42                            1                            


PLANS SECTION
---------------------------------------------------------------------------------------------

- Original
-----------------------------
 Plan Hash Value  : 3956758376 

-----------------------------------------------------------------------------
| Id | Operation           | Name          | Rows | Bytes | Cost | Time     |
-----------------------------------------------------------------------------
|  0 | SELECT STATEMENT    |               |      |       |  395 |          |
|  1 |   TABLE ACCESS FULL | AUTO_IDX_TEST |    1 |   132 |  395 | 00:00:01 |
-----------------------------------------------------------------------------

- With Auto Indexes
-----------------------------
 Plan Hash Value  : 1880695109 

-------------------------------------------------------------------------------------------------------
| Id  | Operation                             | Name                 | Rows | Bytes | Cost | Time     |
-------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                      |                      |    1 |   132 |    4 | 00:00:01 |
|   1 |   TABLE ACCESS BY INDEX ROWID BATCHED | AUTO_IDX_TEST        |    1 |   132 |    4 | 00:00:01 |
| * 2 |    INDEX RANGE SCAN                   | SYS_AI_5afppmx4u3fjp |    1 |       |    3 | 00:00:01 |
-------------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
------------------------------------------
* 2 - access("OBJECT_NAME"='AUTO_IDX_TEST' AND "OBJECT_TYPE"='TABLE')


Notes
-----
- Dynamic sampling used for this statement ( level = 11 )


-------------------------------------------------------------------------------
 Parsing Schema Name  : SYS                                                     
 SQL ID               : 6mmkppzn4pyd8                                           
 SQL Text             : SELECT OBJECT_ID FROM TTS_BINHU.AUTO_IDX_TEST WHERE     
                      OBJECT_ID=TRUNC(DBMS_RANDOM.VALUE(0,10000))               
 Improvement Factor   : 1418x                                                   

Execution Statistics:
-----------------------------
                    Original Plan                 Auto Index Plan              
                    ----------------------------  ---------------------------- 
 Elapsed Time (s):  720172                        22908                        
 CPU Time (s):      717822                        9823                         
 Buffer Gets:       2838                          2                            
 Optimizer Cost:    412                           1                            
 Disk Reads:        0                             1                            
 Direct Writes:     0                             0                            
 Rows Processed:    1                             1                            
 Executions:        2                             1                            


PLANS SECTION
---------------------------------------------------------------------------------------------

- Original
-----------------------------
 Plan Hash Value  : 3956758376 

-----------------------------------------------------------------------------
| Id | Operation           | Name          | Rows | Bytes | Cost | Time     |
-----------------------------------------------------------------------------
|  0 | SELECT STATEMENT    |               |      |       |  412 |          |
|  1 |   TABLE ACCESS FULL | AUTO_IDX_TEST |    1 |     5 |  412 | 00:00:01 |
-----------------------------------------------------------------------------

- With Auto Indexes
-----------------------------
 Plan Hash Value  : 252073491 

------------------------------------------------------------------------------------
| Id  | Operation          | Name                 | Rows | Bytes | Cost | Time     |
------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |                      |    1 |     5 |    1 | 00:00:01 |
| * 1 |   INDEX RANGE SCAN | SYS_AI_0nfqz048chrwz |    1 |     5 |    1 | 00:00:01 |
------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
------------------------------------------
* 1 - access("OBJECT_ID"=TRUNC("DBMS_RANDOM"."VALUE"(0,10000)))


Notes
-----
- Dynamic sampling used for this statement ( level = 11 )


-------------------------------------------------------------------------------
 Parsing Schema Name  : SYS                                                     
 SQL ID               : acdnsvfpdjnw0                                           
 SQL Text             : select * from tts_binhu.auto_idx_test where object_name 
                      = 'MV_WH_COMPANY' and object_type='TABLE'                 
 Improvement Factor   : 1418x                                                   

Execution Statistics:
-----------------------------
                    Original Plan                 Auto Index Plan              
                    ----------------------------  ---------------------------- 
 Elapsed Time (s):  535958                        874                          
 CPU Time (s):      469385                        0                            
 Buffer Gets:       60976                         4                            
 Optimizer Cost:    395                           4                            
 Disk Reads:        0                             1                            
 Direct Writes:     0                             0                            
 Rows Processed:    43                            1                            
 Executions:        43                            1                            


PLANS SECTION
---------------------------------------------------------------------------------------------

- Original
-----------------------------
 Plan Hash Value  : 3956758376 

-----------------------------------------------------------------------------
| Id | Operation           | Name          | Rows | Bytes | Cost | Time     |
-----------------------------------------------------------------------------
|  0 | SELECT STATEMENT    |               |      |       |  395 |          |
|  1 |   TABLE ACCESS FULL | AUTO_IDX_TEST |    1 |   132 |  395 | 00:00:01 |
-----------------------------------------------------------------------------

- With Auto Indexes
-----------------------------
 Plan Hash Value  : 1880695109 

-------------------------------------------------------------------------------------------------------
| Id  | Operation                             | Name                 | Rows | Bytes | Cost | Time     |
-------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                      |                      |    2 |   264 |    4 | 00:00:01 |
|   1 |   TABLE ACCESS BY INDEX ROWID BATCHED | AUTO_IDX_TEST        |    2 |   264 |    4 | 00:00:01 |
| * 2 |    INDEX RANGE SCAN                   | SYS_AI_5afppmx4u3fjp |    1 |       |    3 | 00:00:01 |
-------------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
------------------------------------------
* 2 - access("OBJECT_NAME"='MV_WH_COMPANY' AND "OBJECT_TYPE"='TABLE')


Notes
-----
- Dynamic sampling used for this statement ( level = 11 )


-------------------------------------------------------------------------------
 Parsing Schema Name  : SYS                                                     
 SQL ID               : c3c69a1dkwucj                                           
 SQL Text             : select * from tts_binhu.auto_idx_test where             
                      object_id=trunc(dbms_random.value(0,10000))               
 Improvement Factor   : 1418x                                                   

Execution Statistics:
-----------------------------
                    Original Plan                 Auto Index Plan              
                    ----------------------------  ---------------------------- 
 Elapsed Time (s):  397948                        17176                        
 CPU Time (s):      397844                        158                          
 Buffer Gets:       1420                          3                            
 Optimizer Cost:    412                           2                            
 Disk Reads:        0                             7                            
 Direct Writes:     0                             0                            
 Rows Processed:    2                             1                            
 Executions:        1                             1                            


PLANS SECTION
---------------------------------------------------------------------------------------------

- Original
-----------------------------
 Plan Hash Value  : 3956758376 

-----------------------------------------------------------------------------
| Id | Operation           | Name          | Rows | Bytes | Cost | Time     |
-----------------------------------------------------------------------------
|  0 | SELECT STATEMENT    |               |      |       |  412 |          |
|  1 |   TABLE ACCESS FULL | AUTO_IDX_TEST |    1 |   132 |  412 | 00:00:01 |
-----------------------------------------------------------------------------

- With Auto Indexes
-----------------------------
 Plan Hash Value  : 3717726382 

-------------------------------------------------------------------------------------------------------
| Id  | Operation                             | Name                 | Rows | Bytes | Cost | Time     |
-------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                      |                      |    1 |   132 |    2 | 00:00:01 |
|   1 |   TABLE ACCESS BY INDEX ROWID BATCHED | AUTO_IDX_TEST        |    1 |   132 |    2 | 00:00:01 |
| * 2 |    INDEX RANGE SCAN                   | SYS_AI_0nfqz048chrwz |    1 |       |    1 | 00:00:01 |
-------------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
------------------------------------------
* 2 - access("OBJECT_ID"=TRUNC("DBMS_RANDOM"."VALUE"(0,10000)))


Notes
-----
- Dynamic sampling used for this statement ( level = 11 )


-------------------------------------------------------------------------------
 Parsing Schema Name  : SYS                                                     
 SQL ID               : dt1tymnjfa8m6                                           
 SQL Text             : select * from tts_binhu.auto_idx_test where             
                      object_id=trunc(dbms_random.value(0,10000))               
 Improvement Factor   : 1418x                                                   

Execution Statistics:
-----------------------------
                    Original Plan                 Auto Index Plan              
                    ----------------------------  ---------------------------- 
 Elapsed Time (s):  5006616                       190                          
 CPU Time (s):      4992671                       190                          
 Buffer Gets:       17018                         3                            
 Optimizer Cost:    412                           2                            
 Disk Reads:        0                             0                            
 Direct Writes:     0                             0                            
 Rows Processed:    9                             1                            
 Executions:        12                            1                            


PLANS SECTION
---------------------------------------------------------------------------------------------

- Original
-----------------------------
 Plan Hash Value  : 3956758376 

-----------------------------------------------------------------------------
| Id | Operation           | Name          | Rows | Bytes | Cost | Time     |
-----------------------------------------------------------------------------
|  0 | SELECT STATEMENT    |               |      |       |  412 |          |
|  1 |   TABLE ACCESS FULL | AUTO_IDX_TEST |    1 |   132 |  412 | 00:00:01 |
-----------------------------------------------------------------------------

- With Auto Indexes
-----------------------------
 Plan Hash Value  : 3717726382 

-------------------------------------------------------------------------------------------------------
| Id  | Operation                             | Name                 | Rows | Bytes | Cost | Time     |
-------------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT                      |                      |    1 |   132 |    2 | 00:00:01 |
|   1 |   TABLE ACCESS BY INDEX ROWID BATCHED | AUTO_IDX_TEST        |    1 |   132 |    2 | 00:00:01 |
| * 2 |    INDEX RANGE SCAN                   | SYS_AI_0nfqz048chrwz |    1 |       |    1 | 00:00:01 |
-------------------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
------------------------------------------
* 2 - access("OBJECT_ID"=TRUNC("DBMS_RANDOM"."VALUE"(0,10000)))


Notes
-----
- Dynamic sampling used for this statement ( level = 11 )


-------------------------------------------------------------------------------
-------------------------------------------------------------------------------

ERRORS
---------------------------------------------------------------------------------------------
No errors found.
---------------------------------------------------------------------------------------------

```





## 刪除自動建立的index

![image-20230212185015654](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200169.png)

> 無法直接刪除





## 更新狀態由Auto由Yes->No

![image-20230212185212945](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200178.png)





## 刪除Index Name有區分大小寫

![image-20230212185256081](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200202.png)





## 結論

> 1.測試的DB版本:Oracle 19c
>
> 2.自動建立index不支援function/partition table
