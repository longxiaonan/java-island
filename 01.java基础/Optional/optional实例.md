参考：https://juejin.im/post/5d33bdeb51882557d67fe513

### 案例1：使用工具类方法改良可能抛出异常的API

Java方法处理异常结果的方式有两种：返回null（或错误码）；抛出异常，例如：Integer.parseInt(String)这个方法——如果无法解析到对应的整型，该方法就抛出一个NumberFormationException，这种情况下我们一般会使用try/catch语句处理异常情况。

一般我们建议将try/catch块单独提取到一个方法中，在这里使用Optional设计这个方法，代码如下。在开发中，可以尝试构建一个OptionalUtility工具类，将这些复杂的try/catch逻辑封装起来。

```
public static Optional<Integer> stringToInt(String a) {
  try{
    return Optional.of(Integer.parseInt(s));
  } catch  (NumberFormationException e) {
    return Optional.empty();
  }
}
复制代码
```

### 案例2：综合案例

现在有个方法，是尝试从一个属性映射中获取某个关键词对应的值，例子代码如下：

```
   public static int readDuration(Properties properties, String name) {
        String value = properties.getProperty(name);
        if (value != null) {
            try {
                int i = Integer.parseInt(value);
                if (i > 0) {
                    return i;
                }
            } catch (NumberFormatException e) {

            }
        }
        return 0;
    }
复制代码
```

使用Optional的写法后，代码如下所示：

```
    public static int readDurationWithOptional(Properties properties, String name) {
        return Optional.ofNullable(properties.getProperty(name))
            .flatMap(OptionalUtility::stringToInt)
            .filter(integer -> integer > 0)
            .orElse(0);
    }
复制代码
```

如果需要访问的属性值不存在，Properites.getProperty(String)方法的返回值就是一个null，使用noNullable工厂方法就可以将该值转换为Optional对象；接下来，可以使用flatMap将一个Optional转换为Optional对象；最后使用filter过滤掉负数，然后就可以使用orElse获取属性值，如果拿不到则返回默认值0。