第2天学习了CSS的选择器方面的知识，CSS的选择器很多，在后面一些样式控制中，其实就是运用的是CSS的选择器，这块的知识要经常使用经常复习，今天学习的是CSS中的文本控制与非常重要的盒子模型

## 一、文本控制

### 1.1 文本基础

`font-family设置字体`

在CSS中我们一般设置多个字体，因为在不同的用户下有不用的浏览器，可能该用户浏览器上没有该字体，所以一般设置多个用来备用

也可以使用`@font-face`将字体格式的url引入，用来引入外部字体，下面介绍一些常用的字体格式

```html
@font-face {
    font-family: 'Courier New', Courier, monospace;;
    src: url("SourceHanSansSC-Light.otf") format("opentype"),	/*指定字体格式*/
}
```



| 字体  | 格式              |
| ----- | ----------------- |
| .otf  | opentype          |
| .woff | woff              |
| .ttf  | truetype          |
| .eot  | Embedded-opentype |

`font-weight字重`

字重指字的粗细定义。取值范围 `normal | bold | bolder | lighter | 100 ~900`。400对应 `normal`，700对应 `bold` ，一般情况下使用 bold 或 normal 较多。

`font-size文本字号`

字号用于控制字符的显示大小，包括 `xx-small | x-small | small | meidum | large | x-large | xx-large`，可以使用百分数是子元素相对于父元素的大小，如父元素是20px，子元素设置为 200%即为你元素的两倍大小。

也可以使用em作为单位，em单位等同于百分数单位

`color文本颜色`

可以使用字符串设置、16进制设置、rgb设置、rgba设置透明色

`line-hright行高`

`font-style字体倾斜风格`

`组合设置`

```html
span {
	/* 加粗 斜体 20号 1.5倍行高 字体*/
	font: bold italic 20px/1.5 'Courier New', Courier, monospace;
}
```

* 必须有字体规则
* 必须有字符大小规则

### 1.2 文本样式

`大小转换`

| CSS样式        | 实例                        | 作用         |
| -------------- | --------------------------- | ------------ |
| font-variant   | font-variant: small-caps;   | 小号大写字母 |
| text-transform | text-transform: capitalize; | 首字母小写   |

`text-decoration文本线条`

`text-shadow阴影控制`

```html
span {
	text-shadow: rgba(13, 6, 89, 0.8) 3px 3px 5px;
}
```

`white-space空白显示`

| 选项     | 说明                                    |
| -------- | --------------------------------------- |
| pre      | 保留文本中的所有空白，类似使用 pre 标签 |
| nowrap   | 禁止文本换行                            |
| pre-wrap | 保留空白，保留换行符                    |
| pre-line | 空白合并，保留换行符                    |

### 1.3 文本溢出处理

在一个div里面放置文本，有时会超出div的大小，那么这时就可以用到文本溢出

`单行文本溢出`

单行文本溢出后，可以使用换行来进行处理

```html
div{
    border: solid 1px blue;
    width: 100px;
    overflow-wrap: break-word;
}
```

`溢出内容后面添加...`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        div{
            border: solid 1px blue;
            width:300px;
            white-space: nowrap;            /*禁止换行*/
            overflow: hidden;
            text-overflow: ellipsis;
        }
    </style>
</head>
<body>
    <div>
        HTML称为超文本标记语言，是一种标记语言。它包括一系列标签．通过这些标签可以将网络上的文档格式统一，使分散的Internet资源连接为一个逻辑整体。HTML文本是由HTML命令组成的描述性文本，HTML命令可以说明文字，图形、动画、声音、表格、链接等。 [1] 
超文本是一种组织信息的方式，它通过超级链接方法将文本中的文字、图表与其他信息媒体相关联。这些相互关联的信息媒体可能在同一文本中，也可能是其他文件，或是地理位置相距遥远的某台计算机上的文件。这种组织信息方式将分布在不同位置的信息资源用随机方式进行连接，为人们查找，检索信息提供方便。
    </div>
</body>
</html>
```

![image-20210202163050541](http://cdn.noteblogs.cn/image-20210202163050541.png)

`控制多行文本溢出时添加 `...

```html
<style>
    div{
        border: solid 1px blue;
        width:300px;
        overflow: hidden;
        display:-webkit-box;
        -moz-box-orient: vertical;
        -webkit-line-clamp: 2;
    }
