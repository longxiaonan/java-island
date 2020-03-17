### group by分组

使用`group by`分组统计之后，我们的select 后面只能跟着group by 的字段，或者是聚合函数。

> 很多时候我们`group by`了以后，还想要查询结果中包含`group by`之外的字段(一般情况下，我们都不可能将group by 涵盖所有的字段)，我们就可以上面那样，将查询后的结果作为子查询，放在外部查询的where 子句后，这样外部查询是可以select 出其他字段的。

```sql
select * from user where id in(
   select min(id) from user where name = 'Java3y' and pv = 20 and time='7-25' group by name,pv,time;
)
```

### case when

我用得比较多的语法如下：

```
CASE WHEN sex = '1' THEN '男'
         WHEN sex = '2' THEN '女'
ELSE '其他' END   
复制代码
```

在when后面可以跟多个表达式，比如说：

```
CASE WHEN sex = '1' and name ='Java3y' THEN '男'
         WHEN sex = '2' and name ='Java4y' THEN '女'
ELSE '其他' END   
复制代码
```

如果要为`case when`表达式取别名，在`end` 关键字后边直接加就好了

### 时间函数

```sql
-- 昨天

SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) <= 1

-- 7天

SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(时间字段名)

-- 近30天

SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)

-- 本月

SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )

-- 上一月

SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1
```

参考：<https://juejin.im/post/5d3f9cc1f265da03a31d1192?utm_source=gold_browser_extension>