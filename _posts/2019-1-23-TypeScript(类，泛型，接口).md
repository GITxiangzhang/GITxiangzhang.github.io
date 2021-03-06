
### TypeScript 第二讲

---
# TypeScript 文档分享

# 接口
在TypeScript中，我们使用接口（Interface）来定义对象的类型。
## 什么是接口？
在面向对象语言中，接口可以实现对**类的一些行为的抽象，**对象的形状**】进行描述。

```typescript
// 对象的形状进行描述
interface Family {
	money:number;
  car:string;
  house:boolean;
}

let family:Family = {
	money:Infinity,
  car:'Lamborghini',
  house:true
}
```

**赋值的时候，变量的形状必须和接口的形状保持一致。**
## 可选属性

```typescript
interface Person {
    name: string;
    age?: number;
}

let pony:Person = {
	name:"Pony",
  age:18, // age 可以不赋值, ? 表示为可选属性
}
```

## 任意属性
有时候我们希望一个接口允许有任意的属性，可以使用如下方式：【**但确定属性类型和可选属性类型都必须是它的子类型**】

```typescript
interface Person {
	name: string;
  age?: number;
  [propName: string]: any; // 如果此处改为 string 则为报错
}

let pony: Person = {
	name:"Pony",
  gender:"male",
}
```

## 只读属性
`readonly` 定义只读属性。

```typescript
interface Person {
  readonly id: number;
  name: string;
  age?: number;
  [propName: string]: any;
}

let pony: Person = {
  id: 89757,
  name: 'Pony',
  gender: 'male'
};

pony.id = 9527; // error，不能再赋值。
// 只读的约束存在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候：
```

# 类
传统方法中，JavaScript通过构造函数实现类的概念，通过原型链实现继承，而在ES6中，我们迎来了 `class` 。<br />TypeScript除了实现了所有ES6中的类的功能外，还添加了一些新的用法。

## ES6中类的用法
### 属性和方法
使用 `class` 定义类，使用 `constructor` 定义构造函数。<br />通过 `new` 生成新实例的时候，会自动调用构造函数。

```javascript
class Animal {
	constructor(name){
  	this.name = name;
  }
  sayHi(){
  	return `My name is ${this.name}`;
  }
}

let a = new Animal('Tom');
console.log(a.sayHi)
```

### 类的继承
使用 `extends` 关键字实现继承，子类中使用 `super` 关键字来调用父类的构造函数和方法。

```javascript
class Cat extends Animal {
    constructor(name) {
        super(name); // 调用父类的 constructor(name)
        console.log(this.name);
    }
    sayHi() {
        return 'Meow, ' + super.sayHi(); // 调用父类的 sayHi()
    }
}
let c = new Cat('Tom'); // Tom
console.log(c.sayHi()); // Meow, My name is Tom
```

### 存取器
使用 getter 和 setter 可以改变属性的赋值和读取行为：<br /><br />
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    get name() {
        return 'Jack';
    }
    set name(value) {
        console.log('setter: ' + value);
    }
}
let a = new Animal('Kitty'); // setter: Kitty
a.name = 'Tom'; // setter: Tom
console.log(a.name); // Jack
```

### 静态方法
使用 `static` 修饰符修饰的方法称为静态方法，它们不需要实例化，而是直接通过类来调用：<br /><br />
```javascript
class Animal {
    static isAnimal(a) {
        return a instanceof Animal;
    }
}
let a = new Animal('Jack');
Animal.isAnimal(a); // true
a.isAnimal(a); // TypeError: a.isAnimal is not a function
```

## ES7中类的用法
### 属性和方法
ES6中实例的属性只能通过构造函数中的this.xxx 来定义，**ES7可以直接在类里面定义。**

```javascript
class Animal {
    name = 'Jack';
    constructor() {
        // ...
    }
}
let a = new Animal();
console.log(a.name); // Jack
```

### 静态属性
ES7中可以使用 `static` 定义一个静态属性，不需要实例化，可以直接通过类来调用。

```javascript
class Animal {
    static num = 42;

    constructor() {
        // ...
    }
}

console.log(Animal.num); // 42
```

## TypeScript中类的用法
### ※ 修饰符 public、private和protected
* public 公有的
* private 私有的，不能再声明它的类的外部访问。
* protected 受保护的，但它在子类中也是被允许访问的。

```typescript
// public
class Animal {
	public name;
  public constructor(name){
  	this.name = name
  }
}
let a = new Animal('Tom');
console.log(a.name) // Tom
// => name设置了public,所以直接访问实例的name属性是可行的。

// private
class Animal {
	private name;
  public constructor(name){
  	this.name = name
  }
}
let a = new Animal('Tom');
console.log(a.name) // name属性为私有属性，不允许在外部访问

// => 使用private修饰的属性和方法，在子类中也是不允许访问的
class Animal {
	private name;
  public constructor(name){
  	this.name = name
  }
}
class Cat extends Animal {
	public constructor(name){
  	super(name); // 报错,因为被private修饰的类，子类中也不可以访问
  }
}

// => 而使用protected
class Animal {
	protected name;
  public constructor(name){
  	this.name = name
  }
}
class Cat extends Animal {
	public constructor(name){
  	super(name); // 被protected修饰的类，子类中可以访问
  }
}
let cat = new Cat('Tom')
console.log(cat.name) // 但同样不可以外部访问，会报错。
```

### 抽象类
`abstract` 用于定义抽象类和其中的抽象方法。<br />**什么是抽象类？**<br />首先，抽象类是不允许被实例化的：

```typescript
abstract class Animal {
  public name;
  public constructor(name) {
    this.name = name;
  }
  public abstract sayHi();
}
// 定义一个抽象类，抽象类中有一个抽象方法。
let a = new Animal('Tom') // 报错，因为抽象类不允许实例化

