### 扁平化：数组扁平化、对象扁平化
- 数组扁平化
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
- 浅拷贝：js中的浅拷贝指的是对象单层级拷贝，操作拷贝出来的新对象不会影响被拷贝的对象，可以使用es6解构赋值进行浅拷贝

```js
var man = {
  name: 'abc',
  age: 18
}
var shallowCopy = { ...man }

```
- 深拷贝
  - 不考虑循环引用和函数时使用JSON.parse(JSON.stringify(obj))，适用于大部分应用场景
  - 考虑循环引用和函数时使用递归，判断 某属性 是否指向 父节点 
  ```js
  function deepCopy(obj, parent = null) {
    let result = {},
      keys = Object.keys(obj),
      key = null, value = null;
    let _parent = parent
    
    while(_parent) {
      if(_parent.currentParent == obj) {
        return obj
      }
      _parent = _parent.parent
    }

    for(let i = 0; i < keys.length; i++) {
      key = keys[i], value = obj[key];

      if(value && typeof temp === 'object') {
        result[key] = deepCopy(value, {
          originParent: obj,
          currentParent: result,
          parent: parent
        })
      } else {
        result[key] = value
      }
    }

    return result
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
      var thatFunc = this;
    
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

### 实现函数式编程：compose函数和pipe函数；compose函数数据执行流是从右到左实现的；从左到右是pipe函数
```js
function compose(...args) {
  return function(x) {
    return args.reduceRight((res, cb) => {
      return cb(res)
    }, x)
  }
}
// 或者
const compose = (...args) => x => args.resuceRight((res, cb) => cb(res), x)

const pipe = (...args) => x => args.reduce((res, cb) => cb(res), x)
```

### 防抖节流


### new的模拟实现

