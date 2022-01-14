# 类型守卫

JavaScript 作为一种动态语言，意味着其中的参数、值可以是多态（多种类型）。因此，我们需要区别对待每一种状态，以此确保对参数、值的操作合法。

举一个常见的场景为例，如下我们定义了一个可以接收字符串或者字符串数组的参数 toUpperCase，并将参数转成大写格式输出的函数 convertToUpperCase。

```ts
{
  const convertToUpperCase = (strOrArray) => {
    if (typeof strOrArray === 'string') {
      return strOrArray.toUpperCase();
    } else if (Array.isArray(strOrArray)) {
      return strOrArray.map(item => item.toUpperCase());
    }
  }
}
```

在示例中的第 3 行、第 5 行，我们分别使用了 typeof、Array.isArray 确保字符串和字符串数组类型的入参在运行时分别进入正确的分支，而不至于入参是数组类型时，调用数组类型并不存在的 toUpperCase 方法，从而抛出一个“strOrArray.toUpperCase is not a function”的错误。

在 TypeScript 中，因为受静态类型检测约束，所以在编码阶段我们必须使用类似的手段确保当前的数据类型支持相应的操作。当然，前提条件是已经显式地注解了类型的多态。

比如如果我们将上边示例中的 convertToUpperCase 函数使用 TypeScript 实现，那么就需要显示地标明 strOrArray 的类型就是 string 和 string[] 类型组成的联合类型，如下代码所示：

```ts
{
  const convertToUpperCase = (strOrArray: string | string[]) => {
    if (typeof strOrArray === 'string') {
      return strOrArray.toUpperCase();
    } else if (Array.isArray(strOrArray)) {
      return strOrArray.map(item => item.toUpperCase());
    }
  }
}
```

在示例中，convertToUpperCase 函数的主体逻辑与 JavaScript 中的逻辑完全一致（除了添加的参数类型注解）。

在 TypeScript 中，第 3 行和第 5 行的 typeof、Array.isArray 条件判断，除了可以保证转译为 JavaScript 运行后类型是正确的，还可以保证第 4 行和第 6 行在静态类型检测层面是正确的。

很明显，第 4 行中入参 strOrArray 的类型因为 typeof 条件判断变成了 string，第 6 行入参 strOrArray 的类型因为 Array.isArray 变成了 string[]，所以没有提示类型错误。而这个类型变化就是类型缩小，这里的 typeof、Array.isArray 条件判断就是类型守卫。

从示例中，我们可以看到类型守卫的作用在于触发类型缩小。实际上，它还可以用来区分类型集合中的不同成员。

类型集合一般包括联合类型和枚举类型，下面我们看看如何区分联合类型。

## 如何区分联合类型？

首先，我们看一下如何使用类型守卫来区分联合类型的不同成员，**常用的类型守卫包括switch、字面量恒等、typeof、instanceof、in 和自定义类型守卫这几种。**

### 1. switch

我们往往会使用 switch 类型守卫来处理联合类型中成员或者成员属性可枚举的场景，即字面量值的集合，如以下示例：

```ts
{
  const convert = (c: 'a' | 1) => {
    switch (c) {
      case 1:
        return c.toFixed(); // c is 1
      case 'a':
        return c.toLowerCase(); // c is 'a'
    }
  }
  const feat = (c: { animal: 'panda'; name: 'China' } | { feat: 'video'; name: 'Japan' }) => {
    switch (c.name) {
      case 'China':
        return c.animal; // c is "{ animal: 'panda'; name: 'China' }"
      case 'Japan':
        return c.feat; // c is "{ feat: 'video'; name: 'Japan' }"
    }
  };
}
```

在上述示例中，因为 convert 函数的参数及 feat 函数参数的 name 属性都是一个可被枚举的集合，所以我们可以使用 switch 来缩小类型。

比如第 5 行中 c 的类型被缩小为数字 1，第 7 行的 c 被缩小为字符串 'Japan'，第 13 和 15 行的 c 也被缩小为相应的接口类型。因此，我们对参数 c 进行相关操作时，也就不会提示类型错误了。

### 2. 字面量恒等

switch 适用的场景往往也可以直接使用字面量恒等比较进行替换，比如前边的 convert 函数可以改造成以下示例：

```ts
const convert = (c: 'a' | 1) => {
  if (c === 1) {
    return c.toFixed(); // c is 1
  } else if (c === 'a') {
    return c.toLowerCase(); // c is 'a'
  }
}
```

在以上示例中，第 3 行、第 5 行的类型相应都缩小为了字面量 1 和 'a'。

**建议：**一般来说，如果可枚举的值和条件分支越多，那么使用 switch 就会让代码逻辑更简洁、更清晰；反之，则推荐使用字面量恒等进行判断。

### 3. typeof

当联合类型的成员不可枚举，比如说是字符串、数字等原子类型组成的集合，这个时候就需要使用 typeof。

typeof 是一个比较特殊的操作符，我们可以使用它对 convert 函数进行改造，如下代码所示：

```ts
const convert = (c: 'a' | 1) => {
  if (typeof c === 'number') {
    return c.toFixed(); // c is 1
  } else if (typeof c === 'string') {
    return c.toLowerCase(); // c is 'a'
  }
}
```

在上述示例中，因为 typeof c 表达式的返回值类型是字面量联合类型 'string' | 'number' | 'bigint' | 'boolean' | 'symbol' | 'undefined' | 'object' | 'function'，所以通过字面量恒等判断我们把在第 2 行和第 4 行的 typeof c 表达式值类型进行了缩小，进而将 c 的类型缩小为明确的 string、number 等原子类型。

### 4. instanceof

此外，联合类型的成员还可以是类。比如以下示例中的第 9 行和第 11 行，我们使用了 instanceof 来判断 param 是 Dog 还是 Cat 类。

```ts
{
  class Dog {
    wang = 'wangwang';
  }
  class Cat {
    miao = 'miaomiao';
  }
  const getName = (animal: Dog | Cat) => {
    if (animal instanceof Dog) {
      return animal.wang;
    } else if (animal instanceof Cat) {
      return animal.miao;
    }
  }
}
```

这里我们可以看到，第 10 行、第 12 行的 animal 的类型也缩小为 Dog、Cat 了。

### 5. in

当联合类型的成员包含接口类型（对象），并且接口之间的属性不同，如下示例中的接口类型 Dog、Cat，我们不能直接通过“ . ”操作符获取 param 的 wang、miao 属性，从而区分它是 Dog 还是 Cat。

```ts
{
  interface Dog {
    wang: string;
  }
  interface Cat {
    miao: string;
  }
  const getName = (animal: Dog | Cat) => {
    if (typeof animal.wang == 'string') { // ts(2339)
      return animal.wang; // ts(2339)
    } else if (animal.miao) { // ts(2339)
      return animal.miao; // ts(2339)
    }
  }
}
```

这里我们看到，在第 9~12 行都提示了一个 ts(2339) Dog | Cat 联合类型没有 wang、miao 属性的错误。

这个时候我们就需要使用 in 操作符来改造一下 getName 函数， 这样就不会提示类型错误了，如下代码所示：

```ts
const getName = (animal: Dog | Cat) => {
  if ('wang' in animal) { // ok
    return animal.wang; // ok
  } else if ('miao' in animal) { // ok
    return animal.miao; // ok
  }
}
```

这里我们可以看到，第 3 行、第 4 行中的 animal 的类型也缩小成 Dog 和 Cat 了。

### 6. 自定义类型守卫

我们将使用类型谓词 is，比如封装一个 isDog 函数来区分 Dog 和 Cat，如下代码所示：

```ts
const isDog = function (animal: Dog | Cat): animal is Dog {
  return 'wang' in animal;
}
const getName = (animal: Dog | Cat) => {
  if (isDog(animal)) {
    return animal.wang;
  }
}
```

