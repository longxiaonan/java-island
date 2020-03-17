# nginx安装，启动，停止，重启

[TOC]

## 安装

<https://blog.csdn.net/wltsysterm/article/details/79631530>

## 校验配置是否正确

```shell
./nginx -t
```

## 启动

### 直接启动

```shell
./nginx
```

### 指定配置文件启动--推荐

```shell
/usr/local/nginx/sbin/nginx  -c  /usr/local/nginx/conf/nginx.conf
```

### 校验配置文件后启动

```shell
/usr/local/nginx/sbin/nginx  -t -c  /usr/local/nginx/conf/nginx.conf
```



## 停止

1. 查看进程号

   ```shell
   ps -ef|grep nginx
   ```

2. 杀死进程

   ```shell
   kill 2012
   ```

> 如果停止失败，可以尝试通过`pkill -9 2012`来删除进程

## 重启

### 方式一：sbin下执行reload进行重启

```shell
./nginx -s reload
```

### 方式二：`kill   -HUP   进程号 ` 进行重启

```shell
kill -HUP 2012
```

 