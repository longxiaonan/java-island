
### <center>java 发送邮件附件 文件名过长导致被截取

// linux 下 程序使用javamail1.4.4 发邮件带附件，若附件名过长，会被截断。

原因如下

参数mail.mime.splitlongparameters 在linux下 会默认为 true，附件名过长，就会被截断

解决方案：

只要在设置邮件属性

new MimeMessage、new MimeMultipart、new MimeBodyPart 之前（一个比较靠前的位置，如果在new MimeMultipart之后添加，有可能无效），添加如下代码。

System.getProperties().setProperty("mail.mime.splitlongparameters", "false");

就可以避免在linux下利用javamail1.4.4发邮件带附件，附件名过长而被被截断，导致接收端解析失败的异常了

如果还出现中文乱码的话，就在获取到附件名的时候进行MimeUtility.encodeText(source.getName());编码，就可以避免中文乱码了，

//中文名过长MimeUtility.encodeText方法会自动给添加下划线,通过查找得知：对文件进行编码时超出长度会自动通过"/r","/n"替换,而MimeUtility.encodeText可能通过"_"进行连接,只要替换了"/r","/n"即可,如下

String filenames=MimeUtility.encodeText(source.getName());
filenames=filenames.replace("\\r","").replace("\\n","");

//设置附件名
bodyPart.setFileName(filenames);

最后修改完之后，重新部署程序，重新启动tomcat，

发送邮件，可以正常显示了

困扰了这么久的问题终于解决了，之前一直以为是编码的问题，尝试了很多编码都不能解决，后面又以为是linux 的编码问题，设置了几次还是不行，今天换了思路附件名过长，终于解决了，痛快，参考了以下几篇博文

https://blog.csdn.net/fl_zxf/article/details/60126910

https://blog.csdn.net/haozhongjun/article/details/24419057

https://blog.csdn.net/albert0707/article/details/69284700