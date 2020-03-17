<center><h2> Mysql 自增Id的使用以及查询优化</h2></center>

### 一、[关于Mysql自增Id](https://mp.weixin.qq.com/s/1HczsSfYPrH8oAexLFfy_A)

#### 自增Id优点

```
1. 数据库自动编号，速度快，而且是增量增长，按顺序存放，对于检索非常有利
2. 数字类型，占用空间很小，易排序，在程序中操作也比较方便
3. 在插入数据的时候，可以无需指定该值，不用担心主键重复的问题
```

#### 自增Id缺点

```
1. 手动指定Id时，比较麻烦；进行数据同步时会产生Id冲突
2. 给主键增加前后缀时，需要改表
3. 影响分表、分库
4. 依赖数据库
```

### 二、Mysql为什么建议使用自增主键？

> - Mysql建议使用自增主键，是因为使用自增Id可以避免页分裂。
> - Mysql（默认mysql的存储引擎为InnoDB ）底层数据结构是B+树，**所谓的索引其实就是一颗B+树，mysql中的数据都是按顺序保存在B+树上的**（所以说索引本身就是有序的）。然后mysql在底层又是以数据页为单位来存储数据的，一个数据页的大小为默认为16k（大小是可以修改的），也就是说一个数据页存满了，mysql就会申请一个新的数据页来存储数据。如果主键为自增Id的话，在写满一个数据页的时候，直接申请另一个新数据页接着写数据就可以了；如果主键为非自增Id，为了确保索引有序，mysql 就需要将每次插入的数据都放到合适的位置上。当往一个快满或已满的数据页中插入数据时，新插入的数据会将数据页写满，mysql 就需要申请新的数据页，并且把上个数据页中的部分数据挪到新的数据页上，这就造成了页分裂，这个大量移动数据的过程是会严重影响插入效率的。
> - 其实对主键 id 还有一个小小的要求，在满足业务需求的情况下，尽量使用占空间更小的主键 id，因为普通索引的叶子节点上保存的是主键 id 的值，如果主键 id 占空间较大的话，那将会成倍增加 mysql 空间占用大小。

### 三、使用自增主键时的注意事项

- REPLACE   INTO... 对主键的影响

```
  REPLACE INTO .. .每次插入的时候如果唯一索引对应的数据已经存在，会删除原数据，然后重新插入新的数据，这也就导致id会增大。就是说实际表中只有20条数据，但是id已经增大到了20以上，如果表中的数据非常多时，有朝一日id会超出范围。
```

- INSERT ...  ON  DUPLICATE  KEY  UPDATE ... 对主键的影响

```
  引用Mysql官方文档中的描述：With ON DUPLICATE KEY UPDATE, the affected-rows value per row is 1 if the row is inserted as a new row, 2 if an existing row is updated, and 0 if an existing row is set to its current values.就是说，如果上面这条语句如果是插入的话，影响行数是1，更新的话影响行数是2，0的话就是存在且更新前后值一样，所以它也会带来和replace into... 一样的结果，因此要谨慎使用。
```

### 四、Mysql SELECT 语句优化

#### 1.WHERE 字句优化

- 删除不必要的括号：
  `((a AND b) AND c OR (((a AND b) AND (c AND d))))`
  -> `(a AND b AND c) OR (a AND b AND c AND d)`
- 恒定折叠:
  `(a<b AND b=c) AND a=5`
  -> `b>5 AND b=c AND a=5`
- 恒定条件去除:
  `(b>=5 AND b=5) OR (b=6 AND 5=5) OR (b=7 AND 5=6)`
  -> `b=5 OR b=6`

