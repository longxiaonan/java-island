### <center>zookeeper安装和启动</center>
下载zookeeper注册中心，下载地址：http://www.apache.org/dyn/closer.cgi/zookeeper/

linux下你直接可以找到链接地址后 
```shell
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5.tar.gz
```



解压:
```shell
tar -xzvf  zookeeper-3.5.5.tar.gz
```

将zookeeper-3.4.6/conf目录下的zoo_sample.cfg文件拷贝一份,命名为为zoo.cfg

```shell
cp zoo_sample.cfg zoo.cfg
```


切换到 zookeeper-3.4.6/bin目录下 


启动:
```shell
  ./zkServer.sh start
```

出现一下信息 说明启动成功
```shell
JMX enabled by default
Using config: /usr1/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```