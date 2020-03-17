

## 阿里云申请证书

我是在阿里云申请的证书，也可以在七牛等平台申请证书。

登陆阿里云-》控制台

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de63790aec?w=1535&h=608&f=png&s=59947)

点击购买可以申请免费的证书

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de63a6f8c9?w=1920&h=842&f=png&s=148832)

申请成功：

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de642c476a?w=1920&h=842&f=png&s=165073)

点击`下载`将证书下载到本地

>  要通过服务器类型选择下载。

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de6a42041e?w=1898&h=804&f=png&s=147815)

## nginx服务器配置证书

```shell
vim nginx.conf
```

配置web服务器：

```shell
location / {
	root   /wms/website;
	index  index.html index.htm;
}
```

> web文件的目录为/wms/website，只能用上面的root和index的方式。当如下配置的时候无法访问，不知道为什么。
>
> ```shell
> location / {
>          #allow all;
>          #root /wms/erp_ui;
>          #index index.html index.htm;
>          alias /wms/erp_ui;
>          try_files $uri $uri/ @router;
>      }
> 
> ```

servicer中指定证书的位置，详细配置如下：

```shell
    server {
        listen       443;
        server_name  localhost zhirui-tech.com www.zhirui-tech.com;

        ssl_certificate     cert/2929712_zhirui-tech.com.pem;
        ssl_certificate_key  cert/2929712_zhirui-tech.com.key;

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl on;

        #ssl_ciphers  HIGH:!aNULL:!MD5;
        #ssl_prefer_server_ciphers  on;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        location / {
            root   /wms/website;
            index  index.html index.htm;
        }
        #location / {
        #    alias /wms/website;
        #    try_files $uri $uri/ @router;
        #}
    }
```

> ssl_开头的都需要配置。`ssl_certificate`部分配置的是相对路径，也就是在nginx的安装目录下：
>
> ```shell
> [root@izwz9hy3mj62nle7573jv5z cert]# pwd
> /etc/nginx/cert
> [root@izwz9hy3mj62nle7573jv5z cert]# ll
> 总用量 12
> -rwxrwxr-x 1 root root 1679 10月 12 17:19 2929712_zhirui-tech.com.key
> -rwxrwxr-x 1 root root 4067 10月 12 17:35 2929712_zhirui-tech.com_nginx.zip
> -rwxrwxr-x 1 root root 3688 10月 12 17:19 2929712_zhirui-tech.com.pem
> ```

重启nginx：

```shell
/usr/sbin/nginx -c /etc/nginx/nginx.conf -s reload
```



## tomcat服务器配置证书

编辑`tomcat/conf/server.xml`文件，进行如下配置：

```xml
<Connector port="443"
		protocol="org.apache.coyote.http11.Http11Protocol"
		SSLEnabled="true"
		scheme="https"
		secure="true"
		keystoreFile="pfx所在地的完整路径"
		keystoreType="PKCS12"
                keystorePass="图中你自己秘钥所在地"
		clientAuth="false"
		SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"
		ciphers="TLS_RSA_WITH_AES_128_CBC_SHA,TLS_RSA_WITH_AES_256_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA,TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_128_CBC_SHA256,TLS_RSA_WITH_AES_256_CBC_SHA256"/>
 
```

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de6b71509e?w=1121&h=440&f=png&s=63719)

配置后重启tomcat即可。

## springboot配置证书

### 证书转换

springboot是内置的tomcat，需要用JKS证书类型。在第一步的 阿里云申请证书 步骤中下载证书的时候，选择tomcat类型，点击 帮助 参考如何转换成JKS证书。

