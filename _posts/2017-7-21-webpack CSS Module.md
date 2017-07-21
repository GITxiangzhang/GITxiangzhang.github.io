---
title: webpack CSS Module

---

#### CSS Module 局部作用域

>CSS Modules 加入了局部作用域和模块依赖。

CSS规则都是全局的，任何一个组件的样式规则对整个页面都有效。

产生局部作用域唯一的办法就是使用一个独一无二的class名字，不会与其他的选择器重名。这就是CSS Modules 的做法。

例如：

{% highlight javascript %}
var React =require('react');
var ReactDOM =require('react-dom');
var style =require('./app.css');
ReactDOM.render(
    <div>
      <h1 className={style.title}>Hello World</h1>
    </div>,
    document.getElementById('example')
)

{% endhighlight %}

上面代码中，我们将 app.css 输入到style对象，然后引用style.title表示一个class。
{% highlight javascript %}
.title{
   color:red;
}
{% endhighlight %}
构建工具会将类名 style.title 编译成 一个哈希字符串。

{% highlight javascript %}
<h1 class="l9td2F4yuAHHvsv8gzFp_ ">
  Hello World
</h1>
{% endhighlight %}

app.css 也同时会被编译

{% highlight javascript %}
.l9td2F4yuAHHvsv8gzFp_  {
  color: red;
}
{% endhighlight %}

这样可以保证这个类名独一无二，只对这个组件有效果。

CSS Module提供各种插件，支持不同的构建工具，建议使用webpack css-loader插件。

webpack相关教程可见>[webpack 整理](https://gitxiangzhang.github.io/xiangzhang.github.io/2017/07/20/webpack-%E6%95%B4%E7%90%86.html)

下面是示例的 webpack.config.js

{% highlight javascript %}
module.exports={
 entry:'./main.js',
 output:{
 filename:'./bundle.js'
 },

  module: {
    loaders:[
      { test: /\.js[x]?$/, exclude: /node_modules/, loader: 'babel-loader?presets[]=es2015&presets[]=react' },
      { test: /\.css$/, loader: 'style-loader!css-loader?modules' }
    ]
  }

}
{% endhighlight %}

上面代码中，关键的是 style-loader!css-loader?modules,后面加了个查询参数 modules,它表示开启CSS Module 功能。



----------------------------------
#### 全局作用域
>CSS Module 允许使用：global(.clasName)的语法，声明一个全局规则。凡是这样声明的calss都不会被编译成哈希字符串。


app.css 加入一个全局的 calss
{% highlight javascript %}
.title{
   color:red;
}
:global(.litle){
 color: green;
}
{% endhighlight %}

main.js 使用普通的class写法,就会引用全局class

{% highlight javascript %}
var React =require('react');
var ReactDOM =require('react-dom');
var style =require('./app.css');
ReactDOM.render(
    <div>
      <h1 className={style.title}>Hello World</h1>
      <h1 className='title'>Hello World</h1>
    </div>,
    document.getElementById('example')
)

{% endhighlight %}

CSS Modules 还提供一种显式的局部作用域语法:local(.className)，等同于.className，所以上面的app.css也可以写成下面这样。

{% highlight javascript %}
:local(.title) {
  color: red;
}

:global(.title) {
  color: green;
}
{% endhighlight %}


----------------------------------
#### 定制哈希类名
>CSS-loader 默认的哈希算法是[ hash:base64 ],将会把.title编译成.l9td2F4yuAHHvsv8gzFp_这样的字符串。

webpack.config.js里面可以定制哈希字符串的格式。localIdentName表示命名规则

{% highlight javascript %}
module:{
 loaders:[
 {
    test:/\.css$/,
    loader:"style-loader!css-loader?modules&localIdentName=[path][name]---[local]---[hash:base64:5]"
 }

 ]

}
{% endhighlight %}

运行这个示例，你会发现.title被编译成 .app---title---l9td2

----------------------------------
#### Class的组合
>在CSS Modules 中，一个选择器可以继承另一个选择器，这称为组合（composition）

在 app.css中，.title 继承 .className

{% highlight javascript %}
.className{
  backgrund-color:bule;
}
.title{
 composes:className;
 color:red;
}
{% endhighlight %}

运行demo06 这个示例，会发现app.css 编译成了：

{% highlight javascript %}
.app---className---1RcNP {
  background-color: blue;
}

.app---title---l9td2 {
  color:red;
}

.title {
  color: blue;
}
{% endhighlight %}

相应的 h1 的class 也被编译成 app---title---l9td2 app---className---1RcNP。

>选择器也可以继承其他的CSS文件里的规则。

{% highlight javascript %}
//another.css
.className {
  background-color: blue;
}
{% endhighlight %}

app.css可以继承another.css里面的规则。

{% highlight javascript %}
.title {
  composes: className from './another.css';
  color: red;
}
{% endhighlight %}

----------------------------------
#### Class的组合
>CSS Modules 支持使用变量，不过需要安装 PostCSS 和 postcss-modules-values。

$ npm install --save postcss-loader postcss-modules-values

把postcss-loader加入webpack.config.js。

{% highlight javascript %}
var values = require('postcss-modules-values');

module.exports = {
  entry: __dirname + '/index.js',
  output: {
    publicPath: '/',
    filename: './bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel',
        query: {
          presets: ['es2015', 'stage-0', 'react']
        }
      },
      {
        test: /\.css$/,
        loader: "style-loader!css-loader?modules!postcss-loader"
      },
    ]
  },
  postcss: [
    values
  ]
};
{% endhighlight %}

接着，在colors.css里面定义变量。

{% highlight javascript %}
@value blue: #0c77f8;
@value red: #ff0000;
@value green: #aaf200;
{% endhighlight %}

app.css可以引用这些变量。
{% highlight javascript %}
@value colors: "./colors.css";
@value blue, red, green from colors;

.title {
  color: red;
  background-color: blue;
}
{% endhighlight %}



