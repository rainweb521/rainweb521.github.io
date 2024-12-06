《深入理解PHP》的笔记

1. 数组的自定义排序已经记录到笔记里。

2. 检查表单是否已经提交

   ```php
   if(($_SERVER['REQUEST_METHOD']=='POST')&&!empty($_POST['task'])){}
   ```

3. 确保表单的值是一个整数，这里使用了过滤扩展来确保这里的值是比1打的整数

   ```php
   if(isset($_POST['parent_id'])&&filter_var($_POST['parent_id'],FILTER_VALIDATE_INT,array('min_range'=>1))){
       $parent_id = $_POST['parent_id'];
   }else{
       $parent_id = 0;
   }
   ```

4. 函数参数的类型约束

   类型约束用于指定函数中的参数是什么类型的变量，例如`function f(array $input)`就指定了参数必须是数组类型，如果被调用时没有提供一个数组参数，将会发生一个错误

5. 引用

   PHP中也支持使用引用，即将变量本身作为参数传递给函数，而不是将变量的值传递给函数，使用时加一个&既可以

6. 关于原型文档的使用方法，由于没有用过，先不做笔记

7. 使用printf和springf

   与c语言中的printf，springf，scanf，sscanf用法一致`printf('%d $0.2f %c',80,80,80)`