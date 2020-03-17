对话框

```powershell
@echo off

::对话框
mshta vbscript:msgbox("我是提示内容我是提示内容",64,"我是提示标题")(window.close)

::读取文件内容发给用户
msg * < config/application.properties

pause
```

暂停

```powershell
pause>nul %>nul的作用是不显示请按任意键继续%
```

