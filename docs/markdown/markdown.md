# Markdown教程

## Markdown基本语法

## 总览
几乎所有的Markdown应用程序都支持John Gruber原始设计文档中概述的基本语法。Markdown处理器之间存在细微的差异，尽可能的注明即可。

::: warning 注意
使用Markdown并不意味着您也不能使用HTML。您可以将`HTML`标签添加到任何Markdown文件中。如果您更喜欢某些`HTML`标记而不是`Markdown`语法，这将很有帮助。例如有人发现将HTML标签用于图像更容易。  
:::

## 标题
要创建标题，请#在单词或短语的前面添加数字符号（）。您使用的数字符号的数量应与标题级别相对应。例如，要创建标题级别三（`<h3>`），请使用三个数字符号（例如### My Header）。

| Markdown 	|   HTML	|  
|:--------	|:--------:	|
| # Heading level 1 |  `<h1>Heading level 1</h1>`	|
| ## Heading level 2  |  `<h2>Heading level 2</h2>` |
| ### Heading level 3  |  `<h3>Heading level 3</h3>` |
| #### Heading level 4 | `<h4>Heading level 4</h4>` |
| ##### Heading level 5 | `<h5>Heading level 5</h5>` |
| ###### Heading level 6| `<h6>Heading level 6</h6>` |



### 代替语法
或者，在文本的下方，添加上任意数量的==标题级别--字符或标题级别2的字符。  
| Markdown 	|   HTML	|  
|:--------	|:--------:	|
| Heading level 1 <br /> =============== |  `<h1>Heading level 1</h1>`	|
| ## Heading level 2 <br /> ---------------------- |  `<h2>Heading level 2</h2>` |

## 段落
要创建段落，使用空白行分隔一行或多行文本。您不应缩进带有空格或制表符的段落。

| Markdown 	|   HTML|  
|:--------	|------	|  
| I really like using Markdown. | `<p>I really like using Markdown.</p>`|  
| I think I'll use it to format <br/> all of my documents from now on. |  `<p>I think I'll use it to format `<br/> `all of my documents from now on.</p>` 	|  

## 换行
| Markdown 	|  HTML |  
|:--------:	| ------|  
|  This is the first line. <br/>And this is the second line.|   `<p>This is the first line.<br/> And this is the second line.</p>`	|  

## 着重
可以通过使文本变为粗体或斜体来增加着重。

### 粗体
要加粗文本，请在单词或短句的前后添加两个星号或下划线。要加粗一个单词的中部以强调，请在字母周围添加两个星号，且各空格之间不加空格。

| Markdown 	|  HTML | 效果 |
|:--------	|---	|:-------:|
| `I just love **bold text**. ` | `I just love <strong>bold text</strong>.`	| I just love **bold text**.|
| `I just love __bold text__.`  | `I just love <strong>bold text</strong>.` | I just love __bold text__. |
| `Love **is** bold `  |   `Love<strong>is</strong>bold`	| Love<strong> is </strong>bold |

### 斜体
| Markdown 	|  HTML |  效果 |
|:--------	|------	| ------|
| `Italicized text is the *cat's meow*. ` | `Italicized text is the <em>cat's meow</em>.` |  Italicized text is the <em>cat's meow</em>. |
| `Italicized text is the _cat's meow_.` | `Italicized text is the <em>cat's meow</em>.`|Italicized text is the <em>cat's meow</em>. |
|  `A*cat*meow` |  `A<em>cat</em>meow` 	| A*cat*meow	|

### 粗体和斜体
要同时突出显示带有粗体和斜体的文本，请在单词或短语的前后添加三个星号或下划线。  
| Markdown 	| HTML|  效果 |
|:--------	|------	|-------|
| `This text is ***really important***.`|  `This text is <strong><em>really important</em></strong>.` 	|  这段文字 ***真的很重要***。 	|
| `This text is ___really important___.`| `This text is <strong><em>really important</em></strong>.`  	|  这段文字 ___真的很重要___。 	|
|  `This text is __*really important*__.` | `This text is <strong><em>really important</em></strong>.`  	|  这段文字 __*真的很重要*__。 	|
|`This text is **_really important_**.`|`This text is <strong><em>really important</em></strong>.`| 这段文字 **_真的很重要_**。|


## 块引用
要创建块引用，请在>段落前面添加一个。
```markdown
> 这是一个快引用
```
效果如下
> 这是一个快引用

### 具有多个段落的块引用
块引用可以包含多个段落。>在段落之间的空白行上添加。
``` markdown
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.    
```
**呈现的效果如下所示：**
> 桃乐丝（Dorothy）跟着她走过了她城堡中许多美丽的房间。
>
> 女巫请她清洗锅碗瓢盆，扫地，并用木柴取火。

### 嵌套块引用
块引用可以嵌套。>>在要嵌套的段落前面添加一个。
```markdown
> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```  
**呈现的效果如下所示：**
> 桃乐丝（Dorothy）跟着她走过了她城堡中许多美丽的房间。
>
>> 女巫请她清洗锅碗瓢盆，扫地，并用木柴取火。

### 具有其他元素的块引用
块引用可以包含其他markdown格式的元素。并非所有元素都可以使用，您需要进行实验以查看那些元素有效。
``` markdown 
> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.
```  
**呈现的效果如下所示：**
>  <h3>The quarterly results look great!</h3>
>
> - Revenue was off the chart.
> - Profits were higher than ever.
    >
    >  *Everything* is going according to **plan**.

## 清单
您可以将项目组织成有序和无序列表。
### 有序列表
要创建有序列表，请在订单项中添加数字和句点。数字不必按数字顺序排列，但列表应以数字开头。

**Markdown 格式**
``` markdown
1. First item
2. Second item
3. Third item
4. Fourth item

