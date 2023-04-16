---
title: script_ftp_restart
date: 2023-04-16 14:56:30
tags:
- [Script]
categories:
- [OS]
---

Ftp檢查檔案Size不相同，執行續傳

<!--more-->



```bash
#!/usr/bin/bash

zip_passwd=`date +%b%d%Y`
dump_folder=/dbback/data_pump
ftp_name=/oradata/schedule/log/dba_ftp_
ftp_name_list=${ftp_name}_list
ftp_ck_name=/oradata/schedule/log/dba_ftp_ck_
ftp_size_name=/oradata/schedule/log/dba_ftp_size_
ftp_reput_name=/oradata/schedule/log/dba_ftp_reput_

ls -ltr ${dump_folder} | grep ${zip_passwd} | tr -s ' ' | cut -d ' ' -f9 > ${ftp_name_list}

cat ${ftp_name_list} | while read line
do
   echo 'user ftpuser ftppasswd' > ${ftp_name}${line};
   echo 'bi' >> ${ftp_name}${line};
   echo 'prompt' >> ${ftp_name}${line};
   echo lcd ${dump_folder} >> ${ftp_name}${line};
   echo put ${line} >> ${ftp_name}${line};
   echo 'bye' >> ${ftp_name}${line};
done

cat ${ftp_name_list} | while read line
do
  echo 'user ftpuser ftppasswd' > ${ftp_ck_name}${line};
  echo dir ${line} >> ${ftp_ck_name}${line};
  echo 'bye' >> ${ftp_ck_name}${line};
done


cat ${ftp_name_list} | while read line
do
  ftp -n FTP_IP < ${ftp_name}${line};
  ftp -n FTP_IP < ${ftp_ck_name}${line} > ${ftp_size_name}${line};
done


cat ${ftp_name_list} | while read line
do
  ftp_sfile=`cat ${ftp_size_name}${line} | grep ${zip_passwd} | tr -s ' ' | cut -d ' ' -f5`
  local_sfile=`ls -ltr ${dump_folder} | grep ${line} | tr -s ' ' | cut -d ' ' -f5`
  if [ ${ftp_sfile} -gt 0 ] && [ ${ftp_sfile} -ne ${local_sfile} ]; then
     echo 'user ftpuser ftppasswd' > ${ftp_reput_name}${line};
     echo 'bi' >> ${ftp_reput_name}${line};
     echo 'prompt' >> ${ftp_reput_name}${line};
     echo lcd ${dump_folder} >> ${ftp_reput_name}${line};
     echo restart ${ftp_sfile} >> ${ftp_reput_name}${line};
     echo put ${line} >> ${ftp_reput_name}${line};
     echo 'bye' >> ${ftp_reput_name}${line};
     ftp -n FTP_IP < ${ftp_reput_name}${line} ;
  fi
done
```

