---

layout: post
title: Javascript 初探
category: posts

---
Javascript中几个非常重要的语言特性——对象、原型继承、闭包
=======
来自 Google Docs [Javascript 初探](https://docs.google.com/document/d/1S7pSryAT3v9jyhrH4040EEVq9hennaJIg1ArnWXwZtk/edit

## 1. 对象(直接声明、直接赋值、直接使用)
> ### (1). 所有变量都是对象(除了null 和 undefined)
```javascript
var name = 'duan';
var duan = {name: 'duan', email: 'duan@easya.cc'};
//读取
duan.name  //成员(点操作符)
duan['name']  //Hash(中括号操作符)
duan.name = 'balabala'  //重新赋值
duan = {}  //使用对象的字面语法 - {} - 可以创建一个简单对象。这个新创建的对象从Object.prototype 继承下面，没有任何自定义属性
```
> ### (2). function 也是对象
```javascript
function sayHi() {
    // 注意这里的 this, 没有它的话就是局部变量或者局部函数了
    alert("My name is" + this.name + ", my email is" + this.email);
}
sayHi instanceof Object  // => true
duan.satHi = sayHi  //添加成员函数
duan.sayHi()  // => "My name is duan, my email is duan@easya.cc"
```

> ### (3). 删除属性
```javascript
delete duan['name'] //删除对象的属性
duan.hasOwnProperty('name')  // => false 注: 删除属性的唯一方法是使用 delete 操作符；设置属性为 undefined 或者null 并不能真正的删除属性， 而仅仅是移除了属性和值的关联。
```

> ### (4). [属性的配置](https://gist.github.com/kenyonduan/4667de99d945764dfc1d)

> ### (5). this
  >> a. [Github Gist](https://gist.github.com/kenyonduan/87d785da835bf5345465)
  
  >> b. setTimeout 函数：
  setTimeout 会牵涉到 [Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html) 机制，简单来说就是将 js 代码分成了两种: 同步任务（synchronous）、异步任务（asynchronous）。同步任务会在在主线程上执行的任务，依次排队执行，而异步任务会进入执行队列，当任务队列通知主线程后才会进入主线程执行。异步任务中就有 Mouse click、Keypress、Network events、setTimeout 等，javascript 引擎会在 Golbal contxt 下执行这些异步任务。[代码示例](http://jsfiddle.net/dposin/okjr81ev/light/)


> ### (6). 没有 class(ES5)


## 2. 继承
  JavaScript 不包含传统的类继承模型，而是使用 prototype 原型模型。
> ### (1). \_\_proto\_\_
  
  a. 示例
  
  ```javascript
  var array1 = [1,2,3]
  array1.push(4)  //哪里来的这个方法?
  [] instanceof Array  // => true
  array1.__proto__ == Array.prototype	// => true
  Object.getPrototypeOf(array1) === Array.prototype // => true
  ```
  ![image alt text]({{ site.github.url }}/images/Javascript初探/image_0.png)

  b. 当访问对象其中的一个成员或方法的时候，如果这个对象中没有这个方法或成员，那么Javascript引擎将会访问这个对象的__proto__成员所指向的另外的一个对象，并在那个对象中查找指定的方法或成员，如果不能找到，那就会继续通过那个对象的__proto__成员指向的对象进行递归查找，直到这个链表结束(PS: 有点像 Ruby 中的方法查找)
  
  c. 所有的东西都由Object衍生而来, 即所有东西原型链的终点指向
  
  ```javascript
  Object.prototype
  function F() {}
  F.prototype.foobar = function() {}
  var i = new F();
  F.prototype.__proto__ == Object.prototype
  ```
  ![image alt text]({{ site.github.url }}/images/Javascript初探/image_1.png)

> ### (2). Prototype
  prototype 成员
  
  ```javascript
  Array 的 prototype:
  Object.getOwnPropertyNames(Array.prototype)//=> ["length", "constructor", "toString", "toLocaleString", "join", "pop", "push", "concat", "reverse", "shift", "unshift", "slice", "splice", "sort", "filter", "forEach", "some", "every", "map", "indexOf", "lastIndexOf", "reduce", "reduceRight", "entries", "keys"]
  function Base() {
    this.id = ‘base’;
  }
  //输出的格式为: 构造函数名 原型
  Base.prototype  //=> Base {}
  Base.prototype.foobar = function() {alert(1)}
  Base.prototype  //=> Base {foobar: function}
  a = new Base()
	a.prototype.foobar = function() {}  //=> TypeError: Cannot set property 'foobar' of undefined
	a.__proto__ == Base.prototype //=> true
  ```
  ![image alt text]({{ site.github.url }}/images/Javascript初探/image_2.png)
  (1)、当一个函数对象被创建时，这个函数对象就具有一个 prototype 成员，这个成员是一个对象，这个对象包含了一个constructor 构造子成员，这个构造子成员会指向这个函数本身
  (2)、实例"只能"查看 __proto__ 来得知自己是基于什么 prototype 被制造出来的，所以"不能"再重新定义实例的 prototype创造出实例的实例对象
  *  new 操作符(用来调用构造函数)
  
  ```javascript
  var obj = new Base()  //new 操作符干了什么?
  var obj  = {};  //创建了一个空对象obj
  obj.__proto__ = Base.prototype;  //将这个空对象的__proto__成员指向了Base函数对象prototype成员对象
  Base.call(obj);  //改变 this
  ```
  
  ![image alt text]({{ site.github.url }}/images/Javascript初探/image_3.png)
  
  ```javascript
  //这个时候如果我们修改 Base 的 prototype 对象，为它添加一些函数:
  Base.prototype.print = function() {
	   return this.id;
  }
  obj.print()  //=> 'foobar'
  //根据上面提到的 __proto__ 的特性，print 这个方法也可以作为 obj 对象的方法被访问到。
  ```
  
  * 方法重写
  
  ```javascript
  obj.print = function() {return 'balabala'}
  obj.print()  //=> 'balabala'    WHY?
  ```
  *  总结  
  > *  构造子中(一般为大写开头的函数)，我们来设置对象的成员变量（例如：上面的 id）
  > *  构造子对象 prototype 中我们来设置‘类’的公共方法。通过函数对象和 __proto__与 prototype 成员以及 new 操作符，模拟出类和类实例化的效果。
  > *  简单描述:
     a. __proto__ 是用来指向指向另外一个对象
     b. prototype就是对一对象进行扩展，其特点在于通过委托的机制来调用原型对象的方法和访问原型对象的属性


> ### (3). Pseudo classical 继承
  ```javascript
  function Derive(id) {
  this.id = id;
  }
  function Base() {
  this.id = ‘base’;
  }
  Base.prototype.toString = function() {
  	return this.id;
  }
  Derive.prototype = new Base();
  Derive.prototype.constructor = Derive //修Derive.prototype.constructor为Derive本身
  Derive.prototype.test = function(id){
    return this.id === id;
  }
  var newObj = new Derive("derive");
  ```
  ![image alt text]({{ site.github.url }}/images/Javascript初探/image_4.png)
  
  问题 
  为什么是 Derive.prototype = new Base() 而不是 Derive.prototype = Base.prototype ？
  
  ```javascript
  function A(){}
  function B(){}
  A.prototype = B.prototype
  A.prototype.test = function() { return 'A.test'; }
  new A().test()  //=> A.test
  new B().test()  //=> A.test
  // 分割线 //
  A.prototype = new B()
	A.prototype.test = function() { return 'A.test'; }
		new A().test()  //=> A.test
    new B().test()  //=> TypeError: (intermediate value).test is not a function
  //WHY? 要实现继承，就必须保证A继承B以后，A所做的修改不能影响到B以及继承自B的其它对象，A.prototype = new B();这个方法，是创建了一个新的对象{}，并且继承了B的原型，这是一个新对象，不是和B同一引用，所以不会污染B。
  ```
  
  *  为什么要 Derive.prototype.constructor = Derive?
  
  ```javascript
  //正常情况下不错修正也不会有什么影响，但是当你需要显式的使用构造函数的时候就会出现问题。
	function Person(){}
  function Women() {}
  Women.prototype = new Person()
  Women.prototype.constructor  //=> function Person(){}  Why?
  //由于 constructor 始终指向的是创建本身的构造函数，所以 Women.prototype.constructor 指向的是这个 Person 对象的构造函数，也就是 Person 函数，这样一来 Women 的 constructor 函数指向就不对了。
  Women.prototype.constructor = Women
  Women.prototype.constructor  //=> function Women() {}
  var women = new Women()
  //如果在不清楚 women 对象是由哪个函数实例化出来的情况下，但是我想 clone 一个怎么办?当我们重置了 Women 函数的 constructor 后我们可以直接访问 women 对象的 constructor 来拿到构造函数。
  women.constructor  //=> function Women() {}
  var women = new women.constructor
  ```

> ### (4). Prototypal 继承
  ```javascript
  function object(old) {
		function F() {};
    F.prototype = old;
    return new F();
  }
  var base = {
    id:"base",
    toString:function(){
      return this.id;
    }
  }
  var derive = object(base);
  ```
  ![image alt text]({{ site.github.url }}/images/Javascript初探/image_5.png)
> ### (5). Object.create 方法(ES5)
  ```javascript
  function A(){}
  function B() {}
  A.prototype.test = function() { return 'A.test'; }
  A.prototype = Object.create(B.prototype)   //Pseudo classical 继承
  A.prototype.constructor = A  //重置构造函数
  var derive = Object.create(base);  //Prototypal 继承
  ```
> ### (6). 区别
*  Pseudo classical  继承是使用了“构造函数” 和 new 操作符来创建对象，并且使用了构造函数给予的 prototype 属性来构造继承链。所有的实例对象都会继承该 prototype 属性。
*  Prototypal 继承是直接从现有对象来创建新的对象


## 3. 闭包(Closure)
> ### 1. 概念
> *  闭包就是能够读取其他函数内部变量的函数。
> *  当在一个函数内定义另外一个函数就会产生闭包。
> *  闭包就是就是函数的“堆栈”在函数返回后并不释放，可以理解为这些函数堆栈并不在栈上分配而是在堆上分配

> ### 2. 变量的作用域
```javascript
//javascript 区分全局变量和局部变量，函数内部可以直接读取全局变量，而函数外部无法读取函数内部的局部变量
	var n = 1;
	function foobar(){
		return n;
  }
  foobar()  //=> 1
  function foobar2(){
	   var n = 2;  //注意这里的 var，如果不加var 实际声明的是一个全局变量(污染全局命名空间)
   }
   alert(n)  //=>n is not defined
```
> ### 3. 如何读取函数内部的局部变量
```javascript
//在函数内部再定义一个函数并返回它
function outer() {
  var n = 999;
  function inner() {
	   alert(n);
   }
   return inner;  //WHY? 且为什么是 inner 而不是 inner()
 }
 var r = outer();
 r();
 //Javascript语言有一个"链式作用域"结构（chain scope），子对象会一级一级地向上寻找所有父对象的变量。所以，父对象的所有变量，对子对象都是可见的，反之则不成立。既然 inner 可以读取 outer 中的局部变量，那我们就将 inner 返回给 outer 的调用者，那我们就可以在 outer 的外部读取它的内部变量，而 inner 就是闭包。
 //...但是看起来好像和这样写区别不大
 function outer() {
   var n = 999;
   alert(n);
 }
```
> ### 4. 闭包的用处
* ####  可以读取函数内部的变量
* ####  让这些变量的值始终保持在内存中
```javascript
//对上面的 outer 再改造一下看一下闭包的好处在哪里
function outer() {
  var n = 999;
  plus = function() {  //注意这里没有加 var 所以 plus 是一个全局变量，而 plus 的值是一个匿名函数(也是一个闭包)，该函数相当于一个 setter, 可以在函数外部操作函数内部的局部变量
	n += 1;
  }
  function inner() {
	   alert(n);
   }
  return inner;
}
var r = outer();
r();  //=> 999
plus()
r();  //=> 1000
//可以看到函数 outer 中的 n 一直保持在内存中，并没有在 outer 调用完成后被清除。而如果不使用闭包:
function outer() {
  var n = 999;
  plus = function() { n += 1;}
  return n;
}
var r = outer();  // => 999
plus();
outer();  //=> 999
//WHY?
//因为 outer 是 inner 的父函数，而 inner 被赋给了 r 这个局部变量(但是它的作用域在 global)，在 r 没有被回收前 inner 会一直存在内存中，而 inner 的存在依赖于 outer,因此 outer 也会始终存在内存中，不会被垃圾回收机制回收。
```

> ### 5. 使用闭包要注意的地方
*  闭包会使得函数中的变量都被保持在内存中，加大内存消耗，滥用闭包会造成网页的性能问题。解决办法: 用完了及时清除，不管闭包赋予的是全局变量还是局部变量。
*  闭包可以在父函数外部，改变父函数内部变量的值(比如刚才的 plus 函数)，所以需要注意内部私有变量被污染。
```javascript
//再次改造一下上面的 outer 函数
  var outer = (function() { //立即执行函数
  	var n = 999;//私有变量
  	function setN(value) { //公共的 setter
  		n = value;
  	}
  	function printN() {
  		console.log(n)
  	}
  	return {
  		setN: setN,
  		printN: printN
  	};
  })();
```
*  几个容易让人混淆的使用方式

```javascript
function buildList(list) {
var result = [];
for (var i = 0; i < list.length; i++) {
	var item = 'item' + list[i];
	result.push( function() {
//console.log(item)
//console.log(i)
alert(item + ' ' + list[i])}
);
}
return result;
}
function testList() {
	var fnlist = buildList([1,2,3]);
	for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
}
}
testList() //?会弹出什么?
//答案是弹出三次 item3 undefined, 原因是因为 result 里添加的是闭包，这个闭包在调用 buildList([1,2,3]) 的时候是 不会执行的，当 fnlist[j]() 这个时候才会执行，而这个时候 buildList 函数内部的局部变量已经发生了变化(见 console.log 的内容)，可以看到 var item = item3, var i = 3，[1,2,3][3] 取出的就是 undefined。
结论:闭包中局部变量是引用而非拷贝。
//   我是分割线  //
var name = "Outer";
var object = {
	name : "Inner",
	getName: function() {
		return function(){
			return this.name;
}
}
}
object.getName()()  //=> 返回的是什么？
返回的是 Outer:
var getNameFunc = object.getName()  //=>function(){ return this.name }
this //=> Window {top: Window, window: Window, location: Location, external: Object, chrome: Object…}
this.name //=> “Outer”
//   我是分割线  //
var name = "Outer";
var object = {
	name : "Inner",
	getName: function() {
		var that = this;
		//console.log(this)
		return function(){
			return that.name;
}
}
}
object.getName()()  //=> 返回的是什么？
返回的是 Inner:
var getNameFunc = object.getNameFunc()  //=> function (){ return that.name; }
而这个时候 that 是调用者 object
```

> ### 6. 应用场景示例
*  立即调用函数表达式

```javascript
(function(window){
var a = 'foo', b = 'bar';
function private(){
     		alert(a + b);
    	}
	window.jQuery = {
        public: private
};
})(this);  //=>  this-> window
jQuery.public()  //=> foobar
//好处是可以避免污染全局命名空间
```

## 4. 参考
1. [Javascript 面向对象编程](http://coolshell.cn/articles/6441.html)  
2. [再谈javascript面向对象编程](http://coolshell.cn/articles/6668.html)
3. [学习 JavaScript 最难点之一 -- 理解prototype(原型)](http://www.douban.com/note/293217333/)
4. [javascript 秘密花园](http://bonsaiden.github.io/JavaScript-Garden/zh/)
5. [理解Javascript的闭包](http://coolshell.cn/articles/6731.html)
6. [学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
7. [ECMAScript5.1](http://yanhaijing.com/es5/)
8. [ECMAScript 6](http://es6.ruanyifeng.com/)

## 5. 版权申明
如果你在哪里看到和本教程十分类似的文章，不要怀疑，就是我抄了他们的作业。


Kenyon
