# 作用域

JavaScript 的作用域通俗来讲，就是指变量能够被访问到的范围，在 JavaScript 中作用域也分为好几种，ES5 之前只有全局作用域和函数作用域两种。ES6 出现之后，又新增了块级作用域。

## 全局作用域

在编程语言中，不论 Java 也好，JavaScript 也罢，变量一般都会分为全局变量和局部变量两种。那么变量定义在函数外部，代码最前面的一般情况下都是全局变量。

在 JavaScript 中，全局变量是挂载在 window 对象下的变量，所以在网页中的任何位置你都可以使用并且访问到这个全局变量。

```js
var globalName = 'global';
function getName() { 
  console.log(globalName) // global
  var name = 'inner'
  console.log(name) // inner
} 
getName();
console.log(name);
console.log(globalName); //global
function setName(){ 
  vName = 'setName';
}
setName();
console.log(vName); // setName
console.log(window.vName) // setName
```

globalName 这个变量无论在什么地方都是可以被访问到的，所以它就是全局变量。而在 getName 函数中作为局部变量的 name 变量是不具备这种能力的。

如果在 JavaScript 中所有没有经过定义，而直接被赋值的变量默认就是一个全局变量，比如上面代码中 setName 函数里面的 vName 变量一样。

我们可以发现全局变量也是拥有全局的作用域，无论你在何处都可以使用它，在浏览器控制台输入 window.vName 的时候，就可以访问到 window 上所有全局变量。

当然全局作用域有相应的缺点，我们定义很多全局变量的时候，会容易引起变量命名的冲突，所以在定义变量的时候应该注意作用域的问题。



## 函数作用域

在 JavaScript 中，函数中定义的变量叫作函数变量，这个时候只能在函数内部才能访问到它，所以它的作用域也就是函数的内部，称为函数作用域。

```js
function getName () {
  var name = 'inner';
  console.log(name); //inner
}
getName();
console.log(name);
```

name 这个变量是在 getName 函数中进行定义的，所以 name 是一个局部的变量，它的作用域就是在 getName 这个函数里边，也称作函数作用域。

除了这个函数内部，其他地方都是不能访问到它的。同时，当这个函数被执行完之后，这个局部变量也相应会被销毁。所以在 getName 函数外面的 name 是访问不到的。



## 块级作用域

ES6 中新增了块级作用域，最直接的表现就是新增的 let 关键词，使用 let 关键词定义的变量只能在块级作用域中被访问，有“暂时性死区”的特点，也就是说这个变量在定义之前是不能被使用的。

```js
console.log(a) //a is not defined
if(true){
  let a = '123'；
  console.log(a)； // 123
}
console.log(a) //a is not defined
```

变量 a 是在 if 语句{...} 中由 let 关键词进行定义的变量，所以它的作用域是 if 语句括号中的那部分，而在外面进行访问 a 变量是会报错的，因为这里不是它的作用域。所以在 if 代码块的前后输出 a 这个变量的结果，控制台会显示 a 并没有定义。



# 函数的执行上下文

**执行上下文（Execution Context）**

> 本质就是一个对象

- 全局执行上下文
- 函数级执行上下文
- eval 执行上下文



**函数执行的阶段**可以分文两个：函数建立阶段、函数执行阶段

- 函数建立阶段

  > 当调用函数时，还没有执行函数内部的代码

  **创建执行上下文对象**

  ```js
  ExecutionContext = {
    variableObject(VO): { a, b, sub, s: undefined, arguments } // 变量对象， 存储 函数中的 arguments、参数、局部成员
    scopeChains:  // 当前函数所在的父级作用域中的活动对象
    this: {}			// 当前函数内部的 this 指向: 函数被调用的时候会确认 this 的指向
  }
  
  function fn (a, b) {
    .....
    function sub () {}
    var s = 56
  }
  fn()
  ```

  

- 函数执行阶段

  ```js
  // 把变量对象转换为活动对象
  ExecutionContext = {
    activationObject(AO):  // 活动对象，函数中的 arguments、参数、局部成员
    scopeChains:  // 当前函数所在的父级作用域中的活动对象
    this: {}			// 当前函数内部的 this 指向
  }
  ```

- [[Scopes]] 作用域链，函数在创建时就会生成该属性，js 引擎才可以访问。这个属性中存储的是所有父级中的变量对象

```js
function fn (a, b) {
  function inner () {
    console.log(a, b)
  }
  console.dir(inner)
  // return inner
}
console.dir(fn)
const f = fn(1, 2)
```



# [闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

可以在另一个作用域中调用一个函数的内部函数并访问到该函数的作用域中的成员

## 概念

> 红宝书闭包的定义：闭包是指有权访问另外一个函数作用域中的变量的函数。
> MDN：一个函数和对其周围状态的引用捆绑在一起（或者说函数被引用包围），这样的组合就是闭包（closure）。也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域。

