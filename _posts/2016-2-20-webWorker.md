---
title: webWorker应用实践-多线程编程

---

#### setTimeout那些事儿之单线程
一直以来，大家都在说Javascript是单线程，浏览器无论在什么时候，都且只有一个线程在运行JavaScript程序。

但是，不知道大家有疑问没——就是我们在编程过程中的setTimeout(类似的还有setInterval、Ajax)，不是异步执行的吗？！！

{% highlight javascript %}
<!DOCTYPE html>
    <head>
        <title>setTimeout</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    </head>
    <body>
        <script>
            console.log("a");
            //利用setTimeout延迟执行匿名函数
            setTimeout(function(){
                console.log("b");
            },100);
            console.log("c");
        </script>
    </body>
</html>
{% endhighlight %}

这个结果很容易理解，因为我setTimeout里的内容是在100ms后执行的嘛，当然是先输出a，再输出c，100ms后再输出setTimeout里的b嘛。

咦，那Javascript这不就不是单线程了嘛，这不就可以实现多线程了？！！

其实，不是的。setTimeout没有打破JavaScript的单线程机制，它其实还是单线程。

为什么这么说呢，那就得理解setTimeout到底是个怎么回事。

{% highlight javascript %}
<!DOCTYPE html>
    <head>
        <title>setTimeout</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    </head>
    <body>
        <script>
            var date = new Date();
            //打印才进入时的时间
            console.log('first time: ' + date.getTime());
            //一秒后打印setTimeout里匿名函数的时间
            setTimeout(function(){
                var date1 = new Date();
                console.log('second time: ' + date1.getTime() );
                console.log( date1.getTime() - date.getTime() );
            },1000);
            //重复操作
            for(var i=0; i < 10000 ; i++){
                console.log(1);
            }
        </script>
    </body>
</html>
{% endhighlight %}

{% highlight javascript %}
first time: 1515652625247
10000VM358:12 1
undefined
VM358:7 second time: 1515652626604
VM358:8 1357
{% endhighlight %}

为何不是1000？
我们再看看下面的代码：

{% highlight javascript %}
<!DOCTYPE html>
    <head>
        <title>setTimeout</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    </head>
    <body>
        <script>
            //一秒后执行setTimeout里的匿名函数，alert下
            setTimeout(function(){
                alert("monkey");
            },1000);
            while(true){};
        </script>
    </body>
</html>
{% endhighlight %}

运行代码后！

我靠，怎么一直刷新，浏览器卡死了呢，并且没有alert！！

按道理，即使我while无限循环，在1秒后也得alert一下啊。

种种问题皆一个原因，JavaScript是单线程 。

要记住JavaScript是单线程，setTimeout没有实现多线程，它背后的真相是这样滴：

[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)

>setTimeout那些事儿之this

{% highlight javascript %}
<!DOCTYPE html>
    <head>
        <title>setTimeout</title>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    </head>
    <body>
        <script>
           var name = '!!';
           var obj = {
               name:'monkey',
               print:function(){
                   console.log(this.name);
               },
               test:function(){
                   //this.print
                   setTimeout(this.print,1000);
               }
           }
           obj.test();
        </script>
    </body>
</html>
{% endhighlight %}


undefined
VM8710:5 !!


咦，它怎么输出的是”!!”呢？不应该是obj里的”monkey”吗？！！

这是因为setTimeout中所执行函数中的this，永远指向window。

不对吧，那上面代码中的setTimeout(this.print,1000)里的this.print怎么指向的是obj呢？！！

注意哦，我这里说的是“延迟执行函数中的this”，而不是setTimeout调用环境下的this。

什么意思？

setTimeout(this.print,1000)，这里的this.print中的this就是调用环境下的；

而this.print=function(){console.log(this.name);}，这个匿名函数就是setTimeout延迟执行函数，其中的this.name也就是延迟执行函数中的this啦。


咦，那有个疑问，比如我想在setTimeout延迟执行函数中的this指向调用的函数呢，而不是window？！！我们该怎么办呢。

常用的方法就是利用that。

