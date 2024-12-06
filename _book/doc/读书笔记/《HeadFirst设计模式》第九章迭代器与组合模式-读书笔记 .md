# 《HeadFirst设计模式》第九章迭代器与组合模式-读书笔记 

### 案例代码链接：https://github.com/rainweb521/My-tutorial/tree/master/Design_patterns

## 1. 菜单的迭代

#### 本例中餐厅和煎饼屋合并了，他们需要在客人点餐的时候，能够把二者的菜单内容都打印出来，但是他们之前分别使用ArrayList和数组来存储菜单内容，这样打印的时候就会有重复，多次调用的问题。

这是基础菜单

```
public class MenuItem {
    String name;
    String description;
    boolean vegetarian;
    double price;
    public MenuItem(String name, String description, boolean vegetarian, double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public boolean getVegetarian() {
        return vegetarian;
    }

    public void setVegetarian(boolean vegetarian) {
        this.vegetarian = vegetarian;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }
}
```



#### 这是煎饼屋的菜单

```
public class PancakeHouseMenu {
    ArrayList menuItems;
    public PancakeHouseMenu() {
        menuItems = new ArrayList();
        addItem("K&B Pancake Breakfast","Pancakes with scrambled eggs , and toast",
                true,2.99);
    }
    public void addItem(String name, String description, boolean vegetarian, double price){
        MenuItem menuItem = new MenuItem(name,description,vegetarian,price);
        menuItems.add(menuItem);
    }
    public ArrayList getMenuItems(){
        return menuItems;
    }
}
```

#### 这是餐厅的菜单

```
public class DinerMenu {
    static final int MAX_ITEMS = 6;
    int numberOfItems = 0 ;
    MenuItem[] menuItems;

    public DinerMenu() {
        menuItems = new MenuItem[MAX_ITEMS];
        addItem("Vegetarian BLT",
                "(Fakin') BAcon with lettuce & tomato on whole wheat",true,2.99);
    }

    private void addItem(String name, String description, boolean vegetarian, double price) {
        MenuItem menuItem = new MenuItem(name,description,vegetarian,price);
        if (numberOfItems >= MAX_ITEMS){
            System.out.println("Sorry,meny is full");
        }else {
            menuItems[numberOfItems]  = menuItem;
            numberOfItems += 1;
        }
    }

    public MenuItem[] getMenuItems() {
        return menuItems;
    }
}
```



> 如果想要打印两份菜单，就需要实例化这两个类，然后通过getMenuItems方法获取菜单组，再分别循环打印出来。
>
> 若以后再增加菜单，就需要继续这样更改代码，实现新的循环。

## 2.封装遍历

我们新建一个对象，将它称为迭代器，利用它来封装“遍历集合内的每个对象的过程”。

创建统一的迭代器接口

```
public interface Iterator {
    boolean hasNext();
    Object next();
}
```

接下来实现具体的迭代器，首先是餐厅的

```
public class DinnerMenuIterator implements Iterator {
    MenuItem[] menuItems;
    int position = 0;

    public DinnerMenuIterator(MenuItem[] menuItems) {
        this.menuItems = menuItems;
    }
    public boolean hasNext() {
        if (position>=menuItems.length || menuItems[position]==null){
            return false;
        }else{
            return true;
        }
    }
    public Object next() {
        MenuItem menuItem = menuItems[position];
        position+=1;
        return menuItem;
    }
}
```

在DinerMenu中加入迭代器方法

```
 public Iterator createIterator(){
        return new DinnerMenuIterator(menuItems);
    }
```

实现煎饼屋的迭代器对象

```
public class PancakeHouseIterator implements Iterator{
    ArrayList<MenuItem> menuItems;
    int position = 0;

    public PancakeHouseIterator(ArrayList<MenuItem> menuItems) {
        this.menuItems = menuItems;
    }
    public boolean hasNext() {
        if (position>=menuItems.size() || menuItems.get(position)==null){
            return false;
        }else{
            return true;
        }
    }
    public MenuItem next() {
        MenuItem menuItem = menuItems.get(position);
        position+=1;
        return menuItem;
    }
}
```

在PancakeHouseMenu中加入迭代器方法

```
public Iterator createIterator(){
        return new PancakeHouseIterator(menuItems);
    }
```

修正女招待的代码

我们需要将迭代器代码整合进女招待中。我们应该摆脱原本冗余的部分。整合的做法相当直接:首先创建一个 printmenuo方法,传入一个迭代器当做此方法的参数,然后对每一个菜单都使用 createlterator()方法来检索迭代器,并将迭代器传入新方法

