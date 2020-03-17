## 判断和获取值

```java
User user = new User();
user.setAge(18);
//user或者获取到的age为null时抛出异常
Integer integer = Optional.ofNullable(user).map(a -> a.getAge()).orElseThrow(() -> new RuntimeException());
System.out.println(integer);//18
```





