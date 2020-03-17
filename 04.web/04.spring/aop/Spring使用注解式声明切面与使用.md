# Spring使用注解式声明切面与使用

> 参考项目javasea-web-aspect
>
> 参考自：<https://juejin.im/post/5d394d4ce51d4550a629b35a?utm_source=gold_browser_extension>

### 1 什么是面向切面

这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。

AOP是Spring提供的关键特性之一。AOP即面向切面编程，是OOP编程的有效补充。使用AOP技术，可以将一些系统性相关的编程工作，独立提取出来，独立实现，然后通过切面切入进系统。从而避免了在业务逻辑的代码中混入很多的系统相关的逻辑——比如权限管理，事物管理，日志记录等等。这些系统性的编程工作都可以独立编码实现，然后通过AOP技术切入进系统即可。从而达到了 将不同的关注点分离出来的效果。

### 2 AOP术语

#### 2.1 通知（Advice）

切面必须要完成的工作即称为通知。通知定义了切面是什么以及什么时候实用。

spring切面可以实用的5种类型通知：

- 前置通知（Before）：在目标方法被调用之前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
- 返回通知（After-returning）：在目标方法成功执行之后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

#### 2.2 连接点（Join point）

我们的应用可能有数以千计的时机应用通知。这些时机被称 为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

#### 2.3 切点（Poincut）

切点定义了从何处切入。切点的定义会匹配通知所要织入的一个或多个连接点。通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

#### 2.4 切面（Aspect）

切面是通知和切点的结合。通知和切点共同定义了切面的全部内容----它是什么，在何时和何处完成其功能。

#### 2.5 引入（Introduction）

引入允许我们向现有的类添加新方法或属性。

#### 2.6 织入（Weaving）

织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。

- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入（load-timeweaving，LTW）就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

### 3 Spring对切面的支持

Spring提供了4种类型的AOP支持：

- 基于代理的经典Spring AOP；
- 纯POJO切面；
- @AspectJ注解驱动的切面；
- 注入式AspectJ切面（适用于Spring各版本）。

前三种都是Spring AOP实现的变体，Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于方法拦截。

### 4 认识切点

在Spring AOP中，要使用AspectJ的切点表达式语言来定义切点。

首先定义一个接口来作为切点：

```
public interface Performance {
    void perform();
}
复制代码
```

假设我们想编写Performance的perform()方法触发的通 知。下面的表达式能够设置当perform()方法执行时触发通知的调用。

```
execution(* com.wtj.springlearn.aop.Performance.perform(..))
复制代码
```

execution()指示器选择Performance的perform()方法。方法表达式以“*”号开始，表明了不关心方法返回值的类型。然后指定了全限定类名和方法名。对于方法参数列表，使用两个点号（..）表明切点要选择任意的perform()方法，无论该方法的入参是什么。

如果我们需要设置切点匹配com.wtj.springlearn.aop包，可以使用within()来限定匹配。

```
execution(* com.wtj.springlearn.aop.Performance.perform(..)） && within(com.wtj.springlearn.aop.*)
复制代码
```

表示com.wtj.springlearn.aop包下任意类的方法被调用时。

使用“&&”操作符把execution()和within()指示器连接在一起形成与（and）关系（切点必须匹配所有的指示器）。类似地，我们可以使用“||”操作符来标识或（or）关系，而使用“!”操作符来标识非（not）操作。

因为“&”在XML中有特殊含义，所以在Spring的XML配置里面描述切点时，我们可以使用and来代替“&&”。同样，or和not可以分别用来代替“||”和“!”。

还可以使用bean的ID来标识bean。bean()使用bean ID或bean名称作为参数来限制切点只匹配特定的bean。

```
execution(* com.wtj.springlearn.aop.Performance.perform(..)) && bean('book')
复制代码
```

这里表示执行perform方法时通知，但是只限于bean的ID为book。

### 5 通过注解创建切面

> 本篇主要介绍注解方式的切面定义方式

通过@Aspect进行标注，表示该Audience不仅是一个POJO还是一个切面。类中的方法表示了切面的具体行为。

Spring提供了五种注解来定义通知时间：



![img](https://user-gold-cdn.xitu.io/2019/7/25/16c27d6f5b7f3529?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



首先创建一个切面：

```
@Aspect
public class Audience {

    //表演前 手机静音
    @Before("execution(* com.wtj.springlearn.aop.Performance.perform(..))")
    public void silenceCellPhone(){
        System.out.println("silence Cell Phone");
    }

    //表演成功-clap
    @AfterReturning("execution(** com.wtj.springlearn.aop.Performance.perform(..))")
    public void clap(){
    System.out.println("clap clap clap");
    }

    //表演失败-退款
    @AfterThrowing("execution(** com.wtj.springlearn.aop.Performance.perform(..))")
    public void refund(){
        System.out.println("refund refund refund");
    }

}
复制代码
```

Performance的实现类：

```
@Component
public class PerformanceImpl implements Performance {
    public void perform() {
    System.out.println("the perform is good");
    }
}
复制代码
```

**最后还需要开启自动代理功能**，通过JavaConfig进行配置，使用`@EnableAspectJAutoProxy`标签开启。

```
@Configuration
@EnableAspectJAutoProxy
@ComponentScan
public class AudienceConfig {
    @Bean
    public Audience audience(){
        return new Audience();
    }
}
复制代码
```

最后通过一个简单的测试用例就可以来验证了。

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AudienceConfig.class)
public class PerformanceTest {

