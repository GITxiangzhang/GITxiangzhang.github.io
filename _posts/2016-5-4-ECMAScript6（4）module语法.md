---
title: ECMAScript6 (4) module语法

---

#### 概述

>JavaScript 一直没有模块（module）体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。
其他语言都有这项功能，比如 Ruby 的require、Python 的import，甚至就连 CSS 都有@import，但是 JavaScript 任何这方面的支持都没有，
这对开发大型的、复杂的项目形成了巨大障碍。

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

输出函数
{% highlight javascript %}
export function multiply(x, y) {
  return x * y;
};
{% endhighlight %}

as关键字重命名
{% highlight javascript %}
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
{% endhighlight %}


export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。
第一种写法直接输出1，第二种写法通过变量m，还是直接输出1。1只是一个值，不是接口。
{% highlight javascript %}
// 报错
export 1;

// 报错
var m = 1;
export m;*/

// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};
{% endhighlight %}

同样的，function和class的输出，也必须遵守这样的写法。

{% highlight javascript %}
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};
{% endhighlight %}

>export语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。
{% highlight javascript %}
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
{% endhighlight %}

CommonJS 模块输出的是值的缓存，不存在动态更新，
export命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错
{% highlight javascript %}
function foo() {
  export default 'bar' // SyntaxError
}
foo()
{% endhighlight %}

----------------------------------
#### import命令
>使用export命令定义了模块的对外接口以后，其他 JS 文件就可以通过import命令加载这个模块。
{% highlight javascript %}
import {firstName, lastName, year,multiply,foo} from './profile';

console.log(firstName);
console.log(lastName);
console.log(year);
{% endhighlight %}

as重命名
{% highlight javascript %}
import {firstName as hehe} from './profile';

console.log(hehe);
{% endhighlight %}
>注意，import命令具有提升效果，会提升到整个模块的头部，首先执行。

{% highlight javascript %}
foo();
import { foo } from './profile';
{% endhighlight %}

>由于import是静态执行，所以不能使用表达式和变量这些只有在运行时才能得到结果的语法结构。

import语句会执行所加载的模块

{% highlight javascript %}
import './profile';
{% endhighlight %}

多次重复执行同一句import语句，那么只会执行一次

目前通过 Babel 转码，CommonJS 模块的require命令和 ES6 模块的import命令，可以写在同一个模块里面，但是最好不要这样做。

----------------------------------
#### 模块的整体加载
>除了指定加载某个输出值，还可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面。

下面是一个circle.js文件，它输出两个方法area和circumference。
{% highlight javascript %}
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}
//main.js
import { area, circumference } from './circle';

console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));


{% endhighlight %}

整体加载方法如下。

{% highlight javascript %}
import * as circle from './circle';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
{% endhighlight %}
>模块整体加载所在的那个对象（上例是circle），应该是可以静态分析的，所以不允许运行时改变。下面的写法都是不允许的。

{% highlight javascript %}
import * as circle from './circle';

// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
{% endhighlight %}

----------------------------------
#### export default
>从之前例子看出，使用import命令时候，你必须知道要加载的变量名和函数名，如果我们不知道但依然想去加载这个模块该怎么弄

为了用户方便，这时候用到export default命令，为模块指定默认输出。

{% highlight javascript %}
// export-default.js
export default function () {
  console.log('foo');
}
{% endhighlight %}

其他模块加载该模块时，import命令可以为该匿名函数指定任意名字。
{% highlight javascript %}
// import-default.js
import customName from './export-default';
customName(); // 'foo'
{% endhighlight %}

>上面代码的import可以用任意名称指向export-default.js输出的方法,这时的import后面不需要用{}。

export default 用在非匿名函数前也是一样的。

{% highlight javascript %}
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;
{% endhighlight %}
>export default命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此export default命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能对应一个方法。

export default本质
{% highlight javascript %}
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as xxx } from 'modules';
// 等同于
// import xxx from 'modules';
{% endhighlight %}

export default命令其实只是输出一个叫做default的变量，所以它后面不能跟变量声明语句。
{% highlight javascript %}
// 正确
export var a = 1;

// 正确
var a = 1;
export default a;

// 错误
export default var a = 1;
{% endhighlight %}

----------------------------------
#### 模块之间的继承以及export与import的复合写法
>模块之间也可以继承，其实就是先输入在输出同一个模块。
假设有一个extend模块，继承了export-default模块。
{% highlight javascript %}
import foo,{PI,circle} from './export-default';
export {PI,circle};
export default foo;
// 可以写成
export {default,PI,circle} from './export-default';
{% endhighlight %}

此处 'export {default,PI,circle} from './export-default';'写法就是复合写法
但是复合写法下模块却不到引用模块的输出。

复合写法整体加载不可用as 重命名 且忽略default



----------------------------------
#### 跨模块常量
>const常量在模块的输入输出并没有不同。

{% highlight javascript %}
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
{% endhighlight %}

但是项目中也许会有很多const常量 ，我们可以建一个constants目录，将各种常量写在不同的文件里，输出出去

{% highlight javascript %}
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
{% endhighlight %}

然后利用继承合并到一个js中，使用时直接加载这个js就可以了。

{% highlight javascript %}
// constants/index.js
export {db} from './db';
export {users} from './users';
// script.js
import {db, users} from './constants';
{% endhighlight %}



