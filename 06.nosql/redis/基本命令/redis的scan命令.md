## 查询出安装位置
```shell
$ whereis redis
redis: /etc/redis.conf
```
## 登陆redis
```
$ cd /etc/
$ redis-cli -h 127.0.0.1 -p 6379 -a zhirui888
```
## 查询和删除
推荐用scan命令去查询，使用keys的话如果key值多会导致卡死
```shell
127.0.0.1:6379[1]> scan 0 match *tree count 500
1) "0"
2) 1) "18772bca48674f1aaf1ea882b8ad1b9dmaterialtypetree"
   2) "b45551e5f8de4efc800e3677ef68cb39materialtypetree"
127.0.0.1:6379[1]> del 18772bca48674f1aaf1ea882b8ad1b9dmaterialtypetree b45551e5f8de4efc800e3677ef68cb39materialtypetree
```

