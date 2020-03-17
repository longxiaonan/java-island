

# boke:maven命令方式  打jar包 & 部署到nexus3

## maven的settings.xml配置：

#### 配置文件的`<servers>`下添加`</server>`

id：用于在mvn命令部署时指定server

username：nexus服务器的用户名

password：nexus服务器的密码

```xml
		<!-- 部署到nexus的时候用于认证 -->
		<server>
			<id>3rd-repo-Releases</id>
			<username>admin</username>
			<password>admin123</password>
		</server>
		<server>
			<id>3rd-repo-Snapshots</id>
			<username>admin</username>
			<password>admin123</password>
		</server>
```

#### 添加镜像的`<mirrors>`下添加`</mirrors>`

一个私服的mirror，一个阿里云的mirro

```shell
		<mirror>
			<id>com.zhirui.group</id>
			<mirrorOf>central</mirrorOf>
			<name>com.zhirui.group</name>
			<url>http://192.168.1.254:8081/repository/maven-public/</url>
		  </mirror>
		<mirror>
			<id>alimaven</id>
			<name>aliyun maven</name>
			<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
			<mirrorOf>central</mirrorOf>
		</mirror>
```

## maven本地打包命令：

### 单项目下直接打包

```shell
clean package -Dmaven.test.skip=true
```

两级项目在父模块打包：

```shell
clean package -pl zhirui-zuul-server/ -am -Dmaven.test.skip=true
```

三级项目在父模块打包：

```shell
clean package -pl zhirui-wms-business/zhirui-handle-file-server/ -am -Dmaven.test.skip=true
```



## 本地jar安装到maven：

mvn install:install-file
-DgroupId=<groupId>       : 设置项目代码的包名(一般用组织名)
-DartifactId=<artifactId> : 设置项目名或模块名
-Dversion=1.0.0           : 版本号
-Dpackaging=jar           : 什么类型的文件(jar包)
-Dfile=<myfile.jar>       : 指定jar文件路径与文件名(同目录只需文件名)

```shell
mvn install:install-file -DgroupId=com.zhirui -DartifactId=j-interop -Dversion=3.0.1 -Dpackaging=jar -Dfile=j-interop-3.0.1.jar
    
mvn install:install-file -DgroupId=com.zhirui -DartifactId=j-interopdeps -Dversion=3.0.1 -Dpackaging=jar -Dfile=j-interopdeps-3.0.1.jar
    
mvn install:install-file -DgroupId=com.zhirui -DartifactId=org.openscada.opc.dcom -Dversion=3.0.1 -Dpackaging=jar -Dfile=org.openscada.opc.dcom-1.0.jar
    
mvn install:install-file -DgroupId=com.zhirui -DartifactId=org.openscada.opc.lib -Dversion=3.0.1 -Dpackaging=jar -Dfile=org.openscada.opc.lib-1.0.jar

mvn install:install-file -DgroupId=com.zhirui -DartifactId=jna-4.2.2 -Dversion=1.0.1 -Dpackaging=jar -Dfile=jna-4.2.2.jar


mvn install:install-file -DgroupId=com.zhirui -DartifactId=open-chart -Dversion=1.0.1 -Dpackaging=jar -Dfile=j-interopdeps-3.0.1.jar
    
mvn install:install-file -DgroupId=com.zhirui -DartifactId=open-common -Dversion=1.0.1 -Dpackaging=jar -Dfile=org.openscada.opc.dcom-1.0.jar
    
mvn install:install-file -DgroupId=com.zhirui -DartifactId=open-real -Dversion=1.0.1 -Dpackaging=jar -Dfile=org.openscada.opc.lib-1.0.jar

mvn install:install-file -DgroupId=com.zhirui -DartifactId=open-report -Dversion=1.0.1 -Dpackaging=jar -Dfile=jna-4.2.2.jar


```



## 本地jar上传到nexus3：

这里的DrepositoryId需要和上文`server`配置的id对应

```shell
mvn deploy:deploy-file -DgroupId=com.zuirui -DartifactId=open-chart -Dversion=1.0.1 -Dpackaging=jar -Dfile=open-chart/1.0.1/open-chart-1.0.1.jar -Durl=http://192.168.1.254:8081/repository/3rd-repo/ -DrepositoryId=3rd-repo-Releases

mvn deploy:deploy-file -DgroupId=com.zuirui -DartifactId=open-common -Dversion=1.0.1 -Dpackaging=jar -Dfile=open-common/1.0.1/open-common-1.0.1.jar -Durl=http://192.168.1.254:8081/repository/maven-releases/ -DrepositoryId=3rd-repo-Releases

mvn deploy:deploy-file -DgroupId=com.zuirui -DartifactId=open-real -Dversion=1.0.1 -Dpackaging=jar -Dfile=open-real/1.0.1/open-real-1.0.1.jar -Durl=http://192.168.1.254:8081/repository/3rd-repo/ -DrepositoryId=3rd-repo-Releases

mvn deploy:deploy-file -DgroupId=com.zuirui -DartifactId=open-report -Dversion=1.0.1 -Dpackaging=jar -Dfile=open-report/1.0.1/open-report-1.0.1.jar -Durl=http://192.168.1.254:8081/repository/3rd-repo/ -DrepositoryId=3rd-repo-Releases
```



## idea部署上传到nexus

### 配置pom

* id`要和上文`server`配置的id对应。
* name随意，不冲突就好。
* url指定需要部署到的nexus定义的仓库。如下图中的`3rd-repo-releases`和`3rd-repo-snapshots`，在仓库中必须存在，否则无法部署到nexus。

```xml
<!--定义snapshots库和releases库的nexus地址-->  
<distributionManagement>
        <repository>
            <id>3rd-repo-Releases</id>
            <name>Local Nexus Repository</name>
            <url>http://192.168.1.254:8081/repository/3rd-repo-releases/</url>
        </repository>
        <snapshotRepository>
            <id>3rd-repo-Snapshots</id>
            <name>Local Nexus Repository</name>
            <url>http://192.168.1.254:8081/repository/3rd-repo-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
```

### 执行部署命令

```shell
mvn clean deploy
```

