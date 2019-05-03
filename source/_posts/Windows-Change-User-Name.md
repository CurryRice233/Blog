---
title: Windows 10 更换用户名
date: 2019-05-03 21:35:55
categories: Windows
tags:
	- Windows
---

作为一个中文用户，在安装软件的时候，特别是一些国外的小众软件，经常会碰到软件不支持中文路径。
碰到这种情况，一般我们换一个全英文的路径就OK了。
但是，偶尔会有一些极端情况。

今天我在 [RStudio](https://www.rstudio.com/) 在运行 `plot()` 的时候出现了下面的错误
```
Error in gzfile(file, "wb") :cannot open the connection

In addition: Warning message:

In gzfile(file, "wb") :

cannot open compressed file'C:/Users/????/AppData/Local/Temp/RtmpCuJjwT/rs-graphics-304ed13f-1958-4eeb-930a-24d61ef9cef2/7c68cc44-8d16-47cd-a3ba-93c854098270.snapshot',probable reason 'Invalid argument'

Graphics error: Plot rendering error

```
问题出在中文用户名上，普通的修改用户名并不会修改路径。需要在注册表里修改
首先需要创建一个新的管理员账户，注销现在这个，登陆刚建的账户。
重命名用户文件夹 `C:\Users\咖喱饭` 改为 `C:\Users\CurryRice`
`win+R` 输入 `regedit` 打开注册表，找到
`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`
在这一页的其中一条里（S-1-5-21...）
将 `ProfileImagePath` 的值改为 `C:\Users\CurryRice`
之后就重新登陆主账户，删掉新建的

这样修改之后 `plot()` 就正常运行不报错了

本以为万事大吉，没想到之后打开 [IBM RSA](https://www.ibm.com/developerworks/cn/rational/kunal/index.html) 的时候
提示 `Invalid Configuration Location` 还是原来的中文路径
在折腾了两个多小时后，找到解决办法，同样是修改注册表，在
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
在这一页有一大堆还是中文路径的，将他们修改为英文的就行了

重新打开 RSA 就能成功运行了
