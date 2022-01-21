# tsconfig.json 配置

> tsconfig.json 是 TypeScript 项目的配置文件。如果一个目录下存在一个 tsconfig.json 文件，那么往往意味着这个目录就是 TypeScript 项目的根目录。
>
> tsconfig.json 包含 TypeScript 编译的相关配置，通过更改编译配置项，我们可以让 TypeScript 编译出 ES6、ES5、node 的代码。

接下来介绍一下 tsconfig.json 中的相关配置选项，并对比较重要的编译选项进行着重介绍

## compilerOptions

编译选项是 TypeScript 配置的核心部分，compilerOptions 内的配置根据功能可以分为 6 个部分。

### 项目选项

这些选项用于配置项目的运行时期望、转译 JavaScript 的输出方式和位置，以及与现有 JavaScript 代码的集成级别。

#### target

target 选项用来指定 TypeScript 编译代码的目标，不同的目标将影响代码中使用的特性是否会被降级。

target 的可选值包括ES3、ES5、ES6、ES7、ES2017、ES2018、ES2019、ES2020、ESNext这几种。

一般情况下，target 的默认值为ES3，如果不配置选项的话，代码中使用的ES6特性，比如箭头函数会被转换成等价的函数表达式。

#### module

module 选项可以用来设置 TypeScript 代码所使用的模块系统。

如果 target 的值设置为 ES3、ES5 ，那么 module 的默认值则为 CommonJS；如果 target 的值为 ES6 或者更高，那么 module 的默认值则为 ES6。

另外，module 还支持 ES2020、UMD、AMD、System、ESNext、None 的选项。

#### jsx

jsx 选项用来控制 jsx 文件转译成 JavaScript 的输出方式。该选项只影响.tsx文件的 JS 文件输出，并且没有默认值选项。

- react: 将 jsx 改为等价的对 React.createElement 的调用，并生成 .js 文件。

- react-jsx: 改为 __jsx 调用，并生成 .js 文件。

- react-jsxdev: 改为 __jsx 调用，并生成 .js 文件。

- preserve: 不对 jsx 进行改变，并生成 .jsx 文件。

- react-native: 不对 jsx 进行改变，并生成 .js 文件。

#### incremental

incremental 选项用来表示是否启动增量编译。incremental 为true时，则会将上次编译的工程图信息保存到磁盘上的文件中。

#### declaration

declaration 选项用来表示是否为项目中的 TypeScript 或 JavaScript 文件生成 .d.ts 文件，这些 .d.ts 文件描述了模块导出的 API 类型。

