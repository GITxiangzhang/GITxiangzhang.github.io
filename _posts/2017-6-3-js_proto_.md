---
title: javascript原型于原型链
---

今天要介绍的是，原型与原型链，看了很多资料一直没弄明白原型原型链的坑。今天总结梳理下。
js中万物皆为对象，对于实例化来说，分为原型(函数对象)和实例化对象（普通对象），首先我们要明确实例化对象是没有prototype属性的，例如：

    function Cat(name,color){
    this.name = name;
    this.color = color;
    }
    var c1=new Cat("二狗","白色");
    alert(c1.prototype)//undefined

在JS中，每当创建一个函数对象Cat 时，该对象中都会内置一些属性，其中包括prototype和__proto__,  prototype即原型对象，它记录着Cat的一些属性和方法。
那么，prototype有什么用呢？
其实prototype的主要作用就是继承。通俗一点讲，prototype中定义的属性和方法都是留给自己的“后代”用的，因此，子类完全可以访问prototype中的属性和方法。
而我们是怎样将prototype属性传递给后代的呢？靠的就是原型链。

----------------------------------

#### 原型链
讲到原型链，其核心就是__proto__属性，他在函数对象和实例对象中都存在，它的作用是保存父类对象的prototype对象。————原型链的核心
当通过操作符new实例化对象时候，通常会把父类的prototype对象赋值给新对象的__proto__对象，这样就形成了一代代的传承。例如：

    function Cat(name,color){
    this.name = name;
    this.color = color;
      this.prototype.say=function(){
      alert('瞄');
      }
    }
    var c1=new Cat("二狗","白色");

    c1.__proto__ →→Cat.prototype
    Cat.prototype也有__proto__ 属性
    Cat.prototype.__proto__ →→ Object.prototype
    Object.prototype.__proto__→→ null

这就是原型链了。当c1.say 执行时c1会先查找自身是否含有该属性，当找不到时，c1就会沿着原型链依次查找。
在上面的例子中，我们在Cat的prototype上定义了say属性，这时c1就会在原型链上找到这个属性并执行。
最后，用几句话总结一下本文中涉及到的重点：
原型链的形成真正是靠__proto__ 而非prototype,当JS引擎执行对象的方法时，先查找对象本身是否存在该方法，如果不存在，会在原型链上查找。
一个对象的__proto__记录着自己的原型链，决定了自身的数据类型，改变__proto__就等于改变对象的数据类型。
函数的prototype不属于自身的原型链，它是子类创建的核心，决定了子类的数据类型，是连接子类原型链的桥梁。
在原型对象上定义方法和属性的目的是为了被子类继承和使用。

----------------------------------


