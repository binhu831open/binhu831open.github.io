---
title: Oracle-EM
date: 2023-03-12 14:49:24
tags:
- [ORA-]
categories:
- [Oracle]
- [Oracle11g]
---

重建11g EM問題處理

<!--more-->



1.環境變數

```
set ORACLE_UNQNAME
set ORACLE_HOME
set ORACLE_HOSTNAME
```





2.刪除EM的檢查

```
emca -deconfig dbcontrol db -repos drop

手動檢查這些物件都要不存在
SYSMAN
MGMT_VIEW
MGMT_USER
MGMT_TARGET_BLACKOUTS
SETEMVIEWUSERCONTEXT
```





3.建立EM

```
emca -config dbcontrol db -repos create

--建立錯誤可考慮刪除舊的目錄
$ORACLE_HOME/<主機名稱>_<Oracle_SID>
```

