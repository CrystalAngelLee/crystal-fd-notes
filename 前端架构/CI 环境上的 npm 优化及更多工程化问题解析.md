# CI环境上的 npm 优化

> [npm ci官网](https://docs.npmjs.com/cli/v8/commands/npm-ci)

## 合理使用 npm ci 和 npm install

`npm ci` 就是专门为 CI 环境准备的安装命令，相比 npm install 它的不同之处在于：

- npm ci 要求项目中必须存在 package-lock.json 或 npm-shrinkwrap.json；

- npm ci 完全根据 package-lock.json 安装依赖，这可以保证整个开发团队都使用版本完全一致的依赖

- 正因为 npm ci 完全根据 package-lock.json 安装依赖，在安装过程中，它不需要计算求解依赖满足问题、构造依赖树，因此安装过程会**更加迅速**

- npm ci 在执行安装时，会先删除项目中现有的 node_modules，然后全新安装

- npm ci 只能一次安装整个项目所有依赖包，无法安装单个依赖包

- 如果 package-lock.json 和 package.json 冲突，那么 npm ci 会直接报错，并非更新 lockfiles

- npm ci 永远不会改变 package.json 和 package-lock.json

基于以上特性，**我们在 CI 环境使用 npm ci 代替 npm install，一般会获得更加稳定、一致和迅速的安装体验**



## 使用 package-lock.json 优化依赖安装时间

对于应用项目，建议上传 package-lock.json 到仓库中，以保证依赖安装的一致性。事实上，如果项目中使用了 package-lock.json 一般还可以显著加速依赖安装时间。这是因为**package-lock.json 中已经缓存了每个包的具体版本和下载链接，不需要再去远程仓库进行查询，即可直接进入文件完整性校验环节，减少了大量网络请求**

除了上面所述内容，CI 环境上，缓存 node_modules 文件也是企业级使用包管理工具常用的优化做法



# 更多工程化相关问题解析

## 为什么要 lockfiles，要不要提交 lockfiles 到仓库？

从 npm v5 版本开始，增加了 package-lock.json 文件。我们知道**package-lock.json 文件的作用是锁定依赖安装结构，目的是保证在任意机器上执行 npm install 都会得到完全相同的 node_modules 安装结果**

需要明确，为什么单一的 package.json 不能确定唯一的依赖树：

- 不同版本的 npm 的安装依赖策略和算法不同；

- npm install 将根据 package.json 中的 semver-range version 更新依赖，某些依赖项自上次安装以来，可能已发布了新版本


因此，**保证能够完整准确地还原项目依赖，就是 lockfiles 出现的原因**

我们了解一下 package-lock.json 的作用机制。package-lock.json 的内容举例：

```json
"@babel/core": {
	  "version": "7.2.0",
	  "resolved": "http://www.npm.com/@babel%2fcore/-/core-7.2.0.tgz",
	  "integrity": "sha1-pN04FJAZmOkzQPAIbphn/voWOto=",
	  "dev": true,
	  "requires": {
	    "@babel/code-frame": "^7.0.0",
	    // ...
	  },
	  "dependencies": {
	    "@babel/generator": {
	      "version": "7.2.0",
	      "resolved": "http://www.npm.com/@babel%2fgenerator/-/generator-7.2.0.tgz",
	      "integrity": "sha1-6vOCH6AwHZ1K74jmPUvMGbc7oWw=",
	      "dev": true,
	      "requires": {
	        "@babel/types": "^7.2.0",
	        "jsesc": "^2.5.1",
	        "lodash": "^4.17.10",
	        "source-map": "^0.5.0",
	        "trim-right": "^1.0.1"
	      }
	    },
	    // ...
	  }
	},
	// ...
}
```

通过以上代码，我们可以了解到一个 package-lock.json 的 dependency 主要由以下部分构成：

- Version：依赖包的版本号
- Resolved：依赖包安装源（可简单理解为下载地址）
- Integrity：表明包完整性的 Hash 值
- Dev：表示该模块是否为顶级模块的开发依赖或者是一个的传递依赖关系
- requires：依赖包所需要的所有依赖项，对应依赖包 package.json 里 dependencies 中的依赖项
- dependencies：依赖包 node_modules 中依赖的包（特殊情况下才存在）

事实上，**并不是所有的子依赖都有 dependencies 属性，只有子依赖的依赖和当前已安装在根目录的 node_modules 中的依赖冲突之后，才会有这个属性**。这就涉及嵌套情况的依赖管理，可以在[这里](./Yarn 的安装理念及如何破解依赖管理困境.md) 做相应的了解

那到底要不要提交 lockfiles 到仓库？这就需要看项目定位决定了

- 如果开发一个应用，我建议把 package-lock.json 文件提交到代码版本仓库。这样可以保证项目组成员、运维部署成员或者 CI 系统，在执行 npm install 后，能得到完全一致的依赖安装内容。
- 如果你的目标是开发一个给外部使用的库，那就要谨慎考虑了，因为**库项目一般是被其他项目依赖的，在不使用 package-lock.json 的情况下，就可以复用主项目已经加载过的包，减少依赖重复和体积**。
- 如果我们开发的库依赖了一个精确版本号的模块，那么提交 lockfiles 到仓库可能会造成同一个依赖不同版本都被下载的情况。如果作为库开发者，真的有使用某个特定版本依赖的需要，一个更好的方式是**定义 peerDependencies**

因此，一个推荐的做法是：**把 package-lock.json 一起提交到代码库中，不需要 ignore。但是执行 npm publish 命令，发布一个库的时候，它应该被忽略而不是直接发布出去**

理解上述概念并不够，对于 lockfiles 的处理，需要更加精细。这里我列出几条建议以供参考

1. 早期 npm 锁定版本的方式是使用 npm-shrinkwrap.json，它与 package-lock.json 不同点在于：npm 包发布的时候默认将 npm-shrinkwrap.json 发布，因此类库或者组件需要慎重。
2. 使用 package-lock.json 是 npm v5.x 版本新增特性，而 npm v5.6 以上才逐步稳定，在 5.0 - 5.6 中间，对 package-lock.json 的处理逻辑进行过几次更新。
3. 在 npm v5.0.x 版本中，npm install 时都会根据 package-lock.json 文件下载，不管 package.json 内容究竟是什么。
4. npm v5.1.0 版本到 npm v5.4.2，npm install 会无视 package-lock.json 文件，会去下载最新的 npm 包并且更新 package-lock.json。
5. npm 5.4.2 版本后：
   - 如果项目中只有 package.json 文件，npm install 之后，会根据它生成一个 package-lock.json 文件；
   - 如果项目中存在 package.json 和 package-lock.json 文件，同时 package.json 的 semver-range 版本 和 package-lock.json 中版本兼容，即使此时有新的适用版本，npm install 还是会根据 package-lock.json 下载；
   - 如果项目中存在 package.json 和 package-lock.json 文件，同时 package.json 的 semver-range 版本和 package-lock.json 中版本不兼容，npm install 时 package-lock.json 将会更新到兼容 package.json 的版本；
   - 如果 package-lock.json 和 npm-shrinkwrap.json 同时存在于项目根目录，package-lock.json 将会被忽略。

以上内容你可以结合[ npm 安装流程](./npm安装机制及企业级部署私服原理.md)进一步理解



## 为什么有 xxxDependencies？

npm 设计了以下几种依赖类型声明：

- dependencies 项目依赖
- devDependencies 开发依赖
- peerDependencies 同版本依赖
- bundledDependencies 捆绑依赖
- optionalDependencies 可选依赖

它们起到的作用和声明意义各不相同。dependencies 表示项目依赖，这些依赖都会成为线上生产环境中的代码组成部分。当它关联的 npm 包被下载时，**dependencies 下的模块也会作为依赖，一起被下载**



<h5>devDependencies 表示开发依赖，不会被自动下载</h5>
因为 devDependencies 一般只在开发阶段起作用或只是在开发环境中需要用到。比如 Webpack，预处理器 babel-loader、scss-loader，测试工具 E2E、Chai 等，这些都是辅助开发的工具包，无须在生产环境使用

这里需要特别说明的是：**并不是只有在 dependencies 中的模块才会被一起打包，而在 devDependencies 中的依赖一定不会被打包**。实际上，依赖是否被打包，**完全取决于项目里是否被引入了该模块**。dependencies 和 devDependencies 在业务中更多的只是一个规范作用，我们自己的应用项目中，使用 npm install 命令安装依赖时，dependencies 和 devDependencies 内容都会被下载



<h5>peerDependencies 表示同版本依赖</h5>

简单来说就是：如果你安装我，那么你最好也安装我对应的依赖。举个例子，假设 react-ui@1.2.2 只提供一套基于 React 的 UI 组件库，它需要宿主环境提供指定的 React 版本来搭配使用，因此我们需要在 React-ui 的 package.json 中配置：

```json
"peerDependencies": {
    "React": "^17.0.0"
}
```

🌰 举一个场景实例，对于插件类 (Plugin) 项目，比如我开发一个 Koa 中间件，很明显这类插件或组件脱离（Koa）本体是不能单独运行且毫无意义的，但是这类插件又无须声明对本体（Koa）的依赖声明，更好的方式是使用宿主项目中的本体（Koa）依赖。这就是**peerDependencies 主要的使用场景**。这类场景有以下特点：

- **插件不能单独运行**
- **插件正确运行的前提是核心依赖库必须先下载安装**
- **我们不希望核心依赖库被重复下载**
- **插件 API 的设计必须要符合核心依赖库的插件编写规范**
- **在项目中，同一插件体系下，核心依赖库版本最好相同**



<h5>bundledDependencies 和 npm pack 打包命令有关</h5>

假设 package.json 中有如下配置：

```json
{
  "name": "test",
  "version": "1.0.0",
  "dependencies": {
    "dep": "^0.0.2",
    ...
  },
  "devDependencies": {
    ...
    "devD1": "^1.0.0"
  },
  "bundledDependencies": [
    "bundleD1",
    "bundleD2"
  ]
}
```

在执行 npm pack 时，就会产出一个 test-1.0.0.tgz 压缩包，且该压缩包中包含了 bundle D1 和 bundle D2 两个安装包。业务方使用 npm install test-1.0.0.tgz 命令时，也会安装 bundle D1 和 bundle D2

这里需要注意的是：**在 bundledDependencies 中指定的依赖包，必须先在 dependencies 和 devDependencies 声明过，否则在 npm pack 阶段会进行报错**



<h5>optionalDependencies 表示可选依赖</h5>

> 就是说即使对应依赖项安装失败了，也不会影响整个安装过程

一般我们很少使用到它，这里**我也不建议大家使用，因为它大概率会增加项目的不确定性和复杂性**



## 再谈版本规范——依赖库锁版本行为解析

npm 遵循 SemVer 版本规范，具体内容可以参考[语义化版本 2.0.0](https://semver.org/lang/zh-CN/)，这里不再展开。这部分内容我希望聚焦到工程建设的一个细节点上——依赖库锁版本行为

[Vue 官方有这样的内容](https://vue-loader.vuejs.org/zh/guide/#手动设置)：

> 每个 vue 包的新版本发布时，一个相应版本的 vue-template-compiler 也会随之发布。编译器的版本必须和基本的 vue 包保持同步，这样 vue-loader 就会生成兼容运行时的代码。这意味着你每次升级项目中的 vue 包时，也应该匹配升级 vue-template-compiler。

据此，我们需要考虑的是：作为库开发者，如何保证依赖包之间的强制最低版本要求？

我们先看看 create-react-app 的做法，在 create-react-app 的核心 react-script 当中，它利用 verify PackageTree 方法，对业务项目中的依赖进行比对和限制。[源码](https://github.com/facebook/create-react-app/blob/37712374bcaa6ccb168eeaf4fe8bd52d120dbc58/packages/react-scripts/scripts/utils/verifyPackageTree.js#L19)如下：

```js
function verifyPackageTree() {
  const depsToCheck = [
    'babel-eslint',
    'babel-jest',
    'babel-loader',
    'eslint',
    'jest',
    'webpack',
    'webpack-dev-server',
  ];
  const getSemverRegex = () =>
    /\bv?(?:0|[1-9]\d*)\.(?:0|[1-9]\d*)\.(?:0|[1-9]\d*)(?:-[\da-z-]+(?:\.[\da-z-]+)*)?(?:\+[\da-z-]+(?:\.[\da-z-]+)*)?\b/gi;
  const ownPackageJson = require('../../package.json');
  const expectedVersionsByDep = {};
  depsToCheck.forEach(dep => {
    const expectedVersion = ownPackageJson.dependencies[dep];
    if (!expectedVersion) {
      throw new Error('This dependency list is outdated, fix it.');
    }
    if (!getSemverRegex().test(expectedVersion)) {
      throw new Error(
        `The ${dep} package should be pinned, instead got version ${expectedVersion}.`
      );
    }
    expectedVersionsByDep[dep] = expectedVersion;
  });

  let currentDir = __dirname;
 
  while (true) {
    const previousDir = currentDir;
    currentDir = path.resolve(currentDir, '..');
    if (currentDir === previousDir) {
      // We've reached the root.
      break;
    }
    const maybeNodeModules = path.resolve(currentDir, 'node_modules');
    if (!fs.existsSync(maybeNodeModules)) {
      continue;
    }
    depsToCheck.forEach(dep => {
      const maybeDep = path.resolve(maybeNodeModules, dep);
      if (!fs.existsSync(maybeDep)) {
        return;
      }
      const maybeDepPackageJson = path.resolve(maybeDep, 'package.json');
      if (!fs.existsSync(maybeDepPackageJson)) {
        return;
      }
      const depPackageJson = JSON.parse(
        fs.readFileSync(maybeDepPackageJson, 'utf8')
      );
      const expectedVersion = expectedVersionsByDep[dep];
      if (!semver.satisfies(depPackageJson.version, expectedVersion)) {
        console.error(//...);
        process.exit(1);
      }
    });
  }
}
```

根据上述代码，我们不难发现，create-react-app 会对项目中的 babel-eslint、babel-jest、babel-loader、ESLint、Jest、webpack、webpack-dev-server 这些核心依赖进行检索——是否符合 create-react-app 对这些核心依赖的版本要求。**如果不符合依赖版本要求，那么 create-react-app 的构建过程会直接报错并退出**

create-react-app 这么做的理由是：**需要上述依赖项的某些确定版本，以保障 create-react-app 源码的相关功能稳定**

作为 create-react-app 的使用者，我们依然可以**通过 SKIP_PREFLIGHT_CHECK 这个环境变量，跳过核心依赖版本检查**，对应[源码](https://github.com/facebook/create-react-app/blob/5bd6e73047ef0ccd2f31616255c79a939d6402c4/packages/react-scripts/scripts/start.js#L27)：

```js
const verifyPackageTree = require('./utils/verifyPackageTree');
if (process.env.SKIP_PREFLIGHT_CHECK !== 'true') {
  verifyPackageTree();
}
```



# 最佳实操建议

1. 优先使用 npm v5.4.2 以上的 npm 版本，以保证 npm 的最基本先进性和稳定性。
2. 项目的第一次搭建使用 npm install 安装依赖包，并提交 package.json、package-lock.json，而不提交 node_modules 目录。
3. 其他项目成员首次 checkout/clone 项目代码后，执行一次 npm install 安装依赖包。
4. 对于升级依赖包的需求：
   - 依靠 npm update 命令升级到新的小版本；
   - 依靠 npm install @ 升级大版本；
   - 也可以手动修改 package.json 中版本号，并执行 npm install 来升级版本；
   - 本地验证升级后新版本无问题，提交新的 package.json、package-lock.json 文件。
5. 对于降级依赖包的需求：执行 npm install @ 命令，验证没问题后，提交新的 package.json、package-lock.json 文件。
6. 删除某些依赖：
   - 执行 npm uninstall 命令，验证没问题后，提交新的 package.json、package-lock.json 文件；
   - 或者手动操作 package.json，删除依赖，执行 npm install 命令，验证没问题后，提交新的 package.json、package-lock.json 文件
7. 任何团队成员提交 package.json、package-lock.json 更新后，其他成员应该拉取代码后，执行 npm install 更新依赖。
8. 任何时候都不要修改 package-lock.json。
9. 如果 package-lock.json 出现冲突或问题，建议将本地的 package-lock.json 文件删除，引入远程的 package-lock.json 文件和 package.json，再执行 npm install 命令
