---
title: 学习笔记——JS基础
date: 2022-10-19 19:44:51
tags: 编程
---

# 学习笔记——JS基础

> 声明：学习笔记是我自学过程中记录的笔记，这意味着我对这方面的知识没有多少了解，也意味着本文中由不少文字是抄了网络文章及书的，请注意，这不是为初学者准备的JS入门教程，这只是我在学习JS过程中将我认为重要的及我欠缺的部分摘出来，简单地列个导航图。放到个人网站也就是方便自己查看，不喜勿喷，侵权必删。

# JS

Javascript是1995年由Netscape公司设计并实现的，基于对象和事件驱动的松散的解释性语言，被广泛用在web领域，是前端开发中的核心语言，网页中炫酷好康的效果都离不开它。



# 引入JS

在HTML中有4种引入JS的方法

## 在域名或者重定向的位置引入

```html
<!-- 在域名中引入，即在a标签的href属性中引入 -->
<a href="javascript:alert('Hello World!')">链接</a>

<!-- 在重定向中引入，即在表单的action属性中引入 -->
<form action="javascript:alert('Hello World!')">
    <input type="submit" name="" value="">
</form>
```

## 在事件中引入

```html
<div onclick="javascript:alert('Hello World!')">Hello World </div>
```

## 在页面中嵌入

```html
<script>
	//js 代码
</script>	
```

## 引入外部JS文件

```html
<script type="text/javascript" src="xxx.js">
    alert(2)//这里不会被执行
</script>
```



# 变量

## 变量声明

### var

```js
var a;
a = 123;
a = "123";
var x = "abc",y=123,z = function();
```

### let

用法与var类似，但是所声明的变量只在let命令所在的代码块内有效。

```js
let a = 123;
//a作用域开始
if(condition){
    let b = "abc";
    //b作用域开始
    //...
    //b作用域结束
}
//...
//a作用域结束

```

### const

const声明一个只读常量，必须在声明的同时赋值，一旦声明其值就不能再更改。

```js
const PI=3.14;
pi=3.15; //Uncaught TypeError: Assignment to constant variable.
const foo; //Missing initializer in const declaration
```

### 注意

- 变量再没有被赋值的情况下会被自动赋值为undefined，undefined也是一种数据类型，不是错误
- 不使用var或者let声明的变量，如果直接赋值，不会报错，这个变量会被当做window对象的属性存在，拥有全局的作用域，不推荐这种写法
- 如何覆盖已有变量？变量再声明之后是可以重新赋值的，使用var的方式可以重新声明和赋值，使用let则只能重新赋值而不能重新声明

## 数据类型

- 初始类型

  - Number 数值
    - js不区分具体的数值类型，Number包括整型、浮点型、二进制、八进制、十进制、十六进制等
  - String 字符串
    - 单双引号来表示字符串，二者效率一样，只能成对出现，不能相互交叉使用，可以互相嵌套
    - ES6增加了模板字符串，用反引号表示，如`a = ${a},b = ${b}`  
  - Boolean 布尔值
  - undefined 未定义
  
    - 变量未定义时的属性
    - 对象未被赋值的属性、函数没有定义的返回值都会被自动赋值未Undefine
  - null 空对象
  - symbol
  
    - 特殊的、不可变的，表示唯一的值
    - 相同参数的Symbol函数的返回值是不一样的
    - 栗子
```js
//无参数
var s1 = Symbol();
var s2 = Symbol();
s1 === s2; //false

//有参数情况
var s3 = Symbol('fun');
var s4 = Symbol('fun');
s3 === s4; //false

有时我们需要同一个symbol
//Symbol.for
var foo = Symbol('foo');
var s1 = Symbol.for('foo');
var s2 = Symbol.for('foo');
foo === s1; // false
s1 === s2; // true
typeof(foo);//symbol
typeof(s1);//Symbol(foo)
```



- 引用类型（后面介绍）

  - object
  - function

# 运算符

js的运算符和python等高级语言有很多类似共通之处，我也就一笔带过。

## 一目运算符

略

## 二目运算符

### == 与===的区别

==会将左右两个比较数转换为同一类型之后，再对数值比较是否相等，===直接对数值及数据类型比较是否相等

```js
var num = 1;
var num1 = '1';
num==num1;//true
num===num1;//false
```

## 三目运算符

result = condition ? case1: case2



# 流程控制

- if switch
- for while break continue



# 函数

