最近在操作Jenkins时，忘记了管理员密码。只好重置了：

先停止tomcat服务，然后 vim /root/.jenkins/users/admin_***/config.xml 文件，

找到 <passwordHash>字段，将里面的内容替换为：

```
#jbcrypt:$2a$10$MiIVR0rr/UhQBqT.bBq0QehTiQVqgNpUGyWW2nJObaVAM/2xSQdSq
```

这样，管理员账号的密码就变成了123456，启动tomcat，打开Jenkins登录。

