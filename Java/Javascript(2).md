# 一、BOM

## 1.概念：

将浏览器的各个组成部分封装成对象。

## 2.组成

- Window：窗口对象
- Navigator：浏览器对象
- Screen：显示器屏幕对象
-  History：历史记录对象
-  Location：地址栏对象

## 3.window对象

### 1.概述

Window 对象表示浏览器中打开的窗口。

### 2.创建

Window对象不需要创建实例对象，Window 对象是全局对象。window.方法名();

或者可以省略window

### 3.方法

1. 与弹出框有关的方法
   1. alert()：显示带有一段消息和一个确认按钮的警告框。
   2. confirm()：显示带有一段消息以及确认按钮和取消按钮的对话框。
      - 如果用户点击确定按钮，则方法返回true
      - 如果用户点击取消按钮，则方法返回false

2. 与打开关闭有关的方法：

   1.  close()：关闭浏览器窗口：

      ​	注意：谁调用，就关谁

   2. open()：打开一个新的浏览器窗口
     
      1. 返回一个新的Window对象

3. 与定时器有关的方法：
   1.  setTimeout(js代码或者方法对象,毫秒值)：在指定的毫秒数后调用函数或计算表达式。
      返回一个唯一标识，用于取消定时器
   2. clearTimeout()：取消由 setTimeout() 方法设置的定时器
   3. setInterval()：按照指定的周期（以毫秒计）来调用函数或计算表达式。
   4. clearInterval()：取消由 setInterval() 设置的定时器。

4. 属性：

   1. 获取其他BOM对象：

       history
      location
      Navigator
       Screen

   2. 获取DOM对象：
       document

## 4. Location：地址栏对象

1. 创建(获取)：
   1. location
   2.  window.location

2.  方法：
   1.  reload()	重新加载当前文档。刷新

3. href	设置或返回完整的 URL。

## 5. History：历史记录对象

1. 创建(获取)：
   1. window.history
   2.  history

2.  方法：

   1. back()	加载 history 列表中的前一个 URL。

   2.  forward()	加载 history 列表中的下一个 URL。

   3. go(参数)	加载 history 列表中的某个具体页面。

      ​	参数：正数：前进几个历史记录
      ​                负数：后退几个历史记录

3. 属性：
  
   1. length	返回当前窗口历史列表中的 URL 数量

# 二、DOM

##  1.概念

1. 概述

 Document Object Model 文档对象模型。将标记语言文档的各个组成部分，封装为对象。可以使用这些对象，对标记语言文档进行CRUD的动态操作

2. DOM树

![](F:\笔记\JAVA\js\1.png)

通过这个对象模型，JavaScript 获得创建动态 HTML 的所有力量：

* JavaScript 能改变页面中的所有 HTML 元素
* JavaScript 能改变页面中的所有 HTML 属性
* JavaScript 能改变页面中的所有 CSS 样式
* JavaScript 能删除已有的 HTML 元素和属性
* JavaScript 能添加新的 HTML 元素和属性
* JavaScript 能对页面中所有已有的 HTML 事件作出反应
* JavaScript 能在页面中创建新的 HTML 事件

3. W3C DOM 标准被分为 3 个不同的部分：

   * Core DOM - 所有文档类型的标准模型

     - Document：文档对象

     * Element：元素对象
     * Attribute：属性对象
     * Text：文本对象
     * Comment:注释对象
     *  Node：节点对象，**其他5个的父对象**

   * XML DOM - XML 文档的标准模型

   * HTML DOM - HTML 文档的标准模型

## 2.Core DOM 

### 1.Document:文档对象

1.  创建(获取)：在html dom模型中可以使用window对象来获取
   1. window.document
   2. document

2. 方法：

   1.  获取Element对象：
      1. getElementById()	： 根据id属性值获取元素对象。id属性值一般唯一
      2. getElementsByTagName()：根据元素名称获取元素对象们。返回值是一个数组
      3. getElementsByClassName():根据Class属性值获取元素对象们。返回值是一个数组
      4. getElementsByName(): 根据name属性值获取元素对象们。返回值是一个数组

   2.  创建其他DOM对象：

      1. createAttribute(name)
      2. createComment()
      3. createElement()
      4. createTextNode()

   3. 其他：

      ​	document.write(text)	写入 HTML 输出流

### 2.Element：元素对象

1. 获取/创建：
   通过document来获取和创建
2.  方法：
   1. removeAttribute()：删除属性
   2.  setAttribute()：设置属性

### 3.Node：节点对象

1. 特点：其他5个的父对象，所有dom对象都可以被认为是一个节点
2. 方法：
   1. appendChild()：向节点的子节点列表的结尾添加新的子节点；
   2. removeChild()：删除（并返回）当前节点的指定子节点；
   3. replaceChild()：用新节点替换一个子节点。

3. 属性：
   1. parentNode 返回节点的父节点。

## 3.HTML DOM

1. 标签体的设置和获取：innerHTML       

# 三、事件

## 1.概念

- 某些组件被执行了某些操作后，触发某些代码的执行。	
- 事件：某些操作。如： 单击，双击，键盘按下了，鼠标移动了
- 事件源：组件。如： 按钮 文本输入框...
- 监听器：代码。
- 注册监听：将事件，事件源，监听器结合在一起。 当事件源上发生了某个事件，则触发执行某个监听器代码。

## 2.常见的事件：

1. 点击事件：
		1. onclick：单击事件
	2. ondblclick：双击事件

2. 焦点事件
   1. onblur：失去焦点
   2. onfocus:元素获得焦点。
3. 加载事件：
   1. onload：一张页面或一幅图像完成加载。
4. 鼠标事件：
   1. onmousedown	鼠标按钮被按下。
   2. onmouseup	鼠标按键被松开。
   3. onmousemove	鼠标被移动。
   4. onmouseover	鼠标移到某元素之上。
   5. onmouseout	鼠标从某元素移开。

5. 键盘事件：
		1. onkeydown	某个键盘按键被按下。	
	2. onkeyup		某个键盘按键被松开。
	3. onkeypress	某个键盘按键被按下并松开。

6. 选择和改变
   1. onchange	域的内容被改变。
   2. onselect	文本被选中。
7. 表单事件：
   1. onsubmit	确认按钮被点击。
   2. onreset	重置按钮被点击。