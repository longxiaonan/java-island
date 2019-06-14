 JDK的安装
1.1     第一步卸载已经安装好的open-jdk
Ø  查看已安装的Java：
    rpm -qa | grep java

 
Ø  卸载OPENJDK

卸载的命令如上图，前半部分如下：
rpm -e --nodeps 包名
 
tip: 如果是自己安装的那么通过rpm -qa | grep java是查看不到的, 那么怎么卸载呢, 通过查看环境变量那里找到安装目录, 然后删除..
1.2     第二步创建jdk的安装路径
Ø  在/usr/local/src/ 下创建java目录
cd /usr/local/src/
mkdir java
 
1.3     第三步上传jdk安装文件到linux
Ø  使用xftp 上传文件到 /usr/local/src/ java目录
 

 

 
 
1.4     第四步解压文件
Ø  解压JDK安装文件
tar -zxvf jdk-7u55-linux-i586.tar.gz
 
注意：如果出现错误，需要安装依赖包 glibc.i686，如下：
yum install glibc.i686
tar -zxvf jdk-7u55-linux-i586.tar.gz
1.5     第五步配置环境变量
Ø  使用vim 编辑 profile文件。（需要记住）
    vim /etc/profile
 
Ø  在行末尾添加如下内容：
#set java environment
JAVA_HOME=/usr/src/java/jdk1.7.0_55
CLASSPATH=.:$JAVA_HOME/lib.tools.jar
PATH=$JAVA_HOME/bin:$PATH
export JAVA_HOME CLASSPATH PATH
Ø  保存退出
1.6     第六步加载配置文件
         配置完成之后，为了让其生效，需要加载配置文件，加载命令如下：
source /etc/profile
 
 
1.7     第七步测试
java -version