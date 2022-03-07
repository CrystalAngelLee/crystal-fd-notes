# Webpack 的出现

在前端初期，前端只需要编写静态页面，对JS也仅需要进行表单验证和动效做出处理，不存在太多的工程化的问题；
伴随着 Ajax 技术的诞生，前端的工作模式发生了很大的变化，前端不需要仅渲染页面，也可以管理数据以及和用户进行交互，也诞生了 JQuery 这样的工具库；
趋于现阶段大前端开发，前端的工作面向多样化、复杂化的发展，不仅要开发PC端的Web界面，还有移动端的Web界面以及小程序公众号等，伴随着事情越来越多，流程也越来越复杂，就出现了一些问题：

- 采用模块化开发
  不同的浏览器对模块化的支持的不同的，而且模块化存在多种实现规范（详见[模块化概述](../前端模块化/模块化概述.md)）都给最终产出产生了一定的影响
- 使用新特性提高效率保证安全性
  面对新特性 less\sass\ES6新特性...浏览器也无法提供原生支持
- 实时监听开发过程使用热更新
- 项目结果打包压缩优化

这个时候我们需要一个工具实现项目工程化（webpack）

# Webpack 使用

> Webpack 是为现代 JS 应用提供静态模块打包的工具。
> 核心功能就是进行打包操作：
>
> - 打包： 将不同类型资源安模块处理进行打包
> - 静态： 打包后最终产出静态资源
> - 模块： webpack支持不同规范的模块化开发

- webpack5 支持零配置使用
- 想要自定义修改webpack配置相关内容可以通过参数的形式进行处理（比如修改配置文件名为a.js, 命令修改为 webpack --config a.js即可）

## Loader

**为什么需要loader**



**loader 是什么**

```tex
webpack4 中对于loader的使用分为三种形式
1. 行内 loader
2. 配置文件中的 loader
3. 命令行（cli 工具）中使用的 loader（webpack5 不建议使用！！）
```

### css-loader

> 将 CSS 语法处理为 JS 可以使用的模块类型，不能将样式放在界面上进行使用

**安装依赖：** `yarn add css-loader --dev`

**行内loader使用：**

```js
import "css-loader!../css/login.css"

function login() {
  ...
}
```

**配置文件中的 loader使用：**

```js
module.exports = {
  // ... 
  /**
   * 放置匹配规则 以及 相应规则下需要添加的属性和属性值
  */
  module: {
    rules: [
      {
        test: /\.css$/, // 一般就是一个正则表达式，用于匹配我们需要处理的文件类型
        use: [
          {
            loader: 'css-loader'
          }
        ]
      },
      // 简写方式 一
      {
        test: /\.css$/,
        loader: 'css-loader'
      },
      // 简写方式 二
      {
        test: /\.css$/,
        use: ['css-loader']
      },
    ]
  }
}
```

### style-loader

> 让 css-loader 处理之后的 css 能在界面上展示出来
> 作用：生成一个 style 标签

**安装依赖：** `yarn add style-loader --dev`

**配置：**

```js
module.exports = {
  // ... 
  /**
   * 放置匹配规则 以及 相应规则下需要添加的属性和属性值
  */
  module: {
    rules: [
      {
        test: /\.css$/,
        // 执行顺序 从右/下往左/上 执行
        use: ['style-loader', 'css-loader']
      },
    ]
  }
}
```

### less-loader

> less-loader 中使用 less 将 less 转换为 css，所以需要同时依赖 less

**安装依赖：** `yarn add less less-loader --dev`

**配置：**

```js
module.exports = {
  // ... 
  /**
   * 放置匹配规则 以及 相应规则下需要添加的属性和属性值
  */
  module: {
    rules: [
      {
        test: /\.css$/,
        // 执行顺序 从右/下往左/上 执行
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.less$/,
        // 执行顺序 从右/下往左/上 执行
        use: ['style-loader', 'css-loader', 'less-loader']
      }
    ]
  }
}
```

### postcss-loader

#### Browserslistrc 工作流程

在 package.json 中配置

![image-20220307150450589](./images/broserlistrc应用实例.png)

或创建 .browserslistrc 文件配置

```json
>0.2%
last 2 version
not dead
```

针对以上实例的配置，我们接下来描述一下其原理。

```tex
当前我们在做前端开发的时候，大多采用的都是工程化的方式；
面对兼容性的问题（CSS新特性 JS新特性 不同平台的支持不同），我们分解开来就是
1. 如何实现兼容： babel processenv
2. 到底要兼容哪些平台: 项目针对性平台 或 市场主流平台 - caniuse.com（(到这里查看市场占有率)[https://caniuse.com/usage-table]）

node_modules/browserslist/index.js
当我们用脚手架创建完了一个项目后，默认会帮我们安装好 browserlist. browserlist 在工作的时候会调用 caniuse 的 lite 工具，到 caniuse.com 发送请求，得到相应数据后结合相应配置做兼容性的处理
```

测试指令（mac 电脑下可以添加相应条件）

```bash
# 逗号表示或者的意思，市场占有率大于 1% 或者某一个平台的最新2个版本
npx browserlist ">1%, last 2 version" 
```

#### postcss 工作流程

> 针对 CSS 的兼容性处理，我们可以使用 postcss
> postcss 是通过 javascript 转换样式的工具

**安装**： `npm i postcss postcss-cli -D`

postcss-cli 的安装就是为了可以在命令行终端使用postcss相关指令

`npx postcss -o ret.css .src/css/test.css`

**安装自动添加前缀的插件**： `npm i autoprefixer -D`

`npx postcss --use autoprefixer -o ret.css .src/css/test.css`

#### postcss-loader

**安装依赖：** `yarn add postcss-loader --dev`

postcss 在 css-loader 之前进行工作

**配置：**

```js
module.exports = {
  // ... 
  /**
   * 放置匹配规则 以及 相应规则下需要添加的属性和属性值
  */
  module: {
    rules: [
      {
        test: /\.css$/,
        // 执行顺序 从右/下往左/上 执行
        use: [
          'style-loader', 
          'css-loader', 
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  require('autoprefixer')
                ]
              }
            }
          }
        ]
      }
    ]
  }
}
```

**新需求**

兼容 color: #12345678 的写法特性

可以使用 postcss-preset-env 插件集进行处理

**安装依赖**： `yarn add postcss-preset-env -D`

**配置：**

```js
module.exports = {
  // ... 
  /**
   * 放置匹配规则 以及 相应规则下需要添加的属性和属性值
  */
  module: {
    rules: [
      {
        test: /\.css$/,
        // 执行顺序 从右/下往左/上 执行
        use: [
          'style-loader', 
          'css-loader', 
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                // 写法一
                plugins: [
                  require('autoprefixer'),
                  require('postcss-preset-env')
                ]
                // 简写
                plugins: ['postcss-preset-env']
              }
            }
          }
        ]
      }
    ]
  }
}
```

**配置简写：**

```js
module.exports = {
  // ... 
  /**
   * 放置匹配规则 以及 相应规则下需要添加的属性和属性值
  */
  module: {
    rules: [
      {
        test: /\.css$/,
        // 执行顺序 从右/下往左/上 执行
        use: [
          'style-loader', 
          'css-loader', 
          'postcss-loader'
        ]
      },
      {
        test: /\.less$/,
        // 执行顺序 从右/下往左/上 执行
        use: ['style-loader', 'css-loader', 'postcss-loader', 'less-loader']
      }
    ]
  }
}
```

**新建文件 postcss.config.js**

```js
module.exports = {
  plugins: [
    require('postcss-preset-env')
  ]
}
```

