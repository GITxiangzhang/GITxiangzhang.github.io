---
title: Javascript之BOM与DOM讲解
---

#### Javascript组成
Javascript的实现包含：
1. 核心（ECMAScript）:描述js语法和基本对象；
2. dom 文档对象模型：处理网页内容的方法和接口；
3. bom 浏览器对象模型：与浏览器交互的方法和接口；

----------------------------------
>ECMAScript扩展知识：

  ① ECMAScript是一个标准，JS只是它的一个实现，其他实现包括ActionScript。

  ② “ECMAScript可以为不同种类的宿主环境提供核心的脚本编程能力……”，即ECMAScript不与具体的宿主环境相绑定，如JS的宿主环境是浏览器，AS的宿主环境是Flash。

  ③ ECMAScript描述了以下内容：语法、类型、语句、关键字、保留字、运算符、对象。

我们都知道， javascript 有三部分构成，ECMAScript，DOM和BOM，根据宿主（浏览器）的不同，具体的表现形式也不尽相同，ie和其他的浏览器风格迥异。

    1. DOM 是 W3C的标准；[所有浏览器公共遵守的标准]
    2. BOM 是 各个浏览器厂商根据 DOM
    在各自浏览器上的实现;[表现为不同浏览器定义有差别,实现方式不同]
    DOM（文档对象模型）是 HTML 和 XML 的应用程序接口（API）。

    BOM 主要处理浏览器窗口和框架，不过通常浏览器特定的 JavaScript 扩展都被看做 BOM 的一部分。这些扩展包括：

    弹出新的浏览器窗口

    移动、关闭浏览器窗口以及调整窗口大小

    提供 Web 浏览器详细信息的定位对象

    提供用户屏幕分辨率详细信息的屏幕对象

    对 cookie 的支持
    IE 扩展了 BOM，加入了ActiveXObject类，可以通过JavaScript实例化ActiveX对象

    javacsript是通过访问BOM（Browser Object Model）对象来访问、控制、修改客户端(浏览器)，由于BOM的window包含了document，
    window对象的属性和方法是直接可以使用而且被感知的，因此可以直接使用window对象的document属性，通过document属性就可以访问、检索、
    修改XHTML文档内容与结构。因为document对象又是DOM（Document Object Model）模型的根节点。可以说，BOM包含了DOM(对象)，浏览器提供出
    来给予访问的是BOM对象，从BOM对象再访问到DOM对象，从而js可以操作浏览器以及浏览器读取到的文档。

    Window对象包含属性：document、location、navigator、screen、history、frames

    Document根节点包含子节点：forms、location、anchors、images、links

    从window.document已然可以看出，DOM的最根本的对象是BOM的window对象的子对象。

    区别：DOM描述了处理网页内容的方法和接口，BOM描述了与浏览器进行交互的方法和接口

----------------------------------
>DOM, DOCUMENT, BOM, WINDOW 区别

1. DOM 全称是 Document Object Model，也就是文档对象模型。是针对XML的基于树的API。描述了处理网页内容的方法和接口，是HTML和XML的API，
DOM把整个页面规划成由节点层级构成的文档。

针对XHTML和HTML的DOM。这个DOM定义了一个HTMLDocument和HTMLElement做为这种实现的基础,就是说为了能以编程的方法操作这个 HTML 的内
容（比如添加某些元素、修改元素的内容、删除某些元素），我们把这个 HTML 看做一个对象树（DOM树），它本身和里面的所有东西比如 <div></div>
这些标签都看做一个对象，每个对象都叫做一个节点（node），节点可以理解为 DOM 中所有 Object 的父类。

DOM 有什么用？就是为了操作 HTML 中的元素，比如说我们要通过 JS 把这个网页的标题改了，直接这样就可以了：
document.title = 'how to make love';这个 API 使得在网页被下载到浏览器之后改变网页的内容成为可能。

 ![Image]({{ site.baseurl }}/images/dom.png)

2. document 当浏览器下载到一个网页，通常是 HTML，这个 HTML 就叫 document（当然，这也是 DOM 树中的一个 node），从上图可以看到，
document 通常是整个 DOM 树的根节点。这个 document 包含了标题（document.title）、URL（document.URL）等属性，可以直接在 JS 中访问到。

在一个浏览器窗口中可能有多个 document，例如，通过 iframe 加载的页面，每一个都是一个 document。
在 JS 中，可以通过 document 访问其子节点（其实任何节点都可以），如
document.body;document.getElementById('xxx');

