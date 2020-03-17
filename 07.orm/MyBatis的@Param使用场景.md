# MyBatis 中 @Param 注解的四种使用场景

[TOC]

## 第一种：方法有多个参数，需要 @Param 注解

例如下面这样：

```
@Mapper
public interface UserMapper {
    Integer insert(@Param("username") String username, @Param("address") String address);
}
复制代码
```

对应的 XML 文件如下：

```
<insert id="insert" parameterType="org.javaboy.helloboot.bean.User">
    insert into user (username,address) values (#{username},#{address});
</insert>
复制代码
```

这是最常见的需要添加 @Param 注解的场景。

## 第二种：方法参数要取别名，需要 @Param 注解

当需要给参数取一个别名的时候，我们也需要 @Param 注解，例如方法定义如下：

```
@Mapper
public interface UserMapper {
    User getUserByUsername(@Param("name") String username);
}
复制代码
```

对应的 XML 定义如下：

```
<select id="getUserByUsername" parameterType="org.javaboy.helloboot.bean.User">
    select * from user where username=#{name};
</select>
复制代码
```

老实说，这种需求不多，费事。

## 第三种：XML 中的 SQL 使用了 $ ，那么参数中也需要 @Param 注解

![会有注入漏洞的问题，但是有的时候你不得不使用](https://juejin.im/equation?tex=%E4%BC%9A%E6%9C%89%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E%E7%9A%84%E9%97%AE%E9%A2%98%EF%BC%8C%E4%BD%86%E6%98%AF%E6%9C%89%E7%9A%84%E6%97%B6%E5%80%99%E4%BD%A0%E4%B8%8D%E5%BE%97%E4%B8%8D%E4%BD%BF%E7%94%A8) 符号，例如要传入列名或者表名的时候，这个时候必须要添加 @Param 注解，例如：

```
@Mapper
public interface UserMapper {
    List<User> getAllUsers(@Param("order_by")String order_by);
}
复制代码
```

对应的 XML 定义如下：

```
<select id="getAllUsers" resultType="org.javaboy.helloboot.bean.User">
    select * from user
    <if test="order_by!=null and order_by!=''">
        order by ${order_by} desc
    </if>
</select>
复制代码
```

前面这三种，都很容易懂，相信很多小伙伴也都懂，除了这三种常见的场景之外，还有一个特殊的场景，经常被人忽略。

## 第四种，那就是动态 SQL ，如果在动态 SQL 中使用了参数作为变量，那么也需要 @Param 注解，即使你只有一个参数。

如果我们在动态 SQL 中用到了 参数作为判断条件，那么也是一定要加 @Param 注解的，例如如下方法：

```
@Mapper
public interface UserMapper {
    List<User> getUserById(@Param("id")Integer id);
}
复制代码
```

定义出来的 SQL 如下：

```
<select id="getUserById" resultType="org.javaboy.helloboot.bean.User">
    select * from user
    <if test="id!=null">
        where id=#{id}
    </if>
</select>
复制代码
```

这种情况，即使只有一个参数，也需要添加 @Param 注解，而这种情况却经常被人忽略！