{% highlight javascript %}
var age = 24;
function Fn(){
    //that在此
    var that = this;
    this.age = 18;
    setTimeout(function(){
        console.log(that);
        console.log(that.age);
    },1000);
}
new Fn();
{% endhighlight %}

还有一种方法就是，利用bind。

如下:
{% highlight javascript %}
var age = 24;
function Fn(){
    this.age = 18;
    //bind传入this
    setTimeout(function(){
        console.log(this);
        console.log(this.age);
    }.bind(this),1000);
}
new Fn();
{% endhighlight %}

----------------------------------
#### HTML5 中工作线程（Web Worker）简介

HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变
JavaScript单线程的本质。

工作线程的出现使得在 Web 页面中进行多线程编程成为可能。众所周知，传统页面中（HTML5 之前）的 JavaScript 的运行都是以单线程的方式工作的，
虽然有多种方式实现了对多线程的模拟（例如：JavaScript 中的 setinterval 方法，setTimeout 方法等），但是在本质上程序的运行仍然是由
JavaScript 引擎以单线程调度的方式进行的。在 HTML5 中引入的工作线程使得浏览器端的 JavaScript 引擎可以并发地执行 JavaScript 代码，
从而实现了对浏览器端多线程编程的良好支持。

----------------------------------
#### 生成 worker

创建一个新的 worker 十分简单。你所要做的就是调用 Worker() 构造函数，指定一个要在 worker 线程内运行的脚本的 URI，如果你希望能够收到
worker 的通知，可以将 worker 的 onmessage 属性设置成一个特定的事件处理函数。

{% highlight javascript %}
var myWorker = new Worker("my_task.js");

myWorker.onmessage = function (oEvent) {
  console.log("Called back by the worker!\n");
};
{% endhighlight %}

或者：

{% highlight javascript %}
var myWorker = new Worker("my_task.js");

myWorker.addEventListener("message", function (oEvent) {
  console.log("Called back by the worker!\n");
}, false);

myWorker.postMessage(""); // start the worker.
{% endhighlight %}

例子中的第一行创建了一个新的 worker 线程。第三行为 worker 设置了 message 事件的监听函数。当 worker 调用自己的 postMessage()
函数时就会调用这个事件处理函数。最后，第七行启动了 worker 线程。


----------------------------------
#### 传递数据
在主页面与 worker 之间传递的数据是通过拷贝，而不是共享来完成的。传递给 worker 的对象需要经过序列化，接下来在另一端还需要反序列化。
页面与 worker 不会共享同一个实例，最终的结果就是在每次通信结束时生成了数据的一个副本。大部分浏览器使用结构化拷贝来实现该特性。

实例：创建一个子线程来计算求和：

{% highlight javascript %}
<!DOCTYPE html>
<html>
<head>
    <title>webWorkers 实例演示</title>
</head>
<body>
    请输入要求和的数：<input type="text" id="num"><br>
    <button onclick="caculate()">计算</button>

    <script type="text/javascript">
        //1.创建计算的子线程
        var worker = new Worker("worker1.js");
        function caculate(){
            var num = parseInt(document.querySelector('#num').value,10);
            //2.将数据传递给子线程
            worker.postMessage(num);
        }

        //3.从子线程接收处理结果并展示
        worker.onmessage = function(event){
            alert('总和为：'+ event.data);
        }
    </script>
</body>
</html>
{% endhighlight %}

可以将比较耗时的处理交给一个后台线程，去处理，处理完之后将结果返回给主页面。

----------------------------------
#### 线程之间进行数据交互
线程间的数据交互是通过发送和接收消息来相互传递信息的，主线程首先创建Worker，通过Worker对象的postMessage方法，将数据传递给后台线程，
而主程序通过onmessage 事件，或者自定义addEventListener 事件来监听后台返回后台线程处理的结果。同样，后台线程通过onmessage事件来接
收主页面传递的数据，通过postMessage将处理结果返回给主页面。


{% highlight javascript %}
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>线程之间进行数据交互</title>
</head>
<body>
<h2>线程之间进行数据交互</h2>
<table id="table" style="color: #FFF;background-color: #ccc;">

