## 一、浮动布局

### 1.1 简单使用

float 属性定义元素在哪个方向浮动。以往这个属性总应用于图像，使文本围绕在图像周围，不过在 CSS 中，任何元素都可以浮动。浮动元素会生成一个块级框，而不论它本身是何种元素。

| 选项  | 说明     |
| ----- | -------- |
| left  | 向左浮动 |
| right | 向右浮动 |
| none  | 不浮动   |

首先理解，普通的文档流对于块级元素是从上到下的，行级元素是从左到右的，例如

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        article div:nth-of-type(1){
            height: 200px;
            width: 200px;
            background-color: green;
            box-shadow: 3px 3px 5px rgba(100, 100, 100, .5);
        }
        article div:nth-of-type(2){
            height: 200px;
            width: 200px;
            background-color: red;
            box-shadow: 3px 3px 5px rgba(100, 100, 100, .5);
        }
    </style>
</head>
<body>
    <article>
        <div></div>
        <div></div>
    </article>
</body>
</html>
```

![image-20210204104119502](http://cdn.noteblogs.cn/image-20210204104119502.png)

如果设置浮动的话，浮动其实是对后面元素的影响，设置浮动后，当前元素朝浮动的方法浮动，后面的元素自然会顶上去，例如设置第一个元素右浮动的话

![image-20210204104354892](http://cdn.noteblogs.cn/image-20210204104354892.png)

### 1.2 丢失空间

如果只给第一个元素设置浮动，第二个元素不设置，后面的元素会占用第一个元素空间。两个元素都设置浮动后，会并排显示

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        article div:nth-of-type(1){
            height: 200px;
            width: 200px;
            background-color: green;
            box-shadow: 3px 3px 5px rgba(100, 100, 100, .5);
            float: left;
        }
        article div:nth-of-type(2){
            height: 200px;
            width: 200px;
            background-color: red;
            box-shadow: 3px 3px 5px rgba(100, 100, 100, .5);
            float: left;
        }
    </style>
</head>
<body>
    <article>
        <div></div>
        <div></div>
    </article>
</body>
</html>
```

![image-20210204104616792](http://cdn.noteblogs.cn/image-20210204104616792.png)

### 1.3 浮动边界

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        main{
            width: 400px;
            border: solid 2px black;
            overflow: auto;
            margin: 50px;
            padding: 50px;
            background-color: blueviolet;
        }
        div{
            width: 100px;
            height: 100px;
           
        }
        div:first-of-type{
            float: left;
            border: red 2px solid;
        }
        div:last-of-type{
            float: left;
            border: green 2px solid;
        }
    </style>
</head>
<body>
    <main>
        <div></div>
        <div></div>
    </main>
</body>
</html>
```

![image-20210204105403739](http://cdn.noteblogs.cn/image-20210204105403739.png)

### 1.4 浮动转块

元素浮动后会变为块元素包括行元素如 `span`，所以浮动后的元素可以设置宽高

### 1.5 清除浮动

CSS提供了 `clear` 规则用于清除元素浮动影响。

| 选项  | 说明               |
| ----- | ------------------ |
| left  | 左边远离浮动元素   |
| right | 右边远离浮动元素   |
| both  | 左右都远离浮动元素 |

 例如：现在的需求是上边一行两个div，下边一个div，使用浮动布局

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        div {
            width: 200px;
            height: 200px;
            margin-bottom: 20px;
        }
        div.green {
            border: solid 2px green;
            float: left;
        }
        div.red {
            border: solid 2px red;
            float: left;
        }
        div.bule{
            border: solid 2px blue;
            background-color: blue;
            clear: both;
        }

    </style>
</head>
<body>
    <main>
        <div class="green"></div>
        <div class="red"></div>
        <div class="bule"></div>
    </main>
</body>
</html>
```

也可以在父元素内部添加一个没有高度的元素，并使用clear:both

### 1.6页面布局实例

