### TypeScript 第一讲

---
> 分享内容
1. 介绍TypeScript
2. TypeScript优势
3. TypeScript字符串，参数，函数的新特性


> typescript简介以及优势
1. 微软开发的脚本语言
2. JavaScript的超集，扩展了JavaScript语法,向下兼容javascript代码
3. 遵循ES6语法规则（指出未来一段时间脚本语言的发展方向，es6语法这里不讲，不清楚的私下系统的去学习一下）
4. 谷歌推的angular^2.0框架用TypeScript编写
5. Vue^3.0 今年将发布，也用TypeScript编写
6. 强大的IDE支持
   1. 类型检查，typescript里可以指定变量类型

        ```
         let a:number='hello'
        <!--Type '"ssss"' is not assignable totype 'number'-->
        <!--这个特性会减少开发阶段犯错误的几率-->
   2. 语法提示等提高开发效率

> TypeScript字符串新特性
1. 多行字符串:``

    ```
    let a="aaa"+
          "bbb"+
          "ccc";
    let b=`aaabbbccc`
    <!--多行字符串申明-->
    <!--可以直接换行，在拼接字符串模板的时候很便利-->
    ```
2. 字符串模板

   1. 在多行字符串里插入一个变量或者方法
        ```
        let myName = `max zhang`
        let getMyName = () => myName

        console.log(`hello ${myName}`)
        console.log(`hello ${getMyName()}`)
        console.log(`<span>${myName}<span>
        <label>${getMyName()}</label>
        <a href='${getMyName()}'>hello</a>`)
        ```
3. 自动拆分字符串
   1. 用字符串模板调用一个方法，把字符串模板里的表达式传给方法的参数。第一个参数是字符串模板的值，第二个参数是第一个表达式的值...
        ```
        let test = (temp, name, age) => {
            console.log(temp)
            console.log(name)
            console.log(age)
        }
        let myName = `max`
        let getAge = () => 18
        test`my name is ${myName},i am ${getAge()}`
        -----------------------------------------------------
        Array(3)
        0: "my name is "
        1: ",i am "
        2: ""
        length: 3
        raw: (3) ["my name is ", ",i am ", ""]
        __proto__: Array(0)

        max
        18
       <!-- 我们传进来的字符串模板根据表达式的数量被做了切割成了一个数组-->
        ```
> TypeScript参数的新特性
1. 在参数名称后面使用冒号来指定参数类型
   1. 编译器会做类型判断，但是编译之后的javascript代码是可以执行的。
        ```
        let a: string
        a=1
        <!--编译器会提示变量类型错误-->
        ```
   2. typescript类型推断机制，在变量第一次赋值的时候，自动去推断这个变量的类型
        ```
        let a=2
        a='hello'
        <!--编译器会提示变量类型错误-->

        let a: any <!--any 允许变量为任何类型-->
        ```
   3. 基础类型
        ```
       let isMan: boolean = false;


       <!--支持2进制8进制16进制-->
       let decLiteral: number = 6;
       let hexLiteral: number = 0xf00d;


       let name: string = "bob"


       <!--数组有两种方法定义
       第一种：可以在元素类型后面接上[]，表示由此类型元素组成的一个数组
       第二种：使用数组泛型，Array<元素类型>（泛型后面单独讲到）
       -->
       let list: number[] = [1, 2, 3];
       let list: Array<number> = [1, 2, 3];


       <!--元祖：允许表示一个已知元素数量和类型的数组，各元素的类型不必相同-->
       let x: [string, number];
       x=['aaa',123]
       x=[123,'aaa'] // Error


       <!--枚举：通过enum定义一个枚举-->
       enum size {XL,L,M,S,XS}
       console.log(size)
       <!--Object
        0: "XL"
        1: "L"
        2: "M"
        3: "S"
        4: "XS"
        L: 1
        M: 2
        S: 3
        XL: 0
        XS: 4
        __proto__: Object-->

       <!--默认从0开始为元素编号，也可以手动赋值-->
       enum size {XL=1,L=3,M=5,S=7,XS=9}

       <!--你可以很方便的由枚举的值得到它的名字或者用名字得到它的值-->
        enum size {XL=1,L=3,M=5,S=7,XS=9}
        let sizeName: string = size[3]

       <!--any 不清楚类型的变量指定一个类型,我们不希望类型检查器对这些值进行检查-->


       <!--Void 它表示没有任何类型,没有返值-->

       function test(): void {
            return 'hello'
        }
       //Error


       <!--类型断言:我们需要在还不确定类型的时候就访问其中一个类型的属性或方法的时候-->
       let getLength=(something: string | number): number =>{
         return something.length;
       }
       //Error

       let getLength=(something: string | number): number =>{
         return (<string>something).length;
       }
       <!--或者-->
       let getLength=(something: string | number): number =>{
         return (something as string).length;
       }
      <!-- 当你在TypeScript里使用JSX时，只有 as语法断言是被允许的。-->

        ```
    4. 自定义类型

        ```
        <!--通过class，interface申明自定义类型 例如-->
        class Person {
            name: string;
            age:number
        }

        let max: Person = new Person()
        max.name = 'max'
        max.age = 18

        ```




2. 默认参数
   1. 参数后面用等号指定默认值

        ```
        let a: string = 'aaaa'

        var hello = (a:string,b:string='max') => {
            console.log(a)
            console.log(b)
        }
        hello('hello','will')
        hello('hello')
        <!--默认值必须放在最后-->

        ```
3. 可选参数
   1. 在方法的参数申明后面用?来标明此参数为可选参数

        ```
         var hello = (a:string,b?:string) => {
            console.log(a)
            console.log(b)
        }
        hello('hello')
        <!--开发中注意对可选参数做处理-->
        <!--必选参数不能再可选参数之后-->

        ```


> TypeScript函数新特性
1. **...** 操作符（rest and spread 操作符）
   1. 用来声明任意数量法参数的方法

        ```
        function fun1(...args) {
            args.forEach(i => {
                console.log(i)
            })
        }
        fun1(1,2,3,4,5)

        ```
   2. 把一个任意长度的数组转化方法的参数调用

        ```

        function fun1(a,b,c) {
            console.log(a)
            console.log(b)
            console.log(c)
        }
        let args1 = [1, 2]
        let args2 = [1, 2, 3, 4, 5]
        fun1(...args1)
        fun1(...args2)

        <!--另外的还有数组深拷贝，字符串转数组等用法，本质还是把数组或类数组对象展开成一系列用逗号隔开的值-->

        ```
2. generator函数
   1. 控制函数执行过程，手工暂定和恢复执行

        ```
        <!--通过在function后加 * 来声明一个generator函数-->

        function* doSomeThing() {
            console.log('start')
            yield
            console.log('finish')
        }

        doSomeThing(); //不执行的

        let fun1 = doSomeThing() //必须通过调用generator函数的next()方法 去执行
        fun1.next()
        //停在yield
        fun1.next()

        ```
3. 箭头函数 **=>**
    1. 用来声明匿名函数，消除传统匿名函数this指针问题

        ```
        <!--无参数，返回只有一行无需{}和return-->
        let a = () => 'a'
        等价于
        let a=()=>{
            return 'a'
        }

        <!--只有一个参数-->
        let a = arg => arg

        <!--常用用法-->
        let arr = [1, 2, 3, 4]
        arr = arr.filter(value => value != 1)
        console.log(arr)

        <!--
        （1）函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

        （2）不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。

        （3）不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

        （4）不可以使用yield命令，因此箭头函数不能用作 Generator 函数。
        -->
            function foo1(id) {
              this.id=id
              setTimeout(() => {
                console.log('id:', this.id);
              }, 100);
            }
            function foo2(id) {
              this.id=id
              setTimeout(function(){
                console.log('id:', this.id);
              }, 100);
            }
            var id = 1;
            new foo1(2);
            new foo2(2);
        ```






