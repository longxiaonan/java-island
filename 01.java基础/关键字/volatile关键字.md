## volatile作用
### volatile变量规则：
对一个变量的写操作先行发生于后面对这个变量的读操作
一旦一个共享变量（类的成员变量、类的静态成员变量）被volatile修饰之后，那么就具备了两层语义：

1） 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。
2）禁止进行指令重排序。

1. 内存可见性
变量使用了volatile之后，所有读取操作全部发生在主存中，也就避免了上述的内存不可见的问题了
2. 防止重排序
volatile关键字禁止指令重排序有两层意思：

1）当程序执行到volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
2）在进行指令优化时，不能将在对volatile变量访问的语句放在其后面执行，也不能把volatile变量后面的语句放到其前面执行。

### volidate 应用场景
#### 1.状态标记
```java
volatile boolean flag = false;
 
while(!flag){
    doSomething();
}
 
public void setFlag() {
    flag = true;
}

volatile boolean inited = false;
//线程1:
context = loadContext();  
inited = true;            
 
//线程2:
while(!inited ){
sleep()
}
doSomethingwithconfig(context);
```
#### 2.双重检查锁 （double check）
```java
class Singleton {
    private volatile static Singleton instance = null;
     
    private Singleton() {
         
    }
     
    public static Singleton getInstance() {
        if(instance==null) {
            synchronized (Singleton.class) {
                if(instance==null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```
关于双重检查锁
volatile 的两个作用

* 可见性
* 防止new Singleton()时重排序

创建对象可以简单分解为三步

1.分配内存空间
2.初始化对象
3.将对象指向刚分配的内存空间

处理器在优化性能时，可能会将第二步和第三步进行重排序，这样会导致第二个线程获取到未初始化完成的对象，导致程序异常。


链接：https://www.jianshu.com/p/573ef81a2548
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。