1. First item
1. Second item
1. Third item
1. Fourth item

1. First item
8. Second item
3. Third item
5. Fourth item

1. First item
2. Second item
3. Third item
    1. Indented item
    2. Indented item
4. Fourth item
```
**HTML格式**

```HTML
<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ol>

<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item
        <ol>
            <li>Indented item</li>
            <li>Indented item</li>
        </ol>
    </li>
    <li>Fourth item</li>
</ol>
```

**渲染输出：**
> 1. 第一项
> 2. 第二项
> 3. 第三项
> 4. 第四项


> 1. 第一项
> 2. 第二项
> 3. 第三项
     >       1. 缩进项
>       2. 缩进项
> 4. 第四项

### 无序列表
要创建无序列表，请在订单项前添加破折号（-），星号（*）或加号（+）。缩进一个或多个项目以创建嵌套列表。  
**Markdown格式：**
``` markdown
- First item
- Second item
- Third item
- Fourth item

* First item
* Second item
* Third item
* Fourth item

+ First item
* Second item
- Third item
+ Fourth item

- First item
- Second item
- Third item
    - Indented item
    - Indented item
- Fourth item
```
**HTML格式：**
``` markdown
<ul>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ul>

<ul>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item
        <ul>
            <li>Indented item</li>
            <li>Indented item</li>
        </ul>
    </li>
    <li>Fourth item</li>
</ul>
```
**显示效果：**
> - 第一项
> - 第二项
> - 第三项
> - 第四项

> - First item
> - Second item
> - Third item
    >    - Indented item
>    - Indented item
>- Fourth item

### 在列表中添加元素
#### 段落
``` markdown
*   This is the first list item.
*   Here's the second list item.

    I need to add another paragraph below the second list item.

*   And here's the third list item.
```
呈现的效果如下：
> * 这是第一个列表项。
> * 这是第二个列表项。
>
> 我需要在第二个列表项下面添加另一段。
> * 这是第三个列表项。

#### 块引用
```markdown
*   This is the first list item.
*   Here's the second list item.

    > A blockquote would look great below the second list item.

*   And here's the third list item.
```

**呈现的效果如下所示：**
> * 这是第一个列表项。
> * 这是第二个列表项。
>
> >在第二个列表项的下方，块引用看起来不错。
>
> * 这是第三个列表项。

#### 代码块
```markdown
1.  Open the file.
2.  Find the following code block on line 21:

        <html>
          <head>
            <title>Test</title>
          </head>

3.  Update the title to match the name of your website.
```

**呈现的效果如下：**

> 1. 打开文件。
> 2. 在第21行找到以下代码块：
>```
>    <html>
>        <head>
>        <title>Test</title>
>        </head>
>    </html>
> ```
> 3. 更新标题以匹配您的网站名称。

#### 图片
``` markdown
1.  Open the file containing the Linux mascot.
2.  Marvel at its beauty.

    ![Tux, the Linux mascot](/actor.png)

3.  Close the file.
```
> 1.  Open the file containing the Linux mascot.
> 2.  Marvel at its beauty.
      >
      >    ![Tux, the Linux mascot](/actor.png)
>
> 3.  Close the file.

## 代码
要将单词或短语表示为代码，请将其括在勾号（`）中。

```markdown
At the command prompt, type `nano`.  
```

```html
At the command prompt, type <code>nano</code>.
```
**呈现效果如下所示：**
> At the command prompt, type `nano`.

### 转义刻度线
如果要表示为代码的单词或短语包含一个或多个刻度线，可以通过将单词或短语括在双刻度线（``）中来对具体进行转义。
``` markdown
``Use `code` in your Markdown file.``
```

``` HTML
<code>Use `code` in your Markdown file.</code>
```

**呈现效果如下所示：**
> ``Use `code` in your Markdown file.``

### 代码块
要创建代码块，请在代码块的每一行缩进至少四个空格或一个制表符。
``` html
    <html>
        <head>
        </head>
    </html>
```

**呈现的效果如下所示：**

    <html>
        <head>
        </head>
    </html>

## 水平线
要创建水平线***，请单独在一行上使用三个或更多的星号（），破折号（---）或下划线（___）。
>
> ``***``
>
> ``---``
>
> ``_________``

