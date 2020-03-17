# idea插件

**1、activate-power-mode 和 Power mode II**

根据Atom的插件activate-power-mode的效果移植到IDEA上

![img](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQyMmJvARKF9eZcvvN6HesArgZIgrIvxxa29tz9Jw4vG9YdPNxtDpBsRAibh7Vftvia971QRuuicuENxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

写代码是整个屏幕都在抖动，activate-power-mode是白的的，Power mode II色彩更酷炫点。

**2、Background Image Plus**

idea背景修改插件，让你的idea与众不同，可以设置自己喜欢的图片作为code背景。

![img](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQyMmJvARKF9eZcvvN6HesArYQ6ArGsj6rIicd8ib05Qab5xIquyR6kYhpSkgian4tOM7saZ8Dfod0aibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

安装成功之后重启，菜单栏的VIew标签>点击Set Background Image(没安装插件是没有这个标签的)，在弹框中路由选择到本地图片，点击OK即可。

**3、Grep console**

自定义日志颜色，idea控制台可以彩色显示各种级别的log，安装完成后，在console中右键就能打开。

![img](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQyMmJvARKF9eZcvvN6HesArCJyRLeBrQp9sCDXQd3gDyQF0yodpSYezTrx5ibC5N8mSX5FqiaJv5UQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

并且可以设置不同的日志级别的显示样式。

![img](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQyMmJvARKF9eZcvvN6HesArBLYjUS8IEYBj11pSSrgBSCpqWpOwgGZ2d9KZlKTEia56q9nfFBRLkUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以直接根据关键字搜索你想要的，搜索条件是支持正则表达式的。

**4、Free Mybatis plugin**

mybatis 插件，让你的mybatis.xml像java代码一样编辑。我们开发中使用mybatis时时长需要通过mapper接口查找对应的xml中的sql语句，该插件方便了我们的操作。

安装完成重启IDEA之后，我们会看到code左侧或多出一列绿色的箭头，点击箭头我们就可以直接定位到xml相应文件的位置。

mapper

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

xml

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**5、MyBatis Log Plugin**

Mybatis现在是java中操作数据库的首选，在开发的时候，我们都会把Mybatis的脚本直接输出在console中，但是默认的情况下，输出的脚本不是一个可以直接执行的。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如果我们想直接执行，还需要在手动转化一下。

MyBatis Log Plugin 这款插件是直接将Mybatis执行的sql脚本显示出来，无需处理，可以直接复制出来执行的，如图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

执行程序后，我们可以很清晰的看到我们执行了哪些sql脚本，而且脚本可以执行拿出来运行。

**6、String Manipulation**

强大的字符串转换工具。使用快捷键，Alt+m。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- 切换样式（camelCase, hyphen-lowercase, HYPHEN-UPPERCASE, snake_case, SCREAMING_SNAKE_CASE, dot.case, words lowercase, Words Capitalized, PascalCase）
- 转换为SCREAMING_SNAKE_CASE (或转换为camelCase)
- 转换为 snake_case (或转换为camelCase)
- 转换为dot.case (或转换为camelCase)
- 转换为hyphen-case (或转换为camelCase)
- 转换为hyphen-case (或转换为snake_case)
- 转换为camelCase (或转换为Words)
- 转换为camelCase (或转换为lowercase words)
- 转换为PascalCase (或转换为camelCase)
- 选定文本大写
- 样式反转

**7、Alibaba Java Coding Guidelines**

阿里巴巴代码规范检查插件，当然规范可以参考《阿里巴巴Java开发手册》。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**8、Lombok**

Java语言，每次写实体类的时候都需要写一大堆的setter，getter，如果bean中的属性一旦有修改、删除或增加时，需要重新生成或删除get/set等方法，给代码维护增加负担，这也是Java被诟病的一种原因。Lombok则为我们解决了这些问题，使用了lombok的注解(@Setter,@Getter,@ToString,@@RequiredArgsConstructor,@EqualsAndHashCode或@Data)之后，就不需要编写或生成get/set等方法，很大程度上减少了代码量，而且减少了代码维护的负担。

安装完成之后，在应用Lombok的时候注意别忘了需要添加依，maven为例：

```

```

**9、Key promoter**

Key promoter 是IntelliJ IDEA的快捷键提示插件，会统计你鼠标点击某个功能的次数，提示你应该用什么快捷键，帮助记忆快捷键，等熟悉了之后可以关闭掉这个插件。

**10、Gsonformat**

可根据json数据快速生成java实体类。

自定义个javaBean(无任何内容，就一个空的类)，复制你要解析的Json，然后alt+insert弹出如下界面或者使用快捷键 Alt+S，在里面粘贴刚刚复制的Json，点击OK即可。

![img](https://mmbiz.qpic.cn/mmbiz_png/eZzl4LXykQyMmJvARKF9eZcvvN6HesArRGdcMltHMAUl1DTpDgXuq82vLOrptnRTfnTdB1pgW5icQuNLwcs8YqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**11、Restfultookit**

Spring MVC网页开发的时候，我们都是通过requestmapping的方式来定义页面的URL地址的，为了找到这个地址我们一般都是cmd+shift+F的方式进行查找，大家都知道，我们URL的命名一个是类requestmapping+方法requestmapping，查找的时候还是有那么一点不方便的，restfultookit就能很方便的帮忙进行查找。

例如：我要找到/user/add 对应的controller,那么只要Ctrl+斜杠 ,（图片来自于网络）

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

就能直接定位到我们想要的controller。这个也是真心方便，当然restfultookit还为我们提供的其他的功能。根据我们的controller帮我们生成默认的测试数据，还能直接调用测试，这个可以是解决了我们每次postman调试数据时，自己傻傻的组装数据的的操作，这个更加清晰，比在console找数据包要方便多了。（图片来自于网络）

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**12、JRebel**

JRebel是一种热部署生产力工具，修改代码后不用重新启动程序，所有的更改便可以生效。它跳过了Java开发中常见的重建、重新启动和重新部署周期。

**三、常用插件推荐**

| 插件名称                       | 插件介绍                                                     | 官网地址                                                     |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Gitee                          | 开源中国的码云插件                                           | https://plugins.jetbrains.com/plugin/8383-gitee              |
| Alibaba Java Coding Guidelines | 阿里巴巴出的代码规范检查插件                                 | https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines |
| IDE Features Trainer           | IntelliJ IDEA 官方出的学习辅助插件                           | https://plugins.jetbrains.com/plugin/8554?pr=idea            |
| Key promoter                   | 快捷键提示                                                   | https://plugins.jetbrains.com/plugin/4455?pr=idea            |
| Grep Console                   | 自定义设置控制台输出颜色                                     | https://plugins.jetbrains.com/idea/plugin/7125-grep-console  |
| String Manipulation            | 驼峰式命名和下划线命名交替变化                               | https://plugins.jetbrains.com/plugin/2162?pr=idea            |
| CheckStyle-IDEA                | 代码规范检查                                                 | https://plugins.jetbrains.com/plugin/1065?pr=idea            |
| FindBugs-IDEA                  | 潜在 Bug 检查                                                | https://plugins.jetbrains.com/plugin/3847?pr=idea            |
| MetricsReloaded                | 代码复杂度检查                                               | https://plugins.jetbrains.com/plugin/93?pr=idea              |
| Statistic                      | 代码统计                                                     | https://plugins.jetbrains.com/plugin/4509?pr=idea            |
| JRebel Plugin                  | 热部署                                                       | https://plugins.jetbrains.com/plugin/?id=4441                |
| CodeGlance                     | 在编辑代码最右侧，显示一块代码小地图                         | https://plugins.jetbrains.com/plugin/7275?pr=idea            |
| GsonFormat                     | 把 JSON 字符串直接实例化成类                                 | https://plugins.jetbrains.com/plugin/7654?pr=idea            |
| Markdown Navigator             | 书写 Markdown 文章                                           | https://plugins.jetbrains.com/plugin/7896?pr=idea            |
| Eclipse Code Formatter         | 使用 Eclipse 的代码格式化风格，在一个团队中如果公司有规定格式化风格，这个可以使用。 | https://plugins.jetbrains.com/plugin/6546?pr=idea            |
| Jindent-Source Code Formatter  | 自定义类、方法、doc、变量注释模板                            | http://plugins.jetbrains.com/plugin/2170?pr=idea             |
| Translation                    | 翻译插件                                                     | https://github.com/YiiGuxing/TranslationPlugin               |
| Maven Helper                   | Maven 辅助插件                                               | https://plugins.jetbrains.com/plugin/7179-maven-helper       |
| Properties to YAML Converter   | 把 Properties 的配置格式改为 YAML 格式                       | https://plugins.jetbrains.com/plugin/8000-properties-to-yaml-converter |
| Git Flow Integration           | Git Flow 的图形界面操作                                      | https://plugins.jetbrains.com/plugin/7315-git-flow-integration |
| Rainbow Brackets               | 对各个对称括号进行着色，方便查看                             | https://github.com/izhangzhihao/intellij-rainbow-brackets    |
| MybatisX                       | mybatis 框架辅助（免费）                                     | https://plugins.jetbrains.com/plugin/10119-mybatisx          |
| Lombok Plugin                  | Lombok 功能辅助插件                                          | https://plugins.jetbrains.com/plugin/6317-lombok-plugin      |
| .ignore                        | 各类版本控制忽略文件生成工具                                 | https://plugins.jetbrains.com/plugin/7495--ignore            |
| mongo4idea                     | mongo客户端                                                  | https://github.com/dboissier/mongo4idea                      |
| iedis                          | redis客户端                                                  | https://plugins.jetbrains.com/plugin/9228-iedis              |
| GenerateAllSetter              | new POJO类的快速生成 set 方法                                | https://plugins.jetbrains.com/plugin/9360-generateallsetter  |