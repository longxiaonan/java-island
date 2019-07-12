[TOC]

# centos虚拟机安装

## 系统安装后ifconfig 命令不存在

这种情况就是我们现在面对的，因为centos7.2的mini版没有安装这个东东，所以我们就直接安装就好了，在终端里面输入：

`yum -y install net-tools`

## centos网络配置方法(网关、dns、ip地址配置)

### 1，配置DNS

`vi /etc/resolv.conf`

添加

```shell
nameserver 192.168.0.1 
nameserver 8.8.8.8
nameserver 1.1.1.1
```

### 2，配置网关：

`vi /etc/sysconfig/network`

添加

```shell
NETWORKING=yes
HOSTNAME=localhost.localdomain
GATEWAY=192.168.0.1
```

### 3，配置ip地址：

`vi /etc/sysconfig/network-scripts/ifcfg-enp0s3`

配置如下

```shell
DEVICE="enp0s3"
HWADDR="00:0C:29:6C:BB:E6"
NM_CONTROLLED="yes"
ONBOOT="no"
NETMASK=255.255.255.0
IPADDR=192.168.0.8
GATEWAY=192.168.0.1
BOOTPROTO=static
ONBOOT=yes
PEERDNS=yes	
```

> 只需要添加ip地址和掩码，修改`BOOTPROTO=static`即可

### 4，重新启动服务：

```shell
/etc/init.d/network restart
# 或使用命令：
service network restart
# 或：
ifdown eth0 and ifup eth0
systemctl restart NetworkManager.service
```

