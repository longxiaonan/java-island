# linux常用命令

[TOC]
### 系统相关命令

#### top命令

https://juejin.im/post/5d590126f265da03db0776b6?utm_source=gold_browser_extension

##### load Average

**在一段时间内，CPU正在处理以及等待CPU处理的进程数之和。**三个数字分别代表了1分钟，5分钟，15分钟的统计值。top中的其他参数具体介绍如下：

```shell

第一行：
top - 20:41:08 up 18 days,  5:24,  2 users,  load average: 0.04, 0.03, 0.05
top：当前时间
up：机器运行了多少时间
users：当前有多少用户
load average：分别是过去1分钟，5分钟，15分钟的负载
```

> 一个CPU在一个时间片里面只能运行一个进程，CPU核数的多少直接影响到这台机器在同时间能运行的进程数。所以一般来说Load Average的数值别超过这台机器的总核数，就基本没啥问题。

```shell
第二行：

Tasks: 216 total,   1 running, 215 sleeping,   0 stopped,   0 zombie

Tasks：当前有多少进程
running：正在运行的进程
sleeping：正在休眠的进程
stopped：停止的进程
zombie：僵尸进程
```

> running越多，服务器自然压力越大。

```shell
第三行：

%Cpu(s):  0.2 us,  0.1 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

us: 用户进程占CPU的使用率
sy: 系统进程占CPU的使用率
ni: 用户进程空间改变过优先级
id: 空闲CPU占用率
wa: 等待输入输出的CPU时间百分比
hi: 硬件的中断请求
si: 软件的中断请求
st: steal time
```

> 这一行代表了CPU的使用情况，us长期过高，表明用户进程占用了大量的CPU时间。us+sy如果长期超过80或者90，可能就代表了CPU性能不足，需要加CPU了

```shell
第四行&第五行
KiB Mem : 65810456 total, 30324416 free,  9862224 used, 25623816 buff/cache
KiB Swap:  7999484 total,  7999484 free,        0 used. 54807988 avail Mem

total：内存总量
free：空闲内存
used：使用的
buffer/cache： 写缓存/读缓存
```

> 第四第五行分别是内存信息和swap信息。所有程序的运行都是在内存中进行的，所以内存的性能对与服务器来说非常重要。不过当内存的free变少的时候，其实我们并不需要太紧张。真正需要看的是Swap中的used信息。Swap分区是由硬盘提供的交换区，当物理内存不够用的时候，操作系统才会把暂时不用的数据放到Swap中。所以当这个数值变高的时候，说明内存是真的不够用了。

```shell
第五行往下

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                                                                                  
19868 root      20   0 19.733g 369980  15180 S   0.7  0.6 129:53.91 java                                                                                                                                                                     
19682 root      20   0 19.859g 5.766g  22252 S   0.3  9.2 139:42.81 java                                                                                                                                                                     
54625 100       20   0   50868  33512   4104 S   0.3  0.1   0:04.68 fluentd                                                                               

PID:进程id
USER:进程所有者
PR:优先级。数值越大优先级越高
NI:nice值，负值表示高优先级，正值表示低优先级
VIRT:进程使用的虚拟内存总量
SWAP:进程使用的虚拟内存中被换出的大小
RES:进程使用的、未被换出的物理内存大小
SHR:共享内存大小
SHR:共享内存大小
S：进程状态。D表示不可中断的睡眠状态；R表示运行；S表示睡眠；T表示跟踪/停止；Z表示僵尸进程。
%CPU:上次更新到现在的CPU占用百分比 ；
%MEM:进程使用的物理内存百分比 ；
TIME+:进程使用的CPU时间总计，单位1/100秒；
COMMAND:命令名/命令行
```

> 这些就是进程信息了，从这里可以看到哪些进程占用系统资源的概况。

### 网络命令netstat

```shell
netstat -ano | grep 1194
```



### 文件相关命令

#### scp命令

文件相关命令相关命令

> 如果是传输的是目录需要`-r`参数

```shell
scp -r bin/ root@192.168.1.112:/wms
```





### 磁盘命令

```shell
[root@zhirui-server wms]# du -h   #查看磁盘下文件的大小
396K	./infile/2019/01/08
2.1M	./infile/2019/01/16
24K	./infile/2019/01/25
4.0K	./infile/2019/01/28
4.0K	./infile/2019/01/29
2.5M	./infile/2019/01
180K	./infile/2019/02/15
24K	./infile/2019/02/18
212K	./infile/2019/02/24
12K	./infile/2019/02/25
240K	./infile/2019/02/28
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
```shell
root@weitexun:/home/MiddleService# du -sh ./*
223M    ./backup
16K    ./conf
12K    ./config
21M    ./debug.log
201M    ./logs
148K    ./middle_main.jar
43M    ./middle_main_lib
8.0K    ./output
4.0K    ./run.bat
4.0K    ./run.sh
4.0K    ./start.sh
4.0K    ./tempExcel
32K    ./templates
root@weitexun:/home/MiddleService# cd logs/
root@weitexun:/home/MiddleService/logs# ll
总用量 205012
drwxrwxrwx 2 root root    192512 10月  1 11:19 ./
drwxrwxrwx 9  777 root      4096 9月  30 17:04 ../
-rw-r--r-- 1 root root 104859480 10月  1 11:18 debug.log3469992168401225.tmp
-rw-r--r-- 1 root root 104864077 10月  1 11:18 debug.log3469997842751189.tmp
root@weitexun:/home/MiddleService/logs# cd ..
root@weitexun:/home/MiddleService# df -hl
文件系统                    容量  已用  可用 已用% 挂载点
udev                          16G    0  16G    0% /dev
tmpfs                        3.2G  362M  2.8G  12% /run
/dev/mapper/ubuntu--vg-root  885G  783G  58G  94% /
tmpfs                        16G  152K  16G    1% /dev/shm
tmpfs                        5.0M  4.0K  5.0M    1% /run/lock
tmpfs                        16G    0  16G    0% /sys/fs/cgroup
/dev/sda1                    472M  108M  340M  25% /boot
tmpfs                        3.2G  48K  3.2G    1% /run/user/1000
tmpfs                        3.2G    0  3.2G    0% /run/user/1001





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