闭包其实就是一个可以访问其他函数内部变量的**函数**。即一个定义在函数内部的函数，或者直接说闭包是个内嵌函数也可以

通常情况下，函数内部变量是无法在外部访问的（即全局变量和局部变量的区别），因此使用闭包的作用，就具备实现了能在外部访问某个函数内部变量的功能，让这些内部变量的值始终可以保存在内存中。

```js
function fun1() {
	var a = 1;
	return function(){
		console.log(a);
	};
}
fun1();
var result = fun1();
result();  // 1
```

可以很清楚地发现，a 变量作为一个 fun1 函数的内部变量，正常情况下作为函数内的局部变量，是无法被外部访问到的。但是通过闭包，我们最后还是可以拿到 a 变量的值。

**本质**

函数在执行的时候会放到一个执行栈上，当函数执行完毕之后会从执行栈上移除，**但是堆上的作用域成员因为被外部引用不能释放**，因此内部函数依然可以访问外部函数的成员



## 产生的原因

作用域链的基本概念：当访问一个变量时，代码解释器会首先在当前的作用域查找，如果没找到，就去父级作用域去查找，直到找到该变量或者不存在父级作用域中，这样的链路就是作用域链。

需要注意的是，每一个子函数都会拷贝上级的作用域，形成一个作用域的链条

```js
var a = 1;
function fun1() {
  var a = 2
  function fun2() {
    var a = 3;
    console.log(a);//3
  }
}
```

fun1 函数的作用域指向全局作用域（window）和它自己本身；fun2 函数的作用域指向全局作用域 （window）、fun1 和它本身；而作用域是从最底层向上找，直到找到全局作用域 window 为止，如果全局还没有的话就会报错。

闭包产生的本质就是：**当前环境中存在指向父级作用域的引用**。



## 闭包的表现形式

**1**. 返回一个函数。

**2**. 在定时器、事件监听、Ajax 请求、Web Workers 或者任何异步中，只要使用了回调函数，实际上就是在使用闭包。

```js
// 定时器
setTimeout(function handler(){
  console.log('1');
}，1000);
// 事件监听
$('#app').click(function(){
  console.log('Event Listener');
});
```

**3**. 作为函数参数传递的形式。

```js
var a = 1;
function foo(){
  var a = 2;
  function baz(){
    console.log(a);
  }
  bar(baz);
}
function bar(fn){
  // 这就是闭包
  fn();
}
foo();  // 输出2，而不是1
```

**4**. IIFE（立即执行函数），创建了闭包，保存了全局作用域（window）和当前函数的作用域，因此可以输出全局的变量。

```js
var a = 2;
(function IIFE(){
  console.log(a);  // 输出2
})();
```

IIFE 这个函数会稍微有些特殊，算是一种自执行匿名函数，这个匿名函数拥有独立的作用域。这不仅可以避免了外界访问此 IIFE 中的变量，而且又不会污染全局作用域，我们经常能在高级的 JavaScript 编程中看见此类函数。



### 比较常见的开发应用场景

<h5>如何解决循环输出问题？</h5>

```js
for(var i = 1; i <= 5; i ++){
  setTimeout(function() {
    console.log(i)
  }, 0)
}
```

上面这段代码执行之后，从控制台执行的结果可以看出来，结果输出的是 5 个 6，那么一般面试官都会先问为什么都是 6？我想让你实现输出 1、2、3、4、5 的话怎么办呢？

```tex
setTimeout 为宏任务，由于 JS 中单线程 eventLoop 机制，在主线程同步任务执行完后才去执行宏任务，因此循环结束后 setTimeout 中的回调才依次执行。

因为 setTimeout 函数也是一种闭包，往上找它的父级作用域链就是 window，变量 i 为 window 上的全局变量，开始执行 setTimeout 之前变量 i 已经就是 6 了，因此最后输出的连续就都是 6。
```



<h5>如何按顺序依次输出 1、2、3、4、5 呢？</h5>

- **利用 IIFE**

  可以利用 IIFE（立即执行函数），当每次 for 循环时，把此时的变量 i 传递到定时器中，然后执行。

  ```js
  for(var i = 1;i <= 5;i++){
    (function(j){
      setTimeout(function timer(){
        console.log(j)
      }, 0)
    })(i)
  }
  ```

- **使用 ES6 中的 let**

  ```js
  for(let i = 1; i <= 5; i++){
    setTimeout(function() {
      console.log(i);
    },0)
  }
  ```

- **定时器传入第三个参数**

  ```js
  for(var i=1;i<=5;i++){
    setTimeout(function(j) {
      console.log(j)
    }, 0, i)
  }
  ```

  
