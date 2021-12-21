# Promise

提供了一种全新的异步编程解决方案，通过链式调用的方式，解决了我们在传统异步编程中回调函数嵌套过深的问题

## 出现的原因

promise 的出现是为了解决回调地狱的问题的



## 概述

简单来说 Promise 就是一个容器，里面保存着某个未来才会结束的事件（通常是异步操作）的结果。

从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。

### 两个特点

1. 对象的状态不受外界影响
   `Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果

   `Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同。事件的特点是：如果你错过了它，再去监听，是得不到结果的。

### 三个缺点

1. 无法取消`Promise`，一旦新建它就会立即执行，无法中途取消。
2. 如果不设置回调函数，`Promise`内部抛出的错误，不会反应到外部。
3. 当处于`pending`状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

### Promise 如何解决回调地狱

首先，回调地狱的两个主要问题是：

1. 多层嵌套的问题；
2. 每种任务的处理结果存在两种可能性（成功或失败），那么需要在每种任务执行结束后分别处理这两种可能性。

Promise 利用了三大技术手段来解决回调地狱：**回调函数延迟绑定、返回值穿透、错误冒泡**。

*回调函数延迟绑定* 就是通过 then 方法传入回调函数

*返回值穿透* 根据 then 回调函数的传入值创建不同类型的 promise, 然后把返回的 Promise 穿透到外层

*错误冒泡*  then 前面产生的错误会一直向后传递，被 catch 接收到进行处理



## 用法

### 基本用法

> promise.then：回调函数延迟绑定

**返回resolve**

```js
const promise = new Promise((resolve, reject) => {
  resolve(100)
})

promise.then((value) => {
  console.log('resolved', value) // resolve 100
},(error) => {
  console.log('rejected', error)
})
```

**返回reject**

```js
const promise = new Promise((resolve, reject) => {
  reject(new Error('promise rejected'))
})

promise.then((value) => {
  console.log('resolved', value)
},(error) => {
  console.log('rejected', error)
  // rejected Error: promise rejected
  //  at E:\professer\lagou\Promise\promise-example.js:4:10
  //  at new Promise (<anonymous>)
})
```



### 链式调用

*常见误区**：*** 嵌套使用的方式是使用Promise最常见的误区。要使用promise的链式调用的方法尽可能保证异步任务的扁平化。

***链式调用的理解***：

1. promise对象then方法，返回了全新的promise对象。可以再继续调用then方法，如果return的不是promise对象，而是一个值，那么这个值会作为resolve的值传递，如果没有值，默认是undefined
2. 后面的then方法就是在为上一个then返回的Promise注册回调
3. 前面then方法中回调函数的返回值会作为后面then方法回调的参数
4. 如果回调中返回的是Promise，那后面then方法的回调会等待它的结束



### 异常处理

> then中回调的onRejected方法
> .catch()（推荐）

promise中如果有异常，都会调用reject方法，还可以使用.catch(); 使用.catch方法更为常见，因为更加符合链式调用

**.catch形式和前面then里面的第二个参数的形式，两者异常捕获的区别：**

.catch()是对上一个.then()返回的promise进行处理，不过第一个promise的报错也顺延到了catch中，而then的第二个参数形式，只能捕获第一个promise的报错，如果当前then的resolve函数处理中有报错是捕获不到的。

**所以.catch是给整个promise链条注册的一个失败回调。推荐使用**

```js
ajax('/api/user.json')
  .then(function onFulfilled(res) {
    console.log('onFulfilled', res)
  }).catch(function onRejected(error) {
    console.log('onRejected', error)
  })
  
// 相当于
ajax('/api/user.json')
  .then(function onFulfilled(res) {
    console.log('onFulfilled', res)
  })
  .then(undefined, function onRejected(error) {
    console.log('onRejected', error)
  })
```



**全局对象上的unhandledrejection事件**

> 在全局对象上注册一个unhandledrejection事件，处理那些代码中没有被手动捕获的promise异常，当然**并不推荐使用**

更合理的是：在代码中明确捕获每一个可能的异常，而不是丢给全局处理

