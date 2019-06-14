 ### <center>linux下jdk安装</center>
####oracle官网下载jdk。

####解压后的路径： 
`/usr/local/base/jdk1.8.0_181`

####设置环境变量：
vi /etc/profile 
在最后添加如下内容：
```shell
#java environment
export JAVA_HOME=/usr/local/base/jdk1.8.0_181
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
```
####重新加载环境变量：
```shell
[root@izwz9hy3mj62nle7573jv5z jdk1.8.0_181]# source /etc/profile
[root@izwz9hy3mj62nle7573jv5z jdk1.8.0_181]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```
####卸载已经安装好的jdk
查看已安装的Java：
`rpm -qa | grep java`
卸载的命令：
`rpm -e --nodeps 包名`
