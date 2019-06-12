## <center>Web服务器入门(了解)</center>
### 一、BS和CS概述
#### 1. 什么是服务器？
就是一台性能比较高计算机，把自己的资源通过网络共享给其它的计算机。

#### 2. CS的特点:
 Client 客户端  Server 服务端
 1) 客户端：
 是需要安装软件的，如QQ，网络游戏。
 2) 服务器升级：
 如果服务器进行升级，客户端也要进行相应的升级。
        
#### 3. BS的特点：
Browser 浏览器  Server 服务器
1) 客户端：
是浏览器，只要客户端有浏览器就可以访问服务器。如：淘宝、银行网站、后台管理系统OA
            
2) 服务器升级：
 如果服务器进行升级，客户端不用升级。
#### 4. 静态网站与动态网站
 1) 静态Web开发技术：
 使用HTML、CSS、JavaScript都是静态网页的技术，如果网站没有人去更新，永远一成不变。
 2) 动态Web开发技术：
 使用PHP、ASP、JSP开发动态网站，代码是运行在服务器上，在浏览器上看到的只是JSP运行的结果。
 不同的时候，看到的结果是不同的。
3) Java Web概念：
 使用Java做为服务器端的语言开发的Web网站，扩展名为.jsp，.do，*.action
    
#### 5. 企业级应用介绍：
        ERP：(Enterprise Resource Planning 企业资源计划)
        CRM：(Customer Relationship Management 客户关系管理系统)
        BPM：(Busniess Process Management 业务流程管理)
        OA：(Office Automation 办公自动化)
        HR:  (Human Resource 人力资源管理系统)
![](images/2019-06-10-22-00-28.png)

### 二、服务器的访问
#### 1. Web技术的特点：
1) 通信协议：Web中的协议HTTP或HTTPS协议，客户端和服务端在通信的时候共同遵守的格式
2) 网页语言： HTML的语言，最终在浏览器显示的还是HTML
3) 访问资源： 用统一资源定位URL(Uniform Resource Locator)在网络上信息的精确定位。
4) 域名解析： DNS域名解析服务器，通过域名来访问网站。www.baidu.com
        
#### 2. 请求和响应
1) 请求：request，要访问服务器，通过请求的方式来访问服务器的资源。
2) 响应：response，服务器通过响应的方式将资源返回给浏览器。
        
#### 3. URL的格式(Uniform Resource Locator 统一资源定位符)
1) 格式：
   http://localhost:8080/mysite/page/index.html
2) 组成部分：
            ● 协议：http，使用的协议
            ● 服务器：
                本机域名： localhost
                dns的访问过程：浏览器先访问dns服务器，(查IP地址)再访问真正的web服务器
            ● 端口：
                HTTP默认端口：80，可以省略
                常用的端口： 110 发邮件端口 25收邮件  FTP 21 MySQL 3306  Oracle 1521
            ● 路径：mysite/page
            ● 资源名称：index.html
        
### 三、Web服务器
#### 1. 服务器分类：
1) Web服务器：
            存放Web网站，如：淘宝，百度
2) 数据库服务器：
            存放数据库：Oracle  MySQL
3) 邮件服务器：
            ● 收邮件服务器： smtp.126.com
            ● 发邮件服务器： pop3.126.com    imap.126.com
#### 2. 常见的Web服务器
1) WebLogic: 
            Oracle公司的产品
2) WebSphere:
            IBM公司的产品
3) GlassFish:    
            Sun公司的产品
4) Tomcat服务器
            Apache 基金会产品