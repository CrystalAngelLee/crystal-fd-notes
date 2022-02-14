# 概述

[Flow](https://flow.org/) 是 JavaScript 的静态类型检查器

# 快速上手

## 安装并使用flow

**安装依赖：** `yarn add --dev flow-bin`

**关闭IDE JavaScript 校验功能：**

1. 在 setting 中搜索 `javascript validate`
2. 找到对应的 Validate
3. 关闭选项

**添加 flow 配置文件**

- 通过命令生成： `yarn flow init`

**在需要检查的文件中添加 @flow**

```js
// @flow
function sum (a: number, b: number) {
  return a + b
}
sum(100, 1)
sum('100', '1')
```

**检测代码**

- `yarn flow`
- 停止 flow 服务： `yarn flow stop`



## 编译移除注解

### 方案一

**安装依赖：** `yarn add --dev flow-remove-types`

**使用命令：** `yarn flow-remove-types src -d dist`

### 方案二

**Babel 安装：** `yarn add --dev @babel/core @babel/cli @babel/preset-flow`

**添加 babel 配置文件：** `.babelrc`

```babelrc
{
  "presets": ["@babel/preset-flow"]
}
```

**使用命令：** `yarn babel src -d dist`



## 开发工具插件

VScode 搜索 `flow ` 找到 `Flow Language Support` 并进行安装

[点击查看编辑器支持情况](https://flow.org/en/docs/editors/)



# 语法

## 类型推断

根据代码中的使用情况去推断出变量类型的特征叫做类型推断

```js
/**
 * 类型推断
 *
 * @flow
 */

function square (n) {
  return n * n
}

// square('100')

square(100)
```



## 类型注解

更明确的限制类型

> 官方类型说明文档： https://flow.org/en/docs/types/
>
> 第三方类型说明手册：https://www.saltycrane.com/cheat-sheets/flow-type/latest/

```js
/**
 * 类型注解
 *
 * @flow
 */

function square (n: number) {
  return n * n
}

let num: number = 100

// num = 'string' // error

function foo (): number {
  return 100 // ok
  // return 'string' // error
}

function bar (): void {
  // return undefined
}
```



### 原始类型

```js
/**
 * 原始类型
 *
 * @flow
 */

const a: string = 'foobar'

const b: number = Infinity // NaN // 100

const c: boolean = false // true

const d: null = null

const e: void = undefined

const f: symbol = Symbol()
```



### 数组类型

```js
/**
 * 数组类型
 *
 * @flow
 */

const arr1: Array<number> = [1, 2, 3]

const arr2: number[] = [1, 2, 3]

// 元组
const foo: [string, number] = ['foo', 100]
```



### 对象类型

```js
/**
 * 对象类型
 *
 * @flow
 */

const obj1: { foo: string, bar: number } = { foo: 'string', bar: 100 }

const obj2: { foo?: string, bar: number } = { bar: 100 }

const obj3: { [string]: string } = {}

obj3.key1 = 'value1'
obj3.key2 = 'value2'
```



### 函数类型

```js
/**
 * 函数类型
 *
 * @flow
 */

function foo (callback: (string, number) => void) {
  callback('string', 100)
}

foo(function (str, n) {
  // str => string
  // n => number
})
```



### 特殊类型

```js
/**
 * 特殊类型
 *
 * @flow
 */

// 字面量类型: 限制某一个变量必须是某一个值
const a: 'foo' = 'foo'

const type: 'success' | 'warning' | 'danger' = 'success'

// ------------------------

// 声明类型

type StringOrNumber = string | number

const b: StringOrNumber = 'string' // 100

// ------------------------

// Maybe 类型 扩展了 null 和 undefined 两种类型
const gender: ?number = undefined
// 相当于
// const gender: number | null | void = undefined
```



### Mixed Any

```js
/**
 * Mixed Any
 *
 * @flow
 */

// 接收任意类型
// string | number | boolean | ....
function passMixed (value: mixed) {
  if (typeof value === 'string') {
    value.substr(1)
  }

  if (typeof value === 'number') {
    value * value
  }
}

passMixed('string')

passMixed(100)

// ---------------------------------
// 接收任意类型
function passAny (value: any) {
  value.substr(1)

  value * value
}

passAny('string')

passAny(100)
```

> any  是弱类型 mixed 是强类型



# 运行环境API

```js
/**
 * 运行环境 API
 *
 * @flow
 */

const element: HTMLElement | null = document.getElementById('app')
```

```tex
https://github.com/facebook/flow/tree/main/lib/core.js
https://github.com/facebook/flow/tree/main/lib/dom.js
https://github.com/facebook/flow/tree/main/lib/bom.js
https://github.com/facebook/flow/tree/main/lib/cssom.js
https://github.com/facebook/flow/tree/main/lib/node.js
```

