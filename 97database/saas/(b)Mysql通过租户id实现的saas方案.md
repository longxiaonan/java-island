## 概况

项目开发到一半，用户突然提出需要多个分公司共同使用，这种需要将系统设计成SaaS架构，将各个分公司的数据进行隔离。

### SaaS实现的方案

- 独立数据库

  每个企业 独立的物理数据库，隔离性好，成本高。

- 共享数据库、独立schema

  就是一台物理机，多个逻辑数据库，oracle叫做schema，mysql叫做database，每个企业独立的schema。

- 共享数据库、数据库表(本次采用)：

  在表中添加“企业”或者“租户”字段区分是哪个企业的数据。操作的时候根据“租户”字段去查询相应的数据。

  优点： 所有租户使用同一数据库，所以成本低廉。

  缺点：隔离级别低，安全性低，需要在开发时加大对安全的开发量，数据备份和恢复最困难。

### 改造思路

1. 本次采用`共享数据库、数据库表`的SaaS方案。改造时需要做以下工作：

- 创建租户信息表。
- 先要将所有的表添加租户id字段`tenant_id`。用于关联租户信息表。
- 将`tenant_id`和原始表id创建联合主键。注意主键的顺序，原表主键必须在左边。
- 将表修改为分区表。  

2. 改造后，在添加租户信息的时候，同时在所有表中添加该租户的分区，分区用于保存该租户的数据。
3. 在后续增加记录时，需要`tenant_id`字段的值，在删改查中，都需要在where条件中以`tenant_id`为条件来操作某个租户的数据。

### 测试环境介绍

测试库中有5张表，我下文使用`sys_log`表进行测试。

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5dc1d4f37a3ab?w=188&h=143&f=png&s=11010)
`sys_log`的建表语句为：

