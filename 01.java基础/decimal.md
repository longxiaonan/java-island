

# decimal

## 代码

``` java
vehicle.setLength(BigDecimal.valueOf(0));
//改成
vehicle.setLength(BigDecimal.ZERO);
```

## 数据库

钱数：

```sql
decimal(15,2)
# 改成
decimal(11,2) #11-2 = 9 ，整数部分已经是亿级，够用了
```

备注：

```sql
varchar(500)
#改成
varchar(200) #大部分备注用不着500
```

MySQL使用`DECIMAL`类型去存储对精度要求比较高的数值，比如金额，也叫定点数，decimal在mysql内存是以二进制存储的。声明语法是`DECIMAL(M,D)`。M是数字最大位数（精度precision），范围1-65；D是小数点右侧数字个数（标度scale），范围0-30，但不得超过M。

占用字节数计算方法 —— 小数和整数分别计算，每9位数占4字节，剩余部分如下表换算：

| Leftover Digits | Number of Bytes |
| --------------- | --------------- |
| 0               | 0               |
| 1–2             | 1               |
| 3–4             | 2               |
| 5–6             | 3               |
| 7–9             | 4               |

比如`DECIMAL(18,9)`，整数部分和小数部分各9位，所以各占4字节，共8bytes。
再比如`DECIMAL(20,6)`，整数14位，需要4字节存9位，还需3字节存5位；小数6位，需3字节。共10bytes。

## 参考

<https://www.cnblogs.com/erisen/p/5970315.html>

