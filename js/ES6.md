### es6特性
- const、let
- 模板字符串： ${name} ` `
- 箭头函数
- iterator 迭代器：一个具有next() 方法的对象，每次调用next()都会返回一个结果对象，该对象有两个属性，value表示当前值，done表示遍历是否结束
- 解构： let { data } = res
- 展开运算符： ... let arr = [...res]
- for...of循环
- Promise: 使得异步回调代码扁平化
- ES6 Module
- 类class
- generate

### const let
- var 在创建时就被初始化了，赋值为undefined

### for...of循环
- for...of只能用在可迭代对象上，获取的是迭代器返回的value值，for...in可以获取所有对象的键名
- for...in 会遍历对象的整个原型链，性能非常差不推荐使用，而for...of只遍历当前对象不会遍历他的原型链

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
- 释义：箭头函数表达式的语法比函数表达式更短，并且不绑定自己的this，arguments，super或 new.target。这些函数表达式最适合用于非方法函数(non-method functions)，并且它们不能用作构造函数。
- 没有this，需要通过查找作用域链来确定this的值
- 没有arguments，可以通过命名参数或者rest参数的形式来访问参数
- 不能通过 new 关键字调用
- 没有 new.target
- 没有原型
- 没有super

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
- Promise 控制反转再反转
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