```js
// 浏览器
window.addEventListener('unhandledrejection', event => {
  const { reason, promise } = event
  console.log(reason, promise)

  //reason => Promise 失败原因，一般是一个错误对象
  //promise => 出现异常的Promise对象

  event.preventDefault()
}, false)

// node
process.on('unhandledRejection', (reason, promise) => {
  console.log(reason, promise)

  //reason => Promise 失败原因，一般是一个错误对象
  //promise => 出现异常的Promise对象
})
```



### 静态方法

#### Promise.resolve()

快速的将一个值转换为 Promise 对象

- 接收一个常量将其转换为 Promise 对象
- 接收一个 Promise 对象将返回自身
- 接收一个带有 then 方法的对象（then 方法类似于 Promise的 then 方法），这个对象也可以作为一个 Promise 被执行

#### Promise.reject()

快速创建一个一定是失败的 Promise 对象

#### Promise.all()

语法： `Promise.all（iterable）`

参数： 一个可迭代对象，如 Array。

描述： 此方法对于汇总多个 promise 的结果很有用，在 ES6 中可以将多个 Promise.all 异步请求并行操作，返回结果一般有下面两种情况。

当所有结果成功返回时按照请求顺序返回成功。

当其中有一个失败方法时，则进入失败方法。

#### Promise.allSettled()

Promise.allSettled 的语法及参数跟 Promise.all 类似，其参数接受一个 Promise 的数组，返回一个新的 Promise。唯一的不同在于，执行完之后不会失败，也就是说当 Promise.allSettled 全部处理完成后，我们可以拿到每个 Promise 的状态，而不管其是否处理成功。

```js
const resolved = Promise.resolve(2);
const rejected = Promise.reject(-1);
const allSettledPromise = Promise.allSettled([resolved, rejected]);
allSettledPromise.then(function (results) {
  console.log(results);
});
// 返回结果：
// [
//    { status: 'fulfilled', value: 2 },
//    { status: 'rejected', reason: -1 }
// ]
```

#### Promise.any()

**语法：** Promise.any（iterable）

**参数：** iterable 可迭代的对象，例如 Array。

**描述：** any 方法返回一个 Promise，只要参数 Promise 实例有一个变成 fulfilled 状态，最后 any 返回的实例就会变成 fulfilled 状态；如果所有参数 Promise 实例都变成 rejected 状态，包装实例就会变成 rejected 状态。

```js
const resolved = Promise.resolve(2);
const rejected = Promise.reject(-1);
const anyPromise = Promise.any([resolved, rejected]);
anyPromise.then(function (results) {
  console.log(results);
});
// 返回结果：
// 2
```

#### Promise.race()

**语法：** Promise.race（iterable）

**参数：** iterable 可迭代的对象，例如 Array。

**描述：** race 方法返回一个 Promise，只要参数的 Promise 之中有一个实例率先改变状态，则 race 方法的返回状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给 race 方法的回调函数。

我们来看一下这个业务场景，对于图片的加载，特别适合用 race 方法来解决，将图片请求和超时判断放到一起，用 race 来实现图片的超时判断。

```js
//请求某个图片资源
function requestImg(){
  var p = new Promise(function(resolve, reject){
    var img = new Image();
    img.onload = function(){ resolve(img); }
    img.src = 'http://www.baidu.com/img/flexible/logo/pc/result.png';
  });
  return p;
}
//延时函数，用于给请求计时
function timeout(){
  var p = new Promise(function(resolve, reject){
    setTimeout(function(){ reject('图片请求超时'); }, 5000);
  });
  return p;
}
Promise.race([requestImg(), timeout()])
.then(function(results){
  console.log(results);
})
.catch(function(reason){
  console.log(reason);
});
```

从上面的代码中可以看出，采用 Promise 的方式来判断图片是否加载成功，也是针对 Promise.race 方法的一个比较好的业务场景。



### 执行时序：宏任务/微任务

回调队列中的任务称为 **宏任务**，宏任务执行过程中可以临时加上一些额外需求，这些额外的需求可以选择作为一个新的宏任务进到队列中排队，也可以作为当前任务的 **微任务（*直接在当前任务执行过后立即执行*）**，Promise 的回调会作为微任务进行执行

