# JS 内存管理

> Memory Management

不管是什么样的计算机程序语言，运行在对应的代码引擎上，对应的使用内存过程大致逻辑是一样的，可以分为这三个步骤：

1. 分配你所需要的系统内存空间；
2. 使用分配到的内存进行读或者写等操作；
3. 不需要使用内存时，将其空间释放或者归还。

与其他需要手动管理内存的语言不太一样的地方是，在 JavaScript 中，当我们创建变量（对象，字符串等）的时候，系统会自动给对象分配对应的内存。

🌰 Demo

```js
var a = 123; // 给数值变量分配栈内存
var etf = "ARK"; // 给字符串分配栈内存
// 给对象及其包含的值分配堆内存
var obj = {
  name: "tom",
  age: 13,
};
// 给数组及其包含的值分配内存（就像对象一样）
var a = [1, null, "PSAC"];
// 给函数（可调用的对象）分配内存
function sum(a, b) {
  return a + b;
}
```

当系统经过一段时间发现这些变量不会再被使用的时候，会通过垃圾回收机制的方式来处理掉这些变量所占用的内存，其实开发者不用过多关心内存问题。

对于简单的数据类型，内存是保存在栈（stack）空间中的；复杂数据类型，内存保存在堆（heap）空间中。简而言之，基本就是说明以下两点

- 基本类型：这些类型在内存中会占据固定的内存空间，它们的值都保存在栈空间中，直接可以通过值来访问这些；
- 引用类型：由于引用类型值大小不固定（比如上面的对象可以添加属性等），栈内存中存放地址指向堆内存中的对象，是通过引用来访问的。

**栈内存中的基本类型，可以通过操作系统直接处理；而堆内存中的引用类型，正是由于可以经常变化，大小不固定，因此需要 JavaScript 的引擎通过垃圾回收机制来处理**。

**❤ WHAT 内存管理介绍**

内存：由可读写单元组成，表示一片可操作空间

管理：人为的去操作一片空间的申请、使用和释放

内存管理：开发者主动申请空间、使用空间、释放空间

管理流程：申请-使用-释放

☆ **JS 中的内存管理**

申请内存空间 使用内存空间 释放内存空间

```js
// 申请
let obj = {};
// 使用
obj.name = "obj";
// 释放
obj = null;
```

# [JS 垃圾回收](./JS垃圾回收.md)

# Performance 工具

## 为什么使用 Performance

> 通过 Performance 时刻监控内存

- GC 的目的是为了实现内存空间的良性循环
- 良性循环的基石是合理使用
- 时刻关注才能确定是否合理
- Performance 提供多种监控方式

## 使用步骤

- 打开浏览器输入目标网址
- 进入开发人员工具面板，选择性能
- 开启录制功能，访问具体页面
- 执行用户行为，一段时间后停止录制
- 分析界面中记录的内存信息

## 内存问题的体现

### 外在表现

- 页面出现延迟加载或经常性暂停【频繁垃圾回收】
- 页面持续性出现糟糕的性能【存在内存膨胀】
- 页面的性能随时间延长越来越差【内存泄漏】

### 界定内存问题的标准

- 内存泄漏：内存使用持续升高
- 内存膨胀：在多数设备上都存在性能问题
- 频繁垃圾回收：通过内存变化图进行分析

```tex
内存泄漏是指 JavaScript 中，已经分配堆内存地址的对象由于长时间未释放或者无法释放，造成了长期占用内存，使内存浪费，最终会导致运行的应用响应速度变慢以及最终崩溃的情况。
内存泄漏的场景：
1. 过多的缓存未释放；
2. 闭包太多未释放；
3. 定时器或者回调太多未释放；
4. 太多无效的 DOM 未释放；
5. 全局变量太多未被发现。
```

### 监控内存的几种方式

#### 浏览器任务管理器

- 快捷键：shift + esc
- 内存：原生内存， DOM 节点所占据的内存
- JS 内存：表示 JS 的堆，小括号里面的值表示当前环境下可达对象的使用空间大小，如果小括号里面的值持续增大，说明当前代码有问题

#### Timeline 时序图记录

#### 堆快照查找分离 DOM

- **堆快照留存 JS 堆照片**
- **分离 DOM**
  - 界面元素存活在 DOM 树上
  - 垃圾对象的 DOM 节点（如果当前的节点从当前的 DOM 树上进行了脱离，并且在 JS 代码中没有再次引用的节点，当前 DOM 为垃圾 DOM）
  - 分离状态的 DOM 节点（如果当前的节点只是从 DOM 树上脱离了，但在 JS 中还有引用的地方，当前 DOM 成为分离 DOM， 在界面中看不见，但是在 JS 中占据了内存， 会存在内存泄露）
- **操作**
  - 打开开发人员工具，选择内存（Memory）
  - 给当前操作拍照
  - deta 去查询当前操作下的分离 dom

#### 判断是否存在频繁的垃圾回收

**why**

- GC 工作时应用程序是停止的
- 频繁且过长的 GC 会导致应用假死
- 用户使用中感知应用卡顿

**how**

- Timeline 中频繁的上升下降
- 任务管理器中数据频繁的增加减小

# 内存问题优化

## 如何精准测试 JS 性能

> 本质上就是采集大量的执行样本进行数学统计和分析

