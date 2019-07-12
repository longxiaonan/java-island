

## centos7.2防火墙配置

[TOC]

#### 检查防火墙的状态：

```shell
[root@test1 ~]# firewall-cmd --state
not running
```

> 从centos7开始使用systemctl来管理服务和程序，包括了service和chkconfig。这些命令都可以查看状态，只是显示的信息不太一样而已

```shell
[root@test1 ~]# systemctl list-unit-files|grep firewalld.service      
firewalld.service                             enabled 
[root@test1 ~]# systemctl status firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2019-07-11 09:49:26 CST; 12h ago

```



#### 关闭防火墙：

```shell
[root@test1 ~]# systemctl stop firewalld.service      #关闭防火墙

[root@test1 ~]# 
[root@test1 ~]# systemctl disable firewalld.service   #禁止开机启动
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

#### 其他启动相关命令：

```shell
重启防火墙：firewall-cmd --reload
启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctl is-enabled firewalld.service;echo $?
查看已启动的服务列表：systemctl list-unit-files|grep enabled
```

#### 开放端口：

```shell
[root@test1 ~]# firewall-cmd --zone=public --add-port=8080/tcp --permanent
success
[root@test1 ~]# firewall-cmd --zone=public --add-port=50000/tcp --permanent
success
[root@test1 ~]# firewall-cmd --reload     #重启防火墙
success
[root@test1 ~]# firewall-cmd --list-ports
8080/tcp 50000/tcp
[root@test1 ~]# 
```

> 命令含义：
>
> –zone #作用域
>
> –add-port=80/tcp #添加端口，格式为：端口/通讯协议
>
> –permanent #永久生效，没有此参数重启后失效