*微任务就是为了提高整体的响应能力*

**举个栗子**

> 有一个大爷去银行存款，排队等到他去存款的时候突然想要开通手机银行，这时候银行柜员会优先完成大爷的需求再开始其他人的业务。
> 大爷去存款：宏任务
> 大爷要开通手机银行： 微任务

常见的宏任务： setTimeout, setInterval..

常见的微任务： promise.then, process.nextTick、MutationObserver



# 手写 Promise

## Promise/A+ 规范

> 官方地址：[Promises/A+](https://link.zhihu.com/?target=https%3A//promisesaplus.com/)

以下为摘录的一些关键部分

### Terminology 【术语】

1. “promise” is an object or function with a `then` method whose behavior conforms to this specification. 【promise是一个具有then 方法的对象或者方法，他的行为符合该规范】
2. “thenable” is an object or function that defines a `then` method.【thenable是一个定义了then方法的对象或者方法】
3. “value” is any legal JavaScript value (including `undefined`, a thenable, or a promise).【value是任何一个合法的JS值（包括 undefined、thenable 或 promise）】
4. “exception” is a value that is thrown using the `throw` statement.【exception是一个被throw语句抛出的值-异常】
5. “reason” is a value that indicates why a promise was rejected.【reason是promise 里reject之后返回的原因】

### Promise States 【Promise状态】

A promise must be in one of three states: pending, fulfilled, or rejected.【一个 Promise 有三种状态：pending、fulfilled 和 rejected】

1. When pending, a promise:

2. 1. may transition to either the fulfilled or rejected state.
   2. 【当状态为 pending 状态时，即可以转换为 fulfilled 或者 rejected 其中之一】

3. When fulfilled, a promise:

4. 1. must not transition to any other state.
   2. must have a value, which must not change.
   3. 【当状态为 fulfilled 状态时，就不能转换为其他状态了，必须返回一个不能再改变的值】

5. When rejected, a promise:

6. 1. must not transition to any other state.
   2. must have a reason, which must not change.
   3. 【当状态为 rejected 状态时，同样也不能转换为其他状态，必须有一个原因的值也不能改变】

Here, “must not change” means immutable identity (i.e. `===`), but does not imply deep immutability.

### The `then` Method【then 方法】

A promise must provide a `then` method to access its current or eventual value or reason.【一个 Promise 必须拥有一个 then 方法来访问它的值或者拒绝原因。】

A promise’s `then` method accepts two arguments:【then 方法有两个参数】

> promise.then(onFulfilled, onRejected)

1. Both `onFulfilled` and `onRejected` are optional arguments:

2. 1. If `onFulfilled` is not a function, it must be ignored.
   2. If `onRejected` is not a function, it must be ignored.
   3. 【onFulfilled 和 onRejected 都是可选参数。】

3. If `onFulfilled` is a function:

4. 1. it must be called after `promise` is fulfilled, with `promise`’s value as its first argument.
   2. it must not be called before `promise` is fulfilled.
   3. it must not be called more than once.
   4. 【如果 onFulfilled 是函数，则当 Promise 执行结束之后必须被调用，最终返回值为 value，其调用次数不可超过一次】

5. If `onRejected` is a function,

6. 1. it must be called after `promise` is rejected, with `promise`’s reason as its first argument.
   2. it must not be called before `promise` is rejected.
   3. it must not be called more than once.
   4. 【如果 onFulfilled 是函数，则当 Promise 执行结束之后必须被调用，最终返回reason，其调用次数不可超过一次】

7. `then` may be called multiple times on the same promise.

8. 1. If/when `promise` is fulfilled, all respective `onFulfilled` callbacks must execute in the order of their originating calls to `then`.
   2. If/when `promise` is rejected, all respective `onRejected` callbacks must execute in the order of their originating calls to `then`.
   3. 【then方法可以被一个Promise多次调用】

9. `then` must return a promise [[3.3](https://link.zhihu.com/?target=https%3A//promisesaplus.com/%23notes)].
   promise2 = promise1.then(onFulfilled, onRejected);

10. 1. If either `onFulfilled` or `onRejected` returns a value `x`, run the Promise Resolution Procedure `[[Resolve]](promise2, x)`.
    2. If either `onFulfilled` or `onRejected` throws an exception `e`, `promise2` must be rejected with `e` as the reason.
    3. If `onFulfilled` is not a function and `promise1` is fulfilled, `promise2` must be fulfilled with the same value as `promise1`.
    4. If `onRejected` is not a function and `promise1` is rejected, `promise2` must be rejected with the same reason as `promise1`.
    5. 【then方法必须返回一个promise】



## Promise实现

首先分析其原理

> 1. promise 就是一个类： 在执行类的时候需要传递一个执行器，执行器会立即执行
> 2. promise 的三种状态： pending, fulfilled, or rejected
> 3. resolve 和 reject 方法用来更改状态
> resolve -> fulfilled / reject -> rejected
> 4. then 方法判断状态：
> 成功状态 -> 调用成功回调， 失败状态 -> 调用失败回调

### 1. 完成一个最简单的Promise

```js
// 三种状态
const PENDING = 'pending', FULFILLED = 'fulfilled', REJECTED = 'rejected'
class MyPromise {
  constructor(excutor) {
    excutor(this.resolve, this.reject);
  }

  status = PENDING;         // 当前的状态
  value = undefined;        // 成功的值
  reason = undefined;       // 失败的值

  resolve = (value) => {
    // must not transition to any other state
    if (this.status !== PENDING) return
    this.status = FULFILLED;
    this.value = value
  }

  reject = (reason) => {
    // must not transition to any other state
    if (this.status !== PENDING) return
    this.status = REJECTED;
    this.reason = reason
  }

  then(onFulfilled, onRejected) {
    switch (this.status) {
      case FULFILLED:
        onFulfilled(this.value);
        break;
      case REJECTED:
        onRejected(this.reason);
        break;
    }
  }
}
```

test

```js
const Promise = require('./index');

const res_1 = new Promise(function(resolve, reject) {
  // resolve('最简单的Promise:resolve');
  reject('最简单的Promise:reject')
})
res_1.then(res => {
  console.log('success', res)
}, err => {
  console.log('rejected', err)
})
```



### 2. 异步Promise

第一种情况下没有处理到异步的情况；异步情况的处理其实就是then方法中对应pending状态的处理，如下所示：

```js
class MyPromise {
  // 添加相应值
  onFulfilled = undefined;    // 存储成功后的回调
  onRejected = undefined;     // 存储失败后的回调

  resolve = (value) => {
    // must not transition to any other state
    if (this.status !== PENDING) return
    this.status = FULFILLED;
    this.value = value;
    // 成功之后立即执行缓存中的回调
    this.onFulfilled && this.onFulfilled(this.value)
  }

  reject = (reason) => {
    // must not transition to any other state
    if (this.status !== PENDING) return
    this.status = REJECTED;
    this.reason = reason
    this.onRejected && this.onRejected(this.reason)
  }

  then(onFulfilled, onRejected) {
    switch (this.status) {
      case FULFILLED:
        onFulfilled(this.value);
        break;
      case REJECTED:
        onRejected(this.reason);
        break;
      default: // pending状态
        this.onFulfilled = onFulfilled
        this.onRejected = onRejected
    }
  }
}
```

test

```js
const res_2 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    // resolve('异步Promise:resolve');
    reject('异步Promise:reject')
  }, 100)
})
res_2.then(res => {
  console.log('success', res)
}, err => {
  console.log('rejected', err)
})
```



### 3. then 方法多次调用添加多个处理函数

我们来看以下的使用：

![img](https://pic1.zhimg.com/80/v2-68292833713ecaadc2be90e6e0de7944_1440w.jpg)

通过上述的这种调用方式【多次调用】我们可以看到右侧的结果显示，当同步Promise的情况下，输出正常，异步Promise下只会处理第一个then，这是不对的，现在我们开始完善以上使用方式

```js
class MyPromise {
  // 修改回调为数组形式
  onFulfilled = [];         // 存储成功后的回调
  onRejected = [];          // 存储失败后的回调

  resolve = (value) => {
    ...
    // 循环取出调用成功方法
    while(this.onFulfilled.length) this.onFulfilled.shift()(this.value)
  }

  reject = (reason) => {
    ...
    while(this.onRejected.length) this.onRejected.shift()(this.reason)
  }

  then(onFulfilled, onRejected) {
    switch (this.status) {
      case FULFILLED:
        onFulfilled(this.value);
        break;
      case REJECTED:
        onRejected(this.reason);
        break;
      default: // pending状态
        this.onFulfilled.push(onFulfilled)
        this.onRejected.push(onRejected)
    }
  }
}
```

test

```js
const res_3 = new Promise(function(resolve, reject) {
  // resolve('同步Promise:resolve');
  // reject('同步Promise:reject')
  setTimeout(() => {
    // resolve('异步Promise:resolve');
    reject('异步Promise:reject')
  }, 100)
})

res_3.then(val => {
  console.log('one', val)
}, err => {
  console.log('one', err)
})

res_3.then(val => {
  console.log('two', val)
}, err => {
  console.log('two', err)
})
```



### 4. promise 的链式调用

可以链式调用的核心就是then 返回一个promise

```js
class MyPromise {
  ...
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      let func = undefined;
      switch (this.status) {
        case FULFILLED:
          func = onFulfilled(this.value);
          resolvePromise(func, resolve, reject)
          break;
        case REJECTED:
          func = onRejected(this.reason);
          resolvePromise(func, resolve, reject)
          break;
        default: // pending状态
          this.onFulfilled.push(() => {
            func = onFulfilled(this.value)
            resolvePromise(func, resolve, reject)
          })
          this.onRejected.push(() => {
            func = onRejected(this.reason)
            resolvePromise(func, resolve, reject)
          })
      }
    })
  }
}

function resolvePromise(func, resolve, reject) {
  // 返回promise
  if (func instanceof MyPromise) {
    func.then(resolve, reject)
  } else {
    resolve(func)
  }
}
```

test

```js
const res_4 = new Promise(function(resolve, reject) {
  // resolve('同步Promise:resolve');
  // reject('同步Promise:reject')
  setTimeout(() => {
    // resolve('异步Promise:resolve');
    reject('异步Promise:reject')
  }, 100)
})
const res_4_test = new Promise(function(resolve, reject) {
  // resolve('testPromise:resolve');
  // // reject('testPromise:reject')
  setTimeout(() => {
    resolve('test异步Promise:resolve');
    // reject('test异步Promise:reject')
  }, 100)
})

res_4.then(val => {
  return 'first'
  // return res_4_test
}, err => {
  return 'first-err'
}).then(val => {
  console.log('two', val)
}, err => {
  console.log('two', err)
})
```



### 5. 识别对象自返回

链式调用下，如果返回自身，会出现死循环，出现卡死现象，所以我们是不允许这种现象出现的，以下，我们来处理这种情况：

```js
  // 处理then 方法
  then(onFulfilled, onRejected) {
    const promise = setTimeout(() => new MyPromise((resolve, reject) => {
      let func = undefined;
      switch (this.status) {
        case FULFILLED:
          func = onFulfilled(this.value);
          resolvePromise(promise, func, resolve, reject)
          break;
        case REJECTED:
          func = onRejected(this.reason);
          resolvePromise(promise, func, resolve, reject)
          break;
        default: // pending状态
          this.onFulfilled.push(() => {
            func = onFulfilled(this.value)
            resolvePromise(promise, func, resolve, reject)
          })
          this.onRejected.push(() => {
            func = onRejected(this.reason)
            resolvePromise(promise, func, resolve, reject)
          })
      }
    }), 0)

    return promise
  }

// 处理resolvePromise
function resolvePromise(promise, func, resolve, reject) {
  // 识别对象自返回
  if (func === promise) {
    return reject(
      new TypeError('Chaining cycle detected for promise #<Promise>')
    )
  }
  // 返回promise
  if (func instanceof MyPromise) {
    func.then(resolve, reject)
  } else {
    resolve(func)
  }
}
```

test

```js
const res_5 = new Promise(function(resolve, reject) {
  resolve('同步Promise:resolve');
  // reject('同步Promise:reject')
  // setTimeout(() => {
  //   // resolve('异步Promise:resolve');
  //   reject('异步Promise:reject')
  // }, 100)
})

const p = res_5.then((res) => {
  console.log(res);
  return p;
});
```

setTimeout: 是为了在MyPromise中获取到声明的对象，以便后续的比较



### 6. 错误捕获：try catch

这一步我们将所有可能发生异常的部分try catch起来，健壮代码

```js
class MyPromise {
  constructor(excutor) {
    try {
      excutor(this.resolve, this.reject);
    } catch (e) {
      this.reject(e)
    }
  }
  ...
  then(onFulfilled = val => val, onRejected = err => {throw err}) {
    const promise = new MyPromise((resolve, reject) => {
      let func = undefined;
      switch (this.status) {
        case FULFILLED:
          setTimeout(() => {
            try {
              func = onFulfilled(this.value);
              resolvePromise(promise, func, resolve, reject)
            } catch (e) {}
          }, 0)
          break;
        case REJECTED:
          setTimeout(() => {
            try {
              func = onRejected(this.reason);
              resolvePromise(promise, func, resolve, reject)
            } catch (e) {}
          }, 0)
          break;
        default: // pending状态
          this.onFulfilled.push(() => {
              setTimeout(() => {
                try {
                  func = onFulfilled(this.value)
                  resolvePromise(promise, func, resolve, reject)
                } catch (e) {}
              }, 0)
            })
          this.onRejected.push(() => {
            setTimeout(() => {
              try {
                func = onRejected(this.reason)
                resolvePromise(promise, func, resolve, reject)
              } catch (e) {}
            }, 0)
          })
      }
    })

    return promise
  }
}
```

test

```js
const res_6 = new Promise(function(resolve, reject) {
  // resolve(new TypeError('哈哈，你错了'));
  setTimeout(() => {
    resolve(new TypeError('哈哈，异步你也错了'));
  }, 100)
})
res_6.then((res) => {
  console.log(res);
});
```



### 7. promise.all 方法实现

```js
  static all (array){
    let res = [];
    return new MyPromise((resolve, reject) => {
      const addData = (key, val) => {
        res[key] = val;
        if (res.length === array.length) {
          resolve(res)
        }
      }
      for (let i = 0, len = array.length; i < len; i++) {
        let cur = array[i];
        if (cur instanceof MyPromise) {
          cur.then((val) => addData(i, val), err => reject(err))
        } else {
          addData(i, cur)
        }
      }
    })
  } 
```

test

```js
const test_1 = new Promise((resolve) => {resolve('test_1')})
const test_2 = new Promise((resolve, reject) => {resolve('test_2')})
const test_3 = new Promise((resolve, reject) => {setTimeout(() => {resolve('test_3')}, 100)})
const test_4 = new Promise((resolve, reject) => {setTimeout(() => {reject('test_4')}, 100)})
Promise.all(['1', test_1, test_2, test_3, '2', test_4]).then(res => {
  console.log("all",res)
}, err => {
  console.log('all-err', err)
})
```



### 8. promise.resolve 方法实现

```js
  static resolve(val) {
    if (val instanceof MyPromise) return val;
    return new MyPromise(resolve => resolve(val))
  }
```

test

```js
Promise.resolve('res_8').then((value) => {
  console.log(value)
})
```



### 9. catch finally 方法实现

```js
  catch(onRejected) {
    return this.then(undefined, onRejected)
  }

  finally(cb) {
    this.then(val => MyPromise.resolve(cb()).then(() => val), err => MyPromise.resolve(cb()).then(() => {throw err}))
  }
```

test

```js
const res_9 = new Promise(function(resolve, reject) {
  resolve(new TypeError('哈哈，你错了'));
  setTimeout(() => {
    resolve(new TypeError('哈哈，异步你也错了'));
  }, 100)
})
res_9.then(val => {
  console.log('9:', val)
}).catch(err => {
  console.log('9-err', err)
}).finally(() => {
  console.log('finally')
})
```



## [完整版](https://github.com/CrystalAngelLee/crystal-js-handwritting/blob/main/promise/promise.js)

```js
/* 手写promise实现 */
/**
 * 1. 完成一个最简单的Promise
 * 2. 异步Promise
 * 3. then 方法多次调用添加多个处理函数
 * 4. promise 的链式调用
 * 5. 识别对象自返回
 * 6. 错误捕获
 * 7. promise.all 方法实现
 * 8. promise.resolve 方法实现
 * 9. finally 方法实现
 */
// 三种状态
const PENDING = 'pending', FULFILLED = 'fulfilled', REJECTED = 'rejected'
class MyPromise {
  constructor(excutor) {
    try {
      excutor(this.resolve, this.reject);
    } catch (e) {
      this.reject(e)
    }
  }

  status = PENDING;         // 当前的状态
  value = undefined;        // 成功的值
  reason = undefined;       // 失败的值
  onFulfilled = [];         // 存储成功后的回调
  onRejected = [];          // 存储失败后的回调

  resolve = (value) => {
    // must not transition to any other state
    if (this.status !== PENDING) return
    this.status = FULFILLED;
    this.value = value;
    while(this.onFulfilled.length) this.onFulfilled.shift()()
  }

  reject = (reason) => {
    // must not transition to any other state
    if (this.status !== PENDING) return
    this.status = REJECTED;
    this.reason = reason
    while(this.onRejected.length) this.onRejected.shift()()
  }

  then(onFulfilled = val => val, onRejected = err => {throw err}) {
    const promise = new MyPromise((resolve, reject) => {
      let func = undefined;
      switch (this.status) {
        case FULFILLED:
          setTimeout(() => {
            try {
              func = onFulfilled(this.value);
              resolvePromise(promise, func, resolve, reject)
            } catch (e) {
              reject(e);
            }
          }, 0)
          break;
        case REJECTED:
          setTimeout(() => {
            try {
              func = onRejected(this.reason);
              resolvePromise(promise, func, resolve, reject)
            } catch (e) {
              reject(e);
            }
          }, 0)
          break;
        default: // pending状态
          this.onFulfilled.push(() => {
            setTimeout(() => {
              try {
                func = onFulfilled(this.value)
                resolvePromise(promise, func, resolve, reject)
              } catch (e) {
                reject(e);
              }
            }, 0)
          })
          this.onRejected.push(() => {
            setTimeout(() => {
              try {
                func = onRejected(this.reason)
                resolvePromise(promise, func, resolve, reject)
              } catch (e) {}
            }, 0)
          })
      }
    })

    return promise
  }

  catch(onRejected) {
    return this.then(undefined, onRejected)
  }

  finally(cb) {
    this.then(val => MyPromise.resolve(cb()).then(() => val), err => MyPromise.resolve(cb()).then(() => {throw err}))
  }

  static all (array) {
    let res = [];
    return new MyPromise((resolve, reject) => {
      const addData = (key, val) => {
        res[key] = val;
        if (res.length === array.length) {
          resolve(res)
        }
      }
      for (let i = 0, len = array.length; i < len; i++) {
        let cur = array[i];
        if (cur instanceof MyPromise) {
          cur.then((val) => addData(i, val), err => reject(err))
        } else {
          addData(i, cur)
        }
      }
    })
  } 
  
  static resolve(val) {
    if (val instanceof MyPromise) return val;
    return new MyPromise(resolve => resolve(val))
  }
}

function resolvePromise(promise, func, resolve, reject) {
  // 识别对象自返回
  if (func === promise) {
    return reject(
      new TypeError('Chaining cycle detected for promise #<Promise>')
    )
  }
  // 返回promise
  if (func instanceof MyPromise) {
    func.then(resolve, reject)
  } else {
    resolve(func)
  }
}

module.exports = MyPromise
```