3. BOM 是 Browser Object Model，浏览器对象模型。
刚才说过 DOM 是为了操作文档出现的接口，那 BOM 顾名思义其实就是为了控制浏览器的行为而出现的接口。
浏览器可以做什么呢？比如跳转到另一个页面、前进、后退等等，程序还可能需要获取屏幕的大小之类的参数。
所以 BOM 就是为了解决这些事情出现的接口。比如我们要让浏览器跳转到另一个页面，只需要
location.href = "http://www.xxxx.com";
这个 location 就是 BOM 里的一个对象。

4. window也是 BOM 的一个对象，除去编程意义上的“兜底对象”之外，通过这个对象可以获取窗口位置、确定窗口大小、弹出对话框等等。例如我要关闭当前窗口：
window.close();



BOM和DOM的结构关系示意图:

 ![Image]({{ site.baseurl }}/images/bomdom.png)

----------------------------------
#### DOM
DOM通过创建树来表示文档，描述了处理网页内容的方法和接口，从而使开发者对文档的内容和结构具有空前的控制力，用DOM API可以轻松地删除、添加和替换节点。


>节点类型

DOM规定文档中的每个成分都是一个节点（Node）,HTML文档可以说由节点构成的集合，DOM节点有:

1. 元素节点（Element）：<html>、<body>、<p>等都是元素节点，即标签。

2. 文本节点（Text）:向用户展示的内容，如<li>...</li>中的JavaScript、DOM、CSS等文本。

3. 属性节点（Attr）:元素属性，元素才有属性,如<a>标签的链接属性href="http://www.baidu.com"。

>DOM节点三大属性（XML DOM）

1. nodeName：元素节点、属性节点、文本节点分别返回元素的名称、属性的名称和#text的字符串。

2. nodeType：元素节点、属性节点、文本节点的nodeType值分别为1、2、3.、

3. nodeValue：元素节点、属性节点、文本节点的返回值分别为null、属性值和文本节点内容。

>DOM常见操作

Node为所有节点的父接口，其定义了一组公共的属性和方法，如下：

 ![Image]({{ site.baseurl }}/images/domapi.png)

1. 获取节点
① 获取元素节点：通过document对象的三个方法获取

document.getElementById("ID")

document.getElementByName("name")

document.getElementsByTagName("p");

② 获取属性节点：属性节点附属于元素节点，可通过元素节点的getAttributeNode(attrName)方法获取属性节点，也可通过getAttribute(attrName)
直接获取属性值。（与之相对的是Element接口的setAttribute(attrName , attrValue)方法，如果属性不存在，则直接添加到元素节点上）

③ 获取文本节点：文本节点为元素节点的子节点，可通过元素节点（Element接口）提供的childnodes()[index] 方法获得。

childNodes //NodeList，所有子节点的列表

firstChild //Node，指向在childNodes列表中的第一个节点

lastChild //Node，指向在childNodes列表中的最后一个节点

parentNode //Node，指向父节点

previousSibling /Node，/指向前一个兄弟节点：如果这个节点就是第一个节点，那么该值为 null

nextSibling //Node，指向后一个兄弟节点：如果这个节点就是最后一个节点，那么该值为null

hasChildNodes()` //Boolean，当childNodes包含一个或多个节点时，返回真值

2. 改变节点

① 改变属性节点的值：可以通过属性节点的nodeValue直接修改属性值，也可通过元素节点的setAttribute()方法改变。

② 改变文本节点的值：通过文本节点的nodeValue直接修改。

在HTML DOM中，获取和改变元素内容最简单方法是使用元素的innerHTML属性（innerText属性返回去掉标签的innerHTML）

3. 删除节点

① 删除元素节点：要想删除元素节点A，需获得A的父节点B，父节点可通过17.1.1中的方法获得，也可以直接通过A的parentNode属性获得（推荐）。调用B的removeChild(A)即可删除A节点。

② 删除属性节点：可通过属性节点所属的元素节点的removeAttribute(attrName)或removeAttributeNode(node)删除。

③ 清空文本节点：最简单也是最常用的方法就是直接设置文本节点的nameNode属性为空串：textNode.nodeValue =””。

混淆点:

<p id="example" title="texts">

  这是一段文本

  <span></span>

</p>

var p = document.getElementById('example');

p.nodeValue //null,p是元素节点，所以nodeValue为null



p.getAttributeNode('id').nodeValue  //example，这里获取到p的id属性的属性节点，nodeValue就是它的属性值



p.childNodes[0].nodeValue   //这是一段文本,

p是含有两个子节点的，插入的文本虽然没有标签，但它依然是一个节点。

其类型是是文本节点，其nodeValue是就是写入在其中的字符串，包含换行和缩进



p.innerHTML//这是一段文本 <span></span>"

这里innerHTML返回了p所包含的全部的节点的所包含的各种值了，以字符串的形式。


 [了解更多](http://blog.csdn.net/qq877507054/article/details/51395830)