这里我们在 getName 函数第 5 行的条件判断中使用了 isDog 将 animal 的类型缩小为 Dog，这样第 6 行就可以直接获取 wang 属性了，而不会提示一个 ts(2339) 的错误。



## 如何区别枚举类型？

> 枚举类型是命名常量的集合，所以我们也需要使用类型守卫区分枚举类型的成员。

回想一下枚举类型的若干特性，因为这将决定使用哪几种类型守卫来区分枚举既是可行的，又是安全的。

**特性 1：**枚举和其他任何枚举、类型都不可比较，除了数字枚举可以与数字类型比较之外；

**特性 2：**数字枚举极其不稳定。

熟悉了这些特性后，得出一个结论：最佳实践时，我们永远不要拿枚举和除了自身之外的任何枚举、类型进行比较。

🌰 看一个具体的示例：

```ts
{
  enum A {
    one,
    two
  }
  enum B {
    one,
    two
  }
  const cpWithNumber = (param: A) => {
    if (param === 1) { // bad
      return param;
    }
  }
  const cpWithOtherEnum = (param: A) => {
    if (param === B.two as unknown as A) { // ALERT bad
      return param;
    }
  }
  const cpWithSelf = (param: A) => {
    if (param === A.two) { // good
      return param;
    }
  }
}
```

在第 10～14 行的函数 cpWithNumber 中，第 11 行我们将类型是枚举 A 的入参 param 和数字字面量 1 进行比较，因为 A 是数字枚举，所以 param 可以和 1 进行比较，而不会提示一个 ts(2367) 条件判断恒为 false 的错误。

因为数字枚举不稳定，所以默认情况下 A.two 的值会是 1，因此第 11 行的条件判断在入参为 A.two 的时候为真。但是，如果我们给枚举 A 的成员 one 指定初始值 1，第 11 行的条件判断在入参为 A.two 的时候就为否了，因为 A.two 值变成了 2，所以这不是一个安全的实践。

**在调用函数 cpWithNumber 的时候，我们使用数字类型做入参也是一种不安全的实践，原因同上。**

示例中第 15 ～ 19 行的函数 cpWithOtherEnum，我们使用了双重类型断言将枚举类型 B 转换为 A，主要是为了避免第 16 行提示一个 ts(2367) 错误，所以这同样也是一种不安全的实践。因为一旦 A 和 B 的结构出现了任何差异（比如给成员指定了不同的初始值、改变了成员的顺序或者个数），都会导致第 16 行的条件判断逻辑时真时否。

```
注意：有时候我们确实避免不了像示例中第 16 行这样使用双重类型断言来绕过 TypeScript 静态类型检测，比如使用基于同一个 Swagger 定义自动生成的两个枚举类型。此时，我们就需要极其谨慎，而且还需要添加警示信息进行说明，比如第 16 行添加的 "ALERT" 注释。
```

最安全的实践是使用第 21 行区分枚举成员的判断方式。

以上结论，同样适用于使用其他类型守卫（例如 switch）来区分枚举成员的场景。

⚠️ **注意**：字面量成员枚举可等价为字面量成员类型组成的联合类型，所以类型守卫可以让字面量成员枚举发生类型缩小。比如第 22 行中 param 的类型是 A.two，此时如果我们在 VS Code 中 hover 到 param 变量上，则会看到一个信息验证提示。



## 失效的类型守卫

> 失效的类型守卫指的是某些类型守卫应用在泛型函数中时不能缩小类型，即失效了。

比如我们改造了一个可以接受泛型入参的 getName 函数

```ts
const getName = <T extends Dog | Cat>(animal: T) => {
  if ('wang' in animal) {
    return animal.wang; // ts(2339)
  }
  return animal.miao; // ts(2339)
};
```

在上述示例中，虽然我们在第 2 行使用了 in 类型守卫，但是它并没有让 animal 的类型如预期那样缩小为 Dog 的子类型，所以第 3 行的 T 类型上没有 wang 属性，从而提示一个 ts(2339) 的错误。所以第 5 行的 animal 也不会缩小为 Cat 的子类型，从而也会提示一个 ts(2339) 的错误。

可一旦我们把 in 操作换成自定义类型守卫 isDog 或者使用 instanceOf，animal 的类型就会缩小成了 Dog 的子类型（T & Dog），所以第 3 行不会提示 ts(2339) 的错误。由此可见，in 和 instanceOf、类型谓词在泛型类型缩小上是有区别的。

```ts
const getName = <T extends Dog | Cat>(animal: T) => {
  if (isDog(animal)) { // instanceOf 亦可
    return animal.wang; // ok
  }
  return animal.miao; // ts(2339)
};
```

但是，在缺省的 else 条件分支里，animal 的类型并没有缩小成 Cat 的子类型，所以第 5 行依旧会提示一个 ts(2339) 的错误（这是一个不太科学的设计，所幸在 TypeScript 4.3.2 里已经修改了）。

这个时候，就需要使用类型断言，如下代码所示：

```ts
const getName = <T extends Dog | Cat>(animal: T) => {
  if (isDog(animal)) { // instanceOf 亦可
    return animal.wang; // ok
  }
  return (animal as Cat).miao; // ts(2339)
};
```

在第 5 行，我们把 animal 的类型断言为 Cat，并获取了它的 miao 属性。



# 类型兼容

TypeScript 中类型的兼容性都是基于结构化子类型的一般原则进行判定的。

## 子类型

<span style="font-weight:800;color:red">从子类型的角度来看，所有的子类型与它的父类型都兼容</span>

```ts
{
  const one = 1;
  let num: number = one; // ok
  interface IPar {
    name: string;
  }
  interface IChild extends IPar {
    id: number;
  }
  let Par: IPar;
  let Child: IChild;
  Par = Child; // ok
  class CPar {
    cname = '';
  }
  class CChild extends CPar {
    cid = 1;
  }
  let ParInst: CPar;
  let ChildInst: CChild;
  ParInst = ChildInst; // ok
  let mixedNum: 1 | 2 | 3 = one; // ok
}
```

在示例中的第 3 行，我们可以把类型是数字字面量类型的 one 赋值给数字类型的 num。在第 12 行，我们可以把子接口类型的变量赋值给 Par。在第 21 行，我们可以把子类实例 ChildInst 赋值给 ParInst。

因为成员类型兼容它所属的类型集合（其实联合类型和枚举都算类型集合，这里主要说的是联合类型），所以在示例中的第 22 行，我们可以把 one 赋值给包含字面类型 1 的联合类型。

举一反三，由子类型组成的联合类型也可以兼容它们父类型组成的联合类型

```ts
let ICPar: IPar | CPar;
let ICChild: IChild | CChild;
ICPar = ICChild; // ok
```

在示例中的第 3 行，因为 IChild 是 IPar 的子类，CChild 是 CPar 的子类，所以 IChild | CChild 也是 IPar | CPar 的子类，进而 ICChild 可以赋值给 ICPar。

## 结构类型

类型兼容性的另一准则是结构类型，即如果两个类型的结构一致，则它们是互相兼容的。比如拥有相同类型的属性、方法的接口类型或类，则可以互相赋值。

看一个具体的示例：

```ts
{
  class C1 {
    name = '1';
  }
  class C2 {
    name = '2';
  }
  interface I1 {
    name: string;
  }
  interface I2 {
    name: string;
  }
  let InstC1: C1;
  let InstC2: C2;
  let O1: I1;
  let O2: I2;
  InstC1 = InstC2; // ok
  O1 = O2; // ok
  InstC1 = O1; // ok
  O2 = InstC2; // ok
}
```

因为类 C1、类 C2、接口类型 I1、接口类型 I2 的结构完全一致，所以在第 18~19 行我们可以把类 C2 的实例 InstC2 赋值给类 C1 的实例 Inst1，把接口类型 I2 的变量 O2 赋值给接口类型 I1 的变量 O1。

