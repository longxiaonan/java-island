# jenkins中的maven单独模块打包

一、Jenkins构建Maven多模块项目时，单独编译子模块

配置：

1、Root POM指向父pom.xml

2、Goals and options指定构建模块的参数：mvn -pl jsoft-web -am clean package，单独构建jsoft-web项目以及它所依赖的其它项目。参考：<http://www.cnblogs.com/EasonJim/p/8350560.html>

二、通过上面的操作之后确实能单独构建了，但可能会同时触发Jenkins上的其它模块的项目，可以通过屏蔽下游项目来限制：

[![img](https://images2017.cnblogs.com/blog/417876/201801/417876-20180125115714209-810785690.png)](https://images2017.cnblogs.com/blog/417876/201801/417876-20180125115714209-810785690.png)

选中即可实现不自动触发下游项目的触发。

[![img](https://images2017.cnblogs.com/blog/417876/201801/417876-20180125115749506-961322665.png)](https://images2017.cnblogs.com/blog/417876/201801/417876-20180125115749506-961322665.png)