使用基于 JSBench: [JavaScript performance benchmarking playground](https://jsbench.me/) 完成

## 慎用全局变量

- 全局变量定义在全局执行上下文，是所有作用域链的顶端（按照层级往上查找的过程，局部作用域中没有找到的变量都会查找到最顶端的全局上下文，在这种情况下，时间消耗比较大，降低了代码的执行效率）
- 全局执行上下文一直存在与上下文执行栈，直到程序退出（对于 GC 工作非常不利）
- 如果某个局部作用于出现了同名变量则会遮蔽或污染全局
- 使用严格模式避免意外创建全局变量

```js
var i,
  str = "";
for (i = 0; i < 1000; i++) {
  str += i;
}

// faster
for (let i = 0; i < 1000; i++) {
  let str = "";
  str += i;
}
```

## 缓存全局变量

将使用中无法避免的全局变量缓存到局部

```js
function getBtn() {
  let oBtn1 = document.getElementById("btn1");
  let oBtn3 = document.getElementById("btn3");
  let oBtn5 = document.getElementById("btn5");
  let oBtn7 = document.getElementById("btn7");
  let oBtn9 = document.getElementById("btn9");
}

// faster
function getBtn() {
  let obj = document;
  let oBtn1 = obj.getElementById("btn1");
  let oBtn3 = obj.getElementById("btn3");
  let oBtn5 = obj.getElementById("btn5");
  let oBtn7 = obj.getElementById("btn7");
  let oBtn9 = obj.getElementById("btn9");
}
```

## 通过原型新增方法

在原型对象上新增实例对象需要的方法

```js
var fn1 = function () {
  this.foo = function () {
    console.log(11111);
  };
};
let f1 = new fn1();

// faster
var fn2 = function () {};
fn2.prototype.foo = function () {
  console.log(11111);
};
let f2 = new fn2();
```

## 避开闭包陷阱

> 闭包: 闭包是一种强大的语法, 使用不当很容易出现内存泄漏, 不要为了闭包而闭包 闭包特点： - 外部具有指向内部的引用 - 在“外”部作用域访问“内部”作用域的数据

```js
// before
function foo() {
  var el = document.getElementById("btn");
  el.onclick = function () {
    console.log(el.id);
  };
}
foo();

// after
function foo() {
  var el = document.getElementById("btn");
  el.onclick = function () {
    console.log(el.id);
  };
  el = null;
}
foo();
```

## 避免属性访问方法使用

> JS 的面向对象 JS 不需要属性的访问方法，所有属性都是外部可见的 使用属性访问方法只会增加一层重定义，没有访问的控制力

```js
function Person() {
  this.name = "icoder";
  this.age = 18;
  this.getAge = function () {
    return this.age;
  };
}
const p1 = new Person();
const a = p1.getAge();

// faster
function Person() {
  this.name = "icoder";
  this.age = 18;
}
const p2 = new Person();
const b = p2.age;
```

## for 循环优化

> use while more better

```js
var arrList = [];
arrList[10000] = "icoder";
for (var i = 0; i < arrList.length; i++) {
  console.log(arrList[i]);
}

// faster
var arrList = [];
arrList[10000] = "icoder";
for (var i = 0, len = arrList.length; i < len; i++) {
  console.log(arrList[i]);
}
```

## 采用最优循环方式

forEach 【faster】

for【second】

for-in【last】

## 节点添加优化

> 节点的添加操作必然会有回流和重绘

```js
for (var i = 0; i < 10; i++) {
  var oP = document.createElement("p");
  oP.innerHTML = i;
  document.body.appendChild(oP);
}

// more better
const fragEle = document.createDocumentFragment();
for (var i = 0; i < 10; i++) {
  var oP = document.createElement("p");
  oP.innerHTML = i;
  fragEle.appendChild(oP);
}

document.body.appendChild(fragEle);
```

## 克隆优化节点操作

```js
for (var i = 0; i < 3; i++) {
  var oP = document.createElement("p");
  oP.innerHTML = i;
  document.body.appendChild(oP);
}

// more better
var oldP = document.getElementById("box1");
for (var i = 0; i < 3; i++) {
  var newP = oldP.cloneNode(false);
  newP.innerHTML = i;
  document.body.appendChild(newP);
}
```

## 直接量替换 Object 操作

```js
var a = [1, 2, 3]; // faster

var a1 = new Array(3);
a1[0] = 1;
a1[1] = 2;
a1[2] = 3;
```

## 减少判断层级

```js
function doSomething(part, chapter) {
  const parts = ["ES2016", "Vue", "react", "node"];
  if (part) {
    if (parts.includes(part)) {
      console.log("属于当前课程");
      if (chapter > 5) {
        console.log("您需要提供VIP");
      }
    }
  } else {
    console.log("请确认模块信息");
  }
}

// faster
function doSomething(part, chapter) {
  const parts = ["ES2016", "Vue", "react", "node"];
  if (!part) {
    console.log("请确认模块信息");
    return;
  }
  if (!parts.includes(part)) return;
  console.log("属于当前课程");
  if (chapter > 5) {
    console.log("您需要提供VIP");
  }
}
```

## 减少作用域链查找层级

```js
var _name = "abc";
function foo() {
  _name = "qwe";
  function func() {
    var age = 18;
    console.log(age);
    console.log(_name);
  }
  func();
}

// quick
var _name = "abc";
function foo() {
  var _name = "qwe";
  function func() {
    var age = 18;
    console.log(age);
    console.log(_name);
  }
  func();
}
```

## 减少数据读取次数

> 将数据做缓存

```js
function hasEle(ele, cls) {
  return ele.className === cls;
}

// faster
function hasEle(ele, cls) {
  var _classname = ele.className;
  return _classname === cls;
}
```

## 减少声明及语句数

```js
// faster
var test = (ele) => {
  return ele.offsetWidth * ele.offsetHeight;
};

var test = (ele) => {
  let w = ele.offsetWidth;
  let h = ele.offsetHeight;
  return w * h;
};
```

## 采用事件委托

```js
item.onclick = showText;

// 推荐
var oul = document.getElementById("ul");
oul.addEventListener("click", showText, true);
```