在第 20~21 行，我们甚至可以把接口类型 I1 的变量 O1 赋值给类 C1 的实例，类 C2 的实例赋值给接口类型 I2 的变量 O2。

另外一个特殊的场景：两个接口类型或者类，如果其中一个类型不仅拥有另外一个类型全部的属性和方法，还包含其他的属性和方法（如同继承自另外一个类型的子类一样），那么前者是可以兼容后者的。

```ts
{
  interface I1 {
    name: string;
  }
  interface I2 {
    id: number;
    name: string;
  }
  class C2 {
    id = 1;
    name = '1';
  }
  let O1: I1;
  let O2: I2;
  let InstC2: C2;
  O1 = O2;
  O1 = InstC2;
}
```

在示例中的第 16~17 行，我们可以把类 C2 的实例 InstC2 和接口类型 I2 的变量 O2 赋值给接口类型 I1 的变量 O1，这是因为类 C2、接口类型 I2 和接口类型 I1 的 name 属性都是 string。不过，因为变量 O2、类 C2 都包含了额外的属性 id，所以我们不能把变量 O1 赋值给实例 InstC2、变量 O2。

这里涉及一个需要特别注意的特性：虽然包含多余属性 id 的变量 O2 可以赋值给变量 O1，但是如果我们直接将一个与变量 O2 完全一样结构的对象字面量赋值给变量 O1，则会提示一个 ts(2322) 类型不兼容的错误（如下示例第 2 行），这就是对象字面的 freshness 特性。

也就是说一个对象字面量没有被变量接收时，它将处于一种 freshness 新鲜的状态。这时 TypeScript 会对对象字面量的赋值操作进行严格的类型检测，只有目标变量的类型与对象字面量的类型完全一致时，对象字面量才可以赋值给目标变量，否则会提示类型错误。

当然，我们也可以通过使用变量接收对象字面量或使用类型断言解除 freshness，如下示例：

```ts
O1 = {
  id: 2, // ts(2322)
  name: 'name'
};
let O3 = {
  id: 2,
  name: 'name'
};
O1 = O3; // ok
O1 = {
  id: 2,
  name: 'name'
} as I2; // ok
```

在示例中，我们在第 5 行和第 13 行把包含多余属性的类型赋值给了变量 O1，没有提示类型错误。

另外，我们还需要注意类兼容性特性：实际上，在判断两个类是否兼容时，我们可以完全忽略其构造函数及静态属性和方法是否兼容，只需要比较类实例的属性和方法是否兼容即可。如果两个类包含私有、受保护的属性和方法，则仅当这些属性和方法源自同一个类，它们才兼容。

```ts
{
  class C1 {
    name = '1';
    private id = 1;
    protected age = 30;
  }
  class C2 {
    name = '2';
    private id = 1;
    protected age = 30;
  }
  let InstC1: C1;
  let InstC2: C2;
  InstC1 = InstC2; // ts(2322)
  InstC2 = InstC1; // ts(2322)
}
{
  class CPar {
    private id = 1;
    protected age = 30;
  }
  class C1 extends CPar {
    constructor(inital: string) {
      super();
    }
    name = '1';
    static gender = 'man';
  }
  class C2 extends CPar {
    constructor(inital: number) {
      super();
    }
    name = '2';
    static gender = 'woman';
  }
  let InstC1: C1;
  let InstC2: C2;
  InstC1 = InstC2; // ok
  InstC2 = InstC1; // ok
}
```

在示例中的第 14~15 行，因为类 C1 和类 C2 各自包含私有和受保护的属性，且实例 InstC1 和 InstC2 不能相互赋值，所以提示了一个 ts(2322) 类型的错误。

在第 38~39 行，因为类 C1、类 C2 的私有、受保护属性都继承自同一个父类 CPar，所以检测类型兼容性时会忽略其类型不相同的构造函数和静态属性 gender，也因此实例 InstC1 和 实例 InstC2 之间可以相互赋值。

## 可继承和可实现

类型兼容性还决定了接口类型和类是否可以通过 extends 继承另外一个接口类型或者类，以及类是否可以通过 implements 实现接口。

```ts
{
  interface I1 {
    name: number;
  }
  interface I2 extends I1 { // ts(2430)
    name: string;
  }
  class C1 {
    name = '1';
    private id = 1;
  }
  class C2 extends C1 { // ts(2415)
    name = '2';
    private id = 1;
  }
  class C3 implements I1 {
    name = ''; // ts(2416)
  }
}
```

在示例中的第 5 行，因为接口类型 I1 和接口类型 I2 包含不同类型的 name 属性不兼容，所以接口类型 I2 不能继承接口类型 I1。

同样，在第 12 行，因为类 C1 和类 C2 不满足类兼容条件，所以类 C2 也不能继承类 C1。

而在第 16 行，因为接口类型 I1 和类 C3 包含不同类型的 name 属性，所以类 C3 不能实现接口类型 I1。

## 泛型

> 泛型类型、泛型类的兼容性实际指的是将它们实例化为一个确切的类型后的兼容性。

[ts基础](./ts基础.md) 中我们介绍过可以通过指定类型入参实例化泛型，且入参只有作为实例化后的类型的一部分时才能影响类型兼容性，下面看一个具体的示例：

```ts
{
  interface I1<T> {
    id: number;
  }
  let O1: I1<string>;
  let O2: I1<number>;
  O1 = O2; // ol
}
```

在示例中的第 7 行，因为接口泛型 I1 的入参 T 是无用的，且实例化类型 I1\<string> 和 I1\<numer> 的结构一致，即类型兼容，所以对应的变量 O2 可以给变量 O1赋值。

而对于未明确指定类型入参泛型的兼容性，例如函数泛型（实际上仅有函数泛型才可以在不需要实例化泛型的情况下赋值），TypeScript 会把 any 类型作为所有未明确指定的入参类型实例化泛型，然后再检测其兼容性，如下代码所示：

```ts
{
  let fun1 = <T>(p1: T): 1 => 1;
  let fun2 = <T>(p2: T): number => 2;
  fun2 = fun1; // ok？
}
```

在示例中的第 4 行，实际上相当于在比较函数类型 (p1: any) => 1 和函数类型 (param: any) => number 的兼容性，那么这两个函数的类型兼容吗？答案：兼容。

### 变型

> 判定函数类型兼容性的基础理论知识

TypeScript 中的变型指的是根据类型之间的子类型关系推断基于它们构造的更复杂类型之间的子类型关系。
比如根据 Dog 类型是 Animal 类型子类型这样的关系，我们可以推断数组类型 Dog[] 和 Animal[] 、函数类型 () => Dog 和 () => Animal 之间的子类型关系。

在描述类型和基于类型构造的复杂类型之间的关系时，我们可以使用数学中函数的表达方式。
比如 Dog 类型，我们可以使用 F(Dog) 表示构造的复杂类型；F(Animal) 表示基于 Animal 构造的复杂类型。

这里的变型描述的就是基于 Dog 和 Animal 之间的子类型关系，从而得出 F(Dog) 和 F(Animal) 之间的子类型关系的一般性质。而这个性质体现为子类型关系可能会被保持、反转、忽略，因此它可以被划分为**协变、逆变、双向协变和不变**这 4 个专业术语。

#### 协变

协变也就是说如果 Dog 是 Animal 的子类型，则 F(Dog) 是 F(Animal) 的子类型，这意味着在构造的复杂类型中保持了一致的子类型关系

```ts
{
  type isChild<Child, Par> = Child extends Par ? true : false;
  interface Animal {
    name: string;
  }
  interface Dog extends Animal {
    woof: () => void;
  }
  type Covariance<T> = T;
  type isCovariant = isChild<Covariance<Dog>, Covariance<Animal>>; // true
}
```

