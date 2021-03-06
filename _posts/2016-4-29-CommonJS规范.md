---
title: CommonJS规范

---

#### 概述

>Node应用由模块组成，采用CommonJS模块规范。根据这个规范，每个文件就是一个模块，
有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。

{% highlight javascript %}
// example.js
var x = 5;
var addX = function (value) {
  return value + x;
};
{% endhighlight %}

上面代码中，变量x和函数addX，是当前文件example.js私有的，其他文件不可见。
如果想在多个文件分享变量，必须定义为global对象的属性。

{% highlight javascript %}
global.warning = true;
{% endhighlight %}
上面代码的warning变量，可以被所有文件读取。当然，这样写法是不推荐的。

CommonJS规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。加载某个模块，其实是加载该模块的module.exports属性。

{% highlight javascript %}
var x = 5;
var addX = function (value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
{% endhighlight %}

上面代码通过module.exports输出变量x和函数addX。

require方法用于加载模块。

{% highlight javascript %}
var example = require('./example.js');
console.log(example.x); // 5
console.log(example.addX(1)); // 6
{% endhighlight %}

CommonJS模块的特点如下。
1. 所有代码都运行在模块作用域，不会污染全局作用域。
2. 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
3. 模块加载的顺序，按照其在代码中出现的顺序。

----------------------------------
#### module对象
>Node内部提供一个Module构建函数。所有模块都是Module的实例。

{% highlight javascript %}
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  // ...
{% endhighlight %}

每个模块内部，都有一个module对象，代表当前模块。它有以下属性。

* module.id 模块的识别符，通常是带有绝对路径的模块文件名。
* module.filename 模块的文件名，带有绝对路径。
* module.loaded 返回一个布尔值，表示模块是否已经完成加载。
* module.parent 返回一个对象，表示调用该模块的模块。
* module.children 返回一个数组，表示该模块要用到的其他模块。
* module.exports 表示模块对外输出的值。

下面是一个示例文件，最后一行输出module变量。

{% highlight javascript %}
// example.js
var jquery = require('jquery');
exports.$ = jquery;
console.log(module);
{% endhighlight %}

执行这个文件，命令行会输出如下信息。

{% highlight javascript %}
{ id: '.',
  exports: { '$': [Function] },
  parent: null,
  filename: '/path/to/example.js',
  loaded: false,
  children:
   [ { id: '/path/to/node_modules/jquery/dist/jquery.js',
       exports: [Function],
       parent: [Circular],
       filename: '/path/to/node_modules/jquery/dist/jquery.js',
       loaded: true,
       children: [],
       paths: [Object] } ],
  paths:
   [ '/home/user/deleted/node_modules',
     '/home/user/node_modules',
     '/home/node_modules',
     '/node_modules' ]
}
{% endhighlight %}

如果在命令行下调用某个模块，比如node something.js，那么module.parent就是undefined。如果是在脚本之中调用，比如require('./something.js')，那么module.parent就是调用它的模块。利用这一点，可以判断当前模块是否为入口脚本。

{% highlight javascript %}
if (!module.parent) {
    // ran with `node something.js`
    app.listen(8088, function() {
        console.log('app listening on port 8088');
    })
} else {
    // used with `require('/.something.js')`
    module.exports = app;
}
{% endhighlight %}

----------------------------------
##### module.exports属性
>module.exports属性表示当前模块对外输出的接口，其他文件加载该模块，实际上就是读取module.exports变量。
{% highlight javascript %}
var EventEmitter = require('events').EventEmitter;
module.exports = new EventEmitter();

setTimeout(function() {
  module.exports.emit('ready');
}, 1000);
{% endhighlight %}

上面模块会在加载后1秒后，发出ready事件。其他文件监听该事件，可以写成下面这样。

{% highlight javascript %}
var a = require('./a');
a.on('ready', function() {
  console.log('module a is ready');
});
{% endhighlight %}

----------------------------------
##### exports变量属性
>为了方便，Node为每个模块提供一个exports变量，指向module.exports。这等同在每个模块头部，有一行这样的命令。

{% highlight javascript %}
var exports = module.exports;
{% endhighlight %}

造成的结果是，在对外输出模块接口时，可以向exports对象添加方法。因为指向module.exports其实就是给module.exports添加方法

{% highlight javascript %}
exports.area = function (r) {
  return Math.PI * r * r;
};

exports.circumference = function (r) {
  return 2 * Math.PI * r;
};
{% endhighlight %}

注意，不能直接将exports变量指向一个值，因为这样等于切断了exports与module.exports的联系。
{% highlight javascript %}
exports = function(x) {console.log(x)};
{% endhighlight %}
上面这样的写法是无效的，因为exports不再指向module.exports了。

下面的写法也是无效的。
{% highlight javascript %}
exports.hello = function() {
  return 'hello';
};
module.exports = 'Hello world';
{% endhighlight %}
上面代码中，hello函数是无法对外输出的，因为module.exports被重新赋值了。

这意味着，如果一个模块的对外接口，就是一个单一的值，不能使用exports输出，只能使用module.exports输出。
{% highlight javascript %}
module.exports = function (x){ console.log(x);};
{% endhighlight %}
如果你觉得，exports与module.exports之间的区别很难分清，一个简单的处理方法，就是放弃使用exports，只使用module.exports。

----------------------------------
#### AMD规范与CommonJS规范的兼容性
>CommonJS规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数。
由于Node.js主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以CommonJS规范比较适用。
但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范。

AMD规范使用define方法定义模块，下面就是一个例子：

{% highlight javascript %}
define(['package/lib'], function(lib){
  function foo(){
    lib.log('hello world!');
  }

  return {
    foo: foo
  };
});
{% endhighlight %}

AMD规范允许输出的模块兼容CommonJS规范，这时define方法需要写成下面这样：

{% highlight javascript %}
define(function (require, exports, module){
  var someModule = require("someModule");
  var anotherModule = require("anotherModule");

  someModule.doTehAwesome();
  anotherModule.doMoarAwesome();

  exports.asplode = function (){
    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();
  };
});
{% endhighlight %}


----------------------------------
#### require命令
>Node使用CommonJS模块规范，内置的require命令用于加载模块文件。
require命令的基本功能是，读入并执行一个JavaScript文件，然后返回该模块的exports对象。如果没有发现指定模块，会报错。
{% highlight javascript %}
// example.js
var invisible = function () {
  console.log("invisible");
}

exports.message = "hi";

exports.say = function () {
  console.log(message);
}
{% endhighlight %}
运行下面的命令，可以输出exports对象。

{% highlight javascript %}
var example = require('./example.js');
example
// {
//   message: "hi",
//   say: [Function]
// }
{% endhighlight %}
如果模块输出的是一个函数，那就不能定义在exports对象上面，而要定义在module.exports变量上面。

{% highlight javascript %}
module.exports = function () {
  console.log("hello world")
}

require('./example2.js')()
{% endhighlight %}
上面代码中，require命令调用自身，等于是执行module.exports，因此会输出 hello world。

----------------------------------
##### 加载规则
>require命令用于加载文件，后缀名默认为.js。

根据参数的不同格式，require命令去不同路径寻找模块文件。

1. 如果参数字符串以“/”开头，则表示加载的是一个位于绝对路径的模块文件。比如，require('/home/marco/foo.js')将加载/home/marco/foo.js。

2. 如果参数字符串以“./”开头，则表示加载的是一个位于相对路径（跟当前执行脚本的位置相比）的模块文件。比如，require('./circle')将加载当前脚本同一目录的circle.js。

3. 如果参数字符串不以“./“或”/“开头，则表示加载的是一个默认提供的核心模块（位于Node的系统安装目录中），或者一个位于各级node_modules目录的已安装模块（全局安装或局部安装）。

举例来说，脚本/home/user/projects/foo.js执行了require('bar.js')命令，Node会依次搜索以下文件。
{% highlight javascript %}
/usr/local/lib/node/bar.js
/home/user/projects/node_modules/bar.js
/home/user/node_modules/bar.js
/home/node_modules/bar.js
/node_modules/bar.js
{% endhighlight %}

这样设计的目的是，使得不同的模块可以将所依赖的模块本地化。
4. 如果参数字符串不以“./“或”/“开头，而且是一个路径，比如require('example-module/path/to/file')，则将先找到example-module的位置，然后再以它为参数，找到后续路径。

5. 如果指定的模块文件没有发现，Node会尝试为文件名添加.js、.json、.node后，再去搜索。.js件会以文本格式的JavaScript脚本文件解析，.json文件会以JSON格式的文本文件解析，.node文件会以编译后的二进制文件解析。

6. 如果想得到require命令加载的确切文件名，使用require.resolve()方法。

----------------------------------
##### 目录的加载规则
>通常，我们会把相关的文件会放在一个目录里面，便于组织。这时，最好为该目录设置一个入口文件，让require方法可以通过这个入口文件，加载整个目录。

在目录中放置一个package.json文件，并且将入口文件写入main字段。下面是一个例子。
{% highlight javascript %}
// package.json
{ "name" : "some-library",
  "main" : "./lib/some-library.js" }
{% endhighlight %}
require发现参数字符串指向一个目录以后，会自动查看该目录的package.json文件，然后加载main字段指定的入口文件。

如果package.json文件没有main字段，或者根本就没有package.json文件，则会加载该目录下的index.js文件或index.node文件。

----------------------------------
##### 模块的缓存
>第一次加载某个模块时，Node会缓存该模块。以后再加载该模块，就直接从缓存取出该模块的module.exports属性。
{% highlight javascript %}
require('./example.js');
require('./example.js').message = "hello";
require('./example.js').message
// "hello"
{% endhighlight %}
上面代码中，连续三次使用require命令，加载同一个模块。第二次加载的时候，为输出的对象添加了一个message属性。
但是第三次加载的时候，这个message属性依然存在，这就证明require命令并没有重新加载模块文件，而是输出了缓存。

如果想要多次执行某个模块，可以让该模块输出一个函数，然后每次require这个模块的时候，重新执行一下输出的函数。

所有缓存的模块保存在require.cache之中，如果想删除模块的缓存，可以像下面这样写。

{% highlight javascript %}
// 删除指定模块的缓存
delete require.cache[moduleName];

// 删除所有模块的缓存
Object.keys(require.cache).forEach(function(key) {
  delete require.cache[key];
})
{% endhighlight %}
注意，缓存是根据绝对路径识别模块的，如果同样的模块名，但是保存在不同的路径，require命令还是会重新加载该模块。
[其他更多关于CommonJS规范参考这里](http://javascript.ruanyifeng.com/nodejs/module.html)