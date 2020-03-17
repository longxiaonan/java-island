# linux环境变量配置

> `vim /etc/profile`在最末尾添加
>
> `source /etc/profile`命令使环境变量重新生效

## java环境变量

```shell
#java environment
#export JAVA_HOME=/usr/local/base/jdk1.8.0_181
exdport JAVA_HOME=/java/jdk/jdk1.8
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin

```

校验环境变量是否生效

```shell
[root@izwz9hy3mj62nle7573jv5z conf]# java -version
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```



## maven环境变量

```shell
#maven environment
MAVEN_HOME=/usr/local/maven
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

校验环境变量是否生效

```shell
[root@izwz9hy3mj62nle7573jv5z conf]# mvn -v
Apache Maven 3.6.0 (97c98ec64a1fdfee7767ce5ffb20918da4f719f3; 2018-10-25T02:41:47+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_181, vendor: Oracle Corporation, runtime: /java/jdk/jdk1.8/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-693.2.2.el7.x86_64", arch: "amd64", family: "unix"

```

## rocktmq环境变量

```shell
#rocktmq环境变量
export ROCKETMQ_HOME=/data/rocket
export PATH=$ROCKETMQ_HOME/bin:$PATH
```