在示例中的第 1 行，我们首先定义了一个用来判断两个类型入参 Child 和 Par 子类型关系的工具类型 isChild，如果 Child 是 Par 的子类型，那么 isChild 会返回布尔字面量类型 true，否则返回 false。

然后在第 3~8 行，我们定义了 Animal 类型和它的子类型 Dog。

在第 9 行，我们定义了泛型 Covariant 是一个复杂类型构造器，因为它原封不动返回了类型入参 T，所以对于构造出来的复杂类型 Covariant\<Dog> 和 Covariant\<Animal> 应该与类型入参 Dog 和 Animal 保持一致的子类型关系。

在第 10 行，因为 Covariant\<Dog> 是 Covariant\<Animal> 的子类型，所以类型 isCovariant 是 true，这就是**协变**。

实际上**接口类型的属性、数组类型、函数返回值的类型都是协变的**

```ts
type isPropAssignmentCovariant = isChild<{ type: Dog }, { type: Animal }>; // true
type isArrayElementCovariant = isChild<Dog[], Animal[]>; // true
type isReturnTypeCovariant  = isChild<() => Dog, () => Animal>; // true
```

在示例中的第1~3 行，我们看到 isPropAssignmentCovariant、isArrayElementCovariant、isReturnTypeCovariant 类型都是 true，即接口类型 { type: Dog } 是 { type: Animal } 的子类型，数组类型 Dog[] 是 Animal[] 的子类型，函数类型 () => Dog 也是 () => Animal 的子类型。

#### 逆变

逆变也就是说如果 Dog 是 Animal 的子类型，则 F(Dog) 是 F(Animal) 的父类型，这与协变正好反过来。

实际场景中，在我们推崇的 TypeScript 严格模式下，函数参数类型是逆变的

```ts
type Contravariance<T> = (param: T) => void;
type isNotContravariance = isChild<Contravariance<Dog>, Contravariance<Animal>>; // false;
type isContravariance = isChild<Contravariance<Animal>, Contravariance<Dog>>; // true;
```

在示例中的第 1 行，我们定义了一个基于类型入参构造函数类型的构造器 Contravariance，且类型入参 T 仅约束返回的函数类型参数 param 的类型。因为 TypeScript 严格模式的设定是函数参数类型是逆变的，所以 Contravariance\<Animal> 会是 Contravariance\<Dog> 的子类型，也因此第 2 行 isNotContravariance 是 false，第 3 行 isContravariance 是 true。

为了更易于理解，我们可以从安全性的角度理解函数参数是逆变的设定。

如果函数参数类型是协变而不是逆变，那么意味着函数类型 (param: Dog) => void 和 (param: Animal) => void 是兼容的，这与 Dog 和 Animal 的兼容一致，所以我们可以用 (param: Dog) => void 代替 (param: Animal) => void 遍历 Animal[] 类型数组。

但是，这样是不安全的，因为它不能确保 Animal[] 数组中的成员都是 Dog（可能混入 Animal 类型的其他子类型，比如 Cat），这就会导致 (param: Dog) => void 类型的函数可能接收到 Cat 类型的入参。

```ts
const visitDog = (animal: Dog) => {
   animal.woof();
 };
let animals: Animal[] = [{ name: 'Cat', miao: () => void 0, }];
animals.forEach(visitDog); // ts(2345)
```

在示例中，如果函数参数类型是协变的，那么第 5 行就可以通过静态类型检测，而不会提示一个 ts(2345) 类型的错误。这样第 1 行定义的 visitDog 函数在运行时就能接收到 Dog 类型之外的入参，并调用不存在的 woof 方法，从而在运行时抛出错误。

正是因为函数参数是逆变的，所以使用 visitDog 函数遍历 Animal[] 类型数组时，在第 5 行提示了类型错误，因此也就不出现 visitDog 接收到一只 cat 的情况。

#### 双向协变

双向协变也就是说如果 Dog 是 Animal 的子类型，则 F(Dog) 是 F(Animal) 的子类型，也是父类型，既是协变也是逆变。

对应到实际的场景，在 TypeScript 非严格模式下，函数参数类型就是双向协变的。**如前边提到函数只有在参数是逆变的情况下才安全，且本课程一直在强调使用严格模式，所以双向协变并不是一个安全或者有用的特性，因此我们不大可能遇到这样的实际场景**。

但在某些资料中有提到，如果函数参数类型是双向协变，那么它是有用的，并进行了举例论证 ：

```ts
interface Event {
  timestamp: number;
}
interface MouseEvent extends Event {
  x: number;
  y: number;
}
function addEventListener(handler: (n: Event) => void) {}
addEventListener((e: MouseEvent) => console.log(e.x + ',' + e.y)); // ts(2769)
```

在示例中，我们在第 4 行定义了接口 MouseEvent 是第 1 行定义的接口 Event 的子类型，在第 8 行定义了函数的 handler 参数是函数类型。如果参数类型是双向协变的，那么我们就可以在第 9 行把参数类型是 Event 子类型（比如说 MouseEvent 的函数）作为入参传给 addEventListener。

这种方式确实方便了很多，但是并不安全，原因见前边 Dog 和 Cat 的示例。而且在严格模式下，参数类型是逆变而不是双向协变的，所以第 9 行提示了一个 ts(2769) 的错误。

由此可以得出，真正有用且安全的做法是使用泛型，如下所示：

```ts
function addEventListener<E extends Event>(handler: (n: E) => void) {}
addEventListener((e: MouseEvent) => console.log(e.x + ',' + e.y)); // ok
```

在示例中的第 1 行，因为我们重新定义了带约束条件泛型入参的 addEventListener，它可以传递任何参数类型是 Event 子类型的函数作为入参，所以在第 2 行传入参数类型是 MouseEvent 的箭头函数作为入参时，则不会提示类型错误。

#### 不变

不变即只要是不完全一样的类型，它们一定是不兼容的。也就是说即便 Dog 是 Animal 的子类型，如果 F(Dog) 不是 F(Animal) 的子类型，那么 F(Animal) 也不是 F(Dog) 的子类型。

对应到实际场景，出于类型安全层面的考虑，在特定情况下我们可能希望数组是不变的（实际上是协变），见示例：

```ts
interface Cat extends Animal {
  miao: () => void; 
}
const cat: Cat = {
  name: 'Cat',
  miao: () => void 0,
};
const dog: Dog = {
  name: 'Dog',
  woof: () => void 0,
};
let dogs: Dog[] = [dog];
animals = dogs; // ok
animals.push(cat); // ok
dogs.forEach(visitDog); // 类型 ok，但运行时会抛出错误
```

在示例中的第 1~3 行，我们定义了一个 Animal 的另外一个子类 Cat。在第 4~8 行，我们分别定义了对象 cat 和对象 dog，并在第 12 行定义了 Dog[] 类型的数组 dogs。

因为数组是协变的，所以我们可以在第 13 行把 dogs 数组赋值给 animals 数组，并且在第 14 行把 cat 对象塞到 animals 数组中。那么问题就来了，因为 animals 和 dogs 指向的是同一个数组，所以实际上我们是把 cat 塞到了 dogs 数组中。

然后，我们在第 15 行使用了 visitDog 函数遍历 dogs 数组。虽然它可以通过静态类型检测，但是运行时 visitDog 遍历数组将接收一个混入的 cat 对象并抛出错误，因为 visitDog 中调用了 cat 上没有 woof 的方法。

因此，对于可变的数组而言，不变似乎是更安全、合理的设定。不过，在 TypeScript 中可变、不变的数组都是协变的，这是需要我们注意的一个陷阱。

