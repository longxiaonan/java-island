# 项目管理利器：jira的安装和配置

jira可以作为项目管理工具和bug提交平台。

上家公司用的禅道，本公司用的jira，感觉还是jira顺手些。

[TOC]

官网地址：

https://cn.atlassian.com/software/jira/features

安装和启动方式：

https://confluence.atlassian.com/adminjiraserver072/installing-jira-applications-on-linux-from-archive-file-857048170.html

第一次启动需要设置数据库连接，需要手动配置jdbc驱动程序：
https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-mysql-5-7-966063305.html

平台配置：

https://confluence.atlassian.com/jirasoftwareserver/doing-more-with-your-agile-projects-938845146.html

## 安装和环境配置

其实官方文档已经非常详细了，我在这里介绍下通过`jira`+`mysql`实现的过程。

### 下载

```shell
cd /data
# wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-8.2.2.tar.gz
```

> 现在最新版本是8.4.1，如果要下载最新版本，将8.2.2替换成新的版本即可。

### 解压

```shell
# tar -xf atlassian-jira-software-8.2.2.tar.gz
# ls -l
drwxr-xr-x. 13 root root      4096 Jun 13 00:38 atlassian-jira-software-8.2.2-standalone
-rw-r--r--.  1 root  root  314934527 Sep 22 21:23 atlassian-jira-software-8.2.2.tar

```

### 修改启动端口

```shell  
vim /data/jira/atlassian-jira-software-8.2.2-standalone/conf/server.xml 
```

将 **Server** port (8005) and the **Connector** port (8080)改成能使用的端口。

我将8005改成了5005,8080改成了5050。

```shell
<Server port="5005" shutdown="SHUTDOWN">
...
   <Service name="Catalina">
		<Connector port="5050" relaxedPathChars="[]|" relaxedQueryChars="[]|{}^&#x5c;&#x60;&quot;&lt;&gt;"
                   maxThreads="150" minSpareThreads="25" connectionTimeout="20000" enableLookups="false"
                   maxHttpHeaderSize="8192" protocol="HTTP/1.1" useBodyEncodingForURI="true" redirectPort="8443"
                   acceptCount="100" disableUploadTimeout="true" bindOnInit="false"/>

```

### 配置工作home目录

创建home目录

```shell
# mkdir /data/jira/jirasoftware-home
```

编辑配置文件，在文件里面指定home目录

```shell
# vim /data/jira/atlassian-jira-software-8.2.2-standalone/atlassian-jira/WEB-INF/classes/jira-application.properties
```

改成如下配置：

```shell
jira.home = /data/jira/jirasoftware-hom
```

### 修改数据库配置文件：
```shell
# vim /data/jira/jirasoftware-home/dbconfig.xml
```

配置为对应的`数据库连接`，`用户名`和`密码`：

```shell
<jira-database-config>
  <name>defaultDS</name>
  <delegator-name>default</delegator-name>
  <database-type>mysql</database-type>
  <jdbc-datasource>
    <url>jdbc:mysql://address=(protocol=tcp)(host=127.0.0.1)(port=3306)/jira?useUnicode=true&amp;characterEncoding=UTF8&amp;sessionVariables=default_storage_engine=InnoDB</url>
    <driver-class>com.mysql.jdbc.Driver</driver-class>
    <username>root</username>
    <password>123456</password>
```

### 设置数据库驱动包

如果使用mysql数据库，jira需要额外添加数据库驱动包才能连接到数据库。

下载驱动包：

```shell
# wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
```

解压：

```shell
# tar -xf mysql-connector-java-5.1.47.tar.gz
```

放置驱动包到安装目录的lib下：

```shell
cp /data/jira/mysql-connector-java-5.1.47/mysql-connector-java-5.1.47.jar  /data/jira/atlassian-jira-software-8.2.2-standalone/lib/
```

### 启动

```shell
/data/jira/atlassian-jira-software-8.2.2-standalone/bin/start-jira.sh
```

> start-jira.sh是启动，stop-jira.sh是停止。



### 访问地址：

```
http://192.168.1.254:5050
```

## 破解：

首先，请支持正版！破解方式也很简单：

1. 将一个jar包复制到jira的指定目录：

```shell
cp /data/jira/atlassian-extras-3.2.jar /data/jira/atlassian-jira-software-8.2.2-standalone/atlassian-jira/WEB-INF/lib/
```

atlassian-extras-3.2.jar包的下载地址：

链接：https://pan.baidu.com/s/1sG-upiJZNnHq13v-fZXKSg 
提取码：t1c2 

> atlassian-extras-3.2.jar包放置后，有两种方式可以选择：
>
> 1. 安装下文的申请license和填写license方式。
> 2. 直接在上面的[网盘地址](https://pan.baidu.com/s/1sG-upiJZNnHq13v-fZXKSg)下载压缩包atlassian-jira-software-8.2.2-standalone_pojie.zip，这个就是破解后的安装程序，是在上面的安装完成的基础上已经进行的破解，已经修改了启动端口为`5050`，配置了home目录`/data/jira/jirasoftware-home`，修改了数据库连接`localhost:5050`，配置了数据库驱动包`mysql-connector-java-5.1.47.tar.gz`。

2. 登陆[官网](<https://www.atlassian.com/software/jira>)，注册，登录。

   ![](https://user-gold-cdn.xitu.io/2019/9/28/16d75f921afdb288?w=200&h=300&f=png&s=30397)

   3. 获取到一个license

![](https://user-gold-cdn.xitu.io/2019/9/28/16d75fea73e39e99?w=1190&h=545&f=png&s=129793)

4. 登陆安装的jira，填入申请的信息即可。

   jira url：http://192.168.1.254:5050

   ![](https://user-gold-cdn.xitu.io/2019/9/28/16d7601d56762104?w=1855&h=611&f=png&s=132148)

## 平台设置

平台功能非常多，[官方文档](https://confluence.atlassian.com/jirasoftwareserver/doing-more-with-your-agile-projects-938845146.html)很详细，我在这里介绍下官网文档中没有的功能：

### 工作流添加`后处理功能`

在项目设置中，编辑工作流，选择一条流程线，点击`后处理功能`。

![](https://oscimg.oschina.net/oscnet/da5b17998c0baa7aaa53aafaa0da126b8a0.jpg)

有很多的问题域可以设置，比如可以在用户点击`任务完成`的时候设置`解决结果`的状态。

![](https://oscimg.oschina.net/oscnet/dbfe697e1204808e9b9ad2eb1e1345b9410.jpg)