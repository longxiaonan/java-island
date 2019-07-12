# jenkins+maven+gitlab实现自动部署springcloud

## 服务器规划

| ip地址        | 类型          | 软件                  | 说明 |
| ------------- | ------------- | --------------------- | ---- |
| 192.168.1.230 | 应用服务器    | jdk                   |      |
| 192.168.1.112 | 应用服务器    | jdk                   |      |
| 192.168.1.106 | jenkins服务器 | jdk git maven jenkins |      |

> * 106和230,112之间需要免密登陆, [免密登陆参考](E:\biji\99.system\linux\common\SSH免密登陆\免密登陆试验.md)

