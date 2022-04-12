# 一、概述

1. 概念： Cascading Style Sheets 层叠样式表
   - 层叠：多个样式可以作用在同一个html的元素上，同时生效

2. 好处：
   1. 功能强大
   2. 将内容展示和样式控制分离
      1. 降低耦合度。解耦
      2. 让分工协作更容易
      3. 提高开发效率

# 二、CSS与html的结合方式

## 1.内联样式（很少用）

- 在标签内使用style属性指定css代码

## 2.内部样式

- 在head标签内，定义style标签，style标签的标签体内容就是css代码

## 3.外部样式

1.  定义css资源文件。
2.  在head标签内，定义link标签，引入外部的资源文件
   - link标签属性：
     - rel：规定当前文档与被链接文档之间的关系。一般都是stylesheet
     - href：CSS文件的路径

# 三、CSS的语法

1. 格式：
   		选择器 {
   			属性名1:属性值1;
   			属性名2:属性值2;
   			...
   		}

2.  选择器：筛选具有相似特征的元素
3. 注意：
   * 每一对属性需要使用；隔开，最后一对属性可以不加；

# 四、CSS的选择器

## 1.基础选择器

1. id选择器：选择具体的id属性值的元素，在一个html页面中id值应该唯一。需要在每个标签中指定id属性

   - /#id{

     属性名1:属性值1;

     属性名2:属性值2;

     ......

     }(/是转义，消除markdown的语法)

2.  元素选择器：选择具有相同标签名称的元素

   - 标签名称{

     属性名1:属性值1;属性名2:属性值2;

     ......

     }

3. 类选择器：选择具有相同的class属性值的元素。需要在每个标签中选择类属性

   - .class+{

     属性名1:属性值1;属性名2:属性值2;

     ......

     }

- 注意：3种选择器的优先级

  1. id选择器的优先级高于类选择器，例如：

     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <title>CSS</title>
        <style>
             .div_c1{
             color: black;
            }
            #div_i1{
             color: blue;
            }
     
         </style>
     </head>
     <body>
         <div id="div_i1" class="div_c1">
             hello html&&CSS
         </div>
     </body>
     </html>
     ```

     结果如下：

     ![](F:\笔记\JAVA\HTML\CSS_1.png)

  2. 类选择器选择器优先级高于元素选择器

## 2.扩展选择器

1.选择所有元素：

   * 语法： *{

     属性名1:属性值1;

     属性名2:属性值2;

     ......

     }

2. 并集选择器：

      * 选择器1,选择器2{

        属性名1:属性值1;

        属性名2:属性值2;

        ......

        }

3. 父代选择器：筛选选择器1元素下的选择器2元素

   语法：  选择器1 选择器2{

   ​	属性名1:属性值1;

   ​    属性名2:属性值2;

   ​    ......

    }

4.  子代选择权：与父代选择器类似

   选择器1 > 选择器2{

   ​	属性名1:属性值1;

   ​    属性名2:属性值2;

   ​    ......

   }

   - 区别：后代指所有后代，而子代单指第一代，而且需要考虑到一些属性可以继承

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>子代选择器与后代选择器</title>
    <style>
        /* 后代选择器 
        .zero li{
            border:1px solid;
        }*/
        /* 子代选择器 */
        .zero > li{
            border:1px solid;
        }
    </style>

</head>
<body>
    <ul class="zero">
        <li>我是祖先</li>
        <ul>
            <li>我是第二代</li>
            <ul>
                <li>我是第三代</li>
            </ul>
        </ul>
    </ul>
</body>
</html>
```

当使用后代选择器时结果

![](F:\笔记\JAVA\HTML\css_2.png)

当使用子代选择器时结果

![](F:\笔记\JAVA\HTML\css_3.png)

当然因为border这个属性不可以继承

5.  属性选择器：选择元素名称，属性名=属性值的元素

   语法：  元素名称[属性名="属性值"]{}

6.  伪类选择器：选择一些元素具有的状态

# 五、属性

1. 字体、文本
		* font-size：字体大小
	* color：文本颜色
	* text-align：对其方式
	* line-height：行高 

2. 背景
   - background：

3. 边框
   -  border：设置边框，符合属性

4. 尺寸
   - width：宽度
   - height：高度

5. 盒子模型：控制布局

   - margin：外边距
   -  padding：内边距
     - 默认情况下内边距会影响整个盒子的大小
     - box-sizing: border-box;  设置盒子的属性，让width和height就是最终盒子的大小

   - float：浮动
     -  left
     - right