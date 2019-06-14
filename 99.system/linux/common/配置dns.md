##<center> linux配置dns</center>
### 1） vi /etc/resolv.conf

添加

nameserver 192.168.59.2       本机的网关地址（路由器的地址）

search localdomain    # search 参数指定域名查找顺序

### 2）设置网关

vi /etc/sysconf/network-scripts/ifcfg-eth0

添加

GATEWAY=192.168.59.2

或者

使用命令设置

route add default gw 192.168.59.2

然后重新启动网络服务：

service network restart

### 3）确保可用DNS解析

[root@localhost Desktop]# grep hosts /etc/nsswitch.conf

参考：http://www.centoscn.com/CentosBug/osbug/2014/0306/2508.html