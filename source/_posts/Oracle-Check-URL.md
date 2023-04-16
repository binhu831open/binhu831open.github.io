---
title: Oracle Check URL
date: 2023-02-11 23:00:10
tags:
- [Script]
categories:
- [Oracle]
---

Oracle檢查URL範本

<!--more-->

## 基本範例

```plsql
CREATE OR REPLACE PROCEDURE DBA_Check_URL_Status(t_url in varchar2)
IS
  request UTL_HTTP.REQ;
  response UTL_HTTP.RESP;
  v_url varchar2(4000);
  v_ck_txt varchar2(4000);
  clob_buff CLOB;
  buff VARCHAR2(4000);
  v_seq number ;
  V_MSG varchar2(4000);
  V_USER_FROM varchar2(4000);  
  c  utl_tcp.connection;  -- TCP/IP connection to the Web server
  ret_val pls_integer;
  v_temp varchar2 (1000);
BEGIN
  EXECUTE IMMEDIATE 'ALTER SESSION SET NLS_LANGUAGE= ''AMERICAN''';
  v_url := t_url;
  v_ck_txt := null;
  
  if instr(lower(v_url),'https:') > 0 then
    --處理HTTPS
    select REGEXP_SUBSTR(REGEXP_REPLACE(v_url,'^http[s]?://','',1),'[a-z,A-Z,0-9,-]+(\.\w+)+') INTO v_temp from dual;
    BEGIN
       c := utl_tcp.open_connection(remote_host => v_temp,
                                    remote_port =>  443,
                                    tx_timeout => 5,
                                    charset     => 'UTF-8');  -- open connection
       ret_val := utl_tcp.write_line(c, 'GET / HTTP/1.1');    -- send HTTP request
       ret_val := utl_tcp.write_line(c);
       BEGIN
         LOOP
           dbms_output.put_line(utl_tcp.get_line(c, TRUE));  -- read result
         END LOOP;
       EXCEPTION
         WHEN utl_tcp.end_of_input THEN
           dbms_output.put_line('Check_SSL_URL_OK End');  -- read result
       END;
       utl_tcp.close_connection(c);
    EXCEPTION
      WHEN OTHERS THEN
        --記錄錯誤和簡訊通知
    END;
  else
    BEGIN  
      --處理HTTP
      UTL_HTTP.SET_RESPONSE_ERROR_CHECK(FALSE);
      request := UTL_HTTP.BEGIN_REQUEST(v_url, 'GET');
      UTL_HTTP.SET_HEADER(request, 'User-Agent', 'Mozilla/4.0');
      response := UTL_HTTP.GET_RESPONSE(request);
      DBMS_OUTPUT.PUT_LINE('HTTP response status code: ' || response.status_code);
      
      IF response.status_code = 200 THEN
            BEGIN
                clob_buff := EMPTY_CLOB;
                LOOP
                    UTL_HTTP.READ_TEXT(response, buff, LENGTH(buff));
                    clob_buff := clob_buff || buff;
                END LOOP;
                UTL_HTTP.END_RESPONSE(response);
            EXCEPTION
                WHEN UTL_HTTP.END_OF_BODY THEN
                        UTL_HTTP.END_RESPONSE(response);
                WHEN OTHERS THEN
                  --記錄錯誤和簡訊通知
             END;
            --DBMS_OUTPUT.PUT_LINE(clob_buff);
            v_ck_txt := substr(clob_buff,1,2000);
            v_ck_txt := trim(v_ck_txt);
            
            IF instr(v_ck_txt,'ORA-') > 0  THEN
               --記錄錯誤和簡訊通知
            elsif length(v_ck_txt)=1 and v_ck_txt||''='1' then
               --判讀回傳特定值就視為正確
            else
              --記錄錯誤和簡訊通知
            END IF;
      ELSE
         --記錄錯誤和簡訊通知
         UTL_HTTP.END_RESPONSE(response);
      END IF;
      
    EXCEPTION
        WHEN Utl_Http.request_failed THEN
        v_ck_txt := substr(Utl_Http.get_detailed_sqlerrm,1,100);
        --記錄錯誤和簡訊通知
    END;
   end if;                           
END;
/
```

