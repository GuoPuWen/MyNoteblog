  <h1>CSS样式学习</h1>

  <h1>JAVA</h1>

  <h1>HTML</h1>前言：2021年1月31号正式开始前端的学习，此前，是做JAVA后端的，前端这块基本使用是没有问题的，也用过一些高级框架如Vue等等，但是总觉得前端知识用起来有点捉襟见肘，所以打算正式的系统的学习一下前端知识，就先从HTML开始

# 一、页面结构

一个HTML页面的基本结构如下，在vscode上使用快捷键! + tab能直接打出下面结构

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="keyword" content="HTML">
    <meta name="description" content="HTML学习">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>标题</title>
</head>
<body>
    
</body>
</html>
```

| 标签        | 说明                                            |
| ----------- | ----------------------------------------------- |
| DOCTYPE     | 声明为HTML文档                                  |
| html        | lang:网页的语言，如en/zh等，非必选项目          |
| head        | 文档说明部分，对搜索引擎提供信息或加载CSS、JS等 |
| title       | 网页标题                                        |
| keyword     | 向搜索引擎说明你的网页的关键词                  |
| description | 向搜索引擎描述网页内容的摘要信息                |
| viewport    | 设置响应式布局                                  |
| body        | 页面主体内容                                    |

# 二、语义化标签

`h1-h6`设置标题，这个不过多演示

`header`标签用于设置文档的页眉

`footer`设置文档的页脚，页脚通常包含文档的作者、版权信息、使用条款链接、联系信息等等

`nav`设置导航链接

`main`html5中main标签标示页面主要区域，一个元素中main出现一次较好

`section`定义一个区块

`aside`用于设置与主要区域无关的内容，比如侧面栏的广告等

`div`通用容器，如有有些块不好通过语义表达，可以使用通用容器

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <header>
        <nav>
            <ul>
                <li><a href="">JAVA</a></li>
                <li><a href="">大数据</a></li>

            </ul>
        </nav>
    </header>
    <main>
        <article>
            <h2>Java后端学习路线</h2>
            <ul>
                <li><a href="">Java基础</a></li>
                <li><a href="">web基础</a></li>
            </ul>
            <section><h2>锁机制</h2></section>
            <section><h2>Sync锁</h2></section>
        </article>
    </main>
    <aside>
        <h2>个人简介</h2>
        <p>JAVA爱好者</p>
    </aside>
    <footer>
        <p>JAVA学习路线</p>
    </footer>
</body>
</html>
```

# 三、文本标签

`p`标记一个段落

`pre`原样显示文本内容，包括空白换行等等

`br`换行

`span`标签与 `div` 标签都是没有语义的，`span` 常用于对某些文本特殊控制，但该文本又没有适合的语义标签。

`small`用于设置描述、声明等文本。

`time` 标签定义日期或时间，或者两者。

`abbr`用于描述一个缩写内容

```html
<p>在WWW上，每一信息资源都有统一的且在网上唯一的地址，该地址就叫 <abbr title="Uniform Resource Locator">URL</abbr> 统一资源定位符。</p>
```

效果：