```
public class Waitress {
    PancakeHouseMenu pancakeHouseMenu;
    DinerMenu dinerMenu;
    public Waitress(PancakeHouseMenu pancakeHouseMenu, DinerMenu dinerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinerMenu = dinerMenu;
    }
    public void printMenu(){
        Iterator pancakeHouseMenuIterator = pancakeHouseMenu.createIterator();
        Iterator dinerMenuIterator = dinerMenu.createIterator();
        printMenu(pancakeHouseMenuIterator);
        printMenu(dinerMenuIterator);
    }
    private void printMenu(Iterator iterator) {
        while (iterator.hasNext()){
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName());
            System.out.println(menuItem.getDescription());
            System.out.println(menuItem.getVegetarian());
        }
    }
}
```



> 到目前为止,我们做了些什么? 
>
> 首先,我们让厨师们非常快乐。他们可以保持他们自己的实现又可以摆平差别。只要我们给他们这两个迭代器( Pancake Housemenulteratori和Dinermenuiterator),他们只需要加入一个 create Iterator()方法,一切就大功告成了。
> 这个过程中,我们也帮了我们自己。女招待将会更容易维护和扩展。

#### 类的变化

![](http://cos.rain1024.com/markdown/image-20191030195935389.png)



#### 进一步完善

因为ArrayList本身是支持Iterator的，所以直接调用自身的iterator方法即可

```
public Iterator createIterator(){
        return (Iterator) menuItems.iterator();
    }
```

我们只需要给菜单一个共同的接口,然后再稍微改一下女招待。这个Menu接口相当简单:可能退早需要在里面多加入一些方法,例如 ladditem(),但是目前,我们还是让厨师控制他们的菜单,不要把那些方法放在公开接口中

```
public interface Menu {
    public Iterator createIterator();
}
```

现在,我们需要让煎饼屋菜单类和餐厂菜单类都实现Menu接口,然后更新女招待的代码如下:

```
public class Waitress {
    Menu pancakeHouseMenu;
    Menu dinerMenu;
    public Waitress(Menu pancakeHouseMenu, Menu dinerMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinerMenu = dinerMenu;
    }
```

这样我们就实现了针对接口编程，而不是针对实现编程。

#### 完整的类图就出现了，整个项目的结构如下

![](http://cos.rain1024.com/markdown/image-20191030200809061.png)

## 3.迭代器模式和单一职责

> 迭代器模式：提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。



这很有意义:这个模式给你提供了一种方法,可以顺序访问一个聚集对象中的元素,而又不用知道内部是如何表示的。你已经在前面的两个菜单实现中看到了这一点。在设计中使用迭代器的影响是明显的:如果你有一个统一的方法访问聚合中的每一个对象,你就可以编写多态的代码和这些聚合搭配,使用如同前面的 printmenu方法一样,只要有了送代器这个方法根本不管菜单项究竟是由数组还是由 Arraylist(或者其他能创建迭代器的东西)来保存的。
另一个对你的设计造成重要影响的,**是迭代器模式把在元素之间游走的责任交给迭代器,而不是聚合对象。**
这不仅让聚合的接口和实现变得更简洁,也可以让聚合更专注在它所应该专注的事情上面(也就是管理对象集合),而不必去理会遍历的事情

> 迭代器模式让我们能游走于聚合内的每一个元素,而又不暴露其内部的表示。
> 把游走的任务放在迭代器上,而不是聚合上。
> 这样简化了聚合的接和突现,也让责任各得其所。

#### 单一职责

> 设计原则：一个类应该只有一个引起变化的原因。

类的每个责任都有改变的潜在区城。超过个责任,意味着超过个改变的区域。
这个原则告诉我们量让每个类保持单一责任。

## 4.新加入的咖啡店

新的需求出现了，咖啡店也加入了这份菜单中，要将它们继续整合在一个框架中。

先看咖啡厅的代码，已经将之前的接口加了进去

```
public class CafeMenu implements Menu {
    Hashtable menuItems = new Hashtable();

    public CafeMenu() {
        addItem("Veggie Burger and Air Fries",
                "Veggie burger on a whole wheat bun , lettuce,tomato , and fries",
                true,3.99);
    }
    private void addItem(String name, String description, boolean vegetarian, double price) {
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
        menuItems.put(menuItem.getName(),menuItem);
    }
    public Iterator createIterator() {
        return (Iterator) menuItems.values().iterator();
    }
}
```

修改女招待的代码

```
public class Waitress {
    Menu pancakeHouseMenu;
    Menu dinerMenu;
    Menu cafeMenu;
    public Waitress(Menu pancakeHouseMenu, Menu dinerMenu,Menu cafeMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinerMenu = dinerMenu;
        this.cafeMenu = cafeMenu;
    }
    public void printMenu(){
        Iterator pancakeHouseMenuIterator = pancakeHouseMenu.createIterator();
        Iterator dinerMenuIterator = dinerMenu.createIterator();
        Iterator cafeMenuIterator = cafeMenu.createIterator();
        printMenu(pancakeHouseMenuIterator);
        printMenu(dinerMenuIterator);
        printMenu(cafeMenuIterator);
    }
    private void printMenu(Iterator iterator) {
        while (iterator.hasNext()){
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName());
            System.out.println(menuItem.getDescription());
            System.out.println(menuItem.getVegetarian());
        }
    }
}
```

这样就加入了新的咖啡厅菜单

#### 但是，这样程序中还是调用了三次printMenu方法，而且每次有新的菜单加入，都必须打开女招待加入更多的代码，好像违反了开放-关闭原则。

这次将这些菜单全都打包进一个 Arraylist,然后取得它的迭代器,遍历每一个菜单。这样再增加菜单也不怕了。

```
public class Waitress {
    ArrayList menus;
    public Waitress(ArrayList menus) {
        this.menus = menus;
    }
    public void printMenu(){
        Iterator iterator = (Iterator) menus.iterator();
        while (iterator.hasNext()){
            Menu menu = (Menu) iterator.next();
            printMenu(menu.createIterator());
        }
        
    }
    private void printMenu(Iterator iterator) {
        while (iterator.hasNext()){
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName());
            System.out.println(menuItem.getDescription());
            System.out.println(menuItem.getVegetarian());
        }
    }
}
```

#### 最后就全部完成了，彻底解耦了。

## 5.添加一份子菜单

我们需要在餐厅菜单中添加一份甜点子菜单，这就需要支持菜单中的菜单了。

如下图，正是我们想要的样子。

![](http://cos.rain1024.com/markdown/image-20191031075616469.png)

但我们不能把甜点菜单赋值给菜单项数组。

这时候就需要重构了

事实是，我们已经到达了一个复杂级别，如果现在不重新设计，就无法容纳未来增加的菜单或子菜单等需求。
所以，在我们的新设计中，真正需要些什么呢?

* 我们需要某种树形结构，可以容纳菜单、子菜
  单和菜单项。
* 我们需要确定能够在每个菜单的各个项之间游:
  走，而且至少要像现在用迭代器一-样方便。
* 我们也需要能够更有弹性地在菜单项之间游走。
  比方说，可能只需要遍历甜点菜单，或者可以
  遍历餐厅的整个菜单( 包括甜点菜单在内)。

## 6. 组合模式

我们要介绍另一个模式解决这个难题。我们并没有放弃迭代器一一它仍然是我们解決方案中的一部分，然而,管理菜单的问题已经到了一个迭代器无法解决的新维度。所以,我们将倒退几步,改用组合模式( Composite Pattern)来实现这一部分。

> 组合模式：允许你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。

组合模式让我们能用树形方式创建对象的结,树里面包含了组合以及个别的对象。

使组合结构,我们能把相同的操作应用在组合和个别对象上。换句话说,在大多数情況下, 我们可以忽略对象组合和个别对象之间的差别。

让我们以菜单为例思考这一切:这个模式能够创建个树形结构,在同一个结构中处理嵌套菜单和菜单项组。通过将菜单和项放在相同的结构中,我们创建了个“整体部分”层次结构,即由菜单和菜单项组成的对象树。但是可以将它视为一个整体,像是一个丰富的大菜单。
且有了丰富的大菜单,我们就可以使用这个模式来“统一处理个别对象和组合对象”。这意味着什么? 它意味着,如果我们有了一个树形结构的菜单、子菜单和可能还带有菜单项的子菜单,那么任何一个菜单都是种“组合”。因为它既可以包含其他菜单,也可以包含菜单项。个别对象只是菜单项一一并未持有其他对象。
就像你将看到的,使用一个遵照组合模式的设计,让我们能够写出简单的代码,就能够对整个菜单结构应用相同的操作(例如打印!)。

## 7.使用组合模式设计菜单

我们要如何在菜单上应用组合模式呢?一开始,我们需要创建一个组件接口来作为菜单和菜单项的共同接口,让我们能够用统一的做法来处理菜单和菜单项。换句话说,我们可以针对单或菜单项调用相同的方法。

#### 实现菜单组件

好了,我们要开始编写菜单组件的抽象类;请记住,菜单组件的角色是为叶节点和组合节点提供一个共同的接口。现在你可能想问:“那么菜单组件不就扮演了两个角色吗?”可能是这样的,我们稍后再讨论这一点。然而,目前我们要为这些方法提供默认的实现,这样,如果菜单项(叶节点)或者菜单(组合)不想实现某些方法的时候(例如叶节点不想实现getchild方法),就可以不实现这些方法。

> 所有的组件都必须实现Menu Component接口; 然而,叶节点和组合节点的角色不同,所以有些方法可能并不适合某种节点。面对这种情况,有时候,你最好是抛出运行时异常。

```
public abstract class MenuComponent {
    public void add(MenuComponent menuComponent){
        throw new UnsupportedOperationException();
    }
    public void remove(MenuComponent menuComponent){
        throw new UnsupportedOperationException();
    }
    public MenuComponent getChild(int i){
        throw new UnsupportedOperationException();
    }
    public String getName(){
        throw new UnsupportedOperationException();
    }
    public String getDescription(){
        throw new UnsupportedOperationException();
    }
    public double getPrice(){
        throw new UnsupportedOperationException();
    }
    public boolean isVegetarian(){
        throw new UnsupportedOperationException();
    }
    public void print(){
        throw new UnsupportedOperationException();
    }

}
```

#### 实现菜单项

```
public class MenuItem extends MenuComponent {
    String name;
    String description;
    boolean vegetarian;
    double price;

    public MenuItem(String name, String description, boolean vegetarian, double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public String getDescription() {
        return description;
    }

    @Override
    public boolean isVegetarian() {
        return vegetarian;
    }

    @Override
    public double getPrice() {
        return price;
    }
    public void print(){
        System.out.println(""+getName());
        System.out.println(getPrice());
        System.out.println(getDescription());
    }
}

```

#### 实现组合菜单

```
public class Menu extends MenuComponent {
    ArrayList menuComponents = new ArrayList();
    String name;
    String description;

    public Menu(String name, String description) {
        this.name = name;
        this.description = description;
    }

    @Override
    public void add(MenuComponent menuComponent) {
        menuComponents.add(menuComponent);
    }

    @Override
    public void remove(MenuComponent menuComponent) {
        menuComponents.remove(menuComponent);
    }

    @Override
    public MenuComponent getChild(int i) {
        return (MenuComponent) menuComponents.get(i);
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public String getDescription() {
        return description;
    }

    @Override
    public void print() {
//        这里使用递归的方式，将所有的子菜单也打印出来
        System.out.println(getName());
        System.out.println(getDescription());
        Iterator iterator = menuComponents.iterator();
        while (iterator.hasNext()){
            MenuComponent menuComponent = (MenuComponent) iterator.next();
            menuComponent.print();
        }
    }
}
```

#### 最后我们更新女招待的代码，让她来打印菜单

```
public class Waitress {
    MenuComponent allMenus;

    public Waitress(MenuComponent allMenus) {
        this.allMenus = allMenus;
    }
    public void printMent(){
        allMenus.print();
    }
}
```

女招待的代码变得很简单，她只要调用最顶层的组件，就可以打印整个菜单层次，太棒了。

#### 编写测试程序

```
public class MenuTestDrive {
    public static void main(String[] args) {
        MenuComponent pancakeHouseMenu = new Menu("PANCAKE HOUSE MENU","Breakfast");
        MenuComponent dinerMenu = new Menu("DINER MENU","Lunch");
        MenuComponent cafeMenu = new Menu("CAFE MENU","Dinner");
        MenuComponent dessertMenu = new Menu("DESSERT MENU","Dessert of course!");
        MenuComponent allMenus = new Menu("ALL MENUS","ALL menus combined");
        allMenus.add(pancakeHouseMenu);
        allMenus.add(dessertMenu);
        allMenus.add(dinerMenu);
        dinerMenu.add(new MenuItem(
                "Pasta",
                "Spaghetti with Marinara Sauce ,and a slice of sourdough bread",
                true,
                3.89
        ));
        dinerMenu.add(dessertMenu);
        dessertMenu.add(new MenuItem(
                "Apple Pie",
                "Apple pie with a flakey crust, topped with vanilla ice cream",
                true,
                1.59
        ));
        Waitress waitress = new Waitress(allMenus);
        waitress.printMent();
    }
}
```