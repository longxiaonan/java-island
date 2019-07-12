# centos基本命令

[TOC]

## 系统相关命令

### 查看系统版本

```shell
[root@izwz9hy7gff0ks6leh846hz bin]# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID:    CentOS
Description:    CentOS Linux release 7.2.1511 (Core) 
Release:    7.2.1511
Codename:    Core
```

### 查看系统内核版本

>  有些服务对内核版本有需求，比如docker，就需要内核版本大于3.0

```shell
[root@izwz9hy7gff0ks6leh846hz bin]# uname -a
Linux izwz9hy7gff0ks6leh846hz 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
```

## 网络相关命令

### 端口相关

#### 查看监听的端口

``` shell
netstat -na|grep 8081
```

