---
title: rman_info
date: 2023-02-12 17:22:46
tags:
- [Rman]
- [Script]
categories:
- [Oracle]
---

rman限制I/O和檢查datafile

<!--more-->

```bash
@echo off

@For /F "tokens=1,2,3 delims=:. " %%A in ('echo %time%') do @(
Set Hour=%%A
Set Min=%%B
Set Sec=%%C
Set p_time=%%A_%%B_%%C
)
rem: xp @For /f "tokens=1-3 delims=/ " %%a in ('date /t') do (set p_date=%%c-%%b-%%a)
rem: windows 2003 /f "tokens=1-4 delims=/ " %%a in ('date /t') do (set p_date=%%c-%%b-%%d)
@For /f "tokens=1-4 delims=/ " %%a in ('date /t') do (set p_date=%%b-%%c)
set DateString=%p_date%-%p_time%

@For /F "tokens=1,2,3 delims=:. " %%A in ('echo %time%') do @(
Set Hour=%%A
Set Min=%%B
Set Sec=%%C
Set mail_time=%%A:%%B:%%C
)
set ScriptHome=f:\dba
set ORACLE_SID=xxxx
set RmanBackupsetFolder=F:\Backup_RMAN_history\%ORACLE_SID%
set RmanRcvFile=%ScriptHome%\Rman\Log\rman_full_db_archivelog_%ORACLE_SID%.rcv
set RmanLogFile=%ScriptHome%\Rman\Log\rman_full_db_and_archivelog-%DateString%_%ORACLE_SID%.log

set GetDateSql=%ScriptHome%\Rman\Log\GetDateString_for_rman_fbrhist.sql
set GetDateTxt=%ScriptHome%\Rman\Log\GetDateString_for_rman_fbrhist.txt

ping 1.1.1.1 -n 1 -w 5000>nul
echo run > %RmanRcvFile%
echo { >> %RmanRcvFile%
echo CONFIGURE RETENTION POLICY TO REDUNDANCY 3; >> %RmanRcvFile%
echo CONFIGURE BACKUP OPTIMIZATION OFF; >> %RmanRcvFile%
echo SET CONTROLFILE AUTOBACKUP FORMAT FOR DEVICE TYPE DISK TO '%RmanBackupsetFolder%\%ORACLE_SID%_CONTROLFILE_%%F'; >> %RmanRcvFile%
echo CONFIGURE CONTROLFILE AUTOBACKUP ON; >> %RmanRcvFile%
echo crosscheck backup; >> %RmanRcvFile%
echo crosscheck archivelog all; >> %RmanRcvFile%
echo delete noprompt expired backup; >> %RmanRcvFile%
echo delete noprompt expired archivelog all; >> %RmanRcvFile%
echo allocate channel t1 type disk rate 30M; >> %RmanRcvFile%
echo allocate channel t2 type disk rate 30M; >> %RmanRcvFile%
echo allocate channel t3 type disk rate 30M; >> %RmanRcvFile%
echo backup as compressed backupset format '%RmanBackupsetFolder%\%ORACLE_SID%_%DateString%_DB_FULL_%%U' database include current controlfile; >> %RmanRcvFile%
echo backup as compressed backupset format '%RmanBackupsetFolder%\%ORACLE_SID%_%DateString%_ARCH_%%U' archivelog all not backed up 1 times; >> %RmanRcvFile%
echo backup as compressed backupset format '%RmanBackupsetFolder%\%ORACLE_SID%_%DateString%_ARCH_%%U' archivelog until time 'sysdate-1' delete input not backed up 1 times; >> %RmanRcvFile%
echo backup as copy current controlfile format '%RmanBackupsetFolder%\%ORACLE_SID%_CONTROLFILE_%DateString%_%%U'; >> %RmanRcvFile%
echo delete noprompt obsolete; >>  %RmanRcvFile%
echo backup validate check logical database; >>  %RmanRcvFile%
echo } >> %RmanRcvFile%

ping 1.1.1.1 -n 1 -w 5000>nul
rman target / @%RmanRcvFile% > %RmanLogFile%
ping 1.1.1.1 -n 1 -w 5000>nul
```