![](https://user-gold-cdn.xitu.io/2019/11/6/16e3ee63a10acec5?w=1873&h=730&f=png&s=138633)

### 拷贝密钥

从下载的压缩包中的 Tomcat 目录将证书（文件后缀为`jks`）拷贝到 SpringBoot 工程的 `resources` 目录下即可：

![](https://user-gold-cdn.xitu.io/2019/11/6/16e3f0ddde8cf56b?w=353&h=238&f=png&s=38638)

### 配置springboot配置文件

证书拷贝完整后，打开配置文件 `application.yml`，然后修改网站的端口为 `443`，`key-store-password` 可以在证书压缩包的 Tomcat 文件夹下的文本文件中找到，此外配置一下 SSL 证书的类型以及路径等信息即可：

```yaml
server:
  address: 0.0.0.0
  port: 443
  ssl:
    enabled: true
    key-store: classpath:javasea.com.jks
    key-store-password: xxxxxxxxxxxx
    key-store-type: JKS
```

至此便可以启动运行项目，可以在地址栏输入 `https://yourdomain` 即可访问。

### 将 HTTP 请求重定向到HTTPS

网站虽然升级到了 HTTPS，但是很多老用户并不知情，当他们访问旧版的 HTTP 地址时会发现网站已经无法访问，需要改造项目将http重定向到https。

#### 添加 HTTP 端口配置

首先在配置文件中添加自定义的 HTTP 端口配置：

```yaml
http-port: 80
```

#### 建立重定向关系

新建一个配置类 `HttpsConfiguration`，在类中将配置文件中自定义的 HTTP 端口和 HTTPS 的端口都注入进来，然后创建一个新的 `Connector` 来处理 HTTP 请求，同时设置 `Connector` 的端口为注入的 HTTP 端口，重定向端口（`setRedirectPort`）为新的 HTTPS 端口。

```java
import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HttpsConfiguration {
    @Value("${http-port}")
    private int port;

    @Value("${server.port}")
    private int sslPort;

    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            //上面代码中需要特别注意的是，TomcatServletWebServerFactory 必须要在其 postProcessContext 方法中添加 HTTP 的匹配范围 addPattern("/*")，否则重定向无效
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(redirectConnector());
        return tomcat;
    }

    private Connector redirectConnector() {
        Connector connector = new Connector(TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
        connector.setScheme("http");
        connector.setPort(port);
        connector.setSecure(false);
        connector.setRedirectPort(sslPort);
        return connector;
    }
}
```



## 七牛云配置证书和CDN

阿里云申请的证书，需要上传到七牛，点击`上传自有证书`，将阿里云下载的证书解压，里面的两个文件里面有内容和私钥，填写到下图的`证书内容`和`证书私钥`即可。

> 具体的填写方式记不清了，多试几次就可以了，错误的时候会有提示。

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037f805731900?w=344&h=98&f=png&s=14783)

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de8d1b671d?w=1920&h=842&f=png&s=80197)

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de8f517845?w=1883&h=561&f=png&s=71237)

### 配置CDN加速

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de94cfbbde?w=1152&h=855&f=png&s=139037)

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de9330e9ab?w=1040&h=883&f=png&s=210400)

配置完成，下图配置了两种加速场景：

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037de99cb2b38?w=1888&h=458&f=png&s=64188)

复制`cname`，下面需要将cname配置到域名服务器

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037f18f9c57a9?w=1000&h=187&f=png&s=35710)

### 域名服务器配置`cname`

加速域名配置完成后，需要到域名申请的地方配置cname，我是在阿里云购买的域名，所以去阿里云配置。

进入阿里云控制台，进入域名管理：

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037dec1823384?w=1166&h=597&f=png&s=47432)

点击解析

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037dec4e89c53?w=1920&h=903&f=png&s=180385)

点击 `添加记录`，配置`cname`, 粘贴上在七牛云复制的`cname`。记录类型选择`cname`，主机记录根据记录值来填写： 

![](https://user-gold-cdn.xitu.io/2019/10/25/16e037dec62776db?w=1566&h=462&f=png&s=70316)

### 测试CDN加速

windows系统可以通过Win+R 或 点击左下角的“开始”按钮打开“开始”菜单，打开“运行”，输入cmd回车。
在命令行模式下输入`nslookup 您的加速域名`，在结果中可以看到您复制的cname值即可。下面测试中域名`zhixxx-xxch.com`从cdn进行解析的，说明加速成功；域名`x.zxxmxo.com`加速失败。

```shell
C:\Users\Administrator>nslookup zhixxx-xxch.com
服务器:  UnKnown
Address:  192.168.0.1

非权威应答:
名称:    tinychinacdnweb.qiniu.com.w.kunlunno.com
Addresses:  240e:95c:2002:1:3::3fd
          240e:95c:2002:1:3::3fe
          124.225.167.226
          116.253.191.223
          116.253.29.227
          121.207.229.171
          116.253.29.232
          116.253.29.226
          116.253.29.225
          121.207.229.172
          121.207.229.204
          121.207.229.203
          171.111.154.208
          124.225.167.231
          124.225.167.232
          171.111.154.207
          116.253.191.222
          124.225.167.225
Aliases:  zhixxx-xxch.com
          zhixxx-xxch.com.qiniudns.com
          dt003.china.line.qiniudns.com


C:\Users\Administrator>nslookup x.zxxmxo.com
服务器:  UnKnown
Address:  192.168.0.1

非权威应答:
名称:    x.zxxmxo.com
Address:  121.40.96.83
```

## 参考：
https://help.aliyun.com/knowledge_detail/95496.html
https://juejin.im/post/5b44b4fef265da0f767530f7