# 《HeadFirst设计模式》第八章模版方法模式-读书笔记

### 案例代码链接：https://github.com/rainweb521/My-tutorial/tree/master/Design_patterns

## 1. 找相同

### 1.1在冲泡咖啡和茶的时候有以下两种操作步骤

咖啡冲泡法

1. 把水煮沸
2. 用沸水冲泡咖啡
3. 把咖啡倒进杯子
4. 加糖和牛奶

茶冲泡法

1. 把水煮沸
2. 用沸水浸泡茶叶
3. 把茶倒进杯子
4. 加柠檬

### 1.2 实现咖啡和茶的冲泡

```
public class Coffee {
    void prepareRecipe(){
        boilWater();
        brewCoffeeGrinds();
        pourInCup();
        addSugarAndMilk();

    }
    public void boilWater(){
        System.out.println("把水煮沸");
    }
    public void brewCoffeeGrinds(){
        System.out.println("用沸水冲泡咖啡");
    }
    public void pourInCup(){
        System.out.println("把咖啡倒进杯子");
    }
    public void addSugarAndMilk(){
        System.out.println("添加糖和牛奶");
    }
}
```



```
public class Tea {
    void prepareRecipe(){
        boilWater();
        steepTeaBag();
        pourInCup();
        addLemon();

    }
    public void boilWater(){
        System.out.println("把水煮沸");
    }
    public void steepTeaBag(){
        System.out.println("用沸水冲泡茶");
    }
    public void pourInCup(){
        System.out.println("把茶倒进杯子");
    }
    public void addLemon(){
        System.out.println("添加柠檬");
    }
}

```



### 2. 抽取共同点

> 我们将“把水煮沸”，“倒进杯子”抽取出来，浸泡和冲泡都属于泡，而加柠檬和加糖，牛奶，都是添加配料的操作。所以根据这些条件，进行抽取，抽象。

### 2.1 类图

![image-20191029083253035](http://cos.rain1024.com/markdown/image-20191029083253035.png)

### 2.2 抽取代码

```
public abstract class CaffeineBeverage {
    final void prepareRecipe(){
        boilWater();
        brew();
        addCondimennts();
        pourInCup();
    }
    abstract void brew();
    abstract void addCondimennts();
    public void boilWater(){
        System.out.println("把水煮沸");
    }
    public void pourInCup(){
        System.out.println("倒进杯子");
    }
}
```

```
public class Coffee extends CaffeineBeverage{
    public void brew(){
        System.out.println("用沸水冲泡咖啡");
    }
    public void addCondimennts(){
        System.out.println("添加糖和牛奶");
    }
}
```

```
public class Tea extends CaffeineBeverage{
    public void brew(){
        System.out.println("用沸水冲泡茶");
    }
    public void addCondimennts(){
        System.out.println("添加柠檬");
    }
}
```



## 3.认识模版方法

### 3.1模板方法定义了一个算法的步骤,并容许子类为一个或多个步骤提供突现。

1. 首先我们需要一个茶对象

   Tea mytea=new Tea()

2. 然后我们调用这个模板方法

   mytea. preparerecipe()

   它会依照算法来制作咖啡因饮料

3. 首先,把水煮沸:

    boilwater()

4. 接下来，我们需要泡茶，这件事情只有子类才知道要怎
   么做:
   brew() ;
5. 现在把茶倒进杯子中;所有的饮料做法都一-样，所以这件事情发生
   在超类中:
   pour InCup() ;
6. 最后，我们加进调料，由于调料是各个饮料独有的，所以由子类来
   实现它:
   addCondiments () ;

### 3.2 定义模版方法模式

> 模版方法模式：在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模版方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。



#### 模版方法的定义

![image-20191029195131095](http://cos.rain1024.com/markdown/image-20191029195131095.png)

## 4.有趣的钩子

> 钩子是一种被声明在抽象类中的方法,但只有空的或者默认的实现。钩子的存在, 可以让子类有能力对算法的不同点进行挂钩。要不要挂钩,由子类自行决定。

有了钩子,我能够决定要不要覆盖方法。如果我不提供自己的方法,抽象类会提供一个默认的突现。

#### 基类

```

public abstract class CaffeineBeverage {
    final void prepareRecipe(){
        boilWater();
        brew();
        pourInCup();
        /**
         * 残们加上了一个小的条件语句,而该条件是否成立,是由一个
         * 具体方法customerWantsCondiments()決定的。
         * 如果顾客“想要”调料,有这时我们才调用addCondimennts()
         */
        if (customerWantsCondiments()){
            addCondimennts();
        }
    }
    /**
     * 残们在这里定义了-个方法, (通常)是空的缺省实现。
     * 这个方法会返回true,不做别的事。
     * 这就是一个钩子,子类可以覆盖这个方法,但不见得一定要这么做。
     * @return
     */
    boolean customerWantsCondiments(){
        return true;
    }
    abstract void brew();
    abstract void addCondimennts();
    public void boilWater(){
        System.out.println("把水煮沸");
    }
    public void pourInCup(){
        System.out.println("倒进杯子");
    }
}
```

#### 子类中

```
public class Coffee extends CaffeineBeverage{
//    用户输入的值
    private String answer;
    public void brew(){
        System.out.println("用沸水冲泡咖啡");
    }
    public void addCondimennts(){
        System.out.println("添加糖和牛奶");
    }
    //覆盖钩子，提供自己的功能
    @Override
    boolean customerWantsCondiments() {
//        让用户根据他们的输入来判断是否需要添加配料
        if (answer.toLowerCase().startsWith("y")){
            return true;
        }else {
            return false;
        }
    }
}
```



> 这个例子实在很酷,钩子竟然能够作为条件控制,影响抽象类中的算法流程。

## 5.好莱坞原则

> 好莱坞原则：别调用(打电话给)我们,我们会调用(打电话给)你。

好莱坞原则可以给我们一种防止“依赖腐败”的方法。

当高层组件依赖低层组件,而低层组件又依赖高层组件,而高层组件又依赖边侧组件,而边侧组件又依赖低层组件时, 依赖腐败就发生了。在这种情况下,没有人可以轻易地搞懂系统是如何设计的。
在好莱坞原则之下,我们允许低层组件将自己挂钩到系统上,但是高层组件会决定什么时候和怎样使用这些低层组件。换句话说,高层组件对待低层组件的方式是“别调用我们,我们会调用你”

#### 好莱坞原则和依赖倒置原则之间的关系如何? 

答依赖倒置原则教我们尽量避免使用具体类,而多使用抽象而好菜坞原则是用在创建框架或组件上的一种技巧,好让低层组件能够被挂钩进计算中,而且又不会让高层组件依赖低层组件。两者的目标都是在于解耦,但是依赖倒置原则更加注重如何在设计中避免依赖。
好菜坞原则教我们一个技巧,创建个有弹性的设计,允许低层结构能够互相操作,而又防止其他类太过依赖它们。