# let与块级作用域

> [作用域](https://www.zhihu.com/search?q=作用域&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A356670027})： 某个成员能够起作用的范围 ES2015 之前，JS有`全局作用域`，`函数作用域` ES2015 添加了 `块级作用域`：用花括号括起来的范围

块级作用域中声明的内容外部无法调用

- 使用let 声明的变量只能在`块`中使用

```js
if (true) {
 let foo = 'zce'
}
console.log(foo) // foo is not defined
```

- let 修复了变量声明提升现象

```js
console.log(foo) // error
let foo = 'zce'
```

- let 处理[闭包](https://www.zhihu.com/search?q=闭包&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A356670027})问题

```js
var elements = [{}, {}, {}]
for (let i = 0; i < elements.length; i++) {
 elements[i].onclick = function () {
 console.log(i)
  }
}
elements[0].onclick()
```



# const

> 声明[恒量](https://www.zhihu.com/search?q=恒量&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A356670027})/常量
> 在let的基础上多了只读的特性（声明过后不允许再修改）

- 恒量要求声明同时赋值

```js
const name = 'zce'
name = 'jack'
```

- 恒量只是要求内层指向不允许被修改

```js
const obj = {}
obj.name = 'zce'
```



# var、let、const 三种声明变量的方式之间的具体差别

|                | var   | let   | const |
| -------------- | ----- | ----- | ----- |
| 允许重复声明   | true  | false | false |
| 存在变量提升   | true  | false | false |
| 存在暂时性死区 | false | true  | true  |
| 声明后允许改变 | true  | true  | false |
| 声明需要初始值 | false | false | true  |
| 作用域指向     | 全局  | 块级  | 块级  |

**其他说明：**

1. let ｜ const 声明的变量只有在它所在的区域里面有效【形成块级作用域】
2. const 声明的变量不允许改变其指向地址

**举例说明：**

```js
// 1. 允许重复声明
var a = "var A";
var a = "var A+";

let b = "let B";
let b = "let B+"; // 报错

const c = "const C";
const c = "const C+"; // 报错

// 2. 存在变量提升 & 作用域指向
{
  console.log(a_var); // undefined
  var a_var = "var A";
}

{
  console.log(a_let); // 报错-const同理
  let a_let = "let A";
}

console.log(a_var); // var A
console.log(a_let); // a_let is not defined

// 3. 存在暂时性死区
var a = "var A";
if (true) {
  console.log(a); // 报错
  let a = "let A";
}

// 4. 声明后允许改变
{
  const a = 1;
  a = 2; // 报错

  const b = [];
  b.push("constB"); // 允许：因为没有改变内存地址指向
}

// 5. 声明需要初始值
{
  var a;
  let b;
  const c; // 报错
  const d = "const D";
}
```