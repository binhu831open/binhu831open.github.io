---
title: sqlldr_hdsize
date: 2023-02-12 18:15:32
tags:
- [Script]
categories:
- [Oracle]
---

sqlldr傳硬碟資訊到table

<!--more-->

```
rem: @echo off
set ORACLE_SID=xxxx
set ScriptHome=F:\dba_for_primary_role\Os
set LogFolder=%ScriptHome%\Log
set OS_HDSize_Info_File=%LogFolder%\OS_HDSize.csv
set SQLLDR_BADFILE=%LogFolder%\OS_HDSize.bad
set SQLLDR_DISCARDFILE=%LogFolder%\OS_HDSize.dsc
set TableName=binhu.dba_HDSIZE_LOG
set SQLLDR_CTL_File=%ScriptHome%\Log\LoadHDSize.ctl
set SQLLDR_CTL_Log=%ScriptHome%\Log\LoadHDSize.log
set GetHOSTIPSql=%ScriptHome%\Log\GetHOSTIP.sql
set GetHOSTIPTxt=%ScriptHome%\Log\GetHOSTIP.txt


set IP=xxx.xxx.xxx.xxx

wmic.exe /OUTPUT:%OS_HDSize_Info_File%  logicaldisk where "drivetype=3" get name,freespace,size 

rem: for windows 2008
set GET_COMPUTERNAME=%COMPUTERNAME%

echo LOAD DATA > %SQLLDR_CTL_File%
echo CHARACTERSET 'AL16UTF16' >> %SQLLDR_CTL_File%
echo INFILE '%OS_HDSize_Info_File%' >> %SQLLDR_CTL_File%
echo BADFILE '%SQLLDR_BADFILE%' >> %SQLLDR_CTL_File%
echo DISCARDFILE '%SQLLDR_DISCARDFILE%' >> %SQLLDR_CTL_File%
echo INTO TABLE %TableName% >> %SQLLDR_CTL_File%
echo APPEND >> %SQLLDR_CTL_File%
echo FIELDS TERMINATED BY WHITESPACE >> %SQLLDR_CTL_File%
echo TRAILING NULLCOLS >> %SQLLDR_CTL_File%
echo (HOSTNAME CONSTANT "%GET_COMPUTERNAME%", >> %SQLLDR_CTL_File%
echo FREE_SIZE_BYTE, >> %SQLLDR_CTL_File%
echo MOUNT_POINT_NAME, >> %SQLLDR_CTL_File%
echo MOUNT_POINT_SIZE_BYTE, >> %SQLLDR_CTL_File%
echo HOSTIP CONSTANT  "%IP%") >> %SQLLDR_CTL_File%

sqlldr / control=%SQLLDR_CTL_File%  log=%SQLLDR_CTL_Log%
```