// 抽象类中的方法必须在子类中实现
class Cat extends Animal {
	public eat(){
  	console.log(`${this.name} eat one fish`);
  }
  // 如果没有sayHi方法实现，会报错。
  public sayHi(){
  	this.eat()
  }
}
let c = new Cat();
console.log(c.sayHi())
```

其次， 即使是抽象方法，TypeScript的编译结果中，仍然会存在这个类。

### ※ 类的类型
给类加上TypeScript的类型很简单，与接口类似：

```typescript
class Animal {
	name:string;
  constructor(name:string) {
  	this.name = name;
  }
  sayHi():string{
  	return `My name is ${this.name}`;
  }
}

let a = new Animal('Tom');
console.log(a.sayHi());
```

# 类与接口
接口可以用来对【对象的形状】进行描述。同时也可以对类的一部分行为进行抽象。
## ※ 类实现接口
**把不同类之间一些共有的特性，提取成接口，用 `implements` 关键字来实现，这也大大提高了面向对象的灵活性。**

```typescript
interface Alarm{
	alert()
}
class Door{}
class SecurityDoor extends Door implements Alarm{
	alert(){
  	console.log("我是由接口抽象的行为方法实现")
  }
}
// => 一个类也可以实现不同接口
interface Light{
	lightOn();
  lightOff();
}
class Car implements Alarm,Light{
	alert(){
  	console.log("我是由接口抽象的行为方法实现")
  }
  lightOn(){
  	console.log("接口抽象实现的灯开了")
  }
  lightOff(){
  	console.log("接口抽象实现的灯关了")
  }
}

```

## ※ 接口继承接口
**接口与接口之间可以是继承关系**

```typescript
interface Alarm{
	alert()
}
interface LightAlarm extends Alarm{
	lightOn();
  lightOff();
}
class Car implements LightAlarm{
	alert(){console.log("接口继承抽象实现的alert")}
  lightOn(){console.log("接口抽象实现的lightOn")}
  lightOff(){console.log("接口抽象实现的lightOff")}
}
```

## ※ 接口继承类

```typescript
class Point{
	x:number;
  y:number;
}
interface Point3d extends Point{
	z:number;
}
let point3d:Point3d = {x:1,y:2,z:3}
```

## 混合类型
**用接口的方式定义一个函数需要符合的形状**

```typescript
interface SearchFunc{
	(source:string,subString:string):boolean;
}
let mySearch:SearchFunc = function(source:string,subString:string){
	return source.indexOf(subString)!==-1
}
```

**一个函数还可以有自己的属性和方法，用接口形式展示**

```typescript
interface Counter{
	(start:number):string;
  interval:number;
	reset():void;
}

function getCounter():Counter{
	let count = <Counter>function(start:number){};
  count.interval = 10;
  count.reset = function(){
    count.interval = 0;
    count(0);
  };
  return count;
}

let c = getCounter();
c(10);
c.interval = 12;
c.reset()
```

# 泛型
**泛型是指在定义函数、接口或者类的时候，不预先制定具体的类型，而在使用的时候再指定类型的一种特性。**

## 数组泛型例子

```typescript
function createArray(length:number,value:any):Array<any>{// 数组泛型 Array<any>
  let arr = [];
  let i:number; // 不要在块级作用域里进行定义变量
  for(i= 0;i<length;i++){
  	arr.push(value);
  }
  return arr
}
```

缺点：Array<any>允许数组的每一项都为任意类型，但应该数组的每一项都应该是输入的 `value` 的类型。
## 泛型写法

```typescript
function createArray<T>(length:number,value:T):Array<T>{// 泛型 T
  let arr:T[] = []; // 数组类型：【类型 + 方括号】表示法
  let i:number; // 不要在块级作用域里进行定义变量
  for(i= 0;i<length;i++){
  	arr.push(value);
  }
  return arr
}
// 用法
createArray<string>(3,'x') // ['x','x','x'];
// 也可以不手动指定，而让类型推论自动推算出来
createArray(3,'x')
```

##  多个参数类型
定义泛型的时候，可以一次定义多个类型参数：

```typescript
function swap<T,U>(tuple:[T,U]):[U,T]{
	return [tuple[1],tuple[0]]
}
swap(['x',3]) // => 元组类型的返回[3,'x']
// 元组reverse()方法会报错，更改返回值类型为:any[]
```

## 泛型约束
### 参数自身约束
用接口对泛型进行约束，只允许传入哪些包含特定属性的变量，这就是泛型约束。

```typescript
interface Lengthwise{
	length:number;
}
function getLength<T extends Lengthwise>(arg:T):T{
	console.log(arg.length);
  return arg;
}
// 要求传入参数arg需要包含length属性。
```

### 参数互相约束

```typescript
function copyFileds<T extends U,U>(target:T,source:U):T{
	for(let id in source){
    // 类型断言 <T>source
  	target[id] = (<T>source)[id]
  }
  return target;
}
let x = {a:1,b:2,c:3,d:4};
copyFileds(x,{b:10,d:20});
```

## 泛型接口

```typescript
interface CreateArrayFunc<T> {
    (length: number, value: T): Array<T>;
}
// 注意我们在使用泛型接口的时候，需要定义泛型的类型。
let createArray: CreateArrayFunc<any>;
createArray = function<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}

createArray(3, 'x'); // ['x', 'x', 'x']
```

## 泛型类
泛型也可以用于类的类型定义中：

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

## 泛型参数的默认类型

```typescript
function createArray<T = string>(length:number,value:T):Array<T>{
	let result:T[] = [];
  let i:number;
  for(i=0;i<length;i++){
  	result[i] = value;
  }
  return result;
}
createArray(3,'x');
```
