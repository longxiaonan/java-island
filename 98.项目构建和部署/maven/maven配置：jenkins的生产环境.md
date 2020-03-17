# maven配置：jenkins的生产环境

第三方的jar放在了私服nexus中，且jenkins打包的时候使用普通的maven配置会包冲突等等问题，后面调整成如下的方式后jenkins打包成功。

步骤如下：

1. 备份之前的配置文件：

```shell
mv settings.xml settings.xml.bak
```

2. 新创建`settings.xml`文件

```shell
vim settings.xml
```

3. 在编辑模式下复制下面的配置粘贴保存。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!-- localRepository
     | The path to the local repository maven will use to store artifacts.
     |
     | Default: ${user.home}/.m2/repository
    <localRepository>/path/to/local/repo</localRepository>
    -->
    <proxies>
    </proxies>
    <servers>
        <server>
            <id>releases</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
        <server>
            <id>snapshots</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>

    <mirrors>
    </mirrors>

    <profiles>
        <!-- 发布版(releases)[根据工程pom.xml打包的version决定，version=XXX-RELEASE]将自动采用dev此条件配置 -->
        <profile>
            <id>dev</id>
            <!-- jar包仓库配置 -->
            <repositories>
                <repository>
                    <id>nexus</id>
                    <url>http://192.168.1.254:8081/repository/maven-public/</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </snapshots>
                </repository>
            </repositories>
            <!-- 插件仓库配置 -->
            <pluginRepositories>
                <pluginRepository>
                    <id>nexus</id>
                    <url>http://192.168.1.254:8081/repository/maven-public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
        <!-- 快照版(snapshots)[根据工程pom.xml打包的version决定，version=XXX-SNAPSHOT]将自动采用dev-snapshots此条件配置 -->
        <profile>
            <id>dev-snapshots</id>
            <!-- jar包仓库配置 -->
            <repositories>
                <repository>
                    <id>nexus-snapshots</id>
                    <url>http://192.168.1.254:8081/repository/maven-public/</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                        <checksumPolicy>warn</checksumPolicy>
                    </snapshots>
                </repository>
            </repositories>
            <!-- 插件仓库配置 -->
            <pluginRepositories>
                <pluginRepository>
                    <id>nexus-snapshots</id>
                    <url>http://192.168.1.254:8081/repository/maven-public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <!-- 激活profile -->
    <activeProfiles>
        <activeProfile>dev</activeProfile>
        <activeProfile>dev-snapshots</activeProfile>
    </activeProfiles>

</settings>

```

