# centos7防火墙配置

## 防火墙概述

centos从7开始默认用的是firewalld，iptables的服务是没安装的。如果存在firewalld的基础上又安装了iptables。那么需要两个防火墙都放通端口或者关闭掉，里面的服务才能访问。

> firewalld的启停命令是： 
>
> ```shell
> systemctl stop firewalld.service && systemctl disable firewalld.service
> 
> systemctl start firewalld.service && systemctl enable firewalld.service
> ```
>
> 如果想要改用iptables的话，则需要安装 
>
> ```shell
> yum install iptables-services 
> systemctl stop iptables && systemctl disable iptables
> systemctl start iptables && systemctl enable iptables
> ```
>
> 也可以用如下命令进行启停：
>
> ```shell
> service iptables start
> service iptables stop
> ```
>
> iptables防火墙的端口操作：
>
> ```
> # 开启6379
> /sbin/iptables` `-I INPUT -p tcp --dport 6379 -j ACCEPT
> # 开启6380
> /sbin/iptables` `-I INPUT -p tcp --dport 6380 -j ACCEPT
> # 保存
> /etc/rc``.d``/init``.d``/iptables` `save
> # centos 7下执行
> service iptables save
> ```

## 状态查看

systemctl unmask firewalld.service



### 检查防火墙的状态：

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

### 查看版本：

```shell
firewall-cmd --version
```

### 查看帮助：

```shell
firewall-cmd --help
```

### 查看所有打开的端口：

```shell
firewall-cmd --zone=public --list-ports
```

## 防火墙启停
### 启停命令：

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

## 端口开放和删除：

```shell
#添加端口
[root@test1 ~]# firewall-cmd --zone=public --add-port=8081/tcp --permanent
#移除端口
[root@test1 ~]# firewall-cmd --zone=public --remove-port=80/tcp --permanent

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