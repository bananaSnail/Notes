### es6特性
- `const、let`
- 模板字符串： ${name} ` `
- `箭头函数`
- `iterator 迭代器：一个具有next() 方法的对象，每次调用next()都会返回一个结果对象，该对象有两个属性，value表示当前值，done表示遍历是否结束`
- `解构： let { data } = res`
- `展开运算符： ... let arr = [...res]`
- for...of循环
- `Promise: 使得异步回调代码扁平化`
- ES6 Module
- `类class`
- generate
- Proxy
- Reflect

### [Proxy](https://zh.javascript.info/proxy)
- Reflect 是一个内建对象，可简化 Proxy 的创建。[[Get]] 和 [[Set]] 等，都只是规范性的，不能直接调用。[proxy](https://zh.javascript.info/proxy#proxy-de-ju-xian-xing)
- Reflect 对象使调用这些内部方法成为了可能。它的方法是内部方法的最小包装。
```js
let numbers = [];

numbers = new Proxy(numbers, { // (*)
  set(target, prop, val) { // 拦截写入属性操作
    if (typeof val == 'number') {
      target[prop] = val;
      return true;
    } else {
      return false;
    }
  }
});

numbers.push(1); // 添加成功
numbers.push(2); // 添加成功
alert("Length is: " + numbers.length); // 2

numbers.push("test"); // TypeError（proxy 的 'set' 返回 false）

```
- 跟defineProperty区别，为什么Proxy会取代Object.defineProperty：
  - Object.defineProperty只能劫持对象的属性，不能监听数组。也不能对 es6 新产生的 Map,Set 这些数据结构做出监听。也不能监听新增和删除操作等等。
  - Proxy可以直接监听整个对象而非属性，可以监听数组的变化，具有多达13中拦截方法。

### Reflect
- Reflect 是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与proxy handlers 的方法相同。Reflect不是一个函数对象，因此它是不可构造的。
- Reflect 上的所有函数都对应一个 Proxy 的陷阱。这些函数接受的参数，返回值的类型，都和 Proxy 上的别无二致，可以说 Reflect 就是 Proxy 拦截的那些操作的原本实现。
- 那 Reflect 存在的意义是什么呢？
  - 上文提到过，**Proxy 上某一些陷阱对处理函数的返回值有要求。如果想让代理对象能正常工作，那就不得不按照 Proxy 的要求去写处理函数。或许会有人觉得只要用 Object 提供的方法不就好了，然而不能这么想当然，因为某些陷阱要求的返回值和 Object 提供的方法拿到的返回值是不同的，而且有些陷阱还会有逻辑上的要求，和 Object 提供的方法的细节也有所出入。**举个简单的例子：**Proxy 的 defineProperty 陷阱要求的返回值是布尔类型，成功就是 true，失败就是 false。而 Object.defineProperty 在成功的时候会返回定义的对象，失败则会报错。**如此应该能够看出为陷阱编写实现的难点，如果要求简单那自然是轻松，但是要求一旦复杂起来那真是想想都头大，大多数时候我们其实只想过滤掉一部分操作而已。**Reflect 就是专门为了解决这个问题而提供的，因为 Reflect 里的函数都和 Proxy 的陷阱配套，返回值的类型也和 Proxy 要求的相同，所以如果我们要实现原本的功能，直接调用 Reflect 里对应的函数就好了。**


### 标签模板
- 标签模板紧跟在一个函数后面，该函数被调用来处理这个模板字符串，称为标签模板
```js
let a = 5;
let b = 10;
tag`Hello ${ a + b } world ${ a * b }`;
// 等同于
tag(['Hello ', ' world ', ''], 15, 50);
```

### es2020新特性
- 可选链操作符(?. ) (user?.name) 不为null 或者 undefined 才能继续下去 
- 空位合并操作符(??) (x ?? 500)
- Promise.allSettled 解决Promise.all的万一崩掉的短路问题
- String.prototype.matchAll：如果一个正则表达式在字符串里面有多个匹配，现在一般使用g修饰符或y修饰符，在循环里面逐一取出
- Dynamic import： import(module) 函数可以在任何地方调用。它返回一个解析为模块对象的 promise。
```js
let module = await import('/modules/my-module.js');
```
- globalThis:  目的就是提供一种标准化方式访问全局对象

### const let
- var 在创建时就被初始化了，赋值为undefined

### for...of循环
- `for...of只能用在可迭代对象上，获取的是迭代器返回的value值`，`for...in可以获取所有对象的键名`
- `for...in 会遍历对象的整个原型链，性能非常差不推荐使用，而for...of只遍历当前对象不会遍历他的原型链`

### Module
```js
// module.js
let x = 10, y = 20
export { x }  // 输出变量的引用
export default y  // 输出的是一个值
export {y as default} // 输出变量的引用
```
- 两种导出的区别：在xxx.js中引入这两个变量后，module.js中因为某些原因x变量被修改了，xxx.js中会收到x的改变，而module.js中y变量改变了，xxx.js中y还是原来的值
- export {<变量>}这种形式导出的模块，即使被重命名为default，仍然导出的是一个*变量的引用*

### 箭头函数
- `ES6中的箭头函数并不会创建自身的执行上下文；箭头函数里面的this取决于它的外部函数，通过查找作用域链来确定`
- 释义：`箭头函数表达式的语法比函数表达式更短，并且不绑定自己的this，arguments，super或 new.target。这些函数表达式最适合用于非方法函数(non-method functions)，并且它们不能用作构造函数。`
- `没有this，需要通过查找作用域链来确定this的值`
- `没有arguments，可以通过命名参数或者rest参数的形式来访问参数`
- `不能通过 new 关键字调用`
- `没有 new.target`
- `没有原型`
- `没有super`

### 模拟实现 Set 数据结构
- 简单版：实现add、delete、has、clear、forEach方法
- 处理NaN
```js
(function(global) {
  var NaNSymbol = Symbol('NaN');

  var encodeVal = function(val) {
    return value !== value ? NaNSymbol : value;
  }

  var decodeVal = function(val) {
    return (value === NaNSymbol) ? NaN : value;
  }

  var makeIterator = function(array, iterator) {
    var nextIndex = 0;

    // new Set(new Set()) 会调用这里
    var obj = {
      next: function() {
        return nextIndex < array.length ? 
          {value: iterator(array[nextIndex++]), done: false} : {value: void 0, done: true}
      }
    }

    // [...set.keys()] 会调用这里
    obj[Symbol.iterator] = function() {
      return obj
    }

    return obj
  }

  function forOf(obj, cb) {
    let iterable, result;

    if(typeof obj[Symbol.iterator] !== 'function') throw new TypeError(obj + " is not iterator")
    if(typeof cb !== "function") throw new TypeError('cb must be callable')

    iterable = obj[Symbol.iterator]()

    result = iterable.next()
    while(!result.done) {
      cb(result.value)
      result = iterable.next()
    }
  }

  function Set(data){
    this._values = [];
    this._size = 0

    forOf(data, (item) => {
      this.add(item)
    })
  }

  Set.prototype['add'] = function(value) {
    value = encodeVal(value)
    if(this._values.indexOf(value) == -1) {
      this._values.push(value)
      this._size++
    }
    return this
  }

  Set.prototype['has'] = function(value) {
    return (this._values.indexOf(encodeVal(value)) !== -1)
  }

  Set.prototype['delete'] = function(value) {
    var idx = this._values.indexOf(encodeVal(value))
    if(idx == -1) return false
    this._values.splice(idx, 1);
    this._size--
    return true
  }

  Set.prototype['clear'] = function(value) {
    this._values = []
    this._size = 0
  }

  Set.prototype['values'] = Set.prototype['keys'] = function() {
    return makeIterator(this._values, function(value) {return decodeVal(value)})
  }

  Set.prototype['entries'] = function() {
    return makeIterator(this._values, function(value) {return [decodeVal(value), decodeVal(value)]})
  }

  Set.prototype[Symbol.iterator] = function() {
    return this.values();
  }

  Set.prototype['forEach'] = function(callbackFn, thisArg) {
    thisArg = thisArg || global;
    var iterator = this.entries();

    forOf(iterator, (item) => {
      callbackFn.call(thisArg, item[1], item[0], this);
    })
  }

  Set.length = 0
  global.Set = Set;
}) 
```

### WeakMap
- WeakMap只接受对象作为键名
- WeakMap的键名所引用的对象时弱引用

### Promise
- Promise对象用于表示一个异步操作的最终完成（或失败），及其结果值
- 回调： 
  - 回调嵌套
  - 控制反转 
  - 回调地狱 -- 难以复用、堆栈信息被断开、借助外层变量
- Promise 解决了：
  - 回调地狱问题
  - 信任问题：控制反转再反转
- Promise 回调未调用问题：Promise 在决议时总是会调用完成回调和拒绝回调中的一个，若Promise本身永远不被决议，即使这样，Promise也提供了解决方案，使用Promise.race函数
```js
const p = Promise.race([
  api.getData(),
  new Promise(function(resolve, reject) {
    setTimeout(() => reject(new Error('timeout')), 5000)
  })
])
p.then(val => {})
.catch(err => {}) // 超时
```

- 红绿灯问题：红灯三秒亮一次，绿灯一秒亮一次，黄灯2秒亮一次；如何让三个灯不断交替重复亮灯？
```js
function red() {
  console.log('red')
}
function green() {
  console.log('green')
}
function yellow() {
  console.log('yellow')
}

var lighten = function(timer, cb) {
  return new Promise(function(resolve, reject) {
    setTimeout(() => {
      cb()
      resolve()
    }, timer)
  })
}

var step = function() {
  Promise.resolve().then(function() {
    lighten(3000, red)
  })
  .then(function() {
    lighten(1000, green)
  })
  .then(function() {
    lighten(2000, yellow)
  })
  .then(function() {
    step()
  })
}

step()
```