# 《HeadFirst设计模式》第五章单件模式-读书笔记

### 案例代码链接：https://github.com/rainweb521/My-tutorial/tree/master/Design_patterns

## 1. 什么是单件

在我们的系统中，有一些对象其实我们只需要一个，比如说：线程池、缓存、对话框、注册表、日志对象、充当打印机、显卡等设备驱动程序的对象。事实上，这一类对象只能有一个实例，如果制造出多个实例就可能会导致一些问题的产生，比如：程序的行为异常、资源使用过量、或者不一致性的结果。

**使用单例模式的好处:**

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
- 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

## 2.spring中的单件模式

**Spring 中 bean 的默认作用域就是 singleton(单例)的。** 除了 singleton 作用域，Spring 中 bean 还有下面几种作用域：

- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
- session : 每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
- global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

## 3.经典的单件模式

![image-20191019084155597](http://cos.rain1024.com/markdown/image-20191019084155597.png)



## 4.设计模式

> **单件模式**：确保一个类只有一个实例，并提供一个全局访问点



## 5.改善多线程

### 5.1使用synchronized

```
//    用getInstance方法实例化对象，并返回这个实例
//    通过增加synchronized关键字到getInstance方法中，我们迫使每个
//    线程在进入这个方法之前，要先等候别的线程离开该方法。也就是说，
//    不会有两个线程可以同时进入这个方法。
    public static synchronized Singleton2 getInstance() {
        if (uniqueInstance==null){
            uniqueInstance = new Singleton2();
        }
        return uniqueInstance;
    }
```



> 同步会降低性能，只有第一次执行此方法时，才真正需要同步。换句话说,一旦设置好 uniquelnstance变量,就不再需要同步这个方法了。之后每次调用这个方法,同步都是一种累赘。
>
> 如果你的应用程序可以接受 getlnstance造成的额外负担,就忘了这件事吧。同步 getinstance的方法既简单又有效。但是你必须知道,同步一个方法可能造成程序执行效率下降100倍。因此,如果将 getinstance的程序使用在频繁运行的地方,你可能就得重新考虑了。

### 5.2使用"急切"创建实例，而不用延迟实例化的做法

> 如果应用程序总是创建并使用单件实例,或者在创建和运行时方面的负担不太繁重,你可能想要急切( eagerly)创建此单件,如下所示。

```
public class Singleton3 {

//    利用一个静态变量来记录Singleton类对唯一实例
//    在静态初始化器中创建单件，这段代码保证了线程安全
    private static Singleton3 uniqueInstance = new Singleton3();

//    把构造器声明为私有的，只有自Singleton类内才可以调用构造器
    private Singleton3() {}

//    用getInstance方法实例化对象，并返回这个实例
    public static Singleton3 getInstance() {
        
        return uniqueInstance;
    }
}
```



> 利用这个做法,我们依赖JVM在加载这个类时马上创建此唯一的单件实例。JWM保证在任何线程访问 uniquelnstancei静态变量之前,一定先创建此实例。

### 5.3“双重检查加锁”,在 getinstance()中少使用同步



> 利用双重检查加锁( double- checked locking),首先检査是否实例已经创建了,如果尚未创建,“オ”进行同步。这样一来,只有第一次会同步,这正是我们想要的。

```
public class Singleton4 {

//    利用一个静态变量来记录Singleton类对唯一实例
//    volatile并健词确保:当uniqueInstance变量被初始化成Singleton实例时,
//    多个线程正确的处理uniqueInstance变量
    private volatile static Singleton4 uniqueInstance;

//    把构造器声明为私有的，只有自Singleton类内才可以调用构造器
    private Singleton4() {}

//    用getInstance方法实例化对象，并返回这个实例
//    检查实例，如果不存在，就进入同步区
    public static Singleton4 getInstance() {
//        只有第一次才彻底执行这里的代码。
        if (uniqueInstance==null){
            synchronized (Singleton4.class){
//                进入区块后，再检查一次，如果仍是null，才创建实例
                if (uniqueInstance == null){
                    uniqueInstance = new Singleton4();
                }
            }
        }
        return uniqueInstance;
    }
}
```



> 如果性能是你关心的重点,那么这个做法可以帮你大大地减少 getlnstanceo的时间耗费。
