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

以基于backbone的单页面应用为例，可以在router中进行配置实现按需加载，如下：

{% highlight JavaScript %}
router.js

var Router = Backbone.Router.extend({
    routes: {
        'a': 'a',
        'b': 'b'
    },

    a: function() {
        require.ensure(['./a'], (require) => {
            let a = require('./a');
            //do something
        })
    },

    b: function() {
        require.ensure(['./b'], (require) => {
            let b = require('./b');
            //do something
        })
    }
})
{% endhighlight %}

如上方式将打包出两个文件，a.js和b.js（当然名字会有所不同），且为按需加载。只有在访问a时，a.js才会被加载，b同理。但是这种做法存在两个问题：

1. 若路由分配不合理，会打包出很多很小的文件，每个文件或许只有几k，却多了很多网络请求，得不偿失。
2. 会造成通用模块的重复打包，比如a模块和b模块都引用了c模块，

对于问题一可以通过AggressiveMergingPlugin插件解决。

//在plugins中添加该插件：
plugins: [
    new webpack.optimize.AggressiveMergingPlugin()
]

对问题二可以用CommonsChunkPlugin打包通用模块解决。