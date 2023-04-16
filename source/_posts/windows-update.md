---
title: windows_update
date: 2023-02-13 23:31:03
tags:
- [Script]
categories:
- [Windows]
---

Update失敗時解決方案1

<!--more-->

```bash
net stop wuauserv
--停止update服務

C:\Windows\SoftwareDistribution
--將目錄更名後，再重建一個SoftwareDistribution

net start wuauserv
--起動update服務

再試看看了~
```

