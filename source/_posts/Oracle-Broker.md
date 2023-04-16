---
title: Oracle-Broker
date: 2023-03-12 14:59:22
tags:
- Install
categories:
- [Oracle]
- [Oracle11g]
---

11g 刪除 broker 設定值

<!--more-->



1.設定dg_broker_start為false

```sql
alter system set dg_broker_start = FALSE ;
```



2.刪除dg_broker_config_file1/2

> 刪除dg_broker_config_file1/2的檔案



3.設定dg_broker_start為true

```sql
alter system set dg_broker_start = TRUE Scope=both;
```



4.重新配置broker

> show configuration應該要會空才對
