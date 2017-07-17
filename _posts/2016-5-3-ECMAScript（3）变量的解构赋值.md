---
title: ECMAScript6(3) 变量的解构赋值
---

#### 数组的解构赋值
##### 基本用法
ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这称为解构（Destructuring）

以前，为变量赋值，只能直接指定值。

{% highlight javascript %}
    let a=1;
    let b=2;
    let c=3;
{% endhighlight %}

ES6 允许这样写：

{% highlight javascript %}
    let [a,b,c]=[1,2,3]
{% endhighlight %}

上面代码表示，可以从数组中提取值，按照对应位置，对变量赋值。

本质上，这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。下面是一些使用嵌套数组进行解构的例子。

{% highlight javascript %}
    let [foo,[[bar],baz]]=[1,[[2],3]];
    foo//1
    bar//2
    baz//3
    let [ , , third] = ["foo", "bar", "baz"];
    third // "baz"

    let [x, , y] = [1, 2, 3];
    x // 1
    y // 3

    let [head, ...tail] = [1, 2, 3, 4];
    head // 1
    tail // [2, 3, 4]

    let [x, y, ...z] = ['a'];
    x // "a"
    y // undefined
    z // []
{% endhighlight %}
如果解构不成功，变量值等于 undefined。

{% highlight javascript %}
let [foo] = [];
let [bar, foo] = [1];
{% endhighlight %}
以上两种情况都属于解构不成功，foo的值都会等于undefined。

另一种情况是不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。这种情况下，解构依然可以成功。

{% highlight javascript %}
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
{% endhighlight %}

上面两个例子，都属于不完全解构，但是可以成功。

如果等号的右边不是数组（或者严格地说，不是可遍历的结构），那么将会报错。

{% highlight javascript %}
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
{% endhighlight %}
上面的语句都会报错，因为等号右边的值，要么转为对象以后不具备 Iterator 接口（前五个表达式），要么本身就不具备 Iterator 接口（最后一个表达式）。

对于 Set 结构，也可以使用数组的解构赋值。
{% highlight javascript %}
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
{% endhighlight %}

事实上，只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。

{% highlight javascript %}
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}
let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
{% endhighlight %}
上面代码中，fibs是一个 Generator 函数（参见《Generator 函数》一章），原生具有 Iterator 接口。解构赋值会依次从这个接口获取值。

##### 默认值
解构赋值允许指定默认值：

{% highlight javascript %}
  let [foo=true]=[];
  foo//true
  let[x,y='b']=['a'];
  //x='a',y='b'
  let[x,y='b']=['a',undefined];
  //x='a',y='b'
{% endhighlight %}
注意，ES6 内部使用严格相等运算符（===），判断一个位置是否有值。所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。
{% highlight javascript %}
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
{% endhighlight %}

上面代码中，如果一个数组成员是null，默认值就不会生效，因为null不严格等于undefined。

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。
{% highlight javascript %}
function f() {
  console.log('aaa');
}

let [x = f()] = [1];
{% endhighlight %}
上面代码中，因为x能取到值，所以函数f 根本不会执行。上面代码等价于：
{% highlight javascript %}
    let x;
    if([1]===undefind){
      x=f();
    }else{
      x=[1]
    }
{% endhighlight %}

默认值可以引用解构赋值的其他变量，但该变量必须已经声明。
{% highlight javascript %}
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
{% endhighlight %}

上面最后一个表达式之所以会报错，是因为x用到默认值y时，y还没有声明。

#### 对象的解构赋值
解构不仅可以用于数组，还可以用于对象。

{% highlight javascript %}
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
{% endhighlight %}

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