在JS中函数是头灯对象，所有事件的操作都是基于函数的。不仅如此函数还可以像任何其他对象一样具有属性和方法，当然区别在于，函数可以被调用而别的对象不能，所以函数可以看作是特殊的对象。

## 函数声明

函数声明有3中形式

### 通过function关键字声明

```js
function func(args){
	//to do something
}
```

### 通过字面量的方式声明（函数没有名字）

```js
var func = function(args){
	//to do something
}
```

### 通过实例化构造函数的方式声明

```js
var func = new Function(arg1,arg2,arg3,...,'body');
//eg.
//注意参数必须加引号
var sum = new Function('num1','num2','return num1+num2');
```

需要注意，函数体内部语句在执行时，一旦执行至return就执行完毕，且会返回一个值，**如果没有return语句，则会默认返回undefined**，return每次只能返回一个值，如想返回多个值需要将多个值放到数组里再返回。



## 函数调用

js函数可以用以下几种方式调用

### 用()调用

用于声明函数的调用，注意传参有坑

```js
func(args);
```

### 自调用

用于匿名函数的调用，匿名函数还可以通过引用变量来调用

```js
(function(){
    arlert(1);
})();
```

### 通过事件调用

```html
<button onclick="func()">按钮</button>
<script>
    func();
</script>
```



## 参数

- JS传参是浅拷贝的值传递，所以要想在函数中修改参数得传入个数组

- 传参时，实参顺序与形参对应

- 实参和形参不统一时

  - 如果形参的个数大于实参的个数，那么多余的形参会被赋值为undefine

  - 如果实参的个数大于形参的个数，则对函数没有影响，可用arguments对象访问实参

    > arguments对象是函数中自带的一个对象，用于保存所有的实参信息，其形式类似数组，拥有length,callee等属性，它是函数创建时才有，不能显式创建

