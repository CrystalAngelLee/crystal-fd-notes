# Events 基本介绍

Node.js的events 模块对外提供了一个 EventEmitter 对象，用于对 Node.js 中的事件进行统一管理。因为 Node.js 采用了事件驱动机制，而 EventEmitter 就是 Node.js 实现事件驱动的基础。在 EventEmitter 的基础上，Node.js 中几乎所有的模块都继承了这个类，以实现异步事件驱动架构。

来看下 EventEmitter的简单使用情况

```js
var events = require('events');
var eventEmitter = new events.EventEmitter();
eventEmitter.on('say',function(name){
    console.log('Hello',name);
})
eventEmitter.emit('say','Jonh');
```

以上代码中，新定义的eventEmitter 是接收 events.EventEmitter 模块 new 之后返回的一个实例，eventEmitter 的 emit 方法，发出 say 事件，通过 eventEmitter 的 on 方法监听，从而执行相应的函数。

这就是 Node.js的events 模块中 EventEmitter 的基本用法，下面来说说 EventEmitter 的各种方法以及功能的介绍。



# 常用的 EventEmitter 模块的 API

<table>
	<tr style="text-align: center">
		<th>方法</th>
		<th>方法描述</th>
	</tr>
	<tr>
		<td>addListener(event,listener)</td>
		<td>为指定事件添加一个监听器到监听器数组的尾部</td>
	</tr>
	<tr>
		<td>prependListener(event,listener)</td>
		<td>与addListener相对，为指定事件添加一个监听器到监听器数组的头部</td>
	</tr>
	<tr>
		<td>on(event,listener)</td>
		<td>addListener的别名</td>
	</tr>
	<tr>
		<td>once(event,listener)</td>
		<td>为指定事件注册一个单次监听器，即监听器最多只会触发一次，触发后立刻解除该监听器</td>
	</tr>
	<tr>
		<td>removeListener(event,listener)</td>
		<td>移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器</td>
	</tr>
	<tr>
		<td>off(event,listener)</td>
		<td>removeListener的别名</td>
	</tr>
	<tr>
		<td>removeAllListener([event])</td>
		<td>移除所有事件的所有监听器，如果指定事件，则移除指定事件的所有监听器</td>
	</tr>
	<tr>
		<td>setMaxListeners(n)</td>
		<td>默认情况下，EventEmitters中如果添加的监听器超过10个就会输出警告信息；setMaxListeners函数用于提高监听器的默认限制的数量</td>
	</tr>
	<tr>
		<td>listeners(event)</td>
		<td>返回指定事件的监听器数组</td>
	</tr>
	<tr>
		<td>emit(event, [arg1],[arg2],[...])</td>
		<td>按参数的顺序执行每个监听器，如果事件有注册监听返回true，否则返回false</td>
	</tr>
</table>

除此之外，还有两个特殊的事件，不需要额外手动添加

| 事件           | 事件描述                                                     |
| -------------- | ------------------------------------------------------------ |
| newListener    | 该事件在添加新事件监听器的时候触发                           |
| removeListener | 从指定监听器数组中删除一个监听器。需要注意的是，此操作将会改变处于被删监听器之后的那些监听器的索引。 |

## addListener 和 removeListener、on 和 off 方法对比

addListener 方法的作用是为指定事件添加一个监听器，其实和 on 方法实现的功能是一样的，on 其实就是 addListener 方法的一个别名。二者实现的作用是一样的，同时 removeListener 方法的作用是为移除某个事件的监听器，同样 off 也是 removeListener 的别名。

❀ **addListener 和removeListener**

```javascript
var events = require('events');
var emitter = new events.EventEmitter();
function hello1(name){
  console.log("hello 1",name);
}
function hello2(name){
  console.log("hello 2",name);
}
emitter.addListener('say',hello1);
emitter.addListener('say',hello2);
emitter.emit('say','John');
//输出hello 1 John 
//输出hello 2 John
emitter.removeListener('say',hello1);
emitter.emit('say','John');
//相应的，监听say事件的hello1事件被移除
//只输出hello 2 John
```

## removeListener 和 removeAllListeners

removeListener 方法是指移除一个指定事件的某一个监听器，而 removeAllListeners 指的是移除某一个指定事件的全部监听器。

```javascript
var events = require('events');
var emitter = new events.EventEmitter();
function hello1(name){
  console.log("hello 1",name);
}
function hello2(name){
  console.log("hello 2",name);
}
emitter.addListener('say',hello1);
emitter.addListener('say',hello2);
emitter.removeAllListeners('say');
emitter.emit('say','John');
//removeAllListeners 移除了所有关于 say 事件的监听
//因此没有任何输出
```

## on 和 once 方法区别

on 和 once 的区别是：on 的方法对于某一指定事件添加的监听器可以持续不断地监听相应的事件；而 once 方法添加的监听器，监听一次后，就会被消除。

```javascript
var events = require('events');
var emitter = new events.EventEmitter();
function hello(name){
  console.log("hello",name);
}
emitter.on('say',hello);
emitter.emit('say','John');
emitter.emit('say','Lily');
emitter.emit('say','Lucy');
//会输出 hello John、hello Lily、hello Lucy，之后还要加也可以继续触发
emitter.once('see',hello);
emitter.emit('see','Tom');
//只会输出一次 hello Tom
```

on 方法监听的事件，可以持续不断地被触发，而 once 方法只会触发一次。



# 手动实现一个 EventEmitter

[详细实现详见☞](../javascript/EventEmitter.md)

```javascript
// https://gitee.com/CrystalAngelLee/handwritting/blob/master/event-emitter-handlewritting/index.js
function EventEmitter() {
  this.__events = {}; // 初始化对象：存放自定义事件&自定义事件的回调函数
}

EventEmitter.prototype.on = function(eventName, listener) {
  if (!eventName || !listener) return;
  if (!isValidListener(listener)) {
    throw new TypeError('listener must be a function')
  }
  var events = this.__events;
  var listeners = events[eventName] = events[eventName] || [];
  var listenerIsWrapped = typeof listener === 'object';

  if (indexOf(listeners, listener) === -1) {
    listeners.push(listenerIsWrapped ? listener : {
      listener,
      once: false
    })
  }

  return this
}

EventEmitter.prototype.emit = function(eventName, args) {
  var listeners = this.__events[eventName];
  if (!listeners) return;
  for (var i = 0, len = listeners.length; i < len; i++) {
    var listener = listeners[i];
    if (!listener) break;
    listener.listener.apply(this, args || []);
    if (listener.once) this.off(eventName, listener.listener)
  }
  return this
}

EventEmitter.prototype.off = function(eventName, listener) {
  var listeners = this.__events[eventName];
  if (!listeners) return;
  var index;
  for (var i = 0, len = listeners.length; i < len; i++) {
    var _listener = listeners[i];
    if (_listener && _listener.listener === listener) {
      index = i;
      break;
    }
  }
  if (typeof index !== 'undefined') {
    listeners.splice(index, 1, null)
  }
  return this;
}

EventEmitter.prototype.once = function(eventName, listener) {
  // 直接调用 on 方法，once 参数传入 true，待执行之后进行 once 处理
  return this.on(eventName, {
    listener: listener,
    once: true
  })
}

EventEmitter.prototype.allOff = function(eventName) {
  if (eventName && this.__events[eventName]) {
    this.__events[eventName] = [];
  } else {
    this.__events = {}
  }
}

// 判断是否是合法的listener
function isValidListener(listener) {
  if (typeof listener === 'function') return true;
  if (listener && typeof listener === 'object') return isValidListener(listener.listener)
  return false
}

// 判断新增自定义事件是否存在
function indexOf(array, item) {
  var result = -1
  item = typeof item === 'object' ? item.listener : item;
  for (var i = 0, len = array.length; i < len; i++) {
    if (array[i].listener === item) {
      result = i;
      break;
    }
  }
  return result;
}
```