    @Autowired
    private Performance performance;

    @Test
    public void perTest(){
        performance.perform();
    }
}
复制代码
```

打印结果：

```
silence Cell Phone
the perform is good
clap clap clap
复制代码
```

#### 5.1 @PointCut声明切点

你会发现上面切面的方法中，切点的声明都是一样的，这种情况下可以使用`@Pointcut`注解来定义切点。

```
@Pointcut("execution(* com.wtj.springlearn.aop.Performance.perform(..))")
    public void per(){};

    //表演前 手机静音
    @Before("per()")
    public void silenceCellPhone(){
        System.out.println("silence Cell Phone");
    }
复制代码
```

per()方法本身并不重要，该方法只是一个标识，供@PointCut注解依附。

#### 5.2 环绕通知

环绕通知是最为强大的通知类型。它能够让你所编写的逻辑将被通知的目标方法完全包装起来。实际上就像在一个通知方法中同时编写前置通知和后置通知。

重写Audience切面，使用环绕通知替代之前多个不同的前置通知和后置通知。

```
@Around("per()")
    public void watch(ProceedingJoinPoint point) throws Throwable {
        try{
            System.out.println("silence Cell Phone");
            point.proceed();
            System.out.println("clap clap clap");
        }catch (Exception e){
            System.out.println("refund refund refund");
        }
    }
复制代码
```

首先注意到的可能是它接受ProceedingJoinPoint作为参数。这个对象是必须要有的，因为你要在通知中通过它来调用被通知的方法。通知方法中可以做任何的事情，当要将控制权交给被通知的方法时，它需要调用ProceedingJoinPoint的proceed()方法。

如果不调proceed()这个方法的话，那么你的通知实际上会阻塞对被通知方法的调用。同样的，你也可以调用多次。

#### 5.3 向通知中传入参数

上面我们创建的切面都很简单，没有任何参数。那么切面能访问和使用传递给被通知方法的参数么？

Performance中新增方法：

```
void perform(String name);
复制代码
```

实现类：

```
public void perform(String name) {
        System.out.println("下面请 "+name+" 开始他的表演");
    }
复制代码
```

修改Audience中的切点和切面

```
@Pointcut("execution(* com.wtj.springlearn.aop.Performance.perform(String)) && args(name)")
    public void per(String name){};

    @Around("per(name)")
    public void toWatch(ProceedingJoinPoint point,String name) throws Throwable {
        try{
            point.proceed();
            System.out.println(name +" 上场啦");
            System.out.println(name +" 演出结束");
        }catch (Exception e){
            System.out.println("refund refund refund");
        }
    }
复制代码
```

表达式`args(name)`限定符，它表示传递给perform(String name)方法的String类型参数也会传到通知中去，参数名与切点中的参数名相同。`perform(String)`指明了传入参数的类型。

然后在`@Around`注解中指明切点与参数名，这样就完成了参数转移。

最后修改一下测试用例就完成了

```
@Test
    public void perTest(){
        performance.perform("渣渣辉");
    }
复制代码
```

打印输出：

```
下面请 渣渣辉 开始他的表演
渣渣辉 上场啦
渣渣辉 演出结束
复制代码
```

#### 5.4 通过注解@DeclareParents引入新方法

如果我们想在一个类上新增方法，通常情况下我们会怎么做呢？最简单的办法就是在此目标类上增加此方法，但是如果原目标类非常复杂，动一发而牵全身。并且有些时候我们是没有目标类的源码的，哪这个时候怎么办呢？

我们可以为需要添加的方法建立一个类，然后建一个代理类，同时代理该类和目标类。用一个图来表示



![img](https://user-gold-cdn.xitu.io/2019/7/25/16c27d73eae52454?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



当引入接口的方法被调用时，代理会把此调用委托给实现了新接口的某个其他对象。

还是上面的例子，假设我们需要让表演者跳起来。

新建Jump接口以及实现类：

```
public interface Jump {
    void duJump();
}
复制代码
public class JumpImpl implements Jump {
    public void duJump() {
        System.out.println("do Jump");
    }
}
复制代码
```

然后我们代理两个类：

```
@Aspect
public class JumpIntroducer {
    @DeclareParents(value = "com.wtj.springlearn.aop.Performance+",defaultImpl = JumpImpl.class)
    public static Jump jump;
}
复制代码
```

@DeclareParents注解由三部分组成：

- value属性指定了哪种类型的bean要引入该接口。在本例中，也就是所有实现Performance的类型。（标记符后面的加号表示是Performance的所有子类型，而不是Performance本 身。）
- defaultImpl属性指定了为引入功能提供实现的类。
- @DeclareParents注解所标注的静态属性指明了要引入了接 口。

**通过配置将JumpIntroducer声明**

```
@ComponentScan
@Configuration
@EnableAspectJAutoProxy
public class JumpConfig {
    @Bean
    public JumpIntroducer jumpIntroducer(){
        return new JumpIntroducer();
    }
}
复制代码
```

或者你也可以在JumpIntroducer类上加入`@Component`注解，就可以不用声明bean了。

最后通过测试用例进行测试：

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = JumpConfig.class)
public class PerformanceTest {

    @Autowired
    private Performance performance;

    @Test
    public void perTest(){
        //类型转换
        Jump jump = (Jump) performance;
        jump.duJump();
    }
}
复制代码
```

打印结果：

```
do Jump
```