介绍完变型相关的术语以及对应的实际场景，我们已经了解了函数参数类型是逆变的，返回值类型是协变的，所以前面的函数类型 (p1: any) => 1 和 (param: any) => number 为什么兼容的问题已经给出答案了。因为返回值类型 1 是 number 的子类型，且返回值类型是协变的，所以 (p1: any) => 1 是 (param: any) => number 的子类型，即是兼容的。

## 函数类型兼容性

因为函数类型的兼容性、子类型关系有着更复杂的考量（它还需要结合参数和返回值的类型进行确定），所以下面我们详细介绍一下函数类型兼容性的一般规则。

### 返回值

前边我们已经讲过返回值类型是协变的，所以在参数类型兼容的情况下，函数的子类型关系与返回值子类型关系一致。也就是说返回值类型兼容，则函数兼容。

### 参数类型

前边我们也讲过参数类型是逆变的，所以在参数个数相同、返回值类型兼容的情况下，函数子类型关系与参数子类型关系是反过来的（逆变）。

### 参数个数

在索引位置相同的参数和返回值类型兼容的前提下，函数兼容性取决于参数个数，参数个数少的兼容个数多

```ts
{
  let lessParams = (one: number) => void 0;
  let moreParams = (one: number, two: string) => void 0;
  lessParams = moreParams; // ts(2322)
  moreParams = lessParams; // ok
}
```

在示例中，lessParams 参数个数少于 moreParams，所以如第 5 行所示 lessParams 和 moreParams 兼容，并可以赋值给 moreParams。(可以试着从安全性角度理解（是参数少的函数赋值给参数多的函数安全，还是参数多的函数赋值给参数少的函数安全）)

### 可选和剩余参数

可选参数可以兼容剩余参数、不可选参数

```ts
let optionalParams = (one?: number, tow?: number) => void 0;
let requiredParams = (one: number, tow: number) => void 0;
let restParams = (...args: number[]) => void 0;
requiredParams = optionalParams; // ok
restParams = optionalParams; // ok
optionalParams = restParams; // ts(2322)
optionalParams = requiredParams; // ts(2322)
restParams = requiredParams; // ok
requiredParams = restParams; // ok
```

在示例中的第 4~5 行，我们可以把可选参数 optionalParams 赋值给不可选参数 requiredParams、剩余参数 restParams ，反过来则提示了一个 ts(2322) 的错误（第 5~6 行）。

在第 8~9 行，不可选参数 requiredParams 和剩余参数 restParams 是互相兼容的；从安全性的角度理解第 9 行是安全的，所以可以赋值。

最让人费解的是，在第 8 行中，把不可选参数 requiredParams 赋值给剩余参数 restParams 其实是不安全的（但是符合类型检测），我们需要从方便性上理解这个设定。

正是基于这个设定，我们才可以将剩余参数类型函数定义为其他所有参数类型函数的父类型，并用来约束其他类型函数的类型范围，比如说在泛型中约束函数类型入参的范围。

```ts
type GetFun<F extends (...args: number[]) => any> = Parameters<F>;
type GetRequiredParams = GetFun<typeof requiredParams>;
type GetRestParams = GetFun<typeof restParams>;
type GetEmptyParams = GetFun<() => void>;
```

在示例中的第 1 行，我们使用剩余参数函数类型 (...args: number[]) => any 约束了入参 F 的类型，而第 2~4 行传入的函数类型入参都是这个剩余参数函数类型的子类型。



#  TypeScript 增强类型系统

增强类型系统，顾名思义就是对 TypeScript 类型系统的增强。在 npm 中，有很多历史悠久的库都是使用 JavaScript 编写的，而 TypeScript 作为 JavaScript 的超集，设计目标之一就是能在 TypeScript 中安全、方便地使用 JavaScript 库。

TypeScript 相较于 JavaScript 而言，其一大特点就是类型。关于类型的定义方法，除了之前的内容之外，我们还可以通过以下方式扩展类型系统。

## 声明

那么，我们如何在 TypeScript 中安全地使用 JavaScript 的库呢？关键的步骤就是使用 TypeScript 中的一个 declare 关键字。

通过使用 declare 关键字，我们可以声明全局的变量、方法、类、对象。下面我们先说一下如何声明全局的变量。

### declare 变量

在运行时，前端代码 \<script> 标签会引入一个全局的库，再导入全局变量。此时，如果你想安全地使用全局变量，那么就需要对变量的类型进行声明。

声明变量的语法： declare (var|let|const) 变量名称: 变量类型 ，具体示例如下：

```ts
declare var val1: string;
declare let val2: number;
declare const val3: boolean;
val1 = '1';
val1 = '2';
val2 = 1;
val2 = '2'; // TS2322: Type 'string' is not assignable to type 'number'.
val3 = true; // TS2588: Cannot assign to 'val3' because it is a constant.
```

在上面的代码示例中，我们分别使用 var、let、const 声明了 3 种不同类型的变量。其中，使用 var、let 声明的变量是可以更改变量赋值的，而使用 const 声明的变量则不可以。同时，对于变量类型不正确的错误，TypeScript 也可以正常检测出来。

当然， declare 关键字还可以用来声明函数、类、枚举类型

### 声明函数

声明函数的语法与声明变量类型的语法相同，不同的是 declare 关键字后需要跟 function 关键字

```ts
declare function toString(x: number): string;
const x = toString(1); // => string
```

需要注意：**使用 declare关键字时，我们不需要编写声明的变量、函数、类的具体实现（因为变量、函数、类在其他库中已经实现了），只需要声明其类型即可**，如下示例：

```ts
// TS1183: An implementation cannot be declared in ambient contexts.
declare function toString(x: number) {
  return String(x);
};
```

在上面的例子中，TypeScript 的报错信息提示：环境声明的上下文不需要实现。也就是说 declare 声明的所有类型只需要表明类型，不需要实现。

### 声明类

声明类时，我们只需要声明类的属性、方法的类型即可。

另外，关于类的可见性修饰符我们也可以在此进行声明

```ts
declare class Person {
  public name: string;
  private age: number;
  constructor(name: string);
  getAge(): number;
}
const person = new Person('Mike');
person.name; // => string
person.age; // TS2341: Property 'age' is private and only accessible within class 'Person'.
person.getAge(); // => number
```

在上面的例子中，我们声明了公共属性 name 以及私有属性 age，此时我们看到无法访问私有属性 age。另外，我们还声明了方法 getAge ，并且 getAge 的返回值是 number 类型，所以 Person 实例调用后返回的类型也是 number 类型。

### 声明枚举

声明枚举只需要定义枚举的类型，并不需要定义枚举的值

```ts
declare enum Direction {
  Up,
  Down,
  Left,
  Right,
}
const directions = [Direction.Up, Direction.Down, Direction.Left, Direction.Right];
```

在上述示例中的第 1~6 行，我们声明了在其他地方定义的枚举 Direction 类型结构，然后在第 8 行就可以直接访问枚举的成员了

**注意：声明枚举仅用于编译时的检查，编译完成后，声明文件中的内容在编译结果中会被删除， **相当于仅剩下面使用的语句

```ts
const directions = [Direction.Up, Direction.Down, Direction.Left, Direction.Right];
```

这里的 Direction 表示引入的全局变量。

除了声明变量、函数、类型、枚举之外，我们还可以使用 declare 增强文件、模块的类型系统。

### declare 模块

在 JavaScript 还没有升级至 ES6 的时候，TypeScript 就提供了一种模块化方案，比如通过使用 module 关键字，我们就可以声明一个内部模块。但是由于 ES6 后来也使用了 module 关键字，为了兼容 ES6，所以 TypeScript 使用 namespace 替代了原来的 module，并更名为命名空间。

<span style="font-weight: 800; color: red">注意：目前，任何使用module关键字声明一个内部模块的地方，我们都应该使用namespace关键字进行替换。</span>

