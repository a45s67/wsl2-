# wsl2 安裝紀錄

由於在安裝過程中實在踩太多雷，紀錄一下


[Windows Subsystem for Linux Installation Guide for Windows 10](https://docs.microsoft.com/zh-tw/windows/wsl/install-win10)

首先照著simple的方式安裝，結果wsl --install始終無法成功，我的wsl裡面根本沒吃這個參數  
在"開啟或關閉windows功能"中關了又開，就是沒成功  


後來改成manual install的部分。  
結果一直噴 `RPC 伺服器無法使用`和`0x800706ba`之類的  
亂狗了一下找到這篇:  
https://dotblogs.com.tw/supershowwei/2017/11/16/143448

怪的是在service.msc中，我明明都有開，就剩防火牆不確定。  
於是我跑到vm中試了一下，發現vm也灌不了，然後開防火牆就可以了?!!哭阿  

但我的本機有灌trend的防毒，他取代了防火牆，我沒辦法改規則，就算要關還要公司管理者的授權碼才能關-.-，  
但我總覺得一定有辦法關，只要有更高權限的介面，那種開機後啥都沒load的乾淨介面，於是我她媽想到了一招殺手鐧，下去安全模式搞。  
結果安全模式防毒是拔掉了沒錯，我的wsl也開不了了......(無法在安全模式啟用這項服務)  

又照關鍵字狗了一下，發現一篇開office的，[這篇](https://docs.microsoft.com/zh-tw/office/troubleshoot/office-suite-issues/office-not-start-safe-mode-windows)  
我發現了新天地，在 msconfig 中用選擇性啟動把trend給拔掉就好。拿掉防毒了omg。  
(過了一天後重新看了一次，其實沒拿掉-.-，還是掛在上面)

結果防火牆設了還是沒辦法......回想起狗到的一篇[Win10安装WSL2及Ubuntu20.04子系统](https://blog.mjyai.com/2020/06/01/win10-wsl2-ubuntu/)，  
開始將注意力放在LcxxManager上，

```
sc stop LxssManager
sc start LxssManager
```
結果照著做還是不行，跑到service.msc中去開他，結果存取被拒，為了確認這到底怎麼開，我又查到[Doc Idea: when you need to restart LxssManager service](https://github.com/microsoft/WSL/issues/634),[issue:Could not let LXSSMANAGER autostart](https://github.com/Microsoft/WSL/issues/3585)，  
原來可以用`sc query LxssManager`去看= =!!!看了確定了，sc start是開不起來的  
於是照著["issue:Could not let LXSSMANAGER autostart"](https://github.com/Microsoft/WSL/issues/3585)設定了登錄檔，牛逼，真的很強，到底怎麼知道登錄檔的  

```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LxssManager]
"Start"=dword:2
```
終於解決，不再噴`RPC 伺服器無法使用`了!!  
其中登錄檔的值代表的意思在[请问dns client无法进行停止/恢复/重新启动等操作（按钮是灰色的）是什么原因?](https://www.zhihu.com/question/264016881)給出了解釋，  

```
2: auto
3: manual
4: stop
```

但之後噴了，`access is denied`，又爬了一堆，最後是將wsl2的vhdx移到別的資料夾

```
wsl --export Ubuntu-20.04 ubuntu.tar
wsl --unregister Ubuntu-20.04
mkdir D:\wsl-linux
wsl --import Ubuntu-20.04 ubuntu.tar D:\wsl-linux
(參數的細節可能有錯，靠印象打的，但大概流程就是這樣)
```

然後`Program files\WhatApps`權限全開(右鍵-內容->安全性->進階設定->改owner改access)

現在沒`access is denied`了，但噴了`0x80041002`
