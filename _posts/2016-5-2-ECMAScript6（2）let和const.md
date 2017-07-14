---
title: ECMAScript6(2) let const以及块级作用域
---

#### let
##### let基本用法
ES6 新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。

{% highlight javascript %}
{
  let a = 10;
  var b = 1;
}
a // ReferenceError: a is not defined.
b // 1
{% endhighlight %}

上面代码在代码块之中，分别用let和var声明了两个变量。然后在代码块之外调用这两个变量，结果let声明的变量报错，var声明的变量返回了正确的值。这表明，let声明的变量只在它所在的代码块有效。

for循环的计数器，就很合适使用let命令。
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined
上面代码中，计数器i只在for循环体内有效，在循环体外引用就会报错。

{% highlight javascript %}
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
{% endhighlight %}
上面代码中，变量i是var命令声明的，在全局范围内都有效，所以全局只有一个变量i。每一次循环，变量i的值都会发生改变，而循环内被赋给数组a的函数内部的console.log(i)，里面的i指向的就是全局的i。

也就是说，所有数组a的成员里面的i，指向的都是同一个i，导致运行时输出的是最后一轮的i的值，也就是10。

以前如果想要输出的是6一般这么写：

{% highlight javascript %}
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
{% endhighlight %}

上面代码中，变量i是let声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。

你可能会问，如果每一轮循环的变量i都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。

另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

{% highlight javascript %}
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// ab
{% endhighlight %}
上面代码正确运行，输出了3次abc。这表明函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域。

##### 不存在变量提升
var命令会发生”变量提升“现象，即变量可以在声明之前使用，值为undefined。这种现象多多少少是有些奇怪的，按照一般的逻辑，变量应该在声明语句之后才可以使用。

为了纠正这种现象，let命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

{% highlight javascript %}
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
{% endhighlight %}

上面代码中，变量foo用var命令声明，会发生变量提升，即脚本开始运行时，变量foo已经存在了，但是没有值，所以会输出undefined。

变量bar用let命令声明，不会发生变量提升。这表示在声明它之前，变量bar是不存在的，这时如果用到它，就会抛出一个错误。

##### 暂时性死区
暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

##### 不允许重复声明
let不允许在相同作用域内，重复声明同一个变量。

{% highlight javascript %}
// 报错
function () {
  let a = 10;
  var a = 1;
}

// 报错
function () {
  let a = 10;
  let a = 1;
}
{% endhighlight %}

因此，不能在函数内部重新声明参数。
{% highlight javascript %}
function func(arg) {
  let arg; // 报错
}

function func(arg) {
  {
    let arg; // 不报错
  }
}
{% endhighlight %}


#### 块级作用域
ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

第一种场景，内层变量可能会覆盖外层变量。
{% highlight javascript %}
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}

f(); // undefined
{% endhighlight %}
上面代码的原意是，if代码块的外部使用外层的tmp变量，内部使用内层的tmp变量。但是，函数f执行后，输出结果为undefined，原因在于变量提升，导致内层的tmp变量覆盖了外层的tmp变量。

第二种场景，用来计数的循环变量泄露为全局变量。
{% highlight javascript %}
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
{% endhighlight %}
上面代码中，变量i只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。

let实际上为 JavaScript 新增了块级作用域。

{% highlight javascript %}
function (){
var a=1;
if(true){
var a=10;
}
console.log(a)//10
}

function (){
let a=1;
if(true){
let a=10;
}
console.log(a)//1
}

{% endhighlight %}

上面两个代码块都申明了a,但是用let申明输出的是 1 ,这表示外层代码块不受内层代码块的影响。
























按照惯例我们来个helloworld入门。

##### 包管理器
npm可以自动管理包的依赖，只需要安装你要的包，不需要考虑依赖。
在 PHP 中, 包管理使用的 Composer, python 中，包管理使用 easy_install 或者 pip，ruby 中我们使用 gem。而在 Node.js中，对应就是 npm，npm 是 Node.js Package Manager 的意思。

##### express框架
express框架是node.js应用最广泛的web框架，现在是4.x的版本。
express官网[http://www.expressjs.com.cn/](http://www.expressjs.com.cn/)，没事经常看看API。

1.新建一个文件夹nodestart进去安装express

![nodestart]({{ site.baseurl }}/images/installexpress.png)

安装完后nodestart目录下会出现一个node_modules文件夹，tree(dos命令符)看看里包含express文件夹说明安装成

2.新建一个app.js,写进去代码：

{% highlight javascript %}

{% endhighlight %}

3.执行node app.js

这时候我们的app 就跑起来了，终端中会输出 app is listening at port 3000,这时我们打开浏览器，访问 http://localhost:3000/，会出现 Hello World。如果没有出现的话，肯定是上述哪一步弄错了，自己调试一下。

#### Tips
##### 端口
端口的作用：通过端口来区分出同一电脑内不同应用或者进程，从而实现一条物理网线(通过分组交换技术-比如internet)同时链接多个程序 Port_(computer_networking)
端口号是一个 16位的 uint, 所以其范围为 1 to 65535 (对TCP来说, port 0 被保留，不能被使用. 对于UDP来说, source端的端口号是可选的， 为0时表示无端口).
app.listen(3000)，进程就被打标，电脑接收到的3000端口的网络消息就会被发送给我们启动的这个进程