</table>
</body>
<script type="text/javascript">
  var nums = new Array(100),
      intStr = "";
  //1.处理非字符串数据
  for(var i = 0; i<100; i++){
    nums[i] = parseInt(Math.random()*100);
    intStr += nums[i] + ";";
  }
  //2.创建新进程
  var worker = new Worker("worker1.js");
  //3.向子进程发送数据
  worker.postMessage(intStr);

  //4.从子线程获取处理结果
  worker.onmessage = function(event){
    var row,
        col,
        tr,
        td,
        table = document.querySelector("#table");
    var numArr = event.data.split(";");

    for(var i = 0; i<numArr.length; i++){
      row = parseInt(i/10);
      col = i%10;
      if (col == 0 ) {
        tr = document.createElement("tr");
        tr.id = "tr" + row;
        table.appendChild(tr);
      }else{
        tr = document.querySelector("#tr" + row);
      }
      td = document.createElement('td');
      tr.appendChild(td);
      td.innerHTML = numArr[i];
      td.width = "30";
    }
  }
</script>
</html>

onmessage = function(event){
    var strNum = event.data;
    var numArr = strNum.split(";");
    var returnNum = "";
    for(var i =0; i<numArr.length; i++){
        if (numArr[i]%3 ==0) {
            returnNum += numArr[i] + ";";
        }
    }
    postMessage(returnNum);

}


{% endhighlight %}



----------------------------------
#### 线程间的嵌套
线程中可以嵌套子线程，这样可以把一个较大的后台线程切分成几个子线程，每个子线程格子完成相对独立的工作。

还是使用上述的实例，构造一个单层子线程嵌套的例子。把之前主页面生成随机数的工作放到后台线程，然后在后台线程中构造一个子线程，
来挑选出可以被3整除的数据。传递的数据采用JSON的数据格式。

{% highlight javascript %}
<!DOCTYPE html>
<html>
<head>
    <title>线程之间进行数据交互</title>
</head>
<body>
    <h2>线程之间进行数据交互</h2>
    <table id="table" style="color: #FFF;background-color: #ccc;">

    </table>
</body>
    <script type="text/javascript">
        var nums = new Array(100),
        intStr = "";
        //1.处理非字符串数据
        for(var i = 0; i<100; i++){
            nums[i] = parseInt(Math.random()*100);
            intStr += nums[i] + ";";
        }
        //2.创建新进程
        var worker = new Worker("worker2.js");
        //3.向子进程发送数据
        worker.postMessage(intStr);

        //4.从子线程获取处理结果
        worker.onmessage = function(event){
            var row,
                col,
                tr,
                td,
                table = document.querySelector("#table");
            var numArr = event.data.split(";");

            for(var i = 0; i<numArr.length; i++){
                row = parseInt(i/10);
                col = i%10;
                if (col == 0 ) {
                    tr = document.createElement("tr");
                    tr.id = "tr" + row;
                    table.appendChild(tr);
                }else{
                    tr = document.querySelector("#tr" + row);
                }
                td = document.createElement('td');
                tr.appendChild(td);
                td.innerHTML = numArr[i];
                td.width = "30";
            }
        }
    </script>
</html>


onmessage=function(event){
    var intArray=new Array(100);    //随机数组
    //生成100个随机数
    for(var i=0;i<100;i++)
        intArray[i]=parseInt(Math.random()*100);
    var worker;
    //创建子线程
    worker=new Worker("worker2.js");
    //把随机数组提交给子线程进行挑选工作
    worker.postMessage(JSON.stringify(intArray));
    worker.onmessage = function(event) {
        //把挑选结果返回主页面
        postMessage(event.data);
    }
}



onmessage = function(event) {
    //还原整数数组
    var intArray= JSON.parse(event.data);
    var returnStr;
    returnStr="";
    for(var i=0;i<intArray.length;i++)
    {
        //能否被3整除
        if(parseInt(intArray[i])%3==0)
        {
            if(returnStr!="")
                returnStr+=";";
            //将能被3整除的数字拼接成字符串
            returnStr+=intArray[i];
        }
    }
    //返回拼接字符串
    postMessage(returnStr);
    //关闭子线程
    close();
}
{% endhighlight %}

