## 后台运行jar程序方式一：

```shell
@echo off 
title 铝模无忧打印客户端
taskkill -f -t -im javaw.exe
start javaw -jar zhirui-tenant-client-0.0.1-SNAPSHOT.jar

echo --------- 铝模无忧打印客户端正在启动中.... ----------
pause
exit
```

> 如果要设置开机启动则需要将bat的快捷方式放置到如下目录：
>
> `C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\startup`



## 后台运行jar程序方式二：

编辑bat文件

* 第一行：启动成功后关闭cmd窗口
* 第二行：删除当前所有的java进程
* 第三行四行：进入指定目录执行java -jar命令
* 第五行：提示程序运行成功

```basic
%1 mshta vbscript:CreateObject("WScript.Shell").Run("%~s0 ::",0,FALSE(window.close)&&exit

tskill java

cd bin

java -jar ../zhirui-tenant-client-0.0.1-SNAPSHOT.jar

msg %username% /w /v 铝模无忧客户端 启动成功！
```



另存时保存为`ANSI`格式

![1563366781099](media/1563366781099.png)