---
title: 컴퓨터 02 - addware 제거
date: 2020-12-01
categories: 
---





``` 
sc stop "PcDrmSvc"
sc config "PcDrmSvc" start=disabled

sc delete "PcDrmSvc"
```







## Reference

windowexe.com/bbs/biard.php