向子线程传递消息时，用worker.postMessage,向主页面提交数据时直接用postMessage.


----------------------------------
#### SharedWorker共享线程
共享线程
共享线程可以由两种方式来定义：一是通过指向 JavaScript 脚本资源的 URL 来创建，而是通过显式的名称。当由显式的名称来定义的时候，
由创建这个共享线程的第一个页面中使用 URL 会被用来作为这个共享线程的 JavaScript 脚本资源 URL。通过这样一种方式，它允许同域中的多个应
用程序使用同一个提供公共服务的共享线程，从而不需要所有的应用程序都去与这个提供公共服务的 URL 保持联系。

无论在什么情况下，共享线程的作用域或者是生效范围都是由创建它的域来定义的。因此，两个不同的站点（即域）使用相同的共享线程名称也不会冲突。

共享线程的创建

创建共享线程可以通过使用 SharedWorker() 构造函数来实现，这个构造函数使用 URL 作为第一个参数，即是指向 JavaScript 资源文件的 URL，
同时，如果开发人员提供了第二个构造参数，那么这个参数将被用于作为这个共享线程的名称。创建共享线程的代码示例如下：

{% highlight javascript %}
var worker = new SharedWorker('sharedworker.js', ’ mysharedworker ’ );
{% endhighlight %}

与共享线程通信

共享线程的通信也是跟专用线程一样，是通过使用隐式的 MessagePort 对象实例来完成的。当使用 SharedWorker() 构造函数的时候，这个对象将
通过一种引用的方式被返回回来。我们可以通过这个引用的 port 端口属性来与它进行通信。发送消息与接收消息的代码示例如下：

实例1：在单个页面中使用sharedWorker

{% highlight javascript %}
<!DOCTYPE html>
<html>
<head>
    <title>单个页面的SharedWorker</title>
</head>
<body>
    <h2>单个页面的SharedWorker</h2>
    <div id="show"></div>
    <script type="text/javascript">
        var worker = new SharedWorker('test.js');
        var div = document.querySelector('#show');

        worker.port.onmessage = function(e){
            div.innerHTML = e.data;
        }

    </script>
</body>
</html>


onconnect = function(e){
    var port = e.ports[0];
    port.postMessage('你好！');
}
{% endhighlight %}

实例2：在多个页面中使用sharedWorker：
{% highlight javascript %}

两个页面相同
<!DOCTYPE HTML>
<html>
<head>
  <meta charset="UTF-8">
  <title>Shared workers: demo 2</title>
</head>
<body>
<div id="log"></div>
<input type="text" name="" id="txt">
<button id="get">get</button>
<button id="set">set</button>
<script>
  var worker = new SharedWorker('shared.js');
  var get = document.getElementById('get');
  var set = document.getElementById('set');
  var txt = document.getElementById('txt');
  var log = document.getElementById('log');

  worker.port.addEventListener('message', function(e) {
    log.innerText = e.data;
  }, false);
  worker.port.start(); // note: need this when using addEventListener

  set.addEventListener('click',function(e){
    worker.port.postMessage(txt.value);
  },false);

  get.addEventListener('click',function(e){
    worker.port.postMessage('get');
  },false);
</script>
</body>
</html>

var data;
onconnect = function(e) {
    var port = e.ports[0];
    port.onmessage = function(e) {

        if(e.data=='get'){
            port.postMessage(data);
        }else{
            data=e.data;
        }
    }
}

{% endhighlight %}

第一种方法中是使用事件句柄的方式将听message事件，不需要调用worker.port.start(),第二种方法是通过addEventListener()方法监听
message事件，需要worker.port.start()方法激活端口。他们不同于worker，当有人和他通信时触发connect事件，然后他的message事件是绑定
在messagePort对象上的，不想worker那样，你可以回头看看worker是怎么做的。