具体的行为可以在[Playground](http://www.typescriptlang.org/play)中编写代码，并在右侧的 .D.TS 观察输出。

#### sourceMap

sourceMap 选项用来表示是否生成[sourcemap](https://developer.mozilla.org/zh-CN/docs/Tools/Debugger/How_to/Use_a_source_map) 文件，这些文件允许调试器和其他工具在使用实际生成的 JavaScript 文件时，显示原始的 TypeScript 代码。

Source map 文件以 .js.map （或 .jsx.map）文件的形式被生成到与 .js 文件相对应的同一个目录下。

#### lib

在 [ts进阶](./ts进阶.md)中我们介绍过，安装 TypeScript 时会顺带安装一个 lib.d.ts 声明文件，并且默认包含了 ES5、DOM、WebWorker、ScriptHost 的库定义。

lib 配置项允许我们更细粒度地控制代码运行时的库定义文件，比如说 Node.js 程序，由于并不依赖浏览器环境，因此不需要包含 DOM 类型定义；而如果需要使用一些最新的、高级 ES 特性，则需要包含 ESNext 类型。

具体的详情你可以在[TypeScript 源码](https://github.com/microsoft/TypeScript/tree/main/lib)中查看完整的列表，并且自定义编译需要的lib类型定义。

### 严格模式
TypeScript 兼容 JavaScript 的代码，默认选项允许相当大的灵活性来适应这些模式。

在迁移 JavaScript 代码时，你可以先暂时关闭一些严格模式的设置。在正式的 TypeScript 项目中，推荐开启 strict 设置启用更严格的类型检查，以减少错误的发生。

#### strict

开启 strict 选项时，一般我们会同时开启一系列的类型检查选项，以便更好地保证程序的正确性。

strict 为 true 时，一般我们会开启以下编译配置。

- alwaysStrict：保证编译出的文件是 ECMAScript 的严格模式，并且每个文件的头部会添加 'use strict'。

- strictNullChecks：更严格地检查 null 和 undefined 类型，比如数组的 find 方法的返回类型将是更严格的 T | undefined。

- strictBindCallApply：更严格地检查 call、bind、apply 函数的调用，比如会检查参数的类型与函数类型是否一致。

- strictFunctionTypes：更严格地检查函数参数类型和类型兼容性。

- strictPropertyInitialization：更严格地检查类属性初始化，如果类的属性没有初始化，则会提示错误。

- noImplicitAny：禁止隐式 any 类型，需要显式指定类型。TypeScript 在不能根据上下文推断出类型时，会回退到 any 类型。

- noImplicitThis：禁止隐式 this 类型，需要显示指定 this 的类型。

### 额外检查

TypeScript 支持一些额外的代码检查，在某种程度上介于编译器与静态分析工具之间。如果你想要更多的代码检查，可能更适合使用 ESLint 这类工具。

- noImplicitReturns：禁止隐式返回。如果代码的逻辑分支中有返回，则所有的逻辑分支都应该有返回。

- noUnusedLocals：禁止未使用的本地变量。如果一个本地变量声明未被使用，则会抛出错误。

- noUnusedParameters：禁止未使用的函数参数。如果函数的参数未被使用，则会抛出错误。

- noFallthroughCasesInSwitch：禁止 switch 语句中的穿透的情况。开启 noFallthroughCasesInSwitch 后，如果 switch 语句的流程分支中没有 break 或 return ，则会抛出错误，从而避免了意外的 swtich 判断穿透导致的问题。

### 模块解析

模块解析部分的编译配置会影响代码中模块导入以及编译相关的配置。

#### moduleResolution

moduleResolution 用来指定模块解析策略。

module 配置值为 AMD、UMD、System、ES6 时，moduleResolution 默认为 classic，否则为 node。在目前的新代码中，我们一般都是使用 node，而不使用classic。

具体的模块解析策略，你可以查看[模块解析策略](https://www.typescriptlang.org/docs/handbook/module-resolution.html#module-resolution-strategies)。

#### baseUrl

baseUrl 指的是基准目录，用来设置解析非绝对路径模块名时的基准目录。比如设置 baseUrl 为 './' 时，TypeScript 将会从 tsconfig.json 所在的目录开始查找文件。

#### paths

paths 指的是路径设置，用来将模块路径重新映射到相对于 baseUrl 定位的其他路径配置。这里我们可以将 paths 理解为 webpack 的 alias 别名配置。

看一个具体的示例：

```ts
{
  "compilerOptions": {
    "paths": {
      "@src/*": ["src/*"],
      "@utils/*": ["src/utils/*"]
    }
  }
}
```

在上面的例子中，TypeScript 模块解析支持以一些自定义前缀来寻找模块，避免在代码中出现过长的相对路径。

**注意：因为 paths 中配置的别名仅在类型检测时生效，所以在使用 tsc 转译或者 webpack 构建 TypeScript 代码时，我们需要引入额外的插件将源码中的别名替换成正确的相对路径。**

#### rootDirs

rootDirs 可以指定多个目录作为根目录。这将允许编译器在这些“虚拟”目录中解析相对应的模块导入，就像它们被合并到同一目录中一样。

#### typeRoots

typeRoots 用来指定类型文件的根目录。

在默认情况下，所有 node_modules/@types 中的任何包都被认为是可见的。如果手动指定了 typeRoots ，则仅会从指定的目录里查找类型文件。

#### types

在默认情况下，所有的 typeRoots 包都将被包含在编译过程中。

手动指定 types 时，只有列出的包才会被包含在全局范围内，如下示例：

```ts
{
  "compilerOptions": {
    "types": ["node", "jest", "express"]
  }
}
```

在上述示例中可以看到，手动指定 types 时 ，仅包含了 node、jest、express 三个 node 模块的类型包。

#### allowSyntheticDefaultImports

allowSyntheticDefaultImports\****允许合成默认导出。

当 allowSyntheticDefaultImports 设置为 true，即使一个模块没有默认导出（export default），我们也可以在其他模块中像导入包含默认导出模块一样的方式导入这个模块，如下示例：

```ts
// allowSyntheticDefaultImports: true 可以使用
import React from 'react';
// allowSyntheticDefaultImports: false
import * as React from 'react';
```

在上面的示例中，对于没有默认导出的模块 react，如果设置了 allowSyntheticDefaultImports 为 true，则可以直接通过 import 导入 react；但如果设置 allowSyntheticDefaultImports 为 false，则需要通过 import * as 导入 react。

#### esModuleInterop

esModuleInterop 指的是 ES 模块的互操作性。

在默认情况下，TypeScript 像 ES6 模块一样对待 CommonJS / AMD / UMD，但是此时的 TypeScript 代码转移会导致不符合 ES6 模块规范。不过，开启 esModuleInterop 后，这些问题都将得到修复。

一般情况下，在启用 esModuleInterop 时，我们将同时启用 allowSyntheticDefaultImports。

### Source Maps

为了支持丰富的调试工具，并为开发人员提供有意义的崩溃报告，TypeScript 支持生成符合 JavaScript Source Map 标准的附加文件（即 .map 文件）。

#### sourceRoot

sourceRoot 用来指定调试器需要定位的 TypeScript 文件位置，而不是相对于源文件的路径。

sourceRoot的取值可以是路径或者 URL。

#### mapRoot

mapRoot 用来指定调试器需要定位的 source map 文件的位置，而不是生成的文件位置。

#### inlineSourceMap

开启 inlineSourceMap 选项时，将不会生成 .js.map 文件，而是将 source map 文件内容生成内联字符串写入对应的 .js 文件中。虽然这样会生成较大的 JS 文件，但是在不支持 .map 调试的环境下将会很方便。

#### inlineSources

开启 inlineSources 选项时，将会把源文件的所有内容生成内联字符串并写入 source map 中。这个选项的用途和 inlineSourceMap 是一样的。

### 实验选项

TypeScript 支持一些尚未在 JavaScript 提案中稳定的语言特性，因此在 TypeScript 中实验选项是作为实验特性存在的。

#### experimentalDecorators

experimentalDecorators 选项会开启[装饰器提案](https://github.com/tc39/proposal-decorators)的特性。

**目前，装饰器提案在 stage 2 仍未完全批准到 JavaScript 规范中，且 TypeScript 实现的装饰器版本可能和 JavaScript 有所不同。**

#### emitDecoratorMetadata

emitDecoratorMetadata 选项允许装饰器使用反射数据的特性。

## 高级选项

#### skipLibCheck

开启 skipLibCheck选项，表示可以跳过检查声明文件。

如果我们开启了这个选项，则可以节省编译期的时间，但可能会牺牲类型系统的准确性。在设置该选项时，推荐值为true**。**

#### forceConsistentCasingInFileNames

TypeScript 对文件的大小写是敏感的。如果有一部分的开发人员在大小写敏感的系统开发，而另一部分的开发人员在大小写不敏感的系统开发，则可能会出现问题。

开启此选项后，如果开发人员正在使用和系统不一致的大小写规则，则会抛出错误。

## include

include 用来指定需要包括在 TypeScript 项目中的文件或者文件匹配路径。如果我们指定了 files 配置项，则 include 的 默认值为 []，否则 include 默认值为 ["\*\*/*"] ，即包含了目录下的所有文件。

如果 glob 匹配的文件中没有包含文件的扩展名，则只有 files 支持的扩展名会被包含。

一般来说，include 的默认值为.ts、.tsx 和 .d.ts。如果我们开启了 allowJs 选项，还包括 .js 和 .jsx 文件。

## exclude

exclude 用来指定解析 include 配置中需要跳过的文件或者文件匹配路径。一般来说，exclude 的默认值为 ["node_modules", "bower_components", "jspm_packages"]。

**需要注意：exclude配置项只会改变include配置项中的结果。**

## files

files 选项用来指定 TypeScript 项目中需要包含的文件列表。

如果项目非常小，那么我们可以使用 files**指定项目的文件，否则更适合使用include指定项目文件**。

## extends

extends 配置项的值是一个字符串，用来声明当前配置需要继承的另外一个配置的路径，这个路径使用 Node.js 风格的解析模式。TypeScript 首先会加载 extends 的配置文件，然后使用当前的 tsconfig.json 文件里的配置覆盖继承的文件里的配置。

TypeScript 会基于当前 tsconfig.json 配置文件的路径解析所继承的配置文件中出现的相对路径。

<div style="color: red">更多的 TypeScript 配置可以在<a href="https://www.typescriptlang.org/tsconfig">TSConfig Reference</a>中查看学习。</div>



# 常见 TypeScript 错误汇总分析

## 常见错误

TypeScript 错误信息由错误码和详细信息组成。其中，错误码是以“TS”开头 + 数字（一般是 4 位数字）结尾这样的格式组成的字符串，用来作为特定类型错误的专属代号。如果你想查看所有的错误信息和错误码，可以点击T[ypeScript 源码仓库](https://github.com/Microsoft/TypeScript/blob/main/src/compiler/diagnosticMessages.json)。当然，随着 TypeScript 版本的更新，也会逐渐增加更多新的类型错误。

下面我们看一下那些常见但在官方文档甚少提及的类型错误。

### TS2456

首先是由于类型别名循环引用了自身造成的 TS2456 类型错误，如下示例：

```ts
// TS2456: Type alias 'T' circularly references itself.
type T = Readonly<T>;
```

在上述示例中，对于 T 这个类型别名，如果 TypeScript 编译器想知道 T 类型是什么，就需要展开类型别名赋值的 Readonly`<T>`。而为了确定 Readonly`<T> `的类型，TypeScript 编译器需要继续判断类型入参 T 的类型，这就形成了一个循环引用。类似函数循环调用自己，如果没有正确的终止条件，就会一直处于无限循环的状态。

当然，如果在类型别名的定义中设定了正确的终止条件，我们就可以使用循环引用的特殊数据结构，如下示例：

```ts
type JSON = string | number | boolean | null | JSON[] | { [key: string]: JSON };
const json1: JSON = 'json';
const json2: JSON = ['str', 1, true, null];
const json3: JSON = { key: 'value' };
```

在上面的例子中，我们定义了 JSON 数据结构的 TypeScript 类型。其中，就有对类型别名 JSON 自身的循环引用，即示例中出现的 JSON[] | { [key: string]: JSON }。与第 1 个例子不同的是，这里的引用最终可以具体展开为 string | number | boolean | null 类型，所以不会出现无限循环的情况。

**📢 注意：第 2 个例子只能在 TypeScript 3.7 以上的版本使用，如果版本小于 3.7 仍会提示 TS2456 错误。**

### TS2554

> 形参和实参个数不匹配

```ts
function toString(x: number | undefined): string {
  if (x === undefined) {
    return '';
  }
  return x.toString();
}

toString(); // TS2554: Expected 1 arguments, but got 0.
toString(undefined);
toString(1);
```

上面例子报错的原因是，在 TypeScript 中，undefined 是一个特殊的类型。由于类型为 undefined，并不代表可缺省，因此示例中的第 8 行提示了 TS2554 错误。

而可选参数是一种特殊的类型，虽然在代码执行层面上，最终参数类型是 undefined 和参数可选的函数，接收到的入参的值都可以是 undefined，但是在 TypeScript 的代码检查中，undefined 类型的参数和可选参数都会被当作不同的类型来对待，如下示例：

```ts
function toString(x?: number): string {
  if (x === undefined) {
    return '';
  }
  return x.toString();
}

function toString(x = ''): string {
  return x.toString();
}
```

因此，如果在编程的过程中函数的参数是可选的，我们最好使用可选参数的语法，这样就可以避免手动传入 undefined 的值，并顺利通过 TypeScript 的检查。

值得一提的是，在 TypeScript 4.1 大版本的更新中，Promise 构造的 resolve 参数不再是默认可选的了，所以如以下示例第 2 行所示，在未指定入参的情况下，调用 resolve 会提示类型错误 **（注意：为了以示区分，官方使用了 TS2794 错误码指代这个错误）**。

```ts
new Promise((resolve) => {
  resolve(); // TS2794: Expected 1 arguments, but got 0. Did you forget to include 'void' in your type argument to 'Promise'? 
});
```

如果我们不需要参数，只需要给 Promise 的泛型参数传入 void 即可，如下示例：

```ts
new Promise<void>((resolve) => {
  resolve();
});
```

在上述示例中，因为我们在第 1 行给泛型类 Promise 指定了 void 类型入参（注意是 void 而不是 undefined），所以在第 3 行调用 resolve 时无须指定入参。

### TS1169

> 在接口类型定义中由于使用了非字面量或者非唯一 symbol 类型作为属性名

```ts
interface Obj {
  [key in 'id' | 'name']: any; // TS1169: A computed property name in an interface must refer to an expression whose type is a literal type or a 'unique symbol' type.
};
```

在上述示例中，因为interface 类型的属性必须是字面量类型(string、number) 或者是 unique symbol 类型，所以在第 2 行提示了 TS1169 错误。

关于接口类型支持的用法如下示例：

```ts
const symbol: unique symbol = Symbol();

interface Obj {
  [key: string]: any;
  [key: number]: any;
  [symbol]: any;
}
```

在上述示例中的第 4~6 行，我们使用了 string、number 和 symbol 作为接口属性，所以不会提示类型错误。

但是，在 type 关键字声明的类型别名中，我们却可以使用映射类型定义属性，如下示例：

```ts
type Obj = {
  [key in 'id' | 'name']: any;
};
```

在示例中的第 2 行，我们定义了一个包含 id 和 name 属性的类型别名 Obj。

### TS2345

> 在传参时由于类型不兼容

```ts
enum A {
  x = 'x',
  y = 'y',
  z = 'z',
}
enum B {
  x = 'x',
  y = 'y',
  z = 'z',
}

function fn(val: A) {}
fn(B.x); // TS2345: Argument of type 'B.x' is not assignable to parameter of type 'A'.
```

如上面的例子所示，函数 fn 参数的 val 类型是枚举 A，在 13 行我们传入了与枚举 A 类似的枚举 B 的值，此时 TypeScript 提示了类型不匹配的错误。这是因为枚举是在运行时真正存在的对象，因此 TypeScript 并不会判断两个枚举是否可以互相兼容。

此时解决这个错误的方式也很简单，我们只需要让这两个枚举类型互相兼容就行，比如使用类型断言绕过 TypeScript 的类型检查，如下示例：

```ts
function fn(val: A) {}
fn((B.x as unknown) as A);
```

在示例中的第 2 行，我们使用了 as 双重类型断言让枚举 B.x 兼容枚举类型 A，从而不再提示类型错误。

### TS2589

> 泛型实例化递归嵌套过深

```ts
type RepeatX<N extends number, T extends any[] = []> = T['length'] extends N
  ? T
  : RepeatX<N, [...T, 'X']>;
type T1 = RepeatX<5>; // => ["X", "X", "X", "X", "X"]
// TS2589: Type instantiation is excessively deep and possibly infinite.
type T2 = RepeatX<50>; // => any
```

在上面的例子中，因为第 1 行的泛型 RepeatX 接收了一个数字类型入参 N，并返回了一个长度为 N、元素都是 'X' 的数组类型，所以第 4 行的类型 T1 包含了 5 个 "X" 的数组类型；但是第 6 行的类型 T2 的类型却是 any，并且提示了 TS2589 类型错误。这是因为 TypeScript 在处理递归类型的时候，最多实例化 50 层，如果超出了递归层数的限制，TypeScript 便不会继续实例化，并且类型会变为 top 类型 any。

对于上面的错误，我们使用 @ts-ignore 注释忽略即可。

### TS2322

> 一个常见的字符串字面量类型的 TS2322 错误

```ts
interface CSSProperties {
  display: 'block' | 'flex' | 'grid';
}
const style = {
  display: 'flex',
};
// TS2322: Type '{ display: string; }' is not assignable to type 'CSSProperties'.
//  Types of property 'display' are incompatible.
//   Type 'string' is not assignable to type '"block" | "flex" | "grid"'.
const cssStyle: CSSProperties = style;
```

在上面的例子中，CSSProperties 的 display 属性的类型是字符串字面量类型 'block' | 'flex' | 'grid'，虽然变量 style 的 display 属性看起来与 CSSProperties 类型完全兼容，但是 TypeScript 提示了 TS2322 类型不兼容的错误。这是因为变量 style 的类型被自动推断成了 { display: string }，string 类型自然无法兼容字符串字面量类型 'block' | 'flex' | 'grid'，所以变量 style 不能赋值给 cssStyle。

如下提供了两种解决这个错误的方法。

```ts
// 方法 1
const style: CSSProperties = {
  display: 'flex',
};
// 方法 2
const style = {
  display: 'flex' as 'flex',
};
// typeof style = { display: 'flex' }
```

在方法 1 中，显式声明了 style 类型为 CSSProperties，因此变量 style 类型与 cssStyle 期望的类型兼容。

在方法 2 中，使用了类型断言声明 display 属性的值为字符串字面量类型 'flex'，因此 style 的类型被自动推断成了 { display: 'flex' }，与 CSSProperties 类型兼容。

### TS2352

> 类型收缩特性的 TS2352 类型错误

```ts
let x: string | undefined;
if (x) {
  x.trim();
  setTimeout(() => {
    x.trim(); // TS2532: Object is possibly 'undefined'.
  });
}
class Person {
  greet() {}
}
let person: Person | string;
if (person instanceof Person) {
  person.greet();
  const innerFn = () => {
    person.greet(); // TS2532: Object is possibly 'undefined'.
  };
}
```

在上述示例中的第 1 行，变量 x 的类型是 sting | undefined。在第 3 行的 if 语句中，变量 x 的类型按照之前讲的类型收缩特性应该是 string，可以看到第 4 行的代码可以通过类型检查，而第 6 行的代码报错 x 类型可能是 undefined（因为 setTimeout 的类型守卫失效，所以 x 的类型不会缩小为 string）。

同样，对于第 10 行的变量 person ，我们可以使用 instanceof 将它的类型收缩为 Person，因此第 16 行的代码通过了类型检查，而第 18 行则提示了 TS2352 错误。这是因为函数中对捕获的变量不会使用类型收缩的结果，因为编译器不知道回调函数什么时候被执行，也就无法使用之前类型收缩的结果。

针对这种错误的处理方式也很简单，将类型收缩的代码放入函数体内部即可，如下示例：

```ts
let x: string | undefined;
setTimeout(() => {
  if (x) {
    x.trim(); // OK
  }
});
class Person {
  greet() {}
}
let person: Person | undefined;
const innerFn = () => {
  if (person instanceof Person) {
    person.greet(); // Ok
  }
};
```

## 单元测试

在单元测试中，我们需要测试的是函数的输出与预计的输出是否相等。在 TypeScript 的类型测试中，我们需要测试的是编写的工具函数转换后的类型与预计的类型是否一致。

我们知道当赋值、传参的类型与预期不一致，TypeScript 就会抛出类型错误，如下示例：

```ts
const x: string = 1; // TS2322: Type 'number' is not assignable to type 'string'
```

在上述示例中可以看到，把数字字面量 1 赋值给 string 类型变量 x 时，会提示 TS2322 错误。

因此，我们可以通过泛型限定需要测试的类型。只有需要测试的类型与预期类型一致时，才可以通过 TypeScript 编译器的检查，如下示例：

```ts
type ExpectTrue<T extends true> = T;
type T1 = ExpectTrue<true>;
type T2 = ExpectTrue<null>; // TS2344: Type 'null' does not satisfy the constraint 'true'.
```

在上面 ExpectTrue 的测试方法中，因为第 1 行预期的类型是 true，所以第 2 行的入参为 true 时不会出现错误提示。但是，因为第 3 行的入参是 null ，所以会提示类型错误。

自 TS 3.9 版本起，官方支持了与 @ts-ignore 注释相反功能的 @ts-expect-error 注释。使用 @ts-expect-error 注释，我们可以标记代码中应该有类型错误的部分。

与 ts-ignore 不同的是，如果下一行代码中没有错误，则会提示 TS2578 的错误，如下示例：

```ts
// @ts-expect-error
const x: number = '42';
// TS2578: Unused '@ts-expect-error' directive.
// @ts-expect-error
const y: number = 42;
```

在上述示例的第 2 行代码处并不会提示类型不兼容的错误，这是因为 @ts-expect-error 注释命令表示下一行应当有类型错误，符合预期。而第 6 行的代码会提示 TS2578 未使用的 @ts-expect-error 命令，这是因为第 6 行的代码没有类型错误。

**`@ts-expect-error`注释命令在编写预期失败的单元测试中很有用处**