TypeScript 与 ES6 一样，任何包含顶级 import 或 export 的文件都会被当作一个模块。我们可以通过声明模块类型，为缺少 TypeScript 类型定义的三方库或者文件补齐类型定义，如下示例：

```ts
// lodash.d.ts
declare module 'lodash' {
  export function first<T extends unknown>(array: T[]): T;
}
// index.ts
import { first } from 'lodash';
first([1, 2, 3]); // => number;
```

在上面的例子中，lodash.d.ts 声明了模块 lodash 导出的 first 方法，然后在 TypeScript 文件中使用了模块 lodash 中的 first 方法。

声明模块的语法: `declare module '模块名' {}`

在模块声明的内部，我们只需要使用 export 导出对应库的类、函数即可。

### declare 文件

在使用 TypeScript 开发前端应用时，我们可以通过 import 关键字导入文件，比如先使用 import 导入图片文件，再通过 webpack 等工具处理导入的文件。

但是，因为 TypeScript 并不知道我们通过 import 导入的文件是什么类型，所以需要使用 declare 声明导入的文件类型

```ts
declare module '*.jpg' {
  const src: string;
  export default src;
}
declare module '*.png' {
  const src: string;
  export default src;
}
```

在上面的例子中，我们使用了 *.xxx 模块通配符匹配一类文件。

这里标记的图片文件的默认导出的类型是 string ，通过 import 使用图片资源时，TypeScript 会将导入的图片识别为 string 类型，因此也就可以把 import 的图片赋值给 的 src 属性，因为它们的类型都是 string，是匹配的。

### declare namespace

不同于声明模块，命名空间一般用来表示具有很多子属性或者方法的全局对象变量。

我们可以将声明命名空间简单看作是声明一个更复杂的变量

```ts
declare namespace $ {
  const version: number;
  function ajax(settings?: any): void;
}
$.version; // => number
$.ajax();
```

在上面的例子中，因为我们声明了全局导入的 jQuery 变量 $，所以可以直接使用 $ 变量的 version 属性以及 ajax 方法。

在 TypeScript 中，我们还可以编写以 .d.ts 为后缀的声明文件来增强（补齐）类型系统。



## 声明文件

在 TypeScript 中，以 .d.ts 为后缀的文件为声明文件。如果你熟悉 C/C++，那么可以把它当作 .h 文件。 在声明文件时，我们只需要定义三方类库所暴露的 API 接口即可。

在 TypeScript 中，存在类型、值、命名空间这 3 个核心概念。

### 类型

- 类型别名声明；

- 接口声明；

- 类声明；

- 枚举声明；

- 导入的类型声明。

上面的每一个声明都创建了一个类型名称。

### 值

值就是在运行时表达式可以赋予的值。

我们可以通过以下 6 种方式创建值：

- var、let、const 声明；

- namespace、module 包含值的声明；

- 枚举声明；

- 类声明；

- 导入的值；

- 函数声明。

### 命名空间

在命名空间中，我们也可以声明类型。比如 const x: A.B.C 这个声明，这里的类型 C 就是在 A.B 命名空间下的。

👁 说明：这种区别微妙而重要，这里的A.B可能代表一个值，也可能代表一个类型。

一个名称 A， 在 TypeScript 中可能表示一个类型、一个值，也可能是一个命名空间。通过类型、值、命名空间的组合，我们也就拥有了表达任意类型的能力。如果你想知道名称A 代表的实际意义，则需要看它所在的上下文。

### Demo

#### 使用声明文件

安装 TypeScript 依赖后，一般我们会顺带安装一个 lib.d.ts 声明文件，这个文件包含了 JavaScript 运行时以及 DOM 中各种全局变量的声明，如下示例：

```ts
// typescript/lib/lib.d.ts
/// <reference no-default-lib="true"/>
/// <reference lib="es5" />
/// <reference lib="dom" />
/// <reference lib="webworker.importscripts" />
/// <reference lib="scripthost" />
```

上面的示例其实就是 TypeScript 中的 lib.d.ts 文件的内容。

其中，/// 是 TypeScript 中三斜线指令，后面的内容类似于 XML 标签的语法，用来指代引用其他的声明文件。通过三斜线指令，我们可以更好地复用和拆分类型声明。no-default-lib="true" 表示这个文件是一个默认库。而最后 4 行的lib="..." 表示引用内部的库类型声明。