- js中函数没有重载的概念，如果声明了多个重名的函数，不管函数的形参个数是否一样， 只有最后一个有效，其他的函数声明都是无效的（[参考](https://blog.csdn.net/weixin_34112181/article/details/92281746)）

- 虽然js没有函数重载，但是可以用arguments对象模拟函数重载

  ```js
  function sum(){
      if(arguments.length == 1){
          //...
      }else{
          //...
      }
  }
  ```

## 回调函数

简单来说，回调函数就是作为参数传给另一个函数使用的函数

### 通过函数指针来调用（直接写函数名）

```js
//fu是函数
function math(num1,num2,fu){
    return fu(num1,num2);
};
```

### 把整个函数作为参数传进去

```js
alert(math(2,2,function (num1,num2){
  return num1 * num2;  
}));
```



## 闭包函数

JS允许函数嵌套，内部函数可以访问定义在外部函数中的所有变量和函数（被当作全局），反之不行。

```js
function fn(){
    var num = 100;//在外部函数中定义了变量
    function comp(){//闭包函数
		return num;//内部函数可以访问外部变量
    }
    return comp;//将内部函数返回，从而将其包裹在外部函数的作用域里
}

var func = fn();
var res = func();
console.log(res);//100
```

我们直观地看这段代码可能会认为它无法运行，因为函数fn生命周期已经结束了，变量num不能再被访问，但它实际上能够正常运行，原因在于，comp函数形成了闭包，func是执行fn时创建的comp函数实例的引用，comp的实例维持了一个对它的**词法环境**的引用，变量num存在于此词法环境中，因此，当func被调用时，变量num依旧可用。

> 闭包(closure)是一个函数以及其捆绑的周边环境状态(Lexical environment，词法环境)的引用的组合。[闭包 - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

再看一个MDN上很有意思的例子

```js
//函数工厂
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12

```

闭包的一个实际用处——模拟私有方法

JS对私有变量、方法没有原生支持，但我们可以用闭包来实现这个功能，下面来自MDN的实例展示了如何使用闭包来定义公共函数，并令其可以访问私有函数和变量。这个方式也称为模块模式（module pattern）：

```js
var Counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }
})();

console.log(Counter.value()); /* logs 0 */
Counter.increment();
Counter.increment();
console.log(Counter.value()); /* logs 2 */
Counter.decrement();
console.log(Counter.value()); /* logs 1 */

```

上述例子，privateCounter就是Counter的私有变量，changeBy()就是Counter的私有方法，increment，decrement，value是Counter的公有方法。

闭包使得一些变量一直保存再内存中，这也就难免有负面影响，一方面内存消耗大，另一方面若没有处理好，容易导致内存泄漏，解决方法就是再推出函数之前，删除不适用的局部变量。

闭包的特性

- 函数嵌套函数
- 函数内部可以引用外部的参数和变量
- 参数和变量不会被垃圾回收机制回收，所以使用不当会造成内存泄漏


要想对闭包有更细致的了解，可以访问网站[闭包 - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)



## 内置顶层函数

内置顶层函数就是ECMAScript自带的函数，可以作用于任何对象。

| 函数          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| escape()      | 对字符串进行编码                                             |
| unescape(str) | 对编码的字符串进行解码                                       |
| Number()      | 将任何数据类型转换为数值类型<br />true -> 1  false -> 0<br />null -> 0 undefined -> NaN<br />'' -> 0<br />明显无法转换的类型 -> NaN |
| parseInt()    | 将任何数据类型转换为整数                                     |
| parseFloat()  | 将任何数据类型转换为浮点数                                   |
| String()      | 将任何数据类型转换为字符串<br />null -> 'null'<br />undefined -> 'undefined'<br />true -> 'true' false -> 'false' |
| Boolean()     | 将任何数据类型转换为布尔值<br />"",null,undefined,0,false,NaN -> false<br />others -> true |
| isNaN()       | 判断一个数据能否转换为数值                                   |

## ES6 新增的函数语法

### 参数默认值

ES6允许为函数的参数设置默认值，直接写在参数定义后面

```js
function log(x, y = 'World'){
    console.log(x,y);
}
log('Hello');//Hello World
log('Hello','China')//Hello China
```

### 函数的name属性

函数的name属性就是返回该函数的函数名

```js
function func(){}
console.log(func.name); //"func"
```

### 箭头函数

ES6允许使用箭头(=>)定义函数

```js
var f = function(v){
    return v;
}
//等价于
var f = v => v;
//-------------------
//多个参数
var sum = function(num1,num2){
    return num1 + num2;
}
//等价于
var sum = (num1,num2) => num1 + num2;
//-------------------
//函数体有多条语句
var sum2 = function(num1,num2,num3){
    let tmp = num1 + num2;
    tmp += num3;
    return tmp;
}
//等价于
var sum2 = (num1,num2,num3) => {
 	   let tmp = num1 + num2;
    	tmp += num3;
    	return tmp;
}
//------------------
//若要返回的是json
var person = age => ({name: "Tom", age: age});
//------------------
//配合变量结构一起用
var full = ({name, age}) => name + '-' + age;
console.log(full({name: "Tom", age: 21})); // Tom-21
```

# 数组

JS中数组与其他语言中的数组有较大的差别，JS中的数组元素无需指定数据类型，且大小可变。

## 数组创建

### 通过对象实例化

```js
var arr = new Array();
//创建时赋值
var arr1 = new Array(1,2,3,4);
//先创建再赋值
var arr2 = new Array(3);
arr2[0] = 1;
arr2[1] = 2;
arr2[2] = 3;

//直接赋值
//由于JS数组长度可变，所以在电脑配置足够的情况下，它可以添加任意多的值
var arr3 = new Array();
arr3[0] = 1;
arr3[99] = 100;
```

### 通过JSON隐式创建

```js
var arr = [];
arr[0] = 1;
arr[1] = 2;
var arr1 = [1,2,3,4];
var arr2 = [[1,2,3],[4,5,6],[7,8,9]];
```

## 数组的访问

数组的访问与许多其他高级语言类似，用[]来访问，注意下表从0开始，如果下标范围超出数组定义范围，则返回undefined



## 数组的遍历

### for

效率一般

```js
var arr = [1,2,3,4];
for(let i = 0;i < arr.length;++i){
    arr[i] = arr[i]/2;
}
```

### for in

效率较低，但是很方便

```js
var arr = [1,2,3,4];
for(var i in arr){
	arr[i] = arr[i] / 2;
}
```

### forEach

简便，性能比for低，不兼容IE

```js
var box = [1,2,3,4];
box.forEach((item,index,arr) => {
   arr[index] = item * 2; 
});//2,4,6,8
```

### for of

```js
var arr = [1,2,3,4];
var sum = 0;
for(var item of arr){
	sum += item;
}
```



# 对象

对象是JS中十分重要的概念，是JS中基本的数据类型。一个对象就是一系列相关属性的集合，这里我也只是简单介绍JS中比较特别的地方，更多关于对象和面向对象（OOP）的内容，大家可以自己搜索。

## 对象的创建

### 通过Object方法创建

```js
var stu = new Object();
stu.name = "Tom";
stu.age = 21;
stu.say = function(){
    console.log("Hello!");
}
```

### 通过JSON创建

```js
//方式1
var stu = {};
stu.name = "Tom";
stu.age = 21;
stu.say = function(){
    console.log("Hello!");
}

//方式2
var stu = {
    name: "Tom",
    age: 21,
    say: function(){
    console.log("Hello!");
	}
}
```

### 通过构造函数创建

以上两种创建对象的方法，都只适用于创建单个对象实例，当我们需要创建多个对象实例时，就需要通过构造函数的方法来创建。由于JS中没有类的概念，所以我们要用函数的方式来模拟类，而这个函数就是构造函数

```js
function Person(name, sex, age){
    this.name = name;
    this.sex = sex;
    this.age = age;
    this.say = function(){
    	console.log(`Hello! I'm ${this.name},${this.sex},${this.age} years old.`);
	}
}

