# boke:mysql数据备份和恢复

本文通过mysqldump命令方式先备份，再进行恢复。

网上很多的关于`数据库迁移`的话题，都是通过拷贝数据库`/var/lib/mysql`目录下的`ibdata1`，`ib_logfile0`，`ib_logfile1`等等操作，我都一一试过，全部失败了。。

## 数据备份：

```shell
$ mysqldump --all-databases --user=root --password --master-data > backupdatabase.sql
```

如果要同时备备份数据表和存储过程等，通过如下参数：

```shell
mysqldump  -E -R --triggers --opt  --single-transaction --master-data=2  --default-character-set=utf8 -uroot -p123456 zmgj > /tmp/zmgj.sql
```

如果备份如下报错：

```shell
$ mysqldump: Error: Binlogging on server not active
```

1. 错误原因

   binlog日志会记录下数据库的所以增删改操作，当不小心删除、清空数据，或数据库系统出错，这时候就可以使用binlog日志来还原数据库，简单来说就是一个记录备份的东西。mysqldump需要这个功能开启。

2. 解决办法

   编辑`my.cnf`：

   ```shell
   $ sudo vim /etc/my.cnf
   ```

   在[mysqld]下添加：

   ```shell
   log-bin=mysql-bin
   #server-id=1    #视情况添加，有时候添加这个导致数据库启动失败
   ```

   重启mysql，执行下面的其中一条即可：

   ```shell
   service mysqld restart
   service mysql restart
   systemctl restart mysql.service
   ```

## 数据恢复：

```shell
$ mysql -u root -p < backupdatabase.sql
```

也可以使用如`SQLyog`工具，连接数据库后，右击`执行sql脚本`。

如果恢复如下报错：

```shell
ERROR 1709 (HY000): Index column size too large. The maximum column size is 767 bytes.
```

1. 错误原因
   由于 MySQL Innodb 引擎表索引字段长度的限制为 767 字节，因此对于多字节字符集的大字段（或者多字段组合索引），创建索引会出现上面的错误。
   以 utf8mb4 字符集 字符串类型字段为例：utf8mb4 是 4 字节字符集，则默认支持的索引字段最大长度是： 767 字节 / 4 字节每字符 = 191 字符，因此在 varchar(255) 或 char(255) 类型字段上创建索引会失败。

2. 解决办法：
   1.调整参数 innodb_large_prefix 为 ON
   将 Innodb_large_prefix 修改为 on 后，对于 Dynamic 和 Compressed 格式的InnoDB 引擎表，其最大的索引字段长度支持到 3072 字节。Mysql5.7默认是ON，一般能成功。

   ```shell
   [mysqld]
   default-storage-engine=INNODB
   innodb_file_format=barracuda
   innodb_file_per_table=true
   innodb_large_prefix=true  
   character-set-server=utf8mb4
   collation-server=utf8mb4_unicode_ci
   max_allowed_packet=500M
   ```

3. 创建表的时候指定表的 row format 格式为 Dynamic

   ```shell
   CREATE TABLE `ao_0ac321_recommendation_ao` (
     `CATEGORY` VARCHAR(255) DEFAULT NULL,
     `CUSTOM_FIELD_ID` BIGINT(20) DEFAULT NULL,
     `ID` VARCHAR(255) NOT NULL,
     `NAME` VARCHAR(255) DEFAULT NULL,
     `PERFORMANCE_IMPACT` DOUBLE DEFAULT NULL,
     `PROJECT_IDS` LONGTEXT,
     `RESOLVED` TINYINT(1) DEFAULT NULL,
     `TYPE` VARCHAR(255) DEFAULT NULL,
     PRIMARY KEY (`ID`)
   ) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;
   ```