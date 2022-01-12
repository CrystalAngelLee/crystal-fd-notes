# 案例分析

```javascript
function foo () {
  console.log(this)
}
foo.call(1) // => 1
foo() // => 全局对象 / globalThis
window.foo()
foo.call(1) // => 1
```

```javascript
const obj1 = {
  foo: function () {
    console.log(this)
  }
}

obj1.foo() // => obj1
const fn = obj1.foo
fn() // => window
```


```javascript
const obj2 = {
  foo: function () {
    function bar () {
      console.log(this)
    }
    bar()
  }
}

obj2.foo()
```

# 关于 this 的总结

### 普通函数

1. this指向最终最近调用他的对象

2. 在定义的时候不能确认this指向

3. function 调用一般分为以下几种情况：

   1. 作为函数调用，即：`foo()`

      指向全局对象（globalThis），注意严格模式问题，严格模式下是 undefined

   2. 作为方法调用，即：`foo.bar()` / `foo.bar.baz()` / `foo['bar']()` / `foo[0]()`

      指向最终调用这个方法的对象

   3. 作为构造函数调用，即：`new Foo()`

      指向一个新对象 `Foo {}`

   4. 特殊调用，即：`foo.call()` / `foo.apply()` / `foo.bind()`

      参数指定成员

4. 找不到所属的 function，就是全局对象

### 箭头函数

作用域指向当前作用域的上一个对象

在定义的时候可以确认this指向

```js
function fn () {
    let arrFn = () => {
        console.log(this)
    }
    arrFn()
}

const obj = {
    name: 'zs',
    fn: fn
}

obj.fn()   
fn()	  
```



# 尝试一下

```javascript
var length = 10
function fn () {
  console.log(this.length)
}

const obj = {
  length: 5,
  method (fn) {
    // arguments[0] === fn
    fn()   // 10
    arguments[0]()  // 相当于 arguments.fn()
  }
}

obj.method(fn, 1, 2)
```

严格模式下原本应该指向全局的 `this` 都会指向 `undefined`