var Lily = new Person('Lily',"female",21);
Lily.say();//Hello! I'm Lily,female,21 years old.
var Tom = new Person('Tom',"male",21);
Tom.say();//Hello! I'm Tom,male,21 years old.
```

## 属性与方法

### 添加

```js
obj = new Object();
//对象.名 = 值
obj.member1 = 111;
obj.func1 = function(){}
//对象["名"] = 值
obj["member2"] = 222;
obj["func2"] = function(){}
```

### 访问

```js
//对象.名
obj.member1;
obj.func1();
//对象["名"]
obj["member2"];
obj["func2"]();
```

## 销毁对象与属性

因为JS的垃圾回收机制会自动释放不被引用的对象，所以可以通过如下方式来删除对象。

```js
obj = null;
```

想要删除对象上的属性，则使用delete

```js
delete stu.name;
delete stu.age;
```

## 对象的储存方式

- 变量保存的仅仅是对象的引用地址
- 对象是保存在堆中的，每创建一个对象就开辟一块内存
- 当JS检测到一个对象没有被引用时，就会把他当作垃圾，等待回收
- 在某一时刻回收垃圾对象

## instanceof

instanceof用来检测某个对象是不是某个构造函数的实例

```js
var arr = [];
arr instanceof Array;  // true
arr instanceof Object; // true
```

## 对象的特性

### 封装

封装时将对象的所有组成部分组合起来，尽可能地隐藏对象的部分细节，使其受到保护，只提供有限的接口与外部发生联系

封装方法

- 工厂函数

  ```js
  function person(name,age){
      var person = {};
      person.name = name;
      person.age = age;
      return person;
  }
  var Tom = person("Tom",21);
  ```

- 构造函数

  ```js
  function person(name,age){
      this.name = name;
      this.age = age;
  }
  var Tom = new person("Tom",21);
  ```

- prototype方法

  ```js
  //会把共享的方法或属性放到代码段中来存储，它不能共享对象
  person.prototype.eat = function(){
      alert("eat");
  }
  var Tom = new person("Tom",21);
  Tom.eat();//eat
  ```

- 混合函数
  将构造函数与prototype相结合

  ```js
  function person(name,age){
      this.name = name;
      this.age = age;
  }
  person.prototype.eat = function(){
      alert("eat");
  }
  var Tom = new person("Tom",21);
  Tom.eat();//eat
  ```

### 继承

继承是一个对象拥有另一个对象的属性与方法。所有的JS对象继承于Object对象，并且继承的属性可通过prototype对象找到，JS作为一门轻量级语言，并不原生支持继承，但是可以用以下几种方式达到继承的效果

**继承的方式**

- 通过prototype来继承
  这种方式很简单，只需要让子类的prototype被赋值为父类的一个实例化对象即可，在此之后，就可以直接使用被继承类的方法和属性了

  ```js
  function person(){
      this.name = "Tom";
      this.age = 18;
  }
  function student(){
      
  }
  student.prototype = new person();
  var Tom = new student();
  alert(Tom.name);//Tom
  ```

- call方法

  ```
  fun.call(obj,arg1,arg2,...)
  ```

  

  ```js
  function parent(){
      this.msg = "I'm Parent";
      this.say = function(){
  		console.log(this.msg);
      }
  }
  
  function son(){
      this.msg = "I'm Son";
  }
  
  var p = new parent();
  var s = new son();
  //从本质上来说，call实际上就是改变了函数say的this的指向，
  //现在this指向son
  p.say.call(s);	//I'm Son
  parent.call(s);
  s.say();		//I'm Parent
  ```

  

- apply方法
  apply用法和call类似，只是参数传递有些不同，就不再赘述

  ```js
  fun.apply(obj,[arg1,arg2,...])
  ```

## 对象的分类

- 内置对象：直接使用即可，不需要实例化
- 本地对象：需要实例化后才能使用，如String(),Boolean(),Array()等
- 宿主对象：BOM和DOM



# ES6中对象的新特性

## 类的支持

对象在JS中有很重要的地位，但是它却不支持类？很别扭是吧！ES6添加了对类的支持，引入了关键字class

### 类的定义

```js
class Person{
    //构造函数
    constructor (name,age){
        this.name = name;
        this.age = age;
    }
    //方法
    sayName(){
        console.log(`My name is ${this.name}`);
    }
}
```

### 类的继承

```js
//接上
//类似JAVA，用extends来表示继承
class Teacher extends Person{
    constructor (name,age,job){
        /*调用父类的构造函数
        需要放在this.job=job之前，否则会报错：
        ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor*/
        super(name,age);
        this.job = job;
    }
    sayJob(){
        console.log(`My job is ${this.job}`)
    }
}
var Tom = Teacher('Tom',28,'Teacher');
Tom.sayName();	//My name is Tom
Tom.sayJob();	//My job is Teacher
```

## 变量的解构赋值

ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这称之为解构。

### 数组的解构

```js
//从数组中提取值
let [a,b,c] = [2,5,3];
console.log(a,b,c);//2 5 3
let [num,bar] = ["abc"];
console.log(num,bar);//abc undefined
let [num,bar='efg'] = ['abc'];
console.log(num,bar);//abc efg
```

### 对象的解构赋值

对象的解构和数组的解构的一个重要不同是，数组元素是按照内容顺序依次进行对应位置的赋值，而对象的属性是没有顺序的，变量必须与属性同名才能进行正确的赋值

```js
var {b,a} = {a:"aaa",b:"bbb"};
console.log(a,b);//aaa bbb
var {c} = {a:"aaa",b:"bbb"};
console.log(c);//undefined

var {a:c} = {a:"aaa",b:"bbb"};
console.log(c);//aaa
console.log(a);//a没有被声明
//上例中，a是匹配模式，c才是变量，真正被赋值的变量是c，而不是模式a
```

### 字符串的解构赋值

```js
const [a,b,c] = 'abc';
console.log(a);//a
console.log(b);//b
console.log(c);//c
//字符串'abc'被转换成了类似数组的对象['a','b','c']
```

### 函数参数的解构赋值

```js
function add([x,y]){
    return x+y;
}
add([10,2]); //12
```

## 扩展运算符spread和rest参数

### 扩展运算符

扩展运算符是三个点（...），它用于需要多个参数（函数回调）、多个元素（数组文本）、多个变量（解构分配）等场景

**数组中**

```js
console.log([1,2,3]);//[1,2,3]
console.log(...[1,2,3]);//1 2 3
console.log([1,2,...[3,4,5],6]);//[1,2,3,4,5,6]

//...可以替换concat函数
var arr3 = arr1.concat(arr2);
//等同于
var arr3 = [...arr1,...arr2];
```

**对象中**

```js
let obj1 = {a:1,b:2};
let obj2 = {c:3,b:{e:4,f:5}};
let obj3 = {...obj1,...obj2};
console.log(obj3);//{a:1, b:{e:4,f:5}, c:3}
//注意，对象的扩展运算，是对对象的浅复制，并且是复制对象的可枚举属性
```

### rest参数

ES6中引入了rest参数，形式是（...args），通常我们需要创建一个可变参数长度的函数需要借助arguments对象，现在用rest也可以实现。

```js
function add(...values){
    let sum = 0;
    for(var val of values){
		sum += val;
    }
    return sum;
}
console.log(add(1,2,3));//6
```

# 参考文章

[1] 优逸客科技有限公司.《全栈开发实战宝典》.机械工业出版社

[2] [JS使用call函数实现继承_KDDA的博客-CSDN博客](https://blog.csdn.net/Dream_On_324/article/details/50576106)

[3] [闭包 - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

[4] [【一句话攻略】彻底理解JS中的回调(Callback)函数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/113069353)