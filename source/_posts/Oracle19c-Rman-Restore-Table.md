---
title: Oracle19c_Rman_Restore_Table
date: 2023-02-12 19:16:37
tags:
- [Rman]
categories:
- [Oracle]
- [Oracle19c]
---

Oracle 19c 新功能 : Rman Restore table

<!--more-->



## Recover Table

```
recover table SchemaName.TableName until time "to_date('2020-06-12 15:07:08','yyyy-mm-dd hh24:mi:ss')" auxiliary destination 'c:\ap\recover_temp';
```





## 目錄架構

![image-20230212192032400](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676200832.png)





## 執行的Log

![image-20230212192558673](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676201158.png)



![image-20230212192647009](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676201207.png)



![image-20230212192711109](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230212_1676201231.png)

暫存目錄檔案沒自動刪除乾淨...





## 結論

> 1.在舊版本就只能restore配合skpi tablespace才能完成這個需求
>
> 2.在還原Table的過程中是需要一定的硬碟空間需求
>
> 3.Table取出後要記清除產生的暫存檔案
