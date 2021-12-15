# 方法基本介绍
JSON.stringify 是日常开发中经常用到的 JSON 对象中的一个方法，JSON 对象包含两个方法：一是用于解析成 JSON 对象的 parse()；二是用于将对象转换为 JSON 字符串方法的 stringify()

## [JSON.parse](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)

JSON.parse() 方法用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的 reviver 函数用以在返回之前对所得到的对象执行变换(操作)。

> 语法为：JSON.parse(text[, reviver])

```javascript
const json = '{"result":true, "count":42}';
const obj = JSON.parse(json);

console.log(obj.count);
// expected output: 42

console.log(obj.result);
// expected output: true
-------
JSON.parse('{"p": 5}', function (k, v) {
    if(k === '') return v;     // 如果到了最顶层，则直接返回属性值，
    return v * 2;              // 否则将属性值变为原来的 2 倍。
});                            // { p: 10 }
```



## [JSON.stringify](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)

JSON.stringify 方法是将一个 JavaScript 对象或值转换为 JSON 字符串，默认该方法其实有三个参数：第一个参数是必选，后面两个是可选参数非必选。第一个参数传入的是要转换的对象；第二个是一个 replacer 函数，比如指定的 replacer 是数组，则可选择性地仅处理包含数组指定的属性；第三个参数用来控制结果字符串里面的间距，后面两个参数整体用得比较少。

> 语法为：JSON.stringify(value[, replacer [, space]])

```js
JSON.stringify({ x: 1, y: 2 });
// "{"x":1,"y":2}"
JSON.stringify({ x: [10, undefined, function(){}, Symbol('')] })
// "{"x":[10,null,null,null]}"
/* 第二个参数的例子 */
function replacer(key, value) {
  if (typeof value === "string") {
    return undefined;
  }
  return value;
}
var foo = {foundation: "Mozilla", model: "box", week: 4, transport: "car", month: 7};
var jsonString = JSON.stringify(foo, replacer);
console.log(jsonString);
// "{"week":4,"month":7}"
/* 第三个参数的例子 */
JSON.stringify({ a: 2 }, null, " ");
/* "{
 "a": 2
}"*/
JSON.stringify({ a: 2 }, null, "");
// "{"a":2}"
```


# 手动实现一个JSON.Stringify

```javascript
function jsonStringify(data) {
  let type = typeof data;

  if(type !== 'object') {
    let result = data;
    //data 可能是基础数据类型的情况在这里处理
    if (Number.isNaN(data) || data === Infinity) {
       //NaN 和 Infinity 序列化返回 "null"
       result = "null";
    } else if (type === 'function' || type === 'undefined' || type === 'symbol') {
      // 由于 function 序列化返回 undefined，因此和 undefined、symbol 一起处理
       return undefined;
    } else if (type === 'string') {
       result = '"' + data + '"';
    }
    return String(result);
  } else if (type === 'object') {
     if (data === null) {
        return "null"  // 第01讲有讲过 typeof null 为'object'的特殊情况
     } else if (data.toJSON && typeof data.toJSON === 'function') {
        return jsonStringify(data.toJSON());
     } else if (data instanceof Array) {
        let result = [];
        //如果是数组，那么数组里面的每一项类型又有可能是多样的
        data.forEach((item, index) => {
        if (typeof item === 'undefined' || typeof item === 'function' || typeof item === 'symbol') {
               result[index] = "null";
           } else {
               result[index] = jsonStringify(item);
           }
         });
         result = "[" + result + "]";
         return result.replace(/'/g, '"');
      } else {
         // 处理普通对象
         let result = [];
         Object.keys(data).forEach((item, index) => {
            if (typeof item !== 'symbol') {
              //key 如果是 symbol 对象，忽略
              if (data[item] !== undefined && typeof data[item] !== 'function' && typeof data[item] !== 'symbol') {
                //键值如果是 undefined、function、symbol 为属性值，忽略
                result.push('"' + item + '"' + ":" + jsonStringify(data[item]));
              }
            }
         });
         return ("{" + result + "}").replace(/'/g, '"');
        }
    }
}
```
测试：

```javascript
let nl = null;
console.log(jsonStringify(nl) === JSON.stringify(nl));
// true
let und = undefined;
console.log(jsonStringify(undefined) === JSON.stringify(undefined));
// true
let boo = false;
console.log(jsonStringify(boo) === JSON.stringify(boo));
// true
let nan = NaN;
console.log(jsonStringify(nan) === JSON.stringify(nan));
// true
let inf = Infinity;
console.log(jsonStringify(Infinity) === JSON.stringify(Infinity));
// true
let str = "jack";
console.log(jsonStringify(str) === JSON.stringify(str));
// true
let reg = new RegExp("\w");
console.log(jsonStringify(reg) === JSON.stringify(reg));
// true
let date = new Date();
console.log(jsonStringify(date) === JSON.stringify(date));
// true
let sym = Symbol(1);
console.log(jsonStringify(sym) === JSON.stringify(sym));
// true
let array = [1,2,3];
console.log(jsonStringify(array) === JSON.stringify(array));
// true
let obj = {
    name: 'jack',
    age: 18,
    attr: ['coding', 123],
    date: new Date(),
    uni: Symbol(2),
    sayHi: function() {
        console.log("hi")
    },
    info: {
        sister: 'lily',
        age: 16,
        intro: {
            money: undefined,
            job: null
        }
    }
}
console.log(jsonStringify(obj) === JSON.stringify(obj));
// true
```
注：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210127141751298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzMzNTkxNzE1,size_16,color_FFFFFF,t_70)

1. 由于 function 返回 'null'， 并且 typeof function 能直接返回精确的判断，故在整体逻辑处理基础数据类型的时候，会随着 undefined，symbol 直接处理了；
2. 由于 typeof null 的时候返回'object'，故 null 的判断逻辑整体在处理引用数据类型的逻辑里面；
3. 关于引用数据类型中的数组，由于数组的每一项的数据类型又有很多的可能性，故在处理数组过程中又将 undefined，symbol，function 作为数组其中一项的情况做了特殊处理；
4. 同样在最后处理普通对象的时候，key （键值）也存在和数组一样的问题，故又需要再针对上面这几种情况（undefined，symbol，function）做特殊处理；
5. 最后在处理普通对象过程中，对于循环引用的问题暂未做检测，如果是有循环引用的情况，需要抛出 Error；
6. 根据官方给出的 JSON.stringify 的第二个以及第三个参数的实现，本段模拟实现的代码并未实现