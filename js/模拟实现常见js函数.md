### 扁平化：数组扁平化、对象扁平化
- 数组扁平化flat
```js
function flattern(arr) {
  var result = []
  for(let i = 0; i < arr.length; i++) {
    if(Array.isArray(arr[i])) {
      result = result.concat(arr[i])
    } else {
      result.push(arr[i])
    }
  }
  return result
}

// es6实现
function flattern(arr) {
  return arr.reduce((result, item)=> {
    return result.concat(Array.isArray(item)? flattern(item) : item)
  }, [])
}

```
- 对象扁平化
```js
function flatternObj(obj) {
  let result = {}, 
    keys = Object.keys(obj);

  for(let i = 0; i < keys.length; i++) {
    let key = keys[i],
      value = obj[key];

    if(value.toString() == '[object Object]') {
      Object.assign(result, fllaternObj(value))
    } else {
      result[key] = value
    }
  }
  return result
}
```

### 拷贝：深拷贝浅拷贝
- 总结写前面：
  - 赋值运算符 = 实现的是浅拷贝，只拷贝对象的引用值
  - javascript 中数组和对象自带的拷贝方法都是‘首层浅拷贝’，如concat,slice
  - JSON.stringify 实现的是深拷贝，但不适用于循环引用和函数
  - 递归实现所有条件下的深拷贝；但是递归性能会不如浅拷贝，酌情使用
- 浅拷贝：浅拷贝只是复制对象的引用，如果拷贝后的对象发生变化，原对象也会发生变化
- 深拷贝
  - 不考虑循环引用和函数时使用JSON.parse(JSON.stringify(obj))，适用于大部分应用场景
  - 考虑循环引用和函数时使用递归，判断 某属性 是否指向 父节点 
 ```js
 function deepCopy(obj) {
   if(typeof obj !== 'object') return;
   var newobj = obj instanceof Array ? [] : {};
   for(var key in obj) {
     if(obj.hasOwnProperty(key)) {
       newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key];
     }
   }
   return newObj;
 }
 ```

### 模拟实现call和apply函数
- call()方法接受的是一个参数列表，apply()方法接受一个包含一个多个参数的数组
```js
// call
Function.prototype.call = function(obj) {
  obj = obj ? Object(obj) : window;
  obj.fn = this;

  let args = [...arguments].slice(1),
    result = obj.fn(...args)

  delete obj.fn
  return result
}

// apply
Function.prototype.apply = function(obj, arr) {
  obj = obj ? Object(obj) : window;
  obj.fn = this;

  let result;
  if(!arr) {
    result = obj.fn()
  } else {
    result = obj.fn(...arr)
  }

  delete obj.fn;
  return result
}
```


### 实现bind函数polyfill
- bind() 方法创建一个新的函数，在bind()被调用时，这个新函数的this被指定为bind()的第一个参数，而其余参数将作为新函数的参数，以供调用
```js
if(!Function.prototype.bind) {

  Function.prototype.bind = function() {

    if(typeof this !== 'function') {
      throw new Error('Function.prototype.bind - what is trying to be bound is not callable')
    }

    var args = Array.prototype.slice.call(arguments, 1),
        thatFunc = this;
    
    var fNOP = function() {}

    var fBound = function() {
      var bindArgs = Array.prototype.slice.call(arguments)
      
      return thatFunc.apply(this instanceof fNOP ? this : context, args.concat(bindArgs))
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound
  }
}

```


### new的模拟实现
```js
// 需要判断构造函数返回值是不是一个对象，若是一个对象，则返回该对象；若不是则该返回什么就返回什么

function objectFactory() {
  var obj = new Object()
  Constructor = [].shift.call(arguments)

  obj._proto_ = Constructor.prototype

  var ret = Constructor.apply(obj, arguments)

  return type ret == 'object' ? ret : obj
}

function Otaku(name, age) {
  this.name = name
  this.age = age

}
var person = objectFactory(Otaku, 'kevin', '18')

```

### 防抖节流
- 防抖
```js
// 需求：立即执行、返回值、取消定时器
function debounce(fn, delay, immediate) {
  var timer = null, result;

  var debounced = function() {
    var ctx = this, args = arguments;

    if(timer) clearTimeout(timer);

    if(immediate) {
      var callNow = !timer

      timer = setTimeout(() => {
        fn.call(ctx, args)
      }, delay)

      if(callNow) result = fn.call(ctx, args)
    } else {
      timer = null;
      timer = setTimeout(() => {
        fn.call(ctx, args)
      }, delay)
    }

    return result
  }
  
  debounced.cancel = function() {
    clearTimeout(timer);
    timer = null;
  }

  return debounced
}

```
- 节流
```js
function throttle(fn, delay) {
  let timer = null;

  var throttled = function() {
    let ctx = this, args = arguments;

    if(timer) return false

    timer = setTimeout(() => {
      clearTimeout(timer);
      timer = null;
      fn.call(ctx, args)
    }, delay)

  }

  throttled.cancel = function() {
    clearTimeout(timer);
    timer = null
  }

  return throttled
}

```


