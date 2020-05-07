---
title: 去掉激活win10水印
date: 2017-11-24 20:30:10
tags: system
---

右键左下角Win图标，打开命令提示符（管理员）。

使用如下命令即可激活Windows 10：
```
slmgr.vbs /upk
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms zh.us.to
slmgr /ato
```
注意是复制一行执行一行！

测试可以^_^