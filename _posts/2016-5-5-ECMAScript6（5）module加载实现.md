---
title: ECMAScript6 (5) module加载实现

---

#### 浏览器加载

>html中通过<script>标签加载JavaScript脚本，默认同步，渲染引擎遇到<script>要等到脚本跑完再继续渲染，外部脚本还要等下载完，如果脚本比较大
会造成阻塞，我们所以一般把<script>放在最后。但其实浏览器可以异步加载的。

{% highlight javascript %}
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
{% endhighlight %}

<script>标签打开defer或async属性，脚本就会异步加载，defer是“渲染完再执行”，async是“下载完就执行”。多个defer脚本，会按照它们在页面出现的顺序加载，
而多个async脚本是不能保证加载顺序的。

----------------------------------
##### 加载规则
>浏览器加载 ES6 模块，也使用<script>标签，但是要加入type="module"属性。
{% highlight javascript %}
<script type="module" src="foo.js"></script>
{% endhighlight %}
ES6 模块默认异步加载，等同于<script>标签的defer属性。

ES6 模块也允许内嵌在网页中，其他和ES5一致。




在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。
前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，
成为浏览器和服务器通用的模块解决方案。

ES6 模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。
CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

{% highlight javascript %}
//CommonJS模块
let {stat,exists,readFile}=require('fs')

//等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
{% endhighlight %}

运行时加载：上面的代码实质上是整体加载fs模块（加载了fs所有方法）,生成一个对象_fs，然后再从这个对象上读取3个方法。
只有运行时才能得到这个对象，完全没办法编译的时候就做 静态优化。

ES6 模块不是对象，而是通过export显示指定输出代码，再通过import 命令输入。

{% highlight javascript %}
//ES6模块
import {stat,exists,readFile} from 'fs'
{% endhighlight %}
编译时加载（静态加载）：上面的代码实际上是从fs 模块加载了3个方法，其他的方法不加载。ES6在编译时候就完成了模块加载，效率比CommonJS模块的加载
方式要高。这也导致了没法引用 ES6模块本生，因为他不是对象。

----------------------------------
#### 严格模式
>ES6 的模块自动采用严格模式，不管你有没有在模块头部加上"use strict";。

严格模式主要有以下限制:

* 变量必须声明后再使用
* 函数的参数不能有同名属性，否则报错
* 不能使用with语句
* 不能对只读属性赋值，否则报错
* 不能使用前缀0表示八进制数，否则报错
* 不能删除不可删除的属性，否则报错
* 不能删除变量delete prop，会报错，只能删除属性delete global[prop]
* eval不会在它的外层作用域引入变量
* eval和arguments不能被重新赋值
* arguments不会自动反映函数参数的变化
* 不能使用arguments.callee
* 不能使用arguments.caller
* 禁止this指向全局对象
* 不能使用fn.caller和fn.arguments获取函数调用的堆栈
* 增加了保留字（比如protected、static和interface）

严格模式是 ES5 引入的,具体哪些参见ES5

其中，尤其需要注意this的限制。ES6 模块之中，顶层的this指向undefined，即不应该在顶层代码使用this。

----------------------------------
#### export命令
>模块功能主要由两个命令构成：export和import。export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能。

一个模块就是一个独立的文件，该文件内部所有的变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用export关键字输出该变量。
下面是一个 JS 文件，里面使用export命令输出变量。

{% highlight javascript %}
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
//还可以写成这样
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;
export {firstName, lastName, year};

{% endhighlight %}

