# @Transactional回滚问题（try catch、嵌套）

> Spring 事务注解 @Transactional 本来可以保证原子性，如果事务内有报错的话，整个事务可以保证回滚，但是加上try catch或者事务嵌套，可能会导致事务回滚失败。测试一波。

### 准备

建两张表，模拟两个数据操作

```
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `age` smallint(3) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

CREATE TABLE `role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `role_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
复制代码
```

### 测试

根据排列组合原理，我们进行四种测试：1、无try catch、无嵌套；2、有try catch、无嵌套；3、无try catch、有嵌套；4、都有。

#### 最简单测试

如果我们单纯@Transactional，事务可以正常回滚吗？

```
    @GetMapping("/saveNormal0")
    @Transactional
    public void saveNormal0() throws Exception {
        int age = random.nextInt(100);
        User user = new User().setAge(age).setName("name:"+age);
        userService.save(user);
        throw new RuntimeException();
    }
复制代码
```

如果事务内报了RuntimeException错误，事务可以回滚。

```
    @GetMapping("/saveNormal0")
    @Transactional
    public void saveNormal0() throws Exception {
        int age = random.nextInt(100);
        User user = new User().setAge(age).setName("name:"+age);
        userService.save(user);
        throw new Exception();
    }
复制代码
```

如果事务内报了Exception错误（非RuntimeException错误），事务不可以回滚。

```
    @GetMapping("/saveNormal0")
    @Transactional( rollbackFor = Exception.class)
    public void saveNormal0() throws Exception {
        int age = random.nextInt(100);
        User user = new User().setAge(age).setName("name:"+age);
        userService.save(user);
        throw new Exception();
    }
复制代码
```

如果是Exception错误（非RuntimeException），加上 rollbackFor = Exception.class 参数也可以实现回滚。

**结论一：对于@Transactional可以保证RuntimeException错误的回滚，如果想保证非RuntimeException错误的回滚，需要加上rollbackFor = Exception.class 参数。**

#### try catch 影响

经过博主多种情况测试，发现try catch对回滚这个事本身没有什么影响，结论一照样成立。try catch只是对异常是否可以被@Transactional **感知** 到有影响。如果错误抛到切面可以感知到的地步，那就可以起作用。

```
    @GetMapping("/saveTryCatch")
    @Transactional( rollbackFor = Exception.class)
    public void saveTryCatch() throws Exception{
        try{
            int age = random.nextInt(100);
            User user = new User().setAge(age).setName("name:"+age);
            userService.save(user);
            throw new Exception();
        }catch (Exception e){
            throw e;
        }
    }
复制代码
```

比如上面一段代码就回滚了。

```
    @GetMapping("/saveTryCatch")
    @Transactional( rollbackFor = Exception.class)
    public void saveTryCatch() throws Exception{
        try{
            int age = random.nextInt(100);
            User user = new User().setAge(age).setName("name:"+age);
            userService.save(user);
            throw new Exception();
        }catch (Exception e){
        }
    }
复制代码
```

然而，将catch中的错误不继续网上抛，切面无法感知到错误，无法进行处理，那么事务就无法回滚了。

**结论二：try catch只是对异常是否可以被@Transactional 感知 到有影响。如果错误抛到切面可以感知到的地步，那就可以起作用。**

#### 事务嵌套 影响

首先经过实验，结论一仍然成立，即，当不加上rollbackFor = Exception.class 的时候，无论内外报RuntimeException，都会回滚；无论内外报 非RuntimeException 错误，都不会回滚。如果加上rollbackFor = Exception.class，无论内外怎么报错，都会回滚。这些代码就不给出了。接下来，试下下面两种情况：

```
    @GetMapping("/out")
    @Transactional( rollbackFor = Exception.class)
    public void out() throws Exception{
        innerService.inner();
        int age = random.nextInt(100);
        User user = new User().setAge(age).setName("name:" + age);
        userService.save(user);
        throw new Exception();
    }
    @Transactional
    public void inner() throws Exception{
        Role role = new Role();
        role.setRoleName("roleName:"+new Random().nextInt(100));
        roleService.save(role);
//        throw new Exception();
    }
复制代码
```

情况一，外面事务加上rollbackFor = Exception.class，里面事务不加，测试内外分别报错的情况（为了简化代码量，只给出了外面报错的代码），都可以回滚。因为，无论如何，错误都抛给了外面那个事务进行处理，而外面那个加上了rollbackFor = Exception.class，具备处理非RuntimeException错误的能力，所以都可以让事务进行正常回滚。

下面看情况二，里面的事务加上rollbackFor = Exception.class，外面不加，外面报错。

```
    @GetMapping("/out")
    @Transactional
    public void out() throws Exception{
        innerService.inner();
        int age = random.nextInt(100);
        User user = new User().setAge(age).setName("name:" + age);
        userService.save(user);
        throw new Exception();
    }
    
    @Transactional( rollbackFor = Exception.class)
    public void inner() throws Exception{
        Role role = new Role();
        role.setRoleName("roleName:"+new Random().nextInt(100));
        roleService.save(role);
    }
复制代码
```

事务都无法回滚，这是我们有个疑问，里面的事务明明有很强的处理能力啊，为什么和外面一起回滚失败呢，别着急，等等聊这个。

然后试下里面报错：

```
    @GetMapping("/out")
    @Transactional
    public void out() throws Exception{
        innerService.inner();
        int age = random.nextInt(100);
        User user = new User().setAge(age).setName("name:" + age);
        userService.save(user);
    }
     @Transactional( rollbackFor = Exception.class)
    public void inner() throws Exception{
        Role role = new Role();
        role.setRoleName("roleName:"+new Random().nextInt(100));
        roleService.save(role);
        throw new Exception();
    }
复制代码
```

咦，这回都进行了正常的回滚。我的天，这回外面没有处理能力，为什么接受里面抛出来的错误，也进行了回滚！！！看上去，就好像里外事务总是同生共死的对不对？原来，@Transactional还有个参数，看下源码，这个注解还有默认值：

```
Propagation propagation() default Propagation.REQUIRED;
复制代码
```

REQUIRED的意思是说，事务嵌套的时候，如果发现已经有事务存在了，就加入这个事务，而不是新建一个事务，所以根本就不存在两个事务，一直只有一个！至于，此参数其他值，本文不进行测试。回到上面的问题，当外面报错的时候，此时查看事务，没有增加rollbackFor = Exception.class参数，即没有处理非RuntimeException能力，所以代码走完，貌似“两个事务”，都回滚失败了。当里面报错的时候，事务已经添加上了处理非RuntimeException能力，所以，代码走完就回滚成功了。

**结论三：由于REQUIRED属性，“两个事务”其实是一个事务，处理能力看报错时刻，是否添加了处理非RuntimeException的能力。**

#### try catch和事务嵌套 共同影响

在结论一二三成立的条件下，探索共同影响的问题就简单多了，由于情况太多，就不进行过多的代码展示了。

### 结论

**结论一：对于@Transactional可以保证RuntimeException错误的回滚，如果想保证非RuntimeException错误的回滚，需要加上rollbackFor = Exception.class 参数。** **结论二：try catch只是对异常是否可以被@Transactional 感知 到有影响。如果错误抛到切面可以感知到的地步，那就可以起作用。** **结论三：由于REQUIRED属性，“两个事务”其实是一个事务，处理能力看报错时刻，是否添加了处理非RuntimeException的能力。**