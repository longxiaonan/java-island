### sql 四舍五入

```sql
SELECT CAST('123.456' as decimal) 将会得到 123（小数点后面的将会被省略掉）。
如果希望得到小数点后面的两位。
则需要把上面的改为
SELECT CAST('123.456' as decimal(38, 2)) ===>123.46

说明1:

这里的decimal英文为: 小数, 十进制
decimal(38,2)
这里的38是这个小数的位数有多少位, 一般最大不超过38位, 所以写38是不会出错的!
如果:
SELECT CAST('123.456' as decimal(2, 2))
就会出错, 为什么呢, 因为这个123.456小数点后是3位值, 所以这个38这个位置最少是3!

说明2:
decimal后面的参数中的2是小数点后取几位, 是2就取两位, 是3就取三位! 并且是四舍五入后的结果!
```

取整:

```sql
SELECT 
  spec.`area` AS specCode,
  detail.`position_name` AS positionName,
  spec.`code` AS specCode,
  CAST(spec.length AS DECIMAL) AS specLength,
  CAST(spec.width AS DECIMAL) AS specWidth,
  detail.`num`,
  spec.`area` * detail.num AS specSubtotalArea 
FROM
  wh_inbound_shelf_up_task_detail detail 
  LEFT JOIN t_product_specification spec 
    ON detail.`spec_code` = spec.`code` 
```



