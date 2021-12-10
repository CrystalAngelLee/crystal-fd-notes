## why



## what



## 实现Iterator

> 思路：就是在这个对象中挂载一个iterator方法，然后在这个方法中返回一个迭代器对象

```js
const obj = {
  [Symbol.iterator]: function() {
    return {
      next: function () {
        return {
          value: '',
          done: true
        }
      }
    }
  }
}
```

