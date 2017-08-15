---
title: jquery extend()
---

#### 第一部分
1.理解jquery extend();
$.extend(dest,src1,src2...)
将后面的合并到dest中，返回的是合并后的dest,这种写法会改变dest的结构的。如果不想改变dest的结构可以：$.extend({},src1,src2...) 将{}作为dest的参数。

    var src=$.extend({},{name:"hello",age:11},{name:"world",age:23,sex:"男"})
    src={name:"world",age:23,sex:"男"}

后面的参数如果和前面的参数存在相同的名称，后面会覆盖前面的参数值



----------------------------------

#### 第二部分
省略dest参数{}，如果省略了，则该方法只能有一个src参数，而且是将src合并到调用extend方法的对象中去，如：
  $.extend(src)
该方方法是将src合并到juqery全局对象中去
如下将hello方法合并到jquery全局对象中去

	$.extend({
		hello:function(){
	   alert('hello world!')
		}
	}
	)
	$.hello();/*调用*/
	$.hello()=function(){
	}
$.fn.extend(src)
该方方法是将src合并到juqery的实例对象中去
如下将hello方法合并到jquery的实例对象中去

   	$.fn.extend({}
		hello:function() {
			alert('hello world!')
		}
	}
	)
	$.fn.hello();
	$.prototype.hello();
调用

----------------------------------

#### 第三部分
常用的扩展空间
将hello方法合并到之前扩展的jquery的net命名空间去

	$.extend({net:{}})
	$.extend($.net,
		hello:function(){
	    alert('hello world!')
		}
		)

Jquery的extend方法还有一个重载原型：extend(boolean,dest,src1,src2,src3...)第一个参数boolean代表是否进行深度拷贝，其余参数和前面介绍的一致，什么叫深层拷贝，我们看一个例子：

	var result=$.extend( true, {},
	{ name: "John", location: {city: "Boston",county:"USA"} },
	{ last: "Resig", location: {state: "MA",county:"China"} } );

我们可以看出src1中嵌套子对象location:{city:"Boston"},src2中也嵌套子对象location:{state:"MA"},第一个深度拷贝参数为true，那么合并后的结果就是：

	result={name:"John",last:"Resig",
	location:{city:"Boston",state:"MA",county:"China"}}

也就是说它会将src中的嵌套子对象也进行合并，而如果第一个参数boolean为false，我们看看合并的结果是什么，如下

	var result=$.extend( false, {},
	{ name: "John", location:{city: "Boston",county:"USA"} },
	{ last: "Resig", location: {state: "MA",county:"China"} }
	);

那么合并后的结果就是:

	result={name:"John",last:"Resig",location:{state:"MA",county:"China"}}

{{ page.date|date_to_string }}