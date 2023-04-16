---
title: Ping主機異常用中華SMS發送通知
date: 2023-02-11 23:28:25
tags:
- [Monitor]
- [Script]
categories:
- [OS]
---

使用中華電訊SMS通知異常的範本

<!--more-->

## Check_Ping_Main.bat

```bash
@echo off
set Check_Host=要檢查的主機IP
set SMS_TELNUM="接收告警電話1,接收告警電話1"
set execFileBASE=f:\dba

call %execFileBASE%\check\check_ping_base.bat %Check_Host% %SMS_TELNUM%
```





## Check_Ping_Base.bat

```bash
@echo off
set Check_Host=%1
set SMS_TN=%2
set SMS_TELNUM=%SMS_TN:"=  %


@For /F "tokens=1,2,3 delims=:. " %%A in ('echo %time%') do @(
Set Hour=%%A
Set Min=%%B
Set Sec=%%C
Set p_time=%%A:%%B:%%C
)
rem: xp @For /f "tokens=1-3 delims=/ " %%a in ('date /t') do (set p_date=%%c-%%b-%%a)
rem: windows 2003
@For /f "tokens=1-4 delims=/ " %%a in ('date /t') do (set p_date=%%d-%%b-%%c)
set DateString=%p_date%-%p_time%

set execFileBASE=f:\dba
set check_file=%execFileBASE%\check\log\check_ping_1.ck
set SMS_MESSAGE=%Check_Host% is not availabe yet. -- %DateString%

rem: 中華電訊的sms程式
set notify=%execFileBASE%\notify2\Release\notify2.exe
set SMS_IP=中華電訊的sms主機IP
set SMS_USER=中華電訊的sms帳號
set SMS_PWD=中華電訊的sms密碼

%SystemRoot%\system32\ping.exe -n 1 %Check_Host% >nul
if errorlevel 1 goto NoServer

echo %Check_Host% is availabe.
if exist %check_file% (del %check_file% /Q)
rem Insert commands here, for example 1 or more net use to connect network drives.
goto :EOF

:NoServer
echo %Check_Host% is not availabe yet.
if exist %check_file% ( for %%i in (%SMS_TELNUM%) do %notify% %SMS_IP% %SMS_USER%  %SMS_PWD% %%i %SMS_MESSAGE% )
echo "error" > %check_file%
goto :EOF
```

