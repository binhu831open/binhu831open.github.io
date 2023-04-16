---
title: SQL_FailOver_Cluster
date: 2023-02-19 21:17:39
tags: 
 - Install
categories:
 - [HA]
 - [MSSQL]
---

前罝作業

<!--more-->

> 1.測SQL FailOver Cluster是要AD和iSCSI
>
> 2.是測AlwaysOn可不用AD，可用工作群組設
>
> 3.方便測試可關畢:密碼政策、防火牆
>
> 4.不論是FailOver Cluster或AlwaysOn都需要Windows Failover Cluster
>
> 5.FailOver Cluster安裝時要留意安裝SQL Home時的硬碟有沒有選錯，進FailOver Cluster Manager確認一下，以免安裝錯誤要調整或重來一次都麻煩
>
> 6.TLS 1.0 要考慮是否要開起，舊版的SQL並不支援TLS 1.2
>
> 7.遠端登入要開起
>
> 8.Windows Update記的要做，以免少Patch
>
> 9.AD有包含DNS的話，記的把C:\Windows\System32\drivers\etc\hosts內容盡可能清空，以免讓DNS和hosts裏面的內容出現衝突



------



## TLS 1.0 開起方式

```
1.regedit
2.HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols
3.建新目錄TLS 1.0，下面再建Client和Server，各別新增2個 DWORD 
  --DisabledByDefault  (值設為0)
  --Enabled  (值設為1)
4.重開OS後，用powershell進行檢查
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client" -Name DisabledByDefault
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Server" -Name DisabledByDefault
```





------

## Windows FailOver Cluster留意事項

### 設定Quorum Disk(仲裁磁碟)

![image-20230219222207793](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230219_1676816546.png)



![image-20230219222254144](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230219_1676816583.png)



![image-20230219222429300](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230219_1676816669.png)

選擇那個硬碟要當仲裁磁碟

![image-20230219222509987](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230219_1676816710.png)

沒錯誤的話，應該會是Disk Witness in Quorum

![image-20230219222636220](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230219_1676816796.png)



------



## SQL FailOver Cluster錯誤處理

### 多張網卡

![image-20230222231334277](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230222_1677078814.png)

> 設定網卡只留一個cluster and client在除錯上會容易很多





### DNS

![image-20230222231803452](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230222_1677079083.png)

> 1.安裝windows failover cluster 的帳號要有異動dns和查詢node主機資源的權限，用較大的權限建立避免一些權限問題
>
> 2.當windows/sql 的 failover cluster的vip無法自動建立在dns裏時，可手動補建。相對來說，如有產生錯誤的dns記錄的話，直接手動刪除就可以
>
> 3.node主機網卡的第一個dns ip建議寫網域的





### Remote Registry 權限

> Run wmimgmt.msc



![image-20230223221906009](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677161946.png)



![image-20230223222215469](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677162135.png)



![image-20230223222403422](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677162243.png)



![image-20230223222511496](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677162311.png)







###　DCOM 權限

> Run dcomcnfg





![image-20230223223048305](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677162648.png)



![image-20230223223240744](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677162760.png)



![image-20230223223331758](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677162811.png)





### WMI 權限

> Run dcomcnfg





![image-20230223223914623](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677163154.png)





![image-20230223224057210](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677163644.png)





![image-20230223224316383](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677163396.png)







### 修正SQL Server Agent是null的情況

![image-20230223230823870](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677164904.png)



#### PowerShell

```
$ClusterName = "sqlcluster"
$FciClusterGroupName = "SQL Server (MSSQLSERVER)"
Add-ClusterResourceType -Name "SQL Server Agent" -Dll "sqagtres.dll" 
```

![image-20230223231450729](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677165290.png)





![image-20230223231647851](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677165408.png)





![image-20230223233943452](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677166783.png)



#### SQL Server Agent進行配置

![image-20230223234023893](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677166824.png)



![image-20230223234103405](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677166863.png)





![image-20230223235918061](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677167958.png)

重起SQL FailOver Cluster後再用SSMS試看看能否正常登入DB



## Add Node to SQL FailOver Cluster不知的錯誤

> 遇到該檢查和確認的都沒問題，但就是在安裝過程中出現錯誤，試的檢查 regedit : Primary DB主機，這些參數都要為1

![image-20230223235555095](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677167755.png)



------

## SQL FailOver Cluster安裝

### Node A

![image-20230220212455255](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676899515.png)



![image-20230220212522748](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676899539.png)



![image-20230220212553903](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676899554.png)





![image-20230220212640200](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676899600.png)





![image-20230220223021981](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903422.png)





![image-20230220223135663](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903495.png)



![image-20230220223208973](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903529.png)



![image-20230220223331211](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903611.png)



我網卡沒配置好，不然應該是要設成固定IP較佳

![image-20230220223353803](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903634.png)



![image-20230220223532356](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903732.png)



![image-20230220223623645](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903783.png)



Share Disk

![image-20230220223713560](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903833.png)



![image-20230220223803736](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903883.png)





![image-20230220223856159](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903936.png)



![image-20230220223929225](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676903969.png)





![image-20230220225643580](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230220_1676905003.png)

> 可先建立起後，再進windows failover cluster manager進行重新檢查sql failover cluster無法起動的原因
>
> 檢查:TLS支援度、各node、vip 主機名稱在DNS裏是不是正確、防火牆、起動sql服務的ad帳號權限是否有node主機的administrator權限



### Node B 

新增node到sql failover cluster

![image-20230222234331936](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230222_1677080612.png)



![image-20230222234401864](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230222_1677080642.png)



![image-20230222234420672](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230222_1677080660.png)



![image-20230222234439731](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230222_1677080679.png)



![image-20230223224829524](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677163709.png)



![image-20230223224932838](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677163787.png)





![image-20230223225026687](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677163826.png)



![image-20230223235005122](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230223_1677167405.png)





![image-20230224000200930](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230224_1677168121.png)





![image-20230224000917091](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230224_1677168557.png)







重起原本的primdy db主機後，可以在另一台主機看到已正確接手

![image-20230224001317302](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230224_1677168797.png)



![image-20230224001327038](https://raw.githubusercontent.com/binhu831open/image/master/2023/02/upgit_20230224_1677168807.png)



## 結論:

> 1.安裝SQL的主機權限要額外設定，不然就算用Administrator安裝也一樣報錯
>
> 2.DCOM的權限是跟SSIS有關
>
> 3.WMI和Reomte Registry的權限問題是有關連的，要一併處理
>
> 4.FailOver Cluster的價錢比AlwaysOn便宜，並且無AlwaysOn切換時有資料不一致問題。如Storage有保護機制的，FailOver Cluster也不是無法使用的HA方案

