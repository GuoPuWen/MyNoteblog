# PHP语法学习

- php变量必须以$符号开始，是弱语言，不必声明类型
- global关键字用于函数内访问全局变量，PHP 将所有全局变量存储在一个名为 $GLOBALS[*index*] 的数组中。 *index* 保存变量的名称。这个数组可以在函数内部访问，也可以直接用来更新全局变量
- static可以让某个局部变量不被删除
- 声明一个类的语法

```php
<?php
class Car
{
  var $color;
  function __construct($color="green") {
    $this->color = $color;
  }
  function what_color() {
    return $this->color;
  }
}
?>
```

- `==`比较只比较值，`===`比较类型和值
- 使用`define`定义一个常量，具有全局属性
- 并置运算符`.`用于连接两个字符串，不是用+号，`strlen()`计算长度，`strpos()`在字符串中查找一个字符或者文本
- `array()`用于创建数组，关联数组类似于map。同时可以创建多维数组，数组中嵌套数组，可以使用一系列sort函数进行排序

```php
<?php
$cars=array("Volvo","BMW","Toyota");
$arrlength=count($cars);
 
for($x=0;$x<$arrlength;$x++)
{
    echo $cars[$x];
    echo "<br>";
}
?>
  
<?php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
echo "Peter is " . $age['Peter'] . " years old.";
?>

<?php
$age=array("Peter"=>"35","Ben"=>"37","Joe"=>"43");
 
foreach($age as $x=>$x_value)
{
    echo "Key=" . $x . ", Value=" . $x_value;
    echo "<br>";
}
?>
```

- php中定义了一些超级全局变量，可以直接引用，例如$_SERVER保存一些web信息，$GLOBALS，$_REQUEST等等

- 魔术常量，例如`__LINE__`显示行数，`__FILE__`显示文件的完整路径和文件名等等

- 命名空间

  

