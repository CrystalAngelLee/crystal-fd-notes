## why

为了解决回调地狱和链式回调的问题

## what

Generator 函数的语法糖，返回一个Promise对象

**async 函数对 Generator 函数的改进，主要体现在以下三点:**

1. 内置执行器：Generator 函数的执行必须靠执行器，因为不能一次性执行完成，所以之后才有了开源的 co 函数库。但是，async 函数和正常的函数一样执行，也不用 co 函数库，也不用使用 next 方法，而 async 函数自带执行器，会自动执行。
2. 适用性更好：co 函数库有条件约束，yield 命令后面只能是 Thunk 函数或 Promise 对象，但是 async 函数的 await 关键词后面，可以不受约束。
3. 可读性更好：async 和 await，比起使用 * 号和 yield，语义更清晰明了。



## how

```js
// readFilePromise 依旧返回 Promise 对象
const readFilePromise = (filename) => {
  return new Promise((resolve, reject) => {
    fs.readFile(filename, (err, data) => {
      if(err) {
        reject(err);
      }else {
        resolve(data);
      }
    })
  }).then(res => res);
}
// 这里把 Generator的 * 换成 async，把 yield 换成 await
const gen = async function() {
  const data1 = await readFilePromise('1.txt')
  console.log(data1.toString())
  const data2 = await readFilePromise('2.txt')
  console.log(data2.toString)
}
```



### 错误处理

> 如果`await`后面的异步操作出错，那么等同于`async`函数返回的 Promise 对象被`reject`

1. 可以使用 `try...catch`
2. 可以使用 `func().catch(err => ...)` 处理