</style>
```



![image-20210202163447332](http://cdn.noteblogs.cn/image-20210202163447332.png)

`控制表格文本溢出`

表格文本溢出控制需要为 table 标签定义 `table-layout: fixed;` css样式，表示列宽是由表格和单元格宽度定义。

```css
<style>
  table {
    table-layout: fixed;
  }

  table tbody tr td {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
</style>
```

`溢出后产生滚动条`

```html
<style>
    div{
        width: 100px;
        height: 300px;
        border: solid red 1px;
        overflow:scroll;
    }
</style>
```





### 1.4 段落控制

`text-indent文本缩进`

`text-align文本对齐` 使用 `left|right|center` 对文本进行对齐操作

`vertical-align`用于定义内容的垂直对齐风格，包括`middle | baseline | sub | super` 等。

## 二、盒子模型

打开浏览器的调试工具，就可以看到下面这张盒子模型图

![image-20210202190050057](http://cdn.noteblogs.cn/image-20210202190050057.png)

基本使用实例：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        a {
            display: inline-block;
            border: solid 2px #dddddd;
            text-align: center;
            padding: 10px 20px;
            margin-top: 30px;
        }
    </style>
</head>
<body>
    <a href="">MYSQL</a>
    <a href="">LINUX</a>
    <a href="">PHP</a>
</body>
</html>
```

边距的定义是上、右、下、左，设置为auto能够随页面动态调整

### 2.1 边距合并

相邻元素的纵向外边距会进行合并，例如

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        main article li{
            border: solid 1px red;
            margin-top: 20px;
            margin-bottom: 20px;
        }
        main article div{
            height: 40px;
            background-color: red;
            margin-left: 40px;
        }
    </style>
</head>
<body>
    <main>
        <article>
            <ul>
                <li>html</li>
                <li>css</li>
                <li>java</li>
            </ul>
            <div>

            </div>
        </article>
    </main>
</body>
</html>
```

![image-20210202191734127](http://cdn.noteblogs.cn/image-20210202191734127.png)

很明显，li标签设置了上边距和下边距，按道理相邻两个li之间应该是40px，但是因为边距合并所以才20px

### 2.2 BOX-SIZING

设置了BOX-SIZING之后，宽度与高度包括内边距与边框。前面的设置宽度和高度都不包括内边距和边框，这可以通过调试查看例如

```html
<style>
    div {
        border: solid 10px #ddd;
        height: 60px;
        width: 200px;
        padding: 50px;
        box-sizing: border-box;
    }
</style>
```

![image-20210202192403177](http://cdn.noteblogs.cn/image-20210202192403177.png)

### 2.3 边框设计

对边框设计都可以只涉及一个方向的边框，例如：

| 规则                | 说明 |
| ------------------- | ---- |
| border-top-style    | 顶边 |
| border-right-style  | 右边 |
| border-bottom-style | 下边 |
| border-left-style   | 左边 |
| border-style        | 四边 |

`border-style边框样式`

| 类型   | 描述                                                  |
| :----- | :---------------------------------------------------- |
| none   | 定义无边框。                                          |
| dotted | 定义点状边框。在大多数浏览器中呈现为实线。            |
| dashed | 定义虚线。在大多数浏览器中呈现为实线。                |
| solid  | 定义实线。                                            |
| double | 定义双线。双线的宽度等于 border-width 的值。          |
| groove | 定义 3D 凹槽边框。其效果取决于 border-color 的值。    |
| ridge  | 定义 3D 垄状边框。其效果取决于 border-color 的值。    |
| inset  | 定义 3D inset 边框。其效果取决于 border-color 的值。  |
| outset | 定义 3D outset 边框。其效果取决于 border-color 的值。 |

样式顺序为上、右、左、下

`border-width边框高度`

`border-color边框颜色`

`border-radius圆角边框`使用 `border-radius` 规则设置圆角，可以使用`px | %` 等单位。也支持四个边分别设置。

| 选项                       | 说明 |
| -------------------------- | ---- |
| border-top-left-radius     | 上左 |
| border-top-right-radius    | 上右 |
| border-bottom-left-radius  | 下左 |
| border-bottom-right-radius | 下右 |

通过边框绘制园

```html
div{
    width: 100px;
    height: 100px;
    border: solid 3px red;
    border-radius: 50%;
}
```

### 2.4 轮廓线

元素在获取焦点时产生，并且轮廓线不占用空间。可以使用伪类 `:focus` 定义样式。

* 轮廓线显示在边框外面
* 轮廓线不影响页面布局

表单项是默认有轮廓线的，可以设置取消

`outline-style轮廓线样式`

| 值     | 描述                                                |
| :----- | :-------------------------------------------------- |
| none   | 默认。定义无轮廓。                                  |
| dotted | 定义点状的轮廓。                                    |
| dashed | 定义虚线轮廓。                                      |
| solid  | 定义实线轮廓。                                      |
| double | 定义双线轮廓。双线的宽度等同于 outline-width 的值。 |
| groove | 定义 3D 凹槽轮廓。此效果取决于 outline-color 值。   |
| ridge  | 定义 3D 凸槽轮廓。此效果取决于 outline-color 值。   |
| inset  | 定义 3D 凹边轮廓。此效果取决于 outline-color 值。   |
| outset | 定义 3D 凸边轮廓。此效果取决于 outline-color 值。   |

`outline-width宽度`

`outline-color线条颜色`

同样可以组合定义

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        input:focus{
            outline:double 10px red;
        }
    </style>
</head>
<body>
    <input type="text">
</body>
</html>
```

### 2.5 display与visibility

display控制元素的显示隐藏

| 选项         | 说明                        |
| ------------ | --------------------------- |
| none         | 隐藏元素                    |
| block        | 显示为块元素                |
| inline       | 显示为行元素，不能设置宽/高 |
| inline-block | 行级块元素，允许设置宽/高f  |

visibility也是控制元素的隐藏，在隐藏后空间也保留

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
            background-color: red;
            visibility: hidden;
        }
        article div:nth-child(2){
            height: 200px;
            width: 200px;
            background-color: green;
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

### 2.6 尺寸定义

可以使用多种方式为元素设置宽、高尺寸。

| 选项           | 说明             |
| -------------- | ---------------- |
| width          | 宽度             |
| height         | 高度             |
| min-width      | 最小宽度         |
| min-height     | 最小高度         |
| max-width      | 最大宽度         |
| max-height     | 最大高度         |
| fill-available | 撑满可用的空间   |
| fit-content    | 根据内容适应尺寸 |