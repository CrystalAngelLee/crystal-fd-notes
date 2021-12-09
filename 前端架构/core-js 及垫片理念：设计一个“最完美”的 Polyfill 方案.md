[core-js](https://github.com/zloirock/core-js) 是一个 JavaScript 标准库，它包含了 ECMAScript 2020 在内的多项特性的 polyfills，以及 ECMAScript 在 proposals 阶段的特性、WHATWG/W3C 新特性等。因此它是一个现代化前端项目的“标准套件”

```tex
- 通过 core-js，我们可以窥见前端工程化的方方面面；
- core-js 又和 Babel 深度绑定，因此学习 core-js，也能帮助开发者更好地理解 babel 生态，进而加深对前端生态的理解；
- 通过对 core-js 的解析，我们正好可以梳理前端一个极具特色的概念——polyfill（垫片/补丁）
```

# core-js 工程一览

core-js 是一个由 [Lerna](https://github.com/lerna/lerna) 搭建的 Monorepo 风格的项目，在它的 [packages](https://github.com/zloirock/core-js/tree/master/packages) 中，我们能看到五个相关包：

- core-js
- core-js-pure
- core-js-compact
- core-js-builder
- core-js-bundle

我们先从 core-js 入手。core-js 实现的基础垫片能力，是整个 core-js 的逻辑核心。

比如我们可以按照 `import 'core-js';`  方式引入全局 polyfills：或者按照 `import 'core-js/features/array/from';` 的方式，按需在业务项目的入口引入某些 polyfills

core-js 为什么有这么多的 packages 呢？实际上，它们各司其职，又紧密配合

**core-js-pure 提供了不污染全局变量的垫片能力**，比如我们可以按照：

```js
import _from from 'core-js-pure/features/array/from';
import _flat from 'core-js-pure/features/array/flat';
```
的方式，来实现独立的导出命名空间，进而避免全局变量的污染。

**core-js-compact 维护了按照browserslist规范的垫片需求数据**，来帮助我们找到“符合目标环境”的 polyfills 需求集合，比如以下代码：

```js
const {
  list, // array of required modules
  targets, // object with targets for each module
} = require('core-js-compat')({
  targets: '> 2.5%'
});
```
就可以筛选出全球使用份额大于 2.5% 的浏览器范围，并提供在这个范围下需要支持的垫片能力。

**core-js-builder 可以结合 core-js-compact 以及 core-js，并利用 webpack 能力，根据需求打包出 core-js 代码**。比如：

```js
require('core-js-builder')({
  targets: '> 0.5%',
  filename: './my-core-js-bundle.js',
}).then(code => {}).catch(error => {});
```