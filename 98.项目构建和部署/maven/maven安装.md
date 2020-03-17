```shell
#------------------------------ Maven 安装 ------------------------------
# 部署Maven项目，必须安装.  下载页：http://maven.apache.org/download.cgi 
wget http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.2/binaries/apache-maven-3.6.2-bin.tar.gz
tar -zxf apache-maven-3.6.2-bin.tar.gz
mv apache-maven-3.6.2
cd apache-maven-3.6.2/conf
vim settings.xml    
# <localRepository>/path/to/local/repo</localRepository> 是下载jar文件时的存储路径

# 在<mirrors>中增加aliyun</mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    
# 再将文件settings.xml  copy 到 /root/.m2 (用户根路径)文件夹下
# 配置环境变量：
vim /ect/profile
#maven environment
export M2_HOME=/opt/maven
export PATH=$M2_HOME/bin:$PATH

source /ect/profile
mvn -v
```