关于更多三斜线指令的内容，查看[链接](https://www.typescriptlang.org/docs/handbook/triple-slash-directives.html?fileGuid=xxQTRXtVcqtHK6j8)。

#### 使用 @types

为库编写类型声明非常耗费精力，且难以在多个项目中复用。[Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped?fileGuid=xxQTRXtVcqtHK6j8)是最流行性的高质量 TypeScript 声明文件类库，正是因为有社区维护的这个声明文件类库，大大简化了 JavaScript 项目迁移 TypeScript 的难度。

目前，社区已经记录了 90% 的 JavaScript 库的类型声明，意味着如果我们想使用的库有社区维护的类型声明，那么就可以通过安装类型声明文件直接使用 JavaScript 编写的类库了。

具体操作：首先，[点击这里的链接](https://www.typescriptlang.org/dt/search?search=&fileGuid=xxQTRXtVcqtHK6j8)搜索你想要导入的类库的类型声明，如果有社区维护的声明文件。然后，我们只需要安装 @types/xxx 就可以在 TypeScript 中直接使用它了。

然而，因为 Definitely Typed 是由社区人员维护的，如果原来的三方库升级，那么 Definitely Typed 所导出的三方库的类型定义想要升级还需要经过 PR、发布的流程，就会导致无法与原库保持完全同步。针对这个问题，在 TypeScript 中，我们可以通过类型合并、扩充类型定义的技巧临时解决。

## 类型合并

在 TypeScript 中，相同的接口、命名空间会依据一定的规则进行合并。

## 合并接口

最简单、常见的声明合并是接口合并，下面我们看一个具体的示例：

```ts
interface Person {
  name: string;
}
interface Person {
  age: number;
}
// 相当于
interface Person {
  name: string;
  age: number;
}
```

在上面的例子中，我们展示了接口合并最简单的情况，这里的合并相当于把接口的属性放入了一个同名的接口中。

**需要注意的是接口的非函数成员类型必须完全一样**，如下示例：

```ts
interface Person {
  age: string;
}
interface Person {
  // TS2717: Subsequent property declarations must have the same type.
  // Property 'age' must be of type 'string', but here has type 'number'.
  age: number;
}
```

在上面的例子中，因为存在两个属性相同而类型不同的接口，所以编译器报了一个 ts(2717) 错误 。

对于函数成员而言，每个同名的函数声明都会被当作这个函数的重载。

**需要注意的是后面声明的接口具有更高的优先级**，下面看一个具体的示例：

```ts
interface Obj {
    identity(val: any): any;
}
interface Obj {
    identity(val: number): number;
}
interface Obj {
    identity(val: boolean): boolean;
}
// 相当于
interface Obj {
  identity(val: boolean): boolean;
  identity(val: number): number;
  identity(val: any): any;
}
const obj: Obj = {
    identity(val: any) {
        return val;
    }
};
const t1 = obj.identity(1); // => number
const t2 = obj.identity(true); // => boolean
const t3 = obj.identity("t3"); // => any
```

在上面的代码中，Obj 类型的 identity 函数成员有 3 个重载，与函数重载的顺序相同，声明在前面的重载类型会匹配。我们分开声明接口的 3 个函数成员，相当于 12~16 行的声明，因为后声明的接口具有更高的优先级，所以 t1、t2 的类型可以被重载为其对应的类型，而不是 any。

接下来我们更改一下顺序，再看看结果。

```ts
interface Obj {
  identity(val: boolean): boolean;
}
interface Obj {
  identity(val: number): number;
}
interface Obj {
  identity(val: any): any;
}
// 相当于
interface Obj {
  identity(val: any): any;
  identity(val: number): number;
  identity(val: boolean): boolean;
}
const obj: Obj = {
  identity(val: any) {
      return val;
  }
};
const t1 = obj.identity(1); // => any
const t2 = obj.identity(true); // => any
const t3 = obj.identity("t3"); // => any
```

在上面的代码中，identity 函数参数为 any 的重载在第一位，因此 t1、t2、t3 的返回值类型都被重载成了 any。

## 合并 namespace

合并 namespace 与合并接口类似，命名空间的合并也会合并其导出成员的属性。不同的是，非导出成员仅在原命名空间内可见。

下面看一个具体的示例：

```ts
namespace Person {
  const age = 18;
  export function getAge() {
    return age;
  }
}
namespace Person {
  export function getMyAge() {
    return age; // TS2304: Cannot find name 'age'.
  }
}
```

在上面的例子，同名的命名空间 Person 中，有一个非导出的属性 age，在第二个命名空间 Person 中没有 age 属性却引用了 age，所以 TypeScript 报出了找不到 age 的错误。这是因为非导出成员仅在合并前的命名空间中可见，上例中即 1~6 行的命名空间中可以访问 age 属性。但是对于 8~12 行中的同名命名空间是不可以访问 age 属性的。

## 不可合并

我们在[ts基础](./ts基础.md) 中知道定义一个类类型，相当于定义了一个类，又定义了一个类的类型。因此，对于类这个既是值又是类型的特殊对象不能合并。

除了可以通过接口和命名空间合并的方式扩展原来声明的类型外，我们还可以通过扩展模块或扩展全局对象来增强类型系统。

## 扩充模块

JavaScript 是一门动态类型的语言，通过 prototype 我们可以很容易地扩展原来的对象。

但是，如果我们直接扩展导入对象的原型链属性，TypeScript 会提示没有该属性的错误，因此我们就需要扩展原模块的属性。

下面看一个具体的示例：

```ts
// person.ts
export class Person {}
// index.ts
import { Person } from './person';
declare module './person' {
  interface Person {
    greet: () => void;
  }
}
Person.prototype.greet = () => {
  console.log('Hi!');
};
```

在上面的例子中，我们声明了导入模块 person 中 Person 的属性，TypeScript 会与原模块的类型合并，通过这种方式我们可以扩展导入模块的类型。同时，我们为导入的 Person 类增加了原型链上的 greet 方法。

```ts
// person.ts
export class Person {}

// index.ts
import { Person } from './person';

declare module './person' {
  interface Person {
    greet: () => void;
  }
}

- declare module './person' {
-   interface Person {
-     greet: () => void;
-   }
- }

+ // TS2339: Property 'greet' does not exist on type 'Person'.
Person.prototype.greet = () => {
  console.log('Hi!');
};
```

在上面的例子中，如果我们删除了扩展模块的声明，第 20 行则会报出 ts(2339) 不存在 greet 属性的类型错误。

对于导入的三方模块，我们同样可以使用这个方法扩充原模块的属性。

## 扩充全局

全局模块指的是不需要通过 import 导入即可使用的模块，如全局的 window、document 等。

对全局对象的扩充与对模块的扩充是一样的，下面看一个具体示例：

```ts
declare global {
  interface Array<T extends unknown> {
    getLen(): number;
  }
}
Array.prototype.getLen = function () {
  return this.length;
};
```

在上面的例子中，因为我们声明了全局的 Array 对象有一个 getLen 方法，所以为 Array 对象实现 getLen 方法时，TypeScript 不会报错。



# TypeScript 全局工具类型

## 操作接口类型

### Partial

> Partial 工具类型可以将一个类型的所有属性变为可选的，且该工具类型返回的类型是给定类型的所有子集

```ts
type Partial<T> = {
  [P in keyof T]?: T[P];
};
interface Person {
  name: string;
  age?: number;
  weight?: number;
}
type PartialPerson = Partial<Person>;
// 相当于
interface PartialPerson {
  name?: string;
  age?: number;
  weight?: number;
}
```

在上述示例中，我们使用映射类型取出了传入类型的所有键值，并将其值设定为可选的。

### Required

> Required 工具类型可以将给定类型的所有属性变为必填的

```ts
type Required<T> = {
  [P in keyof T]-?: T[P];
};
type RequiredPerson = Required<Person>;
// 相当于
interface RequiredPerson {
  name: string;
  age: number;
  weight: number;
}
```

在上述示例中，映射类型在键值的后面使用了一个 - 符号，- 与 ? 组合起来表示去除类型的可选属性，因此给定类型的所有属性都变为了必填。

### Readonly

> Readonly 工具类型可以将给定类型的所有属性设为只读，这意味着给定类型的属性不可以被重新赋值

```ts
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};
type ReadonlyPerson = Readonly<Person>;
// 相当于
interface ReadonlyPerson {
  readonly name: string;
  readonly age?: number;
  readonly weight?: number;
}
```

在上述示例中，经过 Readonly 处理后，ReadonlyPerson 的 name、age、weight 等属性都变成了 readonly 只读。

### Pick

> Pick 工具类型可以从给定的类型中选取出指定的键值，然后组成一个新的类型

```ts
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};
type NewPerson = Pick<Person, 'name' | 'age'>;
// 相当于
interface NewPerson {
  name: string;
  age?: number;
}
```

在上述示例中，Pick工具类型接收了两个泛型参数，第一个 T 为给定的参数类型，而第二个参数为需要提取的键值 key。有了参数类型和需要提取的键值 key，我们就可以通过映射类型很容易地实现 Pick 工具类型的功能。

### Omit

> Omit 工具类型的功能是返回去除指定的键值之后返回的新类型

```ts
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
type NewPerson = Omit<Person, 'weight'>;
// 相当于
interface NewPerson {
  name: string;
  age?: number;
}
```

在上述示例中，Omit 类型的实现使用了前面介绍的 Pick 类型。我们知道 Pick 类型的作用是选取给定类型的指定属性，那么这里的 Omit 的作用应该是选取除了指定属性之外的属性，而 Exclude 工具类型的作用就是从入参 T 属性的联合类型中排除入参 K 指定的若干属性。

<span style="font-weight:800; color: red">Tips：</span>操作接口类型这里的工具类型都使用了映射类型。通过映射类型，我们可以对原类型的属性进行重新映射，从而组成想要的类型。



## 联合类型

### Exclude

通过使用 Exclude 类型，我们从接口的所有属性中去除了指定属性，因此，Exclude 的作用就是从联合类型中去除指定的类型。

```TS
type Exclude<T, U> = T extends U ? never : T;
type T = Exclude<'a' | 'b' | 'c', 'a'>; // => 'b' | 'c'
type NewPerson = Omit<Person, 'weight'>;
// 相当于
type NewPerson = Pick<Person, Exclude<keyof Person, 'weight'>>;
// 其中
type ExcludeKeys = Exclude<keyof Person, 'weight'>; // => 'name' | 'age'
```

在上述示例中，Exclude 的实现使用了条件类型。如果类型 T 可被分配给类型 U ，则不返回类型 T，否则返回此类型 T ，这样我们就从联合类型中去除了指定的类型。

再回看之前的 NewPerson 类型的例子，我们也就很明白了。在 ExcludeKeys 中，如果 Person 类型的属性是我们要去除的属性，则不返回该属性，否则返回其类型。

### Extract

Extract 主要用来从联合类型中提取指定的类型，类似于操作接口类型中的 Pick 类型。

```ts
type Extract<T, U> = T extends U ? T : never;
type T = Extract<'a' | 'b' | 'c', 'a'>; // => 'a'
```

通过上述示例，我们发现 Extract 类型相当于取出两个联合类型的交集。

此外，我们还可以基于 Extract 实现一个获取接口类型交集的工具类型

```ts
type Intersect<T, U> = {
  [K in Extract<keyof T, keyof U>]: T[K];
};
interface Person {
  name: string;
  age?: number;
  weight?: number;
}
interface NewPerson {
  name: string;
  age?: number;
}
type T = Intersect<Person, NewPerson>;
// 相当于
type T = {
  name: string;
  age?: number;
};
```

在上述的例子中，我们使用了 Extract 类型来提取两个接口类型属性的交集，并使用映射类型生成了一个新的类型。

#### NonNullable

> NonNullable 的作用是从联合类型中去除 null 或者 undefined 的类型。

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
// 等同于使用 Exclude
type NonNullable<T> = Exclude<T, null | undefined>;
type T = NonNullable<string | number | undefined | null>; // => string | number
```

在上述示例中，如果 NonNullable 传入的类型可以被分配给 null 或是 undefined ，则不返回该类型，否则返回其具体类型。

#### Record

> Record 的作用是生成接口类型，然后我们使用传入的泛型参数分别作为接口类型的属性和值。

```ts
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
type MenuKey = 'home' | 'about' | 'more';
interface Menu {
  label: string;
  hidden?: boolean;
}
const menus: Record<MenuKey, Menu> = {
  about: { label: '关于' },
  home: { label: '主页' },
  more: { label: '更多', hidden: true },
};
```

在上述示例中，Record 类型接收了两个泛型参数：第一个参数作为接口类型的属性，第二个参数作为接口类型的属性值。

⭐️ 需要注意：这里的实现限定了第一个泛型参数继承自`keyof any`

在 TypeScript 中，keyof any 指代可以作为对象键的属性，如下示例：

```ts
type T = keyof any; // => string | number | symbol
```

说明：目前，JavaScript 仅支持string、number、symbol作为对象的键值。



## 函数类型

### ConstructorParameters

ConstructorParameters 可以用来获取构造函数的构造参数，而 ConstructorParameters 类型的实现则需要使用 infer 关键字推断构造参数的类型。

关于 infer 关键字，我们可以把它当成简单的模式匹配来看待。如果真实的参数类型和 infer 匹配的一致，那么就返回匹配到的这个类型。

```ts
type ConstructorParameters<T extends new (...args: any) => any> = T extends new (
  ...args: infer P
) => any
  ? P
  : never;
class Person {
  constructor(name: string, age?: number) {}
}
type T = ConstructorParameters<typeof Person>; // [name: string, age?: number]
```

在上述示例中，ConstructorParameters 泛型接收了一个参数，并且限制了这个参数需要实现构造函数。于是，我们通过 infer 关键字匹配了构造函数内的构造参数，并返回了这些参数。因此，可以看到第 11 行匹配了 Person 构造函数的两个参数，并返回了一个元组类型 [string, number] 给类型别名 T。

### Parameters

Parameters 的作用与 ConstructorParameters 类似，Parameters 可以用来获取函数的参数并返回序对

```ts
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
type T0 = Parameters<() => void>; // []
type T1 = Parameters<(x: number, y?: string) => void>; // [x: number, y?: string]
```

在上述示例中，Parameters 的泛型参数限制了传入的类型需要满足函数类型。

### ReturnType

ReturnType 的作用是用来获取函数的返回类型

```ts
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
type T0 = ReturnType<() => void>; // => void
type T1 = ReturnType<() => string>; // => string
```

在上述示例中，`ReturnType`的泛型参数限制了传入的类型需要满足函数类型。

### ThisParameterType

ThisParameterType 可以用来获取函数的 this 参数类型。

```ts
type ThisParameterType<T> = T extends (this: infer U, ...args: any[]) => any ? U : unknown;
type T = ThisParameterType<(this: Number, x: number) => void>; // Number
```

在上述示例的第 1 行中，因为函数类型的第一个参数声明的是 this 参数类型，所以我们可以直接使用 infer 关键字进行匹配并获取 this 参数类型。在示例的第 2 行，类型别名 T 得到的类型就是 Number。

### ThisType

ThisType 的作用是可以在对象字面量中指定 this 的类型。ThisType 不返回转换后的类型，而是通过 ThisType 的泛型参数指定 this 的类型

😮 注意：如果你想使用这个工具类型，那么需要开启noImplicitThis的 TypeScript 配置。

```ts
type ObjectDescriptor<D, M> = {
  data?: D;
  methods?: M & ThisType<D & M>; // methods 中 this 的类型是 D & M
};
function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {};
  let methods: object = desc.methods || {};
  return { ...data, ...methods } as D & M;
}
const obj = makeObject({
  data: { x: 0, y: 0 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx; // this => D & M
      this.y += dy; // this => D & M
    },
  },
});
obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```

在上述示例子中，methods 属性的 this 类型为 D & M，在上下文中指代 { x: number, y: number } & { moveBy(dx: number, dy: number): void }。

ThisType 工具类型只是提供了一个空的泛型接口，仅可以在对象字面量上下文中被 TypeScript 识别

```ts
interface ThisType<T> {}
```

也就是说该类型的作用相当于任意空接口。

### OmitThisParameter

OmitThisParameter 工具类型主要用来去除函数类型中的 this 类型。如果传入的函数类型没有显式声明 this 类型，那么返回的仍是原来的函数类型。

```ts
type OmitThisParameter<T> = unknown extends ThisParameterType<T>
  ? T
  : T extends (...args: infer A) => infer R
  ? (...args: A) => R
  : T;