```sql
CREATE TABLE `sys_log` (
  `log_id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `type` TINYINT(1) DEFAULT NULL COMMENT '类型',
  `content` VARCHAR(255) DEFAULT NULL COMMENT '内容',
  `create_id` BIGINT(18) DEFAULT NULL COMMENT '创建人ID',
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `tenant_id` INT NOT NULL,
  PRIMARY KEY (`log_id`,`tenant_id`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC COMMENT='系统日志'
```

## 表添加租户id字段

### 找出未添加`租户id(tenant_id)字段`的表。

```sql
SELECT 
    table_name 
  FROM
    INFORMATION_SCHEMA.TABLES
  WHERE table_schema = 'my'   -- my 是我的测试数据库名称
    AND table_name NOT IN 
    (SELECT 
      t.table_name 
    FROM
      (SELECT 
        table_name,
        column_name 
      FROM
        information_schema.columns 
      WHERE table_name IN 
        (SELECT 
          table_name 
        FROM
          INFORMATION_SCHEMA.TABLES 
        WHERE table_schema = 'my')) t 
    WHERE t.column_name = 'tenant_id') ;
```

执行，找到两个符合条件的表，在数据库进行确认，确实表中没`tenant_id`字段。

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5dc36869d0350?w=256&h=159&f=png&s=19031)

### 创建租户信息表

> 仅供参考，用于保存租户信息

```sql
CREATE TABLE `t_tenant` (
  `tenant_id` varchar(40) NOT NULL DEFAULT 'c12dee54f652452b88142a0267ec74b7' COMMENT '租户id',
  `tenant_code` varchar(100) DEFAULT NULL COMMENT '租户编码',
  `name` varchar(50) DEFAULT NULL COMMENT '租户名称',
  `desc` varchar(500) DEFAULT NULL COMMENT '租户描述',
  `logo` varchar(255) DEFAULT NULL COMMENT '公司logo地址',
  `status` smallint(6) DEFAULT NULL COMMENT '状态1有效0无效',
  `create_by` varchar(100) DEFAULT NULL COMMENT '创建者',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `last_update_by` varchar(100) DEFAULT NULL COMMENT '最后修改人',
  `last_update_time` datetime DEFAULT NULL COMMENT '最后修改时间',
  `street_address` varchar(200) DEFAULT NULL COMMENT '街道楼号地址',
  `province` varchar(20) DEFAULT NULL COMMENT '一级行政单位,如广东省,上海市等',
  `city` varchar(20) DEFAULT NULL COMMENT '城市, 如广州市,佛山市等',
  `district` varchar(20) DEFAULT NULL COMMENT '行政区,如番禺区,天河区等',
  `link_man` varchar(50) DEFAULT NULL COMMENT '联系人',
  `link_phone` varchar(50) DEFAULT NULL COMMENT '联系电话',
  `longitude` decimal(10,6) DEFAULT NULL COMMENT '经度',
  `latitude` decimal(10,6) DEFAULT NULL COMMENT '纬度',
  `adcode` varchar(8) DEFAULT NULL COMMENT '区域编码,用于通过区域id快速匹配后展示, 如广州是440100',
  PRIMARY KEY (`tenant_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='租户的基本信息表';
```

### 将所有表添加`tenant_id`字段

```sql
DROP PROCEDURE IF EXISTS addColumn ;

DELIMITER $$

CREATE PROCEDURE addColumn () 
BEGIN
  -- 定义表名变量
  DECLARE s_tablename VARCHAR (100) ;
  /*显示表的数据库中的所有表
 SELECT table_name FROM information_schema.tables WHERE table_schema='databasename' Order by table_name ;
 */
  #显示所有
  DECLARE cur_table_structure CURSOR FOR 
  SELECT 
    table_name 
  FROM
    INFORMATION_SCHEMA.TABLES
  WHERE table_schema = 'my'     -- my = 我的测试数据库名称
    AND table_name NOT IN 
    (SELECT 
      t.table_name 
    FROM
      (SELECT 
        table_name,
        column_name 
      FROM
        information_schema.columns 
      WHERE table_name IN 
        (SELECT 
          table_name 
        FROM
          INFORMATION_SCHEMA.TABLES 
        WHERE table_schema = 'my')) t 
    WHERE t.column_name = 'tenant_id') ;
  DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET s_tablename = NULL ;
  OPEN cur_table_structure ;
  FETCH cur_table_structure INTO s_tablename ;
  WHILE
    (s_tablename IS NOT NULL) DO SET @MyQuery = CONCAT(
      "alter table `",
      s_tablename,
      "` add COLUMN `tenant_id` INT not null COMMENT '租户id'"
    ) ;
    PREPARE msql FROM @MyQuery ;
    EXECUTE msql ;
    #USING @c; 
    FETCH cur_table_structure INTO s_tablename ;
  END WHILE ;
  CLOSE cur_table_structure ;
END $$

DELIMITER ;

#执行存储过程
CALL addColumn () ;
```

## 实现表分区

实现的目标：在添加租户的时候实现对所有表添加分区

需要的条件：

- 表必须是分区表，如果不是分区表，那么需要改成分区表。
- `tenant_id`必须和原表`log_id`主键组成联合主键。

### 将表修改成分区表

表中添加分区有三种方式：

- 创建临时分区表`sys_log_copy`，copy数据过来后，删除旧的`sys_log`，再将`sys_log_copy`修改为`sys_log`（本次采用，详见下文）
- 直接将表修改为分区表，需要原表中无数据，否则无法成功：

```sql
-- 如果表中没数据，可以直接将表进行分区
ALTER TABLE sys_log PARTITION BY LIST COLUMNS (tenant_id)
(
    PARTITION a1 VALUES IN (1) ENGINE = INNODB,
    PARTITION a2 VALUES IN (2) ENGINE = INNODB,
    PARTITION a3 VALUES IN (3) ENGINE = INNODB
);
```

- 在分区表中添加新分区，需要表已经是分区表，否则无法成功：

```sql
-- 已经是分区表中添加分区
ALTER TABLE sys_log_copy ADD PARTITION
(
    PARTITION a4 VALUES IN (4) ENGINE = INNODB,
    PARTITION a5 VALUES IN (5) ENGINE = INNODB,
    PARTITION a6 VALUES IN (6) ENGINE = INNODB
);
```

#### 通过`创建临时分区表`的方式将原表转换成分区表

1. 查看表建表语句：

```sql
SHOW CREATE TABLE `sys_log`;
```

2. 参考建表语句，创建copy表：

```sql
CREATE TABLE `sys_log_copy` (
  `log_id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `type` TINYINT(1) DEFAULT NULL COMMENT '类型',
  `content` VARCHAR(255) DEFAULT NULL COMMENT '内容',
  `create_id` BIGINT(18) DEFAULT NULL COMMENT '创建人ID',
  `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `tenant_id` INT NOT NULL,
  PRIMARY KEY (`log_id`,`tenant_id`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='系统日志'
PARTITION BY LIST COLUMNS (tenant_id)
(
    PARTITION a1 VALUES IN (1) ENGINE = INNODB,
    PARTITION a2 VALUES IN (2) ENGINE = INNODB,
    PARTITION a3 VALUES IN (3) ENGINE = INNODB
);
```

> 注意上文中的`DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC`
> - `CHARSET=utf8mb4`是因为utf8在mysql中是不健全的编码。
> - `ROW_FORMAT=DYNAMIC`是为了避免所以长度过大后导致如下报错：
>
> ```sql
> ERROR 1709 (HY000): Index column size too large. The maximum column size is 767 bytes.
> ```
>
> 也可以在my.ini配置文件中设置为true解决这个问题，但是要重启数据库，会比较麻烦。
>
> ```sql
> [mysqld]
> innodb_large_prefix=true
> ```

3. 验证分区情况：

```sql
SELECT 
  partition_name part,
  partition_expression expr,
  partition_description descr,
  table_rows 
FROM
  information_schema.partitions 
WHERE TABLE_SCHEMA = SCHEMA() 
  AND TABLE_NAME = 'sys_log_copy' ;
```

可以查看到添加的3个分区

![](https://user-gold-cdn.xitu.io/2019/11/12/16e5f299ae608d04?w=1117&h=177&f=png&s=50631)

4. 将数据复制到copy表中

```sql
INSERT INTO `sys_log_copy` SELECT * FROM `sys_log`
```

5. 删除表`sys_log`，再修改`sys_log_copy`表中的名字为`sys_log`。

### 编写自动创建分区的仓储过程

通过存储过程实现，在分区表中添加分区

```sql
DELIMITER $$

USE `my`$$

DROP PROCEDURE IF EXISTS `add_table_partition`$$

CREATE DEFINER=`root`@`%` PROCEDURE `add_table_partition`(IN _tenantId INT)
BEGIN
  DECLARE IS_FOUND INT DEFAULT 1 ;
  -- 用于记录游标中存在分区的表名
  DECLARE v_tablename VARCHAR (200) ;
  -- 用于缓存添加分区时候的sql
  DECLARE v_sql VARCHAR (5000) ;
  -- 分区名称定义
  DECLARE V_P_VALUE VARCHAR (100) DEFAULT CONCAT('P', REPLACE(_tenantId, '-', '')) ;
  DECLARE V_COUNT INT ;
  DECLARE V_LOONUM INT DEFAULT 0 ;
  DECLARE V_NUM INT DEFAULT 0 ;
  -- 定义游标，值是所有分区表的表名
  DECLARE curr CURSOR FOR 
  (SELECT 
    t.TABLE_NAME 
  FROM
    INFORMATION_SCHEMA.partitions t 
  WHERE TABLE_SCHEMA = SCHEMA() 
    AND t.partition_name IS NOT NULL 
  GROUP BY t.TABLE_NAME) ;
  -- 如果没影响的记录，程序也继续执行
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET IS_FOUND=0;
  -- 获取上一步中的游标中获取到的表名的个数
  SELECT 
    COUNT(1) INTO V_LOONUM 
  FROM
    (SELECT 
      t.TABLE_NAME 
    FROM
      INFORMATION_SCHEMA.partitions t 
    WHERE TABLE_SCHEMA = SCHEMA() 
      AND t.partition_name IS NOT NULL 
    GROUP BY t.TABLE_NAME) A ;
  -- 只有在存在分区表的时候才打开游标
  IF V_LOONUM > 0 
  THEN -- 打开游标
  OPEN curr ;
  -- 循环
  read_loop :
  LOOP
    -- 声明结束的时候
    IF V_NUM >= V_LOONUM 
    THEN LEAVE read_loop ;
    END IF ;
    -- 取游标的值给变量
    FETCH curr INTO v_tablename ;
    -- 依次判断分区表是否存在改分区，如果不存在则添加分区
    SET V_NUM = V_NUM + 1 ;
    SELECT 
      COUNT(1) INTO V_COUNT 
    FROM
      INFORMATION_SCHEMA.partitions t 
    WHERE LOWER(T.TABLE_NAME) = LOWER(v_tablename) 
      AND T.PARTITION_NAME = V_P_VALUE 
      AND T.TABLE_SCHEMA = SCHEMA() ;
    IF V_COUNT <= 0 
    THEN SET v_sql = CONCAT(
      '  ALTER TABLE ',
      v_tablename,
      ' ADD PARTITION (PARTITION ',
      V_P_VALUE,
      ' VALUES IN(',
      _tenantId,
      ') ENGINE = INNODB) '
    ) ;
    SET @v_sql = v_sql ;
    -- 预处理需要执行的动态SQL，其中stmt是一个变量
    PREPARE stmt FROM @v_sql ;
    -- 执行SQL语句
    EXECUTE stmt ;
    -- 释放掉预处理段
    DEALLOCATE PREPARE stmt ;
    END IF ;
    -- 结束循环
  END LOOP read_loop;
  -- 关闭游标
  CLOSE curr ;
  END IF ;
END$$

DELIMITER ;
```

调用存储过程测试

```shell
CALL add_table_partition (8) ;
```

> - 如果表还不是分区表，那么调用存储过程会有如下报错：
>
> ```shell
> 错误代码： 1505
> Partition management on a not partitioned table is not possible
> ```
>
> 翻译出来的意思是：“在未分区的表上进行分区管理是不可能的”。
>
> - 可能会报错如下：
>
> ```shell
> 错误代码： 1329
> No data - zero rows fetched, selected, or processed
> ```
>
> 但如果通过查询下面的`information_schema.partitions`无误，那就是添加分区成功。
>
> 可以通过在定义游标后，打开游标之前，添加如下方式解决：
>
> ```sql
> DECLARE CONTINUE HANDLER FOR NOT FOUND SET IS_FOUND=0;
> ```

```sql
SELECT 
  partition_name part,
  partition_expression expr,
  partition_description descr,
  table_rows 
FROM
  information_schema.partitions 
WHERE TABLE_SCHEMA = SCHEMA() 
  AND TABLE_NAME = 'sys_log' ;
```

### 通过`mybatis`调用存储过程

```sql
<select id="testProcedure" statementType="CALLABLE" useCache="false" parameterType="string">
        <![CDATA[
                call add_table_partition (
                        #{_tenantId,mode=IN,jdbcType=VARCHAR});
        ]]>
</select>
```

## 实现简单的数据权限

我们可能需要这种场景需求

1. 集团公司有多个子公司，集团公司和每个子公司各自是一个租户，但是子公司下还有子公司。
2. 不管是集团公司还是说旗下的子公司都有相应的用户(t_user)。
3. 用户需要有查看自己公司下的数据和下面子公司的数据的权限。

从上面的场景需求中，我们知道，`t_tenant`表需要设计成树桩结构。下面我们进行测试。

### 修改上文的`t_tenant`表为：

```sql
CREATE TABLE `t_tenant` (
  `tenant_id` VARCHAR(40) NOT NULL DEFAULT '0' COMMENT '租户id',
  `path` VARCHAR(200) DEFAULT NOT NULL COMMENT '从根节点开始的id路径树，如：0-2-21-211-2111，通过"-"隔开，最末尾为自己id',
  `tenant_code` VARCHAR(100) DEFAULT NULL COMMENT '租户编码',
  `name` VARCHAR(50) DEFAULT NULL COMMENT '租户名称',
  `logo` VARCHAR(255) DEFAULT NULL COMMENT '公司logo地址',
  `status` SMALLINT(6) DEFAULT NULL COMMENT '状态1有效0无效',
  `create_by` VARCHAR(100) DEFAULT NULL COMMENT '创建者',
  `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
  `last_update_by` VARCHAR(100) DEFAULT NULL COMMENT '最后修改人',
  `last_update_time` DATETIME DEFAULT NULL COMMENT '最后修改时间',
  `street_address` VARCHAR(200) DEFAULT NULL COMMENT '街道楼号地址',
  PRIMARY KEY (`tenant_id`) USING BTREE
) ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='租户的基本信息表'
```

修改的地方有：

* 为了演示，删除了些感觉没是没用的字段
* 添加了`path`字段，实现租户和子租户的树形结构

### 添加测试数据

新增租户信息：

> 通过path缓存着`t_tenant`树的路径。

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7e9dad59e98a2?w=1284&h=294&f=png&s=150224)

创建用户表(`t_user`)，添加测试用户：

> 测试的用户id和tenant_id需要对应

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7ea9b2bfd30ef?w=412&h=320&f=png&s=55258)

创建附件表(`t_file`)，添加测试业务数据：

> 创建人字段(`create_by`）关联用户表(`t_user`)，也要关联到租户(`tenant_id`)，指明是哪个子公司的数据。

![](https://user-gold-cdn.xitu.io/2019/11/18/16e7eadc0e420ddf?w=667&h=202&f=png&s=66557)



### 进行测试

* 查看`tenant_id`是"211"的租户信息和其下的子租户信息

  ```sql
  SELECT 
    tt.`tenant_id`,
    tt.path 
  FROM
    t_tenant tt 
  WHERE 
    (SELECT 
      INSTR(tt.path, "211")) ;
  ```

* 查看`tenant_id`是"211"的附件和其下的子租户的附件信息

  ```sql
  SELECT 
    * 
  FROM
    t_file tf 
  WHERE tf.`tenant_id` IN 
    (SELECT 
      tt.`tenant_id` 
    FROM
      t_tenant tt 
    WHERE 
      (SELECT 
        INSTR(tt.path, "211"))) ;
  ```

  ![](https://user-gold-cdn.xitu.io/2019/11/18/16e7eb1ecf27225e?w=677&h=495&f=png&s=107395)

* 查看`tenant_id`是"2"的附件和其下的子租户的附件信息

  ![](https://user-gold-cdn.xitu.io/2019/11/18/16e7eb3feac8c2d5?w=657&h=516&f=png&s=105060)

  

### 通过mybatis拦截器实现查看子租户的数据权限

编写拦截器：

```java
package com.iee.orm.mybatis.common;

import com.baomidou.mybatisplus.core.toolkit.PluginUtils;
import com.iee.orm.mybatis.common.UserHelper;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlCommandType;
import org.apache.ibatis.mapping.StatementType;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.SystemMetaObject;
import org.springframework.context.annotation.Configuration;

import java.sql.Connection;
import java.util.Properties;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * 实现 拦截select语句，实现尾部拼接sql来查询本租户和子租户信息
 * @author longxiaonan@aliyun.com
 */
@Slf4j
@Configuration
@Intercepts({
        @Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class})
})
public class SqlInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        StatementHandler statementHandler = PluginUtils.realTarget(invocation.getTarget());
        MetaObject metaObject = SystemMetaObject.forObject(statementHandler);
        // 非select语句或者是存储过程 则跳过)
        MappedStatement mappedStatement = (MappedStatement) metaObject.getValue("delegate.mappedstatement");
        if (SqlCommandType.SELECT != mappedStatement.getSqlCommandType()
                || StatementType.CALLABLE == mappedStatement.getStatementType()) {
            return invocation.proceed();
        }
        // 拼接sql执行
        getSqlByInvocation(metaObject, invocation);
        return invocation.proceed();
    }

    /**
     * 拼接sql执行
     * @param metaObject
     * @param invocation
     * @return
     */
    private String getSqlByInvocation(MetaObject metaObject, Invocation invocation) throws NoSuchFieldException, IllegalAccessException {
        //在原始的sql中拼装sql，方式一
        String originalSql = (String) metaObject.getValue(PluginUtils.DELEGATE_BOUNDSQL_SQL);
        metaObject.setValue(PluginUtils.DELEGATE_BOUNDSQL_SQL, originalSql);
        String targetSql = addDataSql(originalSql);
        return targetSql;

        //在原始的sql中拼装sql，方式二
//        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
//        BoundSql boundSql = statementHandler.getBoundSql();
//        String sql = boundSql.getSql();
//        Field field = boundSql.getClass().getDeclaredField("sql");
//        field.setAccessible(true);
//        field.set(boundSql, addDataSql(sql));
//        return sql;
    }

    /**
     * 将原始sql进行拼接
     * @param sql
     * @return
     */
    static String addDataSql(String sql) {
        //需要去掉“;" 因为“;"表示sql结束，不去掉那么后面的拼接会受到影响
        sql = StringUtils.replace(sql, ";", "");
        StringBuilder sb = new StringBuilder(sql);
        String tenantId = UserHelper.getTenantId();

        //用于查看子租户信息的sql后缀
        String suffSql = " `tenant_id` IN " +
                "(SELECT " +
                "tt.`tenant_id` " +
                "FROM " +
                "t_tenant tt " +
                "WHERE " +
                "(SELECT " +
                "INSTR(tt.path," + tenantId + "))) ";

        String regex = "(.*)(where)(.*)";
        Pattern compile = Pattern.compile(regex);
        Matcher matcher = compile.matcher(sql);
        if (matcher.find()) {
            String whereLastSql = matcher.group(matcher.groupCount());
            //where 的条件存在 且是括号对的情况下，是不能再加“where”的, 但是需要添加“and”
            int left = StringUtils.countMatches(whereLastSql, "(");
            int right = StringUtils.countMatches(whereLastSql, ")");
            if(left == right){
                sb.append(" and ");
                sb.append(suffSql);
                log.info("数据权限替换后sql:--->" + sb.toString());
                return sb.toString();
            }
        }
        //其他情况下需要添加where
        sb.append(" where ");
        sb.append(suffSql);
        log.info("数据权限替换后sql:--->" + sb.toString());
        return sb.toString();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, new SqlInterceptor());
    }

    @Override
    public void setProperties(Properties properties) {
    }
}
```

用到了mybatis-plus的工具类

```sql
/*
 * Copyright (c) 2011-2020, baomidou (jobob@qq.com).
 * <p>
 * Licensed under the Apache License, Version 2.0 (the "License"); you may not
 * use this file except in compliance with the License. You may obtain a copy of
 * the License at
 * <p>
 * https://www.apache.org/licenses/LICENSE-2.0
 * <p>
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
 * WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
 * License for the specific language governing permissions and limitations under
 * the License.
 */
package com.baomidou.mybatisplus.core.toolkit;

import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.SystemMetaObject;

import java.lang.reflect.Proxy;
import java.util.Properties;

/**
 * 插件工具类
 *
 * @author TaoYu , hubin
 * @since 2017-06-20
 */
public final class PluginUtils {
    public static final String DELEGATE_BOUNDSQL_SQL = "delegate.boundSql.sql";

    private PluginUtils() {
        // to do nothing
    }

    /**
     * 获得真正的处理对象,可能多层代理.
     */
    @SuppressWarnings("unchecked")
    public static <T> T realTarget(Object target) {
        if (Proxy.isProxyClass(target.getClass())) {
            MetaObject metaObject = SystemMetaObject.forObject(target);
            return realTarget(metaObject.getValue("h.target"));
        }
        return (T) target;
    }

    /**
     * 根据 key 获取 Properties 的值
     */
    public static String getProperty(Properties properties, String key) {
        String value = properties.getProperty(key);
        return StringUtils.isEmpty(value) ? null : value;
    }
}
```

测试的时候发现只要是select语句就会去关联查询出子租户的信息。

测试代码见：

<https://github.com/longxiaonan/java-sea/tree/master/javasea-orm/javasea-orm-mybatis-Interceptor>

**<center><font color="red">您的鼓励我的动力，请点赞支持下，谢谢！</font></center>**