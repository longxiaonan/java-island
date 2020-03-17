

# sql优化

## 优化方式-服务器硬件

机械硬盘该固态硬盘，提升10倍

## 优化方式-MySql服务器优化

windows -》linux

## 优化方式-SQL本身优化

### 不要用关联子查询

#### 查询用户信息(包括部门名称)

关联子查询，执行时间`10.777秒`：

> 关联子查询：使用了外部表的作为条件

```sql
SELECT 
  u.`name`,
  u.`addr`,
  u.`age`,
  (SELECT 
    d.`name` 
  FROM
    `dept` d 
  WHERE d.id = u.`dept_id`) 
FROM
  `user` u 
```

关联查询（跟inner join一样），执行时间`1.284秒`：

```sql
SELECT 
  u.`name`,
  u.`addr`,
  u.`age`,
  d.`name`
FROM
  `user` u , `dept` d
  WHERE u.`dept_id` = d.`id`
```

执行速度提升一倍，结论：在工作中不要使用关联子查询！

## 优化方式-反范式优化

对数据库三大范式的违反。允许少量冗余，使用空间换时间。

数据库三大范式：

第一范式：原子性，一列只解决描述一件事情。

第二范式：每列直接依赖主键。

第三范式：

数据库三大范式减少了数据冗余，但是增加了查询复杂度。

## 优化方式-物理设计优化（字段类型、长度设计、仓储引擎选择）

字段类型选择优先程度为：

int 》bigint 》Date 》datetime 》varchar 》其他类型

varchar尽量用字符数量小的，varchar（2）比varchar(200)快

固定小数长度的用int或者bigint，比如钱用bigint。转换的时候除100即可。

## 优化方式-索引优化

### 数据库使用的b+tree进行存储

mysql默认存储引擎innodb只显式支持B-Tree（从技术上说是B+Tree）引擎

### 索引类型：

* 普通索引

  一个索引只包含单个列，一个表可以有多个单列索引。

* 唯一索引

  索引的列必须唯一，但允许有空值。列中只能一个空值。

* 复合索引

  一个索引包含多个列

* 聚簇索引（聚集索引）

  并不是一种单独的索引类型，而是一种仓储方式。具体细节取决于不同的实现。innodb 聚集索引。

* 非聚簇索引

  不是聚簇索引，就是非聚簇索引。myisam 非聚集索引。

### 通过执行计划来分析索引的使用

explain

是否使用索引：key

是否充分使用：key_len



## 优化方式-锁的优化

锁的类型：

表锁，行锁

## 实战

### 创建表

一共两个表，用户表和部门表

部门表：

```sql
CREATE TABLE `dept` (
  `id` bigint(20) DEFAULT NULL,
  `name` varchar(10) DEFAULT NULL COMMENT '部门名称'
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

用户表：

```sql
CREATE TABLE `user` (
  `id` bigint(20) DEFAULT NULL,
  `name` varchar(5) DEFAULT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `dept_id` bigint(20) DEFAULT NULL COMMENT '部门',
  `addr` varchar(20) DEFAULT NULL COMMENT '地址'
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

### 准备测试数据

编写生产批量测试数据的仓储过程和仓储函数。

分别对user表和dept表生产约10万条数据。

```sql


# -- 同样的可以通过delimiter;再设置;为结束标记

-- -----------------定义函数rand_string()-----start----------------------
# -- 定义一个新的命令结束符号，默认是以;为结束标记
DELIMITER $$ 

-- 命令结束符
# -- 删除函数rand_string
DROP FUNCTION rand_string $$

# 创建函数rand_string(n); 随机产生n个字符组成的字符串
CREATE FUNCTION rand_string (n INT) RETURNS VARCHAR (255) 
BEGIN
  DECLARE chars_str VARCHAR (100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ' ;
  DECLARE return_str VARCHAR (255) DEFAULT '' ;
  DECLARE i INT DEFAULT 0 ;
  WHILE
    i < n DO SET return_str = CONCAT(
      return_str,
      SUBSTRING(chars_str, FLOOR(1 + RAND() * 52), 1)
    ) ;
    SET i = i + 1 ;
  END WHILE ;
  RETURN return_str ;
END $$
DELIMITER ;
-- -----------------定义函数rand_string()-----end----------------------

-- -----------------定义函数rand_num()--------start--------------------
DELIMITER $$
DROP FUNCTION rand_num $$
# -- 自定义函数rand_num()：随机生成一个整数
CREATE FUNCTION rand_num () RETURNS INT (5)

BEGIN
  DECLARE i INT DEFAULT 0 ;
  SET i = FLOOR(10 + RAND() * 50) ;
  RETURN i ;
END $$
DELIMITER ;
-- -----------------定义函数rand_num()--------end--------------------

-- -----------------定义仓储过程 insert_pro --------start--------------------
DELIMITER $$
-- 命令结束符
DROP PROCEDURE insert_pro $$

CREATE PROCEDURE insert_pro (
  IN start_num INT (10),
  IN max_num INT (10)
) 
BEGIN
  DECLARE i INT DEFAULT start_num ;
  DECLARE aname VARCHAR (16) DEFAULT 'a' ;
  DECLARE age VARCHAR (16) DEFAULT 'b' ;
  DECLARE addr VARCHAR (16) DEFAULT 'c' ;
  --  declare bizseqno varchar (64) default '' ;
  WHILE
    i < max_num DO SET i = i + 1 ;
    SET aname = CONCAT('a', CAST(i AS CHAR)) ;
    SET age = i ;
    SET addr = CONCAT('c', rand_string (5)) ;
    INSERT INTO `user` (id, `name`, `age`, `dept_id`, `addr`) 
    VALUES
      (i, aname, age, 2, addr) ;
  END WHILE ;
  COMMIT ;
END $$

DELIMITER ;
-- ----------------- 定义仓储过程 insert_pro --------end--------------------

-- -----------------调用存储过程 insert_pro ----------------------------
-- 参数1：开始的数量，参数2：最大的数量
CALL insert_pro (1, 100000) ;
```

<font color=red>只需要修改上面的仓储过程`insert_pro`即可大批量生产指定表的测试记录。</font>

> 将上面仓储过程的insert into语句替换成如下，即可用于生成部门数据：
>
> ```sql
>     INSERT INTO `dept` (id, `name`) 
>     VALUES
>       (i, 2) ;
> ```

### 