**这三种格式输出的效果类似：**
***

## 链接
要创建链接，请将链接文本括在方括号（例如[你好啊]）中，然后立刻在URL后面加上括号（例如(https://fredomli.com)）中的URL。
```
My blog address is [blog address](https://fredoli.com)
```
**呈现效果如下：**
> My blog address is [Fredomli Blog Address](https://fredoli.com)

### 添加标题
您可以选择为链接添加标题。当用户将鼠标悬停在链接时，这将显示为工具提示。要添加标题，请将其包括在URL后的括号中。
``` markdown
My personal site is [personal site go](https://fredomli.com "personal site address")
```  
**呈现效果如下所示：**
> My personal site is [personal site go](https://fredomli.com "personal site address")

### 网址和电子邮件地址
要将URL或电子邮件地址快速转换为链接，将其包括在尖括号中。

``` markdown
<https://fredomli.com>
<fredomli@163.com>
```
呈现效果如下所示：
> <https://fredomli.com>  
> <fredomli@163.com>

### 格式化链接
为了强调链接，请在方括号和括号之间和之后添加星号。
``` markdown
learn in **[Fredomli Go](https://fredomli.com)**
```

**呈现效果如下所示：**
> learn in **[Fredomli Go](https://fredomli.com)**

### 参考样式链接
引用样式链接是一种特殊的链接，它使`URL`在`Markdown`中更易于显示和阅读。引用样式的链接分为两部分：与文本保持内联的部分以及在文件中其他位置储存的部分，以使文本易于阅读。

#### 格式化链接的第一部分
参考样式链接的第一部分使用两组括号进行格式化。第一组方括号包围应显示为链接的文本。第二组括号显示了一个标签，该标签用于指向您存储在文档其他位置的链接。

尽管不是必须的，但您可以在第一组和第二组支架之间包含一个空格。第二组括号中的标签不区分大小写。可以包含字母，数字，空格或标点符号。如下所示：
``` markdown
[title][tag]
[title] [tag]
```
#### 格式化链接第二部分
引用样式链接的第二部分使用以下属性设置格式：
1. 标签放在方括号中，后紧跟冒号和至少一个空格`（例如[label]: ）`。
2. 链接的URL，您可以选择将其括在尖括号中。
3. 链接的可选标题，您可以将其括在双引号，单引号或中括号中。

这意味着以下示例对于链接的第二部分都是等效的:
``` markdown
[1]: https://fredomli.com  
[2]: https://fredomli.gitee.io/fredomli-personal-blog/  
```

您可以将链接的第二部分放在Markdown文档中的任何位置。有些人将它们放在出现的段落之后，而后其他人则将它们放在文档的末尾（例如尾注或脚注）。

#### 示例
假设您添加一个URL作为到段落的标准URL链接，在Markdown中看起来像这样：

``` markdown
[markdown 段落](http://localhost:8080/fredomli-personal-blog/guide/markdown.html#%E6%AE%B5%E8%90%BD "Markdown 段落")
```
``` markdown
[markdown 段落][test]

[test]: <http://localhost:8080/fredomli-personal-blog/guide/markdown.html#%E6%AE%B5%E8%90%BD> ("Markdown 段落")
```
**呈现的效果如下所示：**
> [markdown 段落](http://localhost:8080/fredomli-personal-blog/guide/markdown.html#%E6%AE%B5%E8%90%BD "Markdown 段落")

链接HTML为：

``` HTML
<a href="http://localhost:8080/fredomli-personal-blog/guide/markdown.html#%E6%AE%B5%E8%90%BD" title="Markdown 段落">Markdown 段落</a>
```

## 图片
要添加图像，请添加感叹号`（!）`，然后在括号中添加替代文本，本在括号中添加图像资源的路径或URL。您可以选择在括号中的URL之后添加标题。
``` markdown
![image title](/path/xxx.png "iamge tag")
```
**呈现效果如下所示：**
> ![image title](/java-standard/pics/actor.png "iamge tag")

### 链接图像
要向图像添加链接，请将图像的markdown括在方括号中，然后在括号中添加链接。
``` markdown
[![picture name](/background/bg.jpg) "link image"](https://fredomli.com)
```

**呈现效果如下所示：**  
[![picture name](https://fredomli.gitee.io/fredomli-picture/static/images/logo/actor.png)](https://fredomli.com)

## 转义字符
要显示原义字符，否则将用于设置Markdown文档中的文本格式\，请在字符前面添加（\）。
``` markdown
\* Without the backslash, this would be a bullet in an unordered list.  
```
**呈现的效果如下：**
> \* Without the backslash, this would be a bullet in an unordered list.

可以转义的字符：  
|字符|名称|
|-----|-----|
|\\|反斜杠|
|`|刻度线|
|*|星号|
|_|下划线|
|{}|大括号|
|[]|中括号|
|()|小括号|
|#|井号|
|+|加号|
|-|减号（连字符）|
|.|点|
|!|感叹号|
|\||管道|
