# nginx指令

## `rewrite`指令

### rewrite 企业应用场景

Nginx的rewrite功能在企业里应用非常广泛：

* 可以调整用户浏览的URL，看起来更规范，合乎开发及产品人员的需求。

*  为了让搜索引擎搜录网站内容及用户体验更好，企业会将动态URL地址伪装成静态地址提供服务。

* 网址换新域名后，让旧的访问跳转到新的域名上。例如，访问京东的360buy.com会跳转到jd.com

* 根据特殊变量、目录、客户端的信息进行URL调整等

### rewrite用法

`rewrite`指令是实际用来进行`rewrite`功能配置的具体指令，使用正则表达式的方法来改变`URI`，可同时存在1个或多个指令，按照顺序依次进行处理。
`rewrite`指令可以在`server`块或`location`块中进行配置，语法结构如下：

``` shell
rewrite regex replacement [flag];
```

参数说明：

- `regex`：正则表达式，使用()标记要截取的内容，使用$1~$9获取截取到的内容；
- `replacement`：匹配成功后用于替换被截取内容的字符串，即重写后的地址，如果该字符串是由http(s)开头的，则直接返回此URI给客户端，不再继续向下处理。
- `flag`：设置rewrite对URI的处理行为，有4种标志可设置，分别为：
  `last`：终止继续在本location块中处理URI，将重写后的URI重新在server块中执行，提供了重写后的URI转入到其它`location`块中的机会；
  `break`：重写后的URI继续在本`location`块中执行，不会转入到其它的location块中；
  `redirect`：重写后的URI返回给客户端，状态代码为203，指明是临时重定向URI，主要用在`replacement`不是以http(s)开头的情况下；
  `permanent`：重写后的URI返回给客户端，状态代码为301，指明是永久重定向URI；

### 举例，80端口跳转到443：

```shell
server {
    listen       80;
    server_name  www.abc.net;
    rewrite ^(.*)$ https://$host$1 permanent; #重定向到443端口
}
```

```shell
server {
    listen       443;
    server_name  www.abc.net;
    rewrite ^(.*)$ https://$host$1 permanent; #重定向到443端口
    location / {
    root /data/www/www;
    index index.html index.htm;
    }
}
```



### regex 常用正则表达式说明

| **字符**      | **描述**                                                     |
| ------------- | ------------------------------------------------------------ |
| **\**         | 将后面接着的字符标记为一个特殊字符或一个原义字符或一个向后引用。如“\n”匹配一个换行符，而“\$”则匹配“$” |
| **^**         | 匹配输入字符串的起始位置                                     |
| **$**         | 匹配输入字符串的结束位置                                     |
| *****         | 匹配前面的字符零次或多次。如“ol*”能匹配“o”及“ol”、“oll”      |
| **+**         | 匹配前面的字符一次或多次。如“ol+”能匹配“ol”及“oll”、“oll”，但不能匹配“o” |
| **?**         | 匹配前面的字符零次或一次，例如“do(es)?”能匹配“do”或者“does”，"?"等效于"{0,1}" |
| **.**         | 匹配除“\n”之外的任何单个字符，若要匹配包括“\n”在内的任意字符，请使用诸如“[.\n]”之类的模式。 |
| **(pattern)** | 匹配括号内pattern并可以在后面获取对应的匹配，常用$0...$9属性获取小括号中的匹配内容，要匹配圆括号字符需要\(Content\) |