![image-20210131200701221](http://cdn.noteblogs.cn/image-20210131200701221.png)

`sub`用于数字的下标内容

`sup`用于数字的上标内容

` del`用于删除内容

` code`用于显示代码块内容，一般需要代码格式化插件完成。

` progress`用于表示完成任务的进度，当游览器不支持时显示内容。

```html
<progress value="60" max="100">完成60%</progress>
```

![image-20210131201127584](前端第1天---HTML.assets/image-20210131201127584.png)

`strong`强调文本

`mark`用于突出显示文本内容，类似我们生活中使用的马克笔。

`cite`标签通常表示它所包含的文本对某个参考文献的引用，或文章作者的名子。

` blockquote`用来定义摘自另一个源的块引用

`q`用于表示行内引用文本，在大部分浏览器中会加上引号。

`address`用于设置联系地址等信息，一般将`address` 放在`footer` 标签中。

# 四、链接与图片

`常见三种图片格式说明`

| 格式     | 说明                                                         | 透明 |
| -------- | ------------------------------------------------------------ | ---- |
| PNG      | 无损压缩格式，适合图标、验证码等。有些小图标建议使用css字体构建。 | 支持 |
| GIF      | 256色，可以产生动画效果（即GIF动图）                         | 支持 |
| JPEG/JPG | 有损压缩的类型，如商品、文章的图片展示                       |      |

选取图片一般原则：

- 图片属性静态文件，不要放在WEB服务器上，而放在云储存服务器上并使用CDN加速
- JPEG类型优先使用，文件尺寸更小
- 网页图标建议使用css字体构建
- 小图片使用PNG，清晰度更高，因为文件尺寸小，文件也不会太大

`img`标签展示图片，图片的大小、边框、倒角效果使用css处理

| 属性 | 说明                     |
| ---- | ------------------------ |
| src  | 图片地址                 |
| alt  | 图像打开异常时的替代文本 |

`a`网页链接

| 选项   | 说明                                 |
| ------ | ------------------------------------ |
| href   | 跳转地址                             |
| target | _blank 新窗口打开 _self 当前窗口打开 |
| title  | 链接提示文本                         |

`锚点链接`锚点可以设置跳转到页面中的某个部分，也可以指定跳转到url的锚点

```html
<body>
    <a href="#commind-1">跳转到评论区</a>
    <div style="height: 1000px;background-color: blue;"></div>

    <div id="commind-1" style="background-color: red;height: 1000px;">评论区</div>
</body>
```

`邮箱链接`除了页面跳转外可以指定其他链接

```html
<a href="mailto:1537990340@qq.com">给我发邮件</a>
```

`拨打电话`如果是移动端，则会弹出拨号面板

```html
<a href="tel:1111111111">联系客服</a>
```

`下载文件`如果遇到链接，浏览器无法处理的情况则会直接弹出下载框

# 五、表单与列表

`from` 用于包裹整个表单项

```html
<form action="/adduser" method="POST">
    <fieldset>
        <legend>测试</legend>
        <input type="text">
    </fieldset>
</form>
```

![image-20210131203722744](http://cdn.noteblogs.cn/image-20210131203722744.png)

`label`描述表单标题，当点击标题后文本框会获得焦点，需要保证使用的ID在页面中是唯一的。

`input`文本框用于输入单行文本使用，下面是常用属性与示例

| 属性        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| type        | 表单类型默认为 `text`                                        |
| name        | 后台接收字段名                                               |
| required    | 必须输入                                                     |
| placeholder | 提示文本内容                                                 |
| value       | 默认值                                                       |
| maxlength   | 允许最大输入字符数                                           |
| size        | 表单显示长度，一般用不使用而用 `css` 控制                    |
| disabled    | 禁止使用，不可以提交到后台                                   |
| readonly    | 只读，可提交到后台                                           |
| capture     | 使用麦克风、视频或摄像头哪种方式获取手机上传文件，支持的值有 microphone, video, camera |

`input`中的type字段可以指定不同的输入内容

| 类型     | 说明                         |
| -------- | ---------------------------- |
| email    | 输入内容为邮箱               |
| url      | 输入内容为URL地址            |
| password | 输入内容为密码项             |
| tel      | 电话号，移动端会调出数字键盘 |
| search   | 搜索框                       |
| hidden   | 隐藏表单                     |
| submit   | 提交表单                     |

```html
<input type="email" name="email" required>
```

`表单提交`

创建提交按钮可以将表单数据提交到后台，有多种方式可以提交数据如使用AJAX，或HTML的表单按钮。使用HTML的表单提交有两种方式，使用input输入框或者使用button按钮，使用input如果设置了name，那么提交数据也会发送到后台，通过这个可以判断是哪个表单提交的，使用button也可以提交

```html
<input type="submit" name="submit" value="提交">
```

使用input提交会将当前的按钮的值也一并提交

![image-20210131204948360](http://cdn.noteblogs.cn/image-20210131204948360.png)

```html
button type="submit" >提交表单</button>
```

`select` 下拉列表可用于多个值

| 选项     | 说明       |
| -------- | ---------- |
| multiple | 支持多选   |
| size     | 列表框高度 |
| optgroup | 选项组     |
| selected | 选中状态   |
| option   | 选项值     |

```html
<form action="">
    <select name="lesson[]" >
        <option value="">== 选择课程 ==</option>
        <optgroup label="后台">
            <option value="php">PHP</option>
            <option value="linux">LINUX</option>
            <option value="mysql">MYSQL</option>
        </optgroup>
        <optgroup label="前台">
            <option value="php">HTML</option>
            <option value="linux">JAVASCRIPT</option>
            <option value="mysql">CSS</option>
        </optgroup>
    </select>
    <input type="submit" value="提交">
</form>
```

`radio` 单选框指只能选择一个选项的表单，如性别的选择`男、女、保密` 只能选择一个

| 选项    | 说明     |
| ------- | -------- |
| checked | 选中状态 |

```html
<form action="">
    <input type="radio" name="sex" value="boy" id="boy" checked>
    <label for="boy">男</label>
    <input type="radio" name="sex" value="girl" id="girl">
    <label for="girl">女</label>
</form>
```

`checkbox`复选框指允许选择多个值的表单。

```html
<form action="">
    <fieldset>
        <legend>复选框</legend>
        <input type="checkbox" name="sex" value="boy" id="boy">
        <label for="boy">PHP</label>

        <input type="checkbox" name="sex" value="girl" id="girl" checked>
        <label for="girl">MYSQL</label>
    </fieldset>
</form>
```

`文件上传`文件上传有多种方式，可以使用插件或JS拖放上传处理。HTML本身也提供了默认上传的功能，只是上传效果并不是很美观

| 选项     | 说明                                              |
| -------- | ------------------------------------------------- |
| multiple | 支持多选                                          |
| accept   | 允许上传类型 `.png,.psd` 或 `image/png,image/gif` |

`日期与时间`

| 属性 | 说明                                                |
| ---- | --------------------------------------------------- |
| step | 间隔：date 缺省是1天，week缺省是1周，month缺省是1月 |
| min  | 最小时间                                            |
| max  | 最大时间                                            |

```html
<input type="date" step="5" min="2020-09-22" max="2025-01-15" name="datetime">
```

`datalist`input表单的输入值选项列表

```html
<form action="" method="post">
    <input type="text" list="lesson">
    <datalist id="lesson">
        <option value="PHP">后台管理语言</option>
        <option value="CSS">美化网站页面</option>
        <option value="MYSQL">掌握数据库使用</option>
    </datalist>
</form>
```

![image-20210131210420021](http://cdn.noteblogs.cn/image-20210131210420021.png)

`无序列表`

```html
<style>
    .li-style1{
        /* 
        circle      空心圆
        disc        实心圆
        square      实心方块
        decimal     数字
        upper-alpha 大写字母
        lower-alpha 小写字母
        upper-roman 大写罗马数字
        lower-roman 小写罗马数字
        */
        list-style-type: square;
    }

    .li-style2{
        /* 取消风格 */
        list-style: none;
    }
    .li-style3{
        /*inside 内部 outside 外部（默认）*/
        list-style-position: inside;
    }
</style>

<ol start="1" class="li-style1">
	<li>HTML</li>
	<li>CSS</li>
	<li>JS</li>
</ol>
```

`有序列表ul`

# 六、表格

`基本使用`

| 属性    | 说明         |
| ------- | ------------ |
| caption | 表格标题     |
| thead   | 表头部分     |
| tbody   | 表格主体部分 |
| tfoot   | 表格尾部     |

```html
<table border="1">
    <caption>表格标题</caption>
    <thead>
        <tr>
            <th>标题</th>
            <th>时间</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>JAVA</td>
            <td>2021-2-22</td>
        </tr>
    </tbody>
</table>
```

效果

![image-20210131211315962](http://cdn.noteblogs.cn/image-20210131211315962.png)

`水平单元格合并`

```html
<table border="1">
    <caption>表格标题</caption>
    <thead>
        <tr>
            <th>标题</th>
            <th>时间</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td colspan="2">JAVA 2021-2-22</td>
        </tr>
    </tbody>
</table>
```

![image-20210131211426012](http://cdn.noteblogs.cn/image-20210131211426012.png)

`垂直单元格合并`

```html
<table border="1">
    <caption>表格标题</caption>
    <thead>
        <tr>
            <th>标题</th>
            <th>时间</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td rowspan="2"> JAVA</td>
            <td>2021-01-31</td>
        </tr>
        <tr>
            <td>2021-01-31</td>
        </tr>
    </tbody>
</table>
```

![image-20210131211646197](http://cdn.noteblogs.cn/image-20210131211646197.png)

