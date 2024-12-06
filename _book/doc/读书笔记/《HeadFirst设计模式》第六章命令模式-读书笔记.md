# 《HeadFirst设计模式》第六章命令模式-读书笔记

### 案例代码链接：https://github.com/rainweb521/My-tutorial/tree/master/Design_patterns

## 1.背景

### 1.1 用餐厅来分析

> 从餐厅开始说起，以此来解释命令模式的流程。
>
> 订单封装了准备餐点的请求
>
> 女招待接受订单，然后调用订单的orderUp()方法
>
> 厨师准备餐点
>
> 这样，顾客和女招待是解耦的，一天内，不同的顾客有不同的订单，女招待知道所有的订单都支持orderUp()方法，每次调用这个方法就是了。
>
> 女招待和厨师也是解耦的，订单封装了餐点的细节，她只调用每个订单的方法即可，而厨师也是根据订单去做对应的餐点。他们之间不需要直接沟通。

![image-20191023073214000](http://cos.rain1024.com/markdown/image-20191023073214000.png)



### 1.2 回到遥控器

这一章需要实现一个遥控器，遥控器上只有七个插槽，而厂商有很多的类，并且以后还会新增，所以需要根据类的特征去抽取共同点，这时就用到了命令模式，将"动作的请求者"从"动作的执行者“对象中解耦。

利用命令对象，把请求（例如打开电灯）封装成一个特定对象（例如客厅电灯对象）。所以，如果对每个按钮都存储一个命令对象，那么当按钮被按下的时候，就可以请命令对象做相关的工作。遥控器并不需要知道工作内容是什么，只要有个命令对象能和正确的对象沟通，把事情做好就可以了。所以，遥控器和电灯对象解耦了。
由于对象之间是如此的解耦，要描述这个模式实际的工作并不容易。
使用这个模式，我们能够创建一个API，将这些命令对象加载到按钮插槽，让遥控器的代码尽量保持简单。而把家电自动化的工作和进行该工作的对象一起封装在命令对象中。

## 2.开始设计

### 2.1第一个命令对象

实现命令接口，让所有的命令对象实现相同的包含一个方法的接口。

```
public interface Command {
    public void execute();
}

```

实现打开电灯的命令

```
public class LightOnCommand implements Command {
    Light light;
//    给构造器传入某个电灯，方便以后调用execute
    public LightOnCommand(Light light){
        this.light = light;
    }
//    调用接受对象的on方法
    public void execute() {
        light.on();
    }
}
```

灯的类

```
public class Light {
		private String name;
    public Light(String living_room) {
        this.name = living_room;
    }
    public Light() {
    }
    public void on(){
        System.out.println("打开灯");
    }
    public void off(){
        System.out.println("关闭灯");
    }
}
```

### 2.2 使用命令对象

假设只有一个遥控器，只有一个按钮和对应的插槽。

```
public class SimpleRemoteController {
//    插槽持有命令，而这个命令控制着一个装置
    Command slot;

    public SimpleRemoteController() { }

//    用来设置插槽控制的命令，如果想要改变命令，可以多次调用
    public void setCommand(Command command) {
        this.slot = command;
    }
//    按下按钮，方法就会被调用
    public void buttonWasPressed(){
        slot.execute();
    }
}
```

### 2.3 测试遥控器

```
public class RemoteControllerTest {
    public static void main(String[] args) {
//        实例化遥控器
        SimpleRemoteController remote = new SimpleRemoteController();
//        创建一个电灯对象，此对象就是请求接受者
        Light light = new Light();
//        创建命令，并将接收者传给它
        LightOnCommand lightOn = new LightOnCommand(light);
//        将命令传给调用者
        remote.setCommand(lightOn);
//        模拟按下按钮
        remote.buttonWasPressed();
    }
}

```



## 3.设计模式

### 3.1 模式定义

> 命令模式：将请求封装成对象，以便使用不用的请求，队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

现在,仔细看这个定义。我们知道一个命令对象通过在特定接收者上绑定一组动作来封装一个请求。要达到这一点,命令对象将动作和接收者包进对象中。这个对象只暴露出一个execute方法,当此方法被调用的时候,接收者就会进行这些动作。从外面来看,其他对象不知道究竟哪个接收者进行了哪些动作,只知道如果调用 execute方法,请求的目的就能达到。

### 3.2 类图

![image-20191023224003672](http://cos.rain1024.com/markdown/image-20191023224003672.png)

## 4.开始实现

### 4.1 实现遥控器

```
public class RemoteControl {
    Command[] onCommands;
    Command[] offCommands;

    /**
     * 初始化这两个开关的数组
     */
    public RemoteControl(){
        onCommands = new Command[7];
        offCommands = new Command[7];

        Command noCommand = new NoCommand();
        for (int i = 0 ;i < 7;i++){
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
    }

    /**
     * 有三个命令，分别是插槽位置，开的命令，关的命令
     * @param slot
     * @param onCommand
     * @param offCommand
     */
    public void setCommand(int slot ,Command onCommand,Command offCommand){
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }
    public void onButtonWasPushed(int slot){
        onCommands[slot].execute();
    }
    public void offButtonWasPushed(int slot){
        offCommands[slot].execute();
    }

    @Override
    public String toString() {
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append("\n-------Remote Control ------\n");
        for (int i = 0 ;i < onCommands.length;i++){
            stringBuffer.append("[slot"+i+"]"+onCommands[i].getClass().getName()+"        "
            +offCommands[i].getClass().getName()+"\n");
        }
        return stringBuffer.toString();
    }
}

```

```
public class NoCommand implements Command {
    public void execute() {

    }
}

```

> Nocommand对象是一个空对象( null object)的例子。当你不想返回一个有意义的对象时,空对象就很有用。客户也可以将处理nul的责任转移给空对象。举例来说,遥控器不可能一出厂就设置了有意义的命令对象,所以提供了 Nocommand对象作为代用品,当调用它的 execute0方法时,这种对象什么事情都不做。
> 在许多设计模式中,都会看到空对象的使用。甚至有些时候,空对象本身也被视为是种设计模式。

### 4.2实现命令

```
public class LightOffCommand implements Command {
    Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.off();
    }
}
```



### 4.3测试遥控器

```
public class RemoteLoader {
    public static void main(String[] args) {
        RemoteControl remoteControl = new RemoteControl();

        Light livingroomLight = new Light("Living Room");
        Light kitchenLight = new Light("Kitchen");
//        ceilingFan = new

        LightOnCommand LivingRoomlightOnCommand = new LightOnCommand(livingroomLight);
        LightOffCommand LivingRoomlightOffCommand = new LightOffCommand(livingroomLight);
        LightOnCommand KitchenlightOnCommand = new LightOnCommand(livingroomLight);
        LightOffCommand KitchenlightOffCommand = new LightOffCommand(kitchenLight);

//        添加命令
        remoteControl.setCommand(0,LivingRoomlightOnCommand,LivingRoomlightOffCommand);

//        打印遥控器里的信息
        System.out.println(remoteControl);

//        执行开关
        remoteControl.onButtonWasPushed(0);
        remoteControl.offButtonWasPushed(0);
    }

}
```

### 4.4 查看我们的类图

经过以上代码的实现，基本完成了所需要的功能，这个时候看一下代码的UML图，这样可以更好的理解代码之间的继承关系。

![image-20191024065748724](http://cos.rain1024.com/markdown/image-20191024065748724.png)



## 5.添加撤销

好了,我们现在需要在遥控器上加上撤销的功能。这个功能使用起来就像是这样的:比方说客厅的电灯是关闭的,然后你按下遥控器上的开启按钮,自然电灯就被打开了。现在如果按下撤销按钮,那么上一个动作将被倒转,在这个例子里,电灯将被关闭。在进入更复杂的例子之前,先让撤销按钮能够处理电灯

在命令接口中加入undo方法

```
public interface Command {
    public void execute();
    public void undo();
}

```

在打开灯的命令中，执行off操作
```
public class LightOnCommand implements Command {
    Light light;
//    给构造器传入某个电灯，方便以后调用execute
    public LightOnCommand(Light light){
        this.light = light;
    }
//    调用接受对象的on方法
    public void execute() {
        light.on();
    }

    public void undo() {
        light.off();
    }
}
```


在关闭灯的命令中，执行on方法
```
public class LightOffCommand implements Command {
    Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.off();
    }

    public void undo() {
        light.on();
    }
}
```

```
public class NoCommand implements Command {
    public void execute() {

    }

    public void undo() {

    }
}
```

对遥控器进行一些修改，每次按下按钮的时候，都记录下来按动的哪个，并且可以随时调用undo方法。

```
public class RemoteControl {
    Command[] onCommands;
    Command[] offCommands;

    Command undoCommand;
    /**
     * 初始化这两个开关的数组
     */
    public RemoteControl(){
        onCommands = new Command[7];
        offCommands = new Command[7];

        Command noCommand = new NoCommand();
        for (int i = 0 ;i < 7;i++){
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand;
    }

    /**
     * 有三个命令，分别是插槽位置，开的命令，关的命令
     * @param slot
     * @param onCommand
     * @param offCommand
     */
    public void setCommand(int slot ,Command onCommand,Command offCommand){
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }
    public void onButtonWasPushed(int slot){
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }
    public void offButtonWasPushed(int slot){
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }

    public void undoButtonWasPushhed(){
        undoCommand.undo();
    }


    @Override
    public String toString() {
        StringBuffer stringBuffer = new StringBuffer();
        stringBuffer.append("\n-------Remote Control ------\n");
        for (int i = 0 ;i < onCommands.length;i++){
            stringBuffer.append("[slot"+i+"]"+onCommands[i].getClass().getName()+"        "
            +offCommands[i].getClass().getName()+"\n");
        }
        return stringBuffer.toString();
    }
}
```

> 书中还加入了，使用状态实现撤销，并实现了电扇撤销的功能，虽然比电灯要复杂一些，但总体思路相同。



## 6.命令模式的更多用途



### 6.1 队列请求

命令可以将运算块打包(一个接收者和一组动作), 然后将它传来传去,就像是一般的对象一样。现在即使在命令对象被创建许久之后,运算依然可以被调用。事实上,它甚至可以在不同的线程中被调用。我们可以利用这样的特性行生一些应用,例如:日程安排( Scheduler)、线程池、工作队列等。
想象有一个工作队列:你在某一端添加命令,然后另端则是线程。线程进行下面的动作:从队列中取出一个命令,调用它的 execute()方法,等待这个调用完成, 然后将此命令对象丢弃,再取出下一个命令

> 请注意,工作队列类和进行计算的对象之间完全是解耦的。此刻线程可能在进行财务运算,下一刻却在读取网络数据。工作队列对象不在平到底做些什么,它们只知道取出命令对象,然后调用其execute方法。类似地,它们只要是实现命令模式的对象,就可以放入队列里,当线程可用时,就调用此对象的 execute方法。

### 6.2 日志请求



某些应用需要我们将所有的动作都记录在日志中，并能在系统死机之后，重新调用这些动作恢复到之前的状态。通过新增两个方法（store（）、load），命令模式就能够支持这一点。在Java中，我们可以利用对象的序列化（Serialization）实现这些方法，但是一般认为序列化最好还是只用在对象的持久化上（persistence）要怎么做呢？当我们执行命令的时候，将历史记录储存在磁盘中。一旦系统死机，我们就可以将命令对象重新加载，并成批地依次调用这些对象的 execute（方法。这种日志的方式对于遥控器来说没有意义然而，有许多调用大型数据结构的动作的应用无法在每次改变发生时被快速地存储。通过使用记录日志，我们可以将上次检查点（checkpoint）之后的所有操作记录下来，如果系统出状况，从检查点开始应用这些操作。比方说，对于电子表格应用，我们可能想要实现的错误恢复方式是将电子表格的操作记录在日志中，而不是每次电子表格一有变化就记录整个电子表格。对更高级的应用而言，这些技巧可以被扩展应用到事务（ transaction）处理中，也就是说，一整群操作必须全部进行完成，或者没有进行任何的操作。