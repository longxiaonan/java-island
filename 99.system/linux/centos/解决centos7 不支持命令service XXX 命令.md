# 解决centos7 不支持命令service XXX 命令

## 现象如下：

```shell
[root@zm 2.0]# service iptables save
The service command supports only basic LSB actions (start, stop, restart, try-restart, reload, force-reload, status). For other actions, please try to use systemctl.
```

## 先停止防火墙

```shell
[root@zm sysconfig]# systemctl stop firewalld
```

## 按照防火墙

```shell
[root@zm sysconfig]# yum install iptables-services
```

## 启用防火墙

```shell
[root@zm sysconfig]# systemctl enable iptables
Created symlink from /etc/systemd/system/basic.target.wants/iptables.service to /usr/lib/systemd/system/iptables.service.
```

## 开启防火墙

```shell
[root@zm sysconfig]# systemctl start iptables 
```

> 还可以试着重启防火墙
>
> ```shell
> [root@zm sysconfig]# service iptables restart
> Redirecting to /bin/systemctl restart iptables.service
> ```

## 添加配置

```shell
[root@zm sysconfig]# iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -j MASQUERADE
[root@zm sysconfig]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```

