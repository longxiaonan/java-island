### <center>linux常用命令</center>
[toc]
### 磁盘命令
```shell
[root@zhirui-server tmp]# du -sh
12K	.
[root@zhirui-server tmp]# du -sh *
0	a.txt
4.0K	err
4.0K	input
4.0K	out
[root@zhirui-server tmp]# df -h
文件系统                                容量  已用  可用 已用% 挂载点
/dev/mapper/centos_zhirui--server-root   50G   46G  4.7G   91% /
devtmpfs                                7.8G     0  7.8G    0% /dev
tmpfs                                   7.8G     0  7.8G    0% /dev/shm
tmpfs                                   7.8G  149M  7.7G    2% /run
tmpfs                                   7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/sda1                              1014M  271M  744M   27% /boot
/dev/mapper/centos_zhirui--server-home  142G   39M  142G    1% /home
tmpfs                                   1.6G   36K  1.6G    1% /run/user/0
[root@zhirui-server tmp]# df -hl
文件系统                                容量  已用  可用 已用% 挂载点
/dev/mapper/centos_zhirui--server-root   50G   46G  4.7G   91% /
devtmpfs                                7.8G     0  7.8G    0% /dev
tmpfs                                   7.8G     0  7.8G    0% /dev/shm
tmpfs                                   7.8G  149M  7.7G    2% /run
tmpfs                                   7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/sda1                              1014M  271M  744M   27% /boot
/dev/mapper/centos_zhirui--server-home  142G   39M  142G    1% /home
tmpfs                                   1.6G   36K  1.6G    1% /run/user/0
[root@zhirui-server tmp]# 
```
### kill命令
杀死指定线程，kill -9强制杀死线程
#### ps、grep和kill联合使用杀掉进程
##### kill -9 `ps -ef|grep 777|awk '{print $2}'` 
杀掉名称中有777的进程，或者说给进程名中有777的发送signal 9信号
打印一行中的第二列
这个比较危险。如果有个进程进程号是777,x777,xx777的都会被杀掉。
ps -ef|grep 777|awk '{print $2}' : 得到用ps查看时,在一行中含有"777"字符的该进程的进程号
kill -9 强制杀死改进程
awk测试：
```shell
[root@zhirui-server wms]# ps -ef | grep we-server
root      47126  47046  0 20:42 pts/1    00:00:00 grep --color=auto we-server
root     104341      1  0 5月22 ?       01:45:26 java -Dlog.rootPath=/wms/logs/ -Dpro.log.level=ERROR -Dconfig.profile=dev -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8 -server -Xms512m -Xmx512m -XX:MaxMetaspaceSize=512m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -jar /wms/services/zhirui-wms-we-server/lib/zhirui-wms-we-server-0.0.1-SNAPSHOT.jar
[root@zhirui-server wms]# ps -ef | grep we-server awk 'NR==1{print $1}'
grep: awk: 没有那个文件或目录
grep: NR==1{print $1}: 没有那个文件或目录
[root@zhirui-server wms]# ps -ef | grep we-server awk 'NR==1{print $1}'
grep: awk: 没有那个文件或目录
grep: NR==1{print $1}: 没有那个文件或目录
[root@zhirui-server wms]# ps | grep we-server awk 'NR==1{print $1}'
grep: awk: 没有那个文件或目录
grep: NR==1{print $1}: 没有那个文件或目录
[root@zhirui-server wms]# ps -ef | grep we-server | awk 'NR==1{print $1}'
root
[root@zhirui-server wms]# ps -ef | grep we-server | awk 'NR==1{print $2}'
47145
[root@zhirui-server wms]# ps -ef | grep we-server | awk '{print $2}'
47176
104341
[root@zhirui-server wms]#
```
##### ps -ef |grep hello |awk '{print $2}'|xargs kill -9
同上个命令, 是上个命令的xargs方式