type T = OmitThisParameter<(this: Number, x: number) => string>; // (x: number) => string
```

在上述示例中， ThisParameterType 类型的实现如果传入的泛型参数无法推断 this 的类型，则会返回 unknown 类型。在OmitThisParameter 的实现中，第一个条件语句如果传入的函数参数没有 this 类型，则返回原类型；否则通过 infer 分别获取函数参数和返回值的类型构造一个新的没有 this 的函数类型，并返回这个函数类型。

## 字符串类型

### 模板字符串

TypeScript 自 4.1版本起开始支持模板字符串字面量类型。为此，TypeScript 也提供了 Uppercase、Lowercase、Capitalize、Uncapitalize这 4 种内置的操作字符串的类型，如下示例：

```ts
// 转换字符串字面量到大写字母
type Uppercase<S extends string> = intrinsic;
// 转换字符串字面量到小写字母
type Lowercase<S extends string> = intrinsic;
// 转换字符串字面量的第一个字母为大写字母
type Capitalize<S extends string> = intrinsic;
// 转换字符串字面量的第一个字母为小写字母
type Uncapitalize<S extends string> = intrinsic;
type T0 = Uppercase<'Hello'>; // => 'HELLO'
type T1 = Lowercase<T0>; // => 'hello'
type T2 = Capitalize<T1>; // => 'Hello'
type T3 = Uncapitalize<T2>; // => 'hello'
```

在上述示例中，这 4 种操作字符串字面量工具类型的实现都是使用 JavaScript 运行时的字符串操作函数计算出来的，且不支持语言区域设置。以下代码是这 4 种字符串工具类型的实际实现。

```ts
function applyStringMapping(symbol: Symbol, str: string) {
  switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
    case IntrinsicTypeKind.Uppercase:
      return str.toUpperCase();
    case IntrinsicTypeKind.Lowercase:
      return str.toLowerCase();
    case IntrinsicTypeKind.Capitalize:
      return str.charAt(0).toUpperCase() + str.slice(1);
    case IntrinsicTypeKind.Uncapitalize:
      return str.charAt(0).toLowerCase() + str.slice(1);
  }
  return str;
}
```

在上述代码中可以看到，字符串的转换使用了 JavaScript 中字符串的 toUpperCase 和 toLowerCase 方法，而不是 toLocaleUpperCase 和 toLocaleLowerCase。其中 toUpperCase 和 toLowerCase 采用的是 Unicode 编码默认的大小写转换规则。
