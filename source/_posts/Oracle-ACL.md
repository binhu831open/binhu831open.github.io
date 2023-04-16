---
title: Oracle ACL設定
date: 2023-02-11 21:14:23
tags:
- [ACL]
- [Email]
- [Script]
categories:
- [Oracle]
description: Oracle ACL設定
---

ACL維護和Mail

<!--more-->

## 刪除ACL設定

```SQL
begin
 dbms_network_acl_admin.drop_acl(acl => 'mail_acl_file.xml'); 
 commit;
end;
```



## 新增ACL設定

```SQL
BEGIN
DBMS_NETWORK_ACL_ADMIN.create_acl (
acl => 'mail_acl_file.xml',
description => 'create by 2020/1/1',
principal => 'User帳號',--改這
is_grant => TRUE,
privilege => 'connect',
start_date => SYSTIMESTAMP,
end_date => NULL);
commit;
END;
/
```



## 新增ACL權限

```sql
begin
  dbms_network_acl_admin.add_privilege (
  acl => 'mail_acl_file.xml',
  principal => 'User帳號',--改這
  is_grant => true,
  privilege => 'connect'
  );
  commit;
  end;
/

begin
dbms_network_acl_admin.add_privilege(
   acl => 'mail_acl_file.xml',
   principal => 'User帳號',--改這
   is_grant => TRUE,
   privilege => 'resolve'
   ); 
  commit;
end;
/ 
```



------



## 設定Email-ACL

```sql
begin
    dbms_network_acl_admin.assign_acl(
    acl => 'mail_acl_file.xml',
    host => 'Mail IP'--改這
    );
    commit;
end;  
/
```



## 相關SQL

```sql
SELECT host, lower_port, upper_port, acl FROM dba_network_acls;

select acl,principal,privilege,is_grant,to_char(start_date, 'dd-mon-yyyy') as start_date,to_char(end_date, 'dd-mon-yyyy') as end_date 
from dba_network_acl_privileges;

select any_path from resource_view where any_path like '/sys/acls/%.xml';
```



------



## Schedule Mail

```sql
select * from DBA_SCHEDULER_GLOBAL_ATTRIBUTE;

execute DBMS_SCHEDULER.SET_SCHEDULER_ATTRIBUTE ('EMAIL_SERVER',' Mail_IP');

execute DBMS_SCHEDULER.SET_SCHEDULER_ATTRIBUTE ('EMAIL_SENDER','寄件者Mail');
```



## Mail_PROCEDURE

```plsql
CREATE OR REPLACE PROCEDURE DBA_MONITOR_SEND_MAIL (
   p_from          IN VARCHAR2,
   p_to            IN VARCHAR2,   
   p_cc            IN VARCHAR2 default null,
   p_bcc           IN VARCHAR2 default null,
   p_reply         IN VARCHAR2 default null,
   p_subject       IN VARCHAR2,   
   p_text          IN VARCHAR2 default null,
   p_html          in VARCHAR2 default null,
   p_smtp_hostname in varchar2,
   p_smtp_portnum  in number,
   p_domain        in varchar2 default null,
   p_login         in varchar2,
   p_pwd           in varchar2,
   p_html_body     in clob 
   )
IS
 --宣告
  l_mail_conn   UTL_SMTP.connection;
  l_boundary    VARCHAR2(50) := '----=*#abc1234321cba#*=';

  
BEGIN
 --設定伺服器
  l_mail_conn := UTL_SMTP.open_connection(p_smtp_hostname, p_smtp_portnum);
  --l_mail_conn := utl_smtp.open_connection('1.1.1.1', 25);
  UTL_SMTP.helo(l_mail_conn, p_smtp_hostname);

 --寄件者
  UTL_SMTP.mail(l_mail_conn, p_from);

 --收件者
  UTL_SMTP.rcpt(l_mail_conn,  p_to);
  if p_cc is not null then    
    UTL_SMTP.rcpt(l_mail_conn,  p_cc);      
  end if;
  UTL_SMTP.open_data(l_mail_conn);

 -- 主旨
  UTL_SMTP.write_data(l_mail_conn, 'From: ' || p_from || UTL_TCP.crlf);
  UTL_SMTP.write_data(l_mail_conn, 'To: ' || p_to || UTL_TCP.crlf);
  if p_cc is not null then    
    UTL_SMTP.write_data(l_mail_conn, 'Cc: ' || p_cc || UTL_TCP.crlf);  
  end if;
  UTL_smtp.write_raw_data(l_mail_conn, utl_raw.cast_to_raw( 'Subject: '|| p_subject || UTL_TCP.crlf));
  UTL_SMTP.write_data(l_mail_conn, 'MIME-Version: 1.0' || UTL_TCP.crlf);
  UTL_SMTP.write_data(l_mail_conn, 'Content-Type: multipart/alternative; boundary="' || l_boundary || '"' || UTL_TCP.crlf || UTL_TCP.crlf);
  UTL_SMTP.write_data(l_mail_conn, '--' || l_boundary || UTL_TCP.crlf);
  UTL_SMTP.write_data(l_mail_conn, 'Content-Type: text/html; charset=utf-8/big5' || UTL_TCP.crlf || UTL_TCP.crlf);
  --utl_smtp.write_data( l_mail_conn, 'Content-Transfer-Encoding: 8bit' || UTL_TCP.crlf );
  
  --UTL_smtp.write_raw_data(l_mail_conn, utl_raw.cast_to_raw( p_html_body ));
  UTL_smtp.write_raw_data(l_mail_conn, utl_raw.cast_to_raw(CONVERT(dbms_lob.substr(p_html_body), 'UTF8')));
  UTL_SMTP.write_data(l_mail_conn, UTL_TCP.crlf || UTL_TCP.crlf);
  UTL_SMTP.write_data(l_mail_conn, '--' || l_boundary || '--' || UTL_TCP.crlf);
  UTL_SMTP.close_data(l_mail_conn);
  UTL_SMTP.quit(l_mail_conn);
END;
/
```



