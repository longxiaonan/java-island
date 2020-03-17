## 第一个vue程序

在电脑上创建一个html结尾的文件，贴上下面的代码，再用浏览器打开该文件可以查看到效果，显示红色"hello vue !"几个红色的字。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .bg{
            color:red;
        }
    </style>
    <script src="https://cdn.bootcss.com/vue/2.6.10/vue.min.js"></script>
</head>
<body>
<div class="bg" id="app">
    {{msg}}
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            msg: 'hello vue !'
        }
    })
</script>
</body>
</html>
```

上面是vue的基本用法，其中vue.js通过cdn加载，在网站"<https://www.bootcdn.cn/vue/>"可以找到。

## 模板语法

### vue文件结构（template，script，style）

### 模板语法包含插值、指令（指令缩写）

