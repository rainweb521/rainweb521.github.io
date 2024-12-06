《HeadFirst设计模式》第十章状态模式-读书笔记

## 1.糖果销售机

这次要实现如图的糖果机，从状态图可以看出我们有四个状态，没有25分钱，有25分钱，糖果售罄，售出糖果，还有四个动作，分别是投入25分钱，退出25分钱，转动曲柄，发放糖果。

![](http://cos.rain1024.com/markdown/image-20191108074709235.png)



定义每个状态的值

```
		final static int SOLD_OUT = 0;
    final static int NO_QUARTER = 1;
    final static int HAS_QUARTER = 2;
    final static int SOLD = 3;
    int state = SOLD_OUT;
```

现在,我们创建了一个类,它的作用就像是一个状态机。对每个动作,我们都创建了一个对应的方法,这些方法利用条件语句来决定在每个状态内什么行为是恰当的。比如对“投入25分钱”这个动作来说,我们可以把对应方法写成下面的样子:

```
public void insetQuarter(){
        if (state == HAS_QUARTER){
            System.out.println("您不能再投入25分钱");
        }else if (state == SOLD_OUT){
            System.out.println("您无法投入25分钱，机器售罄");
        }else if (state == SOLD){
            System.out.println("请稍等，我们已经在给您一个糖果");
        }else if (state == NO_QUARTER){
            state = HAS_QUARTER;
            System.out.println("您投入了25分钱");
        }
    }
```

### 开始实现糖果机

现在我们来实现糖果机。我们知道要利用实例变量持有当前的状态,然后需要处理所有可能发生的动作、行为和状态的转换。我们需要实现的动作包括:投入25分钱、退回25分钱、转动曲柄和发放糖果;也要检査糖果是否售罄。

```
public class GumballMachine {
//    每个状态都用一个不同的整数代表
    final static int SOLD_OUT = 0;
    final static int NO_QUARTER = 1;
    final static int HAS_QUARTER = 2;
    final static int SOLD = 3;
//    持有当前的状态，最开始设置为糖果售罄的状态，因为糖果机开始安装时并没有装糖果
    int state = SOLD_OUT;
//    糖果数量
    int count = 0;

//    初始化糖果数量
    public GumballMachine(int count) {
        this.count = count;
        if (count>0){
            state = NO_QUARTER;
        }
    }

//    投入25分钱
    public void insetQuarter(){
        if (state == HAS_QUARTER){
            System.out.println("您不能再投入25分钱");
        }else if (state == SOLD_OUT){
            System.out.println("您无法投入25分钱，机器售罄");
        }else if (state == SOLD){
            System.out.println("请稍等，我们已经在给您一个糖果");
        }else if (state == NO_QUARTER){
            state = HAS_QUARTER;
            System.out.println("您投入了25分钱");
        }
    }
//    试着退钱
    public void ejectQuarter(){
        if (state == HAS_QUARTER){
            System.out.println("退出25分钱");
            state = NO_QUARTER;
        }else if (state == NO_QUARTER){
            System.out.println("尚未投入25分钱");
        }else if (state == SOLD){
            System.out.println("对不起，你已经拿到糖果了");
        }else if (state == SOLD_OUT){
            System.out.println("糖果售罄，尚未投入25分钱");
        }
    }
//    转动曲柄
    public void turnCrank(){
        if (state == SOLD){
            System.out.println("别想拿到两次糖果");
        }else if (state == NO_QUARTER){
            System.out.println("你需要先投入25分钱");
        }else if (state == SOLD_OUT){
            System.out.println("已经没有糖果了");
        }else if (state == HAS_QUARTER){
            System.out.println("请取出糖果");
            state = SOLD;
            dispense();
        }
    }

//    发放糖果
    private void dispense() {
        if (state == SOLD){
            System.out.println("正在售出糖果");
            count = count - 1;
            if (count == 0){
                System.out.println("糖果售罄");
                state = SOLD_OUT;
            }else{
                state = NO_QUARTER;
            }
        }else if (state == NO_QUARTER){
            System.out.println("你需要投入25分钱");
        }else if (state == SOLD_OUT){
            System.out.println("操作错误");
        }else if (state == HAS_QUARTER){
            System.out.println("操作错误");
        }
    }
    
    @Override
    public String toString() {
        return "GumballMachine{" +
                "state=" + state +
                ", count=" + count +
                '}';
    }
}

```

### 测试代码

```
public class GumballMachineTestDrive {
    public static void main(String[] args) {
        GumballMachine gumballMachine = new GumballMachine(5);
        System.out.println(gumballMachine.toString());
        gumballMachine.insetQuarter();
        gumballMachine.turnCrank();
        System.out.println(gumballMachine.toString());

        gumballMachine.insetQuarter();
        gumballMachine.ejectQuarter();
        gumballMachine.turnCrank();
        System.out.println(gumballMachine.toString());
    }
}
```

```
GumballMachine{state=1, count=5}
您投入了25分钱
请取出糖果
正在售出糖果
GumballMachine{state=1, count=4}
您投入了25分钱
退出25分钱
你需要先投入25分钱
GumballMachine{state=1, count=4}
```

## 2.变更需求

这一次他们想要把购买糖果变成一个游戏，这样能大大增加销售量，当曲柄转动时，有10%的几率掉下来的是两颗糖果，多送客户一颗。当修改代码时，很难继续去扩展，需要改动的地方太多了，每个方法都要加入一个新的条件去判断来处理“赢家的状态”。

我们需要重构这份代码,以便我们能容易地维护和修改它。
我们应该试着局部化每个状态的行为,这样一来,如果我们针对某个状态做了改变,就不会把其他的代码给搞乱了。
换句话说,遵守“封装变化”原则。
如果我们将毎个状态的行为都放在各自的类中,那么毎个状态只要实现它自己的动作就可以了。
糖果机只需要委托给代表当前状态的状态对象。

### 新的设计

我们的计划是这样的:不要维护我们现有的代码，我们重写它以便于将状态对象封装在各自的类中，然后在动作发生时委托给当前状态。
我们在这里遵照我们的设计原则，所以最后应该得到一个容易维护的设计。我们要做的事情是:
➊首先，我们定义一-个State接口。在这个接口内，糖果机的每个动作都有一个对应的方法。
❷然后为机器中的每个状态实现状态类。这些类将负责在对应的状态下进行机器的行为。
❸最后，我们要摆脱旧的条件代码，取而代之的方式是，将动作委托到状态类。
你将会看到，我们不仅遵守了设计原则，实际上我们还实现了状态模式。在重新完成代码之后我们再来了解状态模式的正式定义

