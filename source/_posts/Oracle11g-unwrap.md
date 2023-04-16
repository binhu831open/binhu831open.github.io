---
title: Oracle11g_unwrap
date: 2023-02-14 23:43:31
tags:
- [Script]
categories:
- [Oracle]
- [Oracle11g]
---

Oracle11g 反解wrap的內容

<!--more-->



## 使用方式

```sql
SET SERVEROUTPUT ON SIZE 1000000
DECLARE
BEGIN    
     SYS.unwrap ('SchemaName', 'Object_Name','Object_Type');
END;
/
```





## 程式碼

```
CREATE OR REPLACE PROCEDURE SYS.unwrap(p_owner IN VARCHAR,
                   p_name  IN VARCHAR,
                   p_type  IN VARCHAR) AS
  
    l_wrap varchar2(32767);
  
    l_inf       clob;
    l_res       clob;
    l_src       clob;
    l_inflate   varchar2(32767);
    l_temp      VARCHAR2(32767);
    l_bt        varchar2(32767);
    l_text      varchar2(32767);
    l_char      varchar2(2);
    l_len       number;
    l_slen      integer;
    l_offset    integer := 1;
    l_chunk_len integer := 10080;
  
    --解密字節對照字典表
    l_dict varchar2(512) := '3D6585B318DBE287F152AB634BB5A05F' ||
                            '7D687B9B24C228678ADEA4261E03EB17' ||
                            '6F343E7A3FD2A96A0FE935561FB14D10' ||
                            '78D975F6BC4104816106F9ADD6D5297E' ||
                            '869E79E505BA84CC6E278EB05DA8F39F' ||
                            'D0A271B858DD2C38994C480755E4538C' ||
                            '46B62DA5AF322240DC50C3A1258B9C16' ||
                            '605CCFFD0C981CD4376D3C3A30E86C31' ||
                            '47F533DA43C8E35E1994ECE6A39514E0' ||
                            '9D64FA5915C52FCABB0BDFF297BF0A76' ||
                            'B449445A1DF0009621807F1A82394FC1' ||
                            'A7D70DD1D8FF139370EE5BEFBE09B977' ||
                            '72E7B254B72AC7739066200E51EDF87C' ||
                            '8F2EF412C62B83CDACCB3BC44EC06936' ||
                            '6202AE88FCAA4208A64557D39ABDE123' ||
                            '8D924A1189746B91FBFEC901EA1BF7CE';
    l_sl   varchar2(512) := '000102030405060708090A0B0C0D0E0F' ||
                            '101112131415161718191A1B1C1D1E1F' ||
                            '202122232425262728292A2B2C2D2E2F' ||
                            '303132333435363738393A3B3C3D3E3F' ||
                            '404142434445464748494A4B4C4D4E4F' ||
                            '505152535455565758595A5B5C5D5E5F' ||
                            '606162636465666768696A6B6C6D6E6F' ||
                            '707172737475767778797A7B7C7D7E7F' ||
                            '808182838485868788898A8B8C8D8E8F' ||
                            '909192939495969798999A9B9C9D9E9F' ||
                            'A0A1A2A3A4A5A6A7A8A9AAABACADAEAF' ||
                            'B0B1B2B3B4B5B6B7B8B9BABBBCBDBEBF' ||
                            'C0C1C2C3C4C5C6C7C8C9CACBCCCDCECF' ||
                            'D0D1D2D3D4D5D6D7D8D9DADBDCDDDEDF' ||
                            'E0E1E2E3E4E5E6E7E8E9EAEBECEDEEEF' ||
                            'F0F1F2F3F4F5F6F7F8F9FAFBFCFDFEFF';
    cursor c_src is
      SELECT src.line, src.text
        FROM DBA_SOURCE src
       WHERE owner = p_owner
         AND Name = p_name
         AND TYPE = p_type
       order by line;
  BEGIN
    dbms_lob.createtemporary(lob_loc => l_inf,
                             cache   => TRUE,
                             dur     => dbms_lob.session);
    --取得package密文
    for rec in c_src loop
      l_src := l_src || rtrim(rec.text);
    end loop;
  
     --out_put('source:<'||l_src||'>');
    l_src := rtrim(substr(l_src, instr(l_src, chr(10), 1, 20) + 1), chr(10));
    l_src := replace(l_src, chr(10), '');
    -- dbms_output.put_line('source:<'||l_src||'>');
    --l_src:=substr(l_src,41);
    
    --base64解碼
    l_len := dbms_lob.getlength(l_src);
    while l_offset < l_len loop
      if (l_len - l_offset) < 10080 then
        l_chunk_len := (l_len - l_offset);
      end if;
      l_temp := dbms_lob.substr(l_src, l_chunk_len, l_offset);
      l_bt   := utl_encode.base64_decode(utl_raw.cast_to_raw(l_temp));
    
      if l_bt is not null then
        if l_offset = 1 then
          --去掉前40位哈希串
          l_bt := substr(l_bt, 41);
        end if;
        -- l_wrap := l_wrap || l_bt;
        --字典表轉換
        l_bt := utl_raw.translate(l_bt, l_sl, l_dict);
      
        l_offset := l_offset + l_chunk_len;
        --    l_inflate := l_inflate || l_bt;
        dbms_lob.writeappend(l_inf, length(l_bt), l_bt);
      end if;
    
    end loop;
  
    /* dbms_output.put_line('base:' || l_wrap);
    dbms_output.put_line('<' || l_inflate || '>');*/
    
    --解壓縮
    l_res := cux_unwrapper_pl.inflate(l_inf);
    DBMS_OUTPUT.PUT_LINE(l_res);
  END unwrap;
/
```





## 結論

> 1.網路上試了好幾個版本，可在11g中使用無錯誤
>
> 2.應該不支援11g之後的版本，Oracle的加密方式已有改變

