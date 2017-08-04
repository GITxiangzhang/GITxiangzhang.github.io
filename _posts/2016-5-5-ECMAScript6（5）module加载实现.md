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


----------------------------------
#### ES6 模块与CommonJS 模块的差异
>CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
>CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，
再根据这个只读引用，到被加载的那个模块里面去取值。


----------------------------------
#### node 加载
>node有自己的ConmmonJS的模块格式，与es6的module不兼容。

一个模块脚本只要有一行import或export语句，Node 就会认为该脚本为 ES6 模块，否则就为 CommonJS 模块。

ES6 模块之中，顶层的this指向undefined；CommonJS 模块的顶层this指向当前模块，这是两者的一个重大差异。


----------------------------------
#### import 加载 CommonJS模块
>import 加载 CommonJS模块，node会把被加载模块的module.exports属性，当做模块的默认输出，等同于export default。（export default 只能有一个啊，所以默认输出最后一个？）

下面是一个CommonJS模块
{% highlight javascript %}
//export-default.js
var PI=3.1415;
function circle(r){
    return PI*r*r;
}
module.exports=circle;
module.exports=PI;
{% endhighlight %}

import 命令加载上面模块，module.exports视为默认输出（export default）。

{% highlight javascript %}
//方法一
import  baz from './export-default';
console.log(baz);//3.1415
//方法二
import * as baz from './export-default';
console.log(baz.default);//baz本身是一个对象，不能当作函数调用或者，只能通过baz.default调用
{% endhighlight %}






