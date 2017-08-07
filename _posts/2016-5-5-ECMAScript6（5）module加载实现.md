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

标签打开defer或async属性，脚本就会异步加载，defer是“渲染完再执行”，async是“下载完就执行”。多个defer脚本，会按照它们在页面出现的顺序加载，
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



----------------------------------
#### require 加载ES6模块
>采用require命令加载 ES6 模块时，ES6 模块的所有输出接口，会成为输入对象的属性。

{% highlight javascript %}
export default function foo() {
    console.log('foo');
};
export var PI=3.1415;
export function circle(r){
    return PI*r*r;
}

var dexport=require('./export-default');
console.log(dexport);
{% endhighlight %}

上面代码中，default接口变成了es_namespace.default属性。另外，由于存在缓存机制，es.js对foo的重新赋值没有在模块外部反映出来。


----------------------------------
#### 循环加载
>a加载b,b加载c,c加载a,两种模块格式CommonJS和ES6，处理“循环加载”的方法是不一样的，返回的结果也不一样。

----------------------------------
##### CommonJS模块的循环加载
>require命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。
即使再次执行require命令，也不会再次执行该模块，而是到缓存之中取值。除非手动清除系统缓存
{% highlight javascript %}
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
{% endhighlight %}
上面代码就是Node内部加载模块后生成的一个对象。该对象的id属性是模块名，exports属性是模块输出的各个接口，loaded属性是一个布尔值，表示该模块的脚本是否执行完毕。

CommonJS 模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。


看个栗子：
{% highlight javascript %}
//a.js

exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');
{% endhighlight %}

a.js 先输出一个done 变量，然后加载另一个脚本文件b.js,此时a.js会等待b.js执行完毕再往下执行。

{% highlight javascript %}
//b.js

exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');
{% endhighlight %}

b.js执行到第二行就回去加载 a.js 这时候发生了循环加载 。系统回去 a.js模块对应对象的exports属性取值，可因为a.js 没执行完，只会输出已经执行
的部分。

此时 a.js 只执行了一行
{% highlight javascript %}
exports.done = false;
{% endhighlight %}

对于b.js 来说 他加载a.js 此时a.js只输入了一个done变量，值为false

然后b.js 接着往下执行 等到执行完后 再把执行权交给a.js, a.js继续执行。

所以最后执行结果为：

{% highlight javascript %}

在 b.js 之中，a.done = false
b.js 执行完毕
在 a.js 之中，b.done = true
a.js 执行完毕
在 main.js 之中, a.done=true, b.done=true

{% endhighlight %}

上面代码说明了两件事。
1. 在b.js中 a.js没有执行完，只跑了一行
2. main.js执行到第二行时，不会再次执行b.js，而是输出了缓存的b.js。

由于CommonJS模块遇到循环加载时，返回的是当前已经执行的部分的值，而不是代码全部执行后的值，两者可能会有差异。

----------------------------------
##### ES6 模块的循环加载
>ES6 处理“循环加载”与CommonJS有本质的不同。ES6模块是动态引用如果使用import从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

再来看个栗子：

{% highlight javascript %}

// a.js如下
import {bar} from './b.js';
console.log('a.js');
console.log(bar);
export let foo = 'foo';

// b.js
import {foo} from './a.js';
console.log('b.js');
console.log(foo);
export let bar = 'bar';

{% endhighlight %}

上面代码中 a.js 第一行是加载b.js 所以执行b.js 。而b.js 第一行又是加载a.js ，a.js已经开始执行，不会重复执行，会继续往下跑b.js
所以第一行输出是b.js

接着，b.js 要打印 foo 这时a.js 还没执行完，取不到foo 所以是 undfined, b.js执行完。跑a.js

{% highlight javascript %}
b.js
undefined
a.js
bar
{% endhighlight %}


再看一个栗子：

{% highlight javascript %}
// a.js
import {bar} from './b.js';
export function foo() {
  console.log('foo');
  bar();
  console.log('执行完毕');
}
foo();

// b.js
import {foo} from './a.js';
export function bar() {
  console.log('bar');
  if (Math.random() > 0.5) {
    foo();
  }
}
{% endhighlight %}

如果按照 CommonJS规范，上面代码，a加载b ,b又加载a ,这时a没有任何执行结果，输出是null ,对于b来说，foo的值等于null
foo()就应该报错。

但执行结果为：

{% highlight javascript %}
foo
bar
执行完毕
{% endhighlight %}

a.js之所以能够执行，原因就在于ES6加载的变量，都是动态引用其所在的模块。只要引用存在，代码就能执行。（和commonjs不同，引用会执行整个脚本，然后在内存生成一个对象）

















