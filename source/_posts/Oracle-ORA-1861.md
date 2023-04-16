---
title: Oracle-ORA-1861
date: 2023-03-02 22:47:17
tags:
- [Script]
categories:
- [Oracle]
- [Oracle11g]
---

RMAN Recovery Session Fails with ORA-1861 (Doc ID 852723.1)

<!--more-->

遇到的情況就是rman list backup出現 ORA-01861: literal does not match format string



查的解決方法

1.設定NLS，失敗

```
Linux :
export NLS_LANG=AMERICAN_AMERICA.WE8ISO8859P1
export NLS_DATE_FORMAT=YYYY_MM_DD HH24:MI:SS

Windows :
set NLS_LANG=AMERICAN_AMERICA.WE8ISO8859P1
set NLS_DATE_FORMAT=YYYY_MM_DD HH24:MI:SS
```

2.重建controlfile，懶的用，所以不採用

3.依Doc ID 852723.1來做最簡單，裏面就只有plb檔要戴入而已

```
loadjava -user sys/sys_pwd pb.plb

sqlplus sys/sys_pwd as sysdba
@del.sql
```





結論:

> 1.DataGuard 環境在Standby DB也要做一次@del.sql
>
> 2.Standby Control File也可用restored的方式