------



## Windows Script

```bash
rem:
rem: WScript Send Mail
rem: cscript SendMail.vbs /P_From:From /P_To:To /P_Subject:Subject /P_TextBody:TextBody /P_File:File
/P_TextBody:TextBody_test /P_File:D:\binhu_folder\temp\README-BR.txt
rem:

SmtpName = "SMTP主機IP"
Login = "Mail帳號"
LoginPassword = "Mail密碼"
Dim P_From
Dim P_To
Dim P_Subject
Dim P_TextBody
Dim P_File

rem: WScript.Echo Wscript.Arguments.Named("P_From")
rem: WScript.Echo Wscript.Arguments.Named("P_To")
rem: WScript.Echo Wscript.Arguments.Named("P_Subject")
rem: WScript.Echo Wscript.Arguments.Named("P_TextBody")
rem: WScript.Echo Wscript.Arguments.Named("P_File")

P_From=Wscript.Arguments.Named("P_From")
P_To=Wscript.Arguments.Named("P_To")
P_Subject=Wscript.Arguments.Named("P_Subject")
P_TextBody=Wscript.Arguments.Named("P_TextBody")
P_File=Wscript.Arguments.Named("P_File")


Set objEmail = CreateObject("CDO.Message")
    objEmail.From = P_From
    objEmail.To = P_To
    objEmail.Subject = P_Subject
    objEmail.Textbody = P_TextBody
    Set objFSO = CreateObject("Scripting.FileSystemObject")
        IF objFSO.FileExists (P_File) then
           objEmail.Addattachment P_File
           rem: WScript.echo "File Exist"
        end if 
    objEmail.Configuration.Fields.Item ("http://schemas.microsoft.com/cdo/configuration/sendusing") = 2
    objEmail.Configuration.Fields.Item ("http://schemas.microsoft.com/cdo/configuration/smtpserver") = SmtpName
    objEmail.Configuration.Fields.Item ("http://schemas.microsoft.com/cdo/configuration/smtpserverport") = 25
    objEmail.Configuration.Fields.Item ("http://schemas.microsoft.com/cdo/configuration/smtpauthenticate") = 1
    objEmail.Configuration.Fields.Item ("http://schemas.microsoft.com/cdo/configuration/sendusername") = Login
    objEmail.Configuration.Fields.Item ("http://schemas.microsoft.com/cdo/configuration/sendpassword") = LoginPassword
    objEmail.Configuration.Fields.Update
    objEmail.Send

Set objEmail = Nothing
```

