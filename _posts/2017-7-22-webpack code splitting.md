---
title: webpack Code Splitting 分批打包、按需下载

---

#### Code Splitting

>首先说，code splitting指什么。我们打包时通常会生成一个大的bundle.js(或者index,看你如何命名)文件，这样所有的模块都会打包到这个bundle.js文件中，
最终生成的文件往往比较大。code splitting就是指将文件分割为块(chunk)，webpack使我们可以定义一些分割点(split point)，根据这些分割点对文件进行分块，并实现按需加载。

1. 第三方类库单独打包。由于第三方类库的内容基本不会改变，可以将其与业务代码分离出来，这样就可以将类库代码缓存在客户端，减少请求。
2. 按需加载。webpack支持定义分割点，通过require.ensure进行按需加载。
3. 通用模块单独打包。我们代码中可能会有一些通用模块，比如弹窗、分页、通用的方法等等。其他业务代码模块常常会有引用这些通用模块。若按照2中做，则会造成通用模块重复打包。这时可以将通用模块单独打包出来。

----------------------------------
#### CommonsChunkPlugin打包第三方类库
>我们项目中常常会用到一些第三方的类库，比如jquery,bootstrap等。可以配置多入口来将第三方类库单独打包，如下：

webpack.config.js

{% highlight JavaScript %}
var webpack= require('webpack')

module.exports={
  entry: {
    app: './main.js',
    vendor: ['jquery'],
  },
  output: {
    filename: 'bundle.js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */'vendor', /* filename= */'vendor.js')
  ]
}

{% endhighlight %}

CommonsChunkPlugin提供两个参数，第一个参数为对应的chunk名（chunk指文件块，对应entry中的属性名），第二个参数为生成的文件名。

这个插件做了两件事：

1. 将vendor配置的模块（jquery,bootstrap）打包到vendor.js中。
2. 将main中存在的jquery, bootstrap模块从文件中移除。这样main中则只留下纯净的业务代码。

源码见webpack-demos demo13。


----------------------------------
#### require.ensure按需加载
>require.ensure 是个啥？有叫代码分割，有叫异步加载，到底用来干嘛的，其实说白了它就是把js模块给独立导出一个.js文件的，然后使用这个
模块的时候，webpack会构造script dom元素，由浏览器发起异步请求这个js文件。

什么意思呢，我们假设一个场景：

比如页面有个按钮点击后可以打开某个地图。打开地图的话就要利用百度地图echar.js,于是我们不得不在首页中把echar.js一起打包进去首页,
echar.js文件是非常大的，假设为1m，于是就造成了我们首页打包的js非常大，用户打开首页的时间就比较长了。有没有什么好的解决方法呢？

方法1：

既然打包成同一个js非常大的话，那么我们完全可以把百度地图js分类出去，利用浏览器的并发请求
js文件处理，这样的话，会比加载一个js文件时间小得多。嗯，这也是个不错的方案。为baidumap.js
配置一个新的入口就行了，这样就能打包成两个js文件，都插入html即可（如果baidumap.js被多个
入口文件引用的话，也可以不用将其设置为入口文件，而且直接利用CommonsChunkPlugin,导出到一个
公共模块即可）

我们可以知道这个地图那还有没有更好的解决方案呢？

方法2：

百度地图是用户点击了才弹出来的，也就是说，这个功能是可选的。那么解决方案就来了，
能不能在用户点击的时候，我在去下载百度地图的js.当然可以。那如何实现用户点击的时候再去下载百度地图的js呢？于是，我们可以写一个按钮的监听器

{% highlight JavaScript %}
mapBtn.click(function() {
  //获取 文档head对象
  var head = document.getElementsByTagName('head')[0];
  //构建 <script>
  var script = document.createElement('script');
  //设置src属性
  script.async = true;
  script.src = "http://map.baidu.com/.js"
  //加入到head对象中
  head.appendChild(script);
})
{% endhighlight %}

可以在点击的时候，才加载百度地图，等百度地图加载完成后，在利用百度地图的对象去执行我们的操作。讲到这里webpack.ensure的原理也就讲的差不多了
它就是把一些js模块给独立出一个个js文件，然后需要用到的时候，在创建一个script对象，加入到document.head对象中即可，
浏览器会自动帮我们发起请求，去请求这个js文件，在写个回调，去定义得到这个js文件后，需要做什么业务逻辑操作。

那么我们就利用webpack的api去帮我们完成这样一件事情。点击后才进行异步加载百度地图js，上面的click加载js时我们自己写的，
webpack可以轻松帮我们搞定这样的事情，而不用我们手写

{% highlight JavaScript %}
mapBtn.click(function() {
  require.ensure([], function() {
    var baidumap = require('./baidumap.js') //baidumap.js放在我们当前目录下
  })
})
{% endhighlight %}

require.ensure的第一个参数：当前这个require.ensure所依赖的其他异步加载的模块。试想A和B都是异步加载的，B中需要A，那么在下载B之前
是要先下载A,这里我们要注意webpack会把参数里面依赖的异步模块和当前需要分离出去的异步模块给一起打包成同一个js文件。

说了半天举个栗子：
![webpack]({{ site.baseurl }}/images/webpack.jpg)

entry.js 依赖三个js。

* Abtn-work.js 是封装了 abtn按钮点击后，才执行的业务逻辑
* Bbtn-work.js 是封装了 bbtn按钮点击后，才执行的业务逻辑
* util.js 是封装了 entry.js需要利用的工具箱

针对上面的需求，优化方案

假设 Abtn-work.js Bbtn-work.js util.js都是非常大的文件
因为 Abtn-work.js Bbtn-work.js 都不是entry.js必须有的，即可能发生的操作，那么我们把
他们利用异步加载，当发生的时候再去加载就行了

util.js是entry.js立即马上依赖的工具箱。但是它又非常的大，所以将其配置打包成一个公共模块，

index.html：

{% highlight JavaScript %}
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>index</title>
</head>
<body>
<div id="aBtn">Abtn</div>
<div id="bBtn">Bbtn</div>
</body>
<script type="text/javascript" src="bundle.js"></script>
</html>
{% endhighlight %}

entry.js

{% highlight JavaScript %}
var util_sync = require('./util.js')

alert(util_sync.data)

document.getElementById("aBtn").onclick = function() {
    require.ensure([], function() {
        var awork = require('./Abtn-work.js')
        alert(awork.data)
        //异步里面再导入同步模块--实际是使用同步中的模块
        var util1 = require('./util.js')
    })
}

document.getElementById("bBtn").onclick = function() {

    require.ensure([], function() {
        var bwork = require('./Bbtn-work.js')
        alert(bwork.data)
    })
}
{% endhighlight %}

可以看到，workA-async.js， workB-async.js 都是点击后才ensure进来的。详情见webpack-demo。