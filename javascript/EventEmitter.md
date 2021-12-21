[EventEmitter 是在Node.js 中 events 模块里封装的](../nodeJs/events.md)，我们今天自己封装一个能在浏览器中跑的EventEmitter，可以帮你实现自定义事件的订阅和发布，从而提升业务开发的便利性。

> 实现一个基础版本的EventEmitter，包含基础的on、 of、emit、once、allof 这几个方法。



# EventEmitter的初始化代码

先初始化一个内部的__events 的对象，用来存放自定义事件，以及自定义事件的回调函数。

```js
function EventEmitter() {
    this.__events = {}
}
EventEmitter.VERSION = '1.0.0';
```



# 实现 EventEmitter的 on 的方法

on 方法的核心思路：当调用订阅一个自定义事件的时候，只要该事件通过校验合法之后，就把该自定义事件 push 到 this.__events 这个对象中存储，等需要出发的时候，则直接从通过获取 __events 中对应事件的 listener 回调函数，而后直接执行该回调方法就能实现想要的效果。

```js
EventEmitter.prototype.on = function(eventName, listener){
	  if (!eventName || !listener) return;
      // 判断回调的 listener 是否为函数
	  if (!isValidListener(listener)) {
	       throw new TypeError('listener must be a function');
	  }
	   var events = this.__events;
	   var listeners = events[eventName] = events[eventName] || [];
	   var listenerIsWrapped = typeof listener === 'object';
       // 不重复添加事件，判断是否有一样的
       if (indexOf(listeners, listener) === -1) {
           listeners.push(listenerIsWrapped ? listener : {
               listener: listener,
               once: false
           });
       }
	   return this;
};
// 判断是否是合法的 listener
 function isValidListener(listener) {
     if (typeof listener === 'function') {
         return true;
     } else if (listener && typeof listener === 'object') {
         return isValidListener(listener.listener);
     } else {
         return false;
     }
}
// 顾名思义，判断新增自定义事件是否存在
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



# 实现 EventEmitter的 emit 的方法

就是拿到对应自定义事件进行 apply 执行，在执行过程中对于一开始 once 方法绑定的自定义事件进行特殊的处理，当once 为 true的时候，再触发 off 方法对该自定义事件进行解绑，从而实现自定义事件一次执行的效果

```js
EventEmitter.prototype.emit = function(eventName, args) {
     // 直接通过内部对象获取对应自定义事件的回调函数
     var listeners = this.__events[eventName];
     if (!listeners) return;
     // 需要考虑多个 listener 的情况
     for (var i = 0; i < listeners.length; i++) {
         var listener = listeners[i];
         if (listener) {
             listener.listener.apply(this, args || []);
             // 给 listener 中 once 为 true 的进行特殊处理
             if (listener.once) {
                 this.off(eventName, listener.listener)
             }
         }
     }
     return this;
};

EventEmitter.prototype.off = function(eventName, listener) {
     var listeners = this.__events[eventName];
     if (!listeners) return;
     var index;
     for (var i = 0, len = listeners.length; i < len; i++) {
	    if (listeners[i] && listeners[i].listener === listener) {
           index = i;
           break;
        }
    }
    // off 的关键
    if (typeof index !== 'undefined') {
         listeners.splice(index, 1, null)
    }
    return this;
};
```



# 实现 once 方法和 alloff

once 方法的本质还是调用 on 方法，只不过传入的参数区分和非一次执行的情况。当再次触发 emit 方法的时候，once 绑定的执行一次之后再进行解绑。

alloff 方法就是对内部的__events 对象进行清空，清空之后如果再次触发自定义事件，也就无法触发回调函数了。

```js
EventEmitter.prototype.once = function(eventName, listener）{
    // 直接调用 on 方法，once 参数传入 true，待执行之后进行 once 处理
     return this.on(eventName, {
         listener: listener,
         once: true
     })
 };
EventEmitter.prototype.allOff = function(eventName) {
     // 如果该 eventName 存在，则将其对应的 listeners 的数组直接清空
     if (eventName && this.__events[eventName]) {
         this.__events[eventName] = []
     } else {
         this.__events = {}
     }
};
```



# 完整实现

```javascript
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