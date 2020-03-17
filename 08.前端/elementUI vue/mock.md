# Element（Vue）+Express（Node）模拟服务器获取本地json数据

> 比如webpack的中间件api-mock，json-server，mockjs，还有express。结合我们项目的情况，逐一尝试，最后选择了express	

## 安装Express

> **express 一般项目中均安装，若未安装则进行安装**

<http://www.expressjs.com.cn/>

```shell
$ npm install express --save
```

配置package.json

![](https://user-gold-cdn.xitu.io/2019/11/7/16e447518e1c7e11?w=1368&h=821&f=png&s=164259)

创建 mock 文件夹，准备server代码编写，此处命名为 mock.js：



## 配置json数据

网上很多教程说需要在build目录下的dev-server.js文件中配置，但目前最新的vue-cli是没有dev-server.js这个文件的，因为已经被合并到webpack.dev.conf.js文件中，所以直接在该文件中配置即可。

`demo/mock/users.json`

```json
{
  "users": [
    {
      "id": 1,
      "name": "龙小东",
      "age": 20,
      "sex": "男",
      "dept": "技术部",
      "floor": 20,
      "area": "东区"
    },{
      "id": 2,
      "name": "龙小南",
      "age": 21,
      "sex": "男",
      "dept": "财务部",
      "floor": 21,
      "area": "东区"
    },{
      "id": 3,
      "name": "龙小西",
      "age": 22,
      "sex": "男",
      "dept": "总裁办",
      "floor": 22,
      "area": "东区"
    },{
      "id": 4,
      "name": "龙小北",
      "age": 23,
      "sex": "男",
      "dept": "研发部",
      "floor": 23,
      "area": "东区"
    }
  ]
}
```

`demo/build/webpack.dev.conf.js`

添加如下代码

```javascript
// mock code start
const express = require('express')
const app = express()

const users = require('../mock/users.json') // 用户列表数据源
const routes = express.Router()
app.use('/api', routes)
// mock code end
```

## 使用第三方http请求库axios进行ajax请求

在项目的cmd命令行(管理员模式)  进行安装`axios`

```shell
$ npm install axios --save-dev
```

main.js中加入以下代码：

```javascript
import axios from 'axios'
Vue.prototype.$http = axios
```



## 测试

测试无法mock。。

## 参考

<https://www.cnblogs.com/dreamsqin/p/9340344.html>

<https://www.cnblogs.com/momo798/p/9924518.html>