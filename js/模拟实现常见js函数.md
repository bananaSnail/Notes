### 将扁平数据转换成树形结构
```js

function buildTree(list){
	let temp = {};
	let tree = {};
	for(let i in list){
    // 将id提取出来作为对象的key
		temp[list[i].id] = list[i];
	}
  
	for(let i in temp){
		if(temp[i].parent_id) {
			if(!temp[temp[i].parent_id].children) {
				temp[temp[i].parent_id].children = new Object();
			}
			temp[temp[i].parent_id].children[temp[i].id] = temp[i];
		} else {
			tree[temp[i].id] =  temp[i];
		}
	}
	return tree;
}

var menu_list = [{
      id: '1',
      menu_name: '设置',
      menu_url: 'setting',
      parent_id: 0
    }, {
      id: '1-1',
      menu_name: '权限设置',
      menu_url: 'setting.permission',
      parent_id: '1'
    }, {
      id: '1-1-1',
      menu_name: '用户管理列表',
      menu_url: 'setting.permission.user_list',
      parent_id: '1-1'
    }, {
      id: '1-1-2',
      menu_name: '用户管理新增',
      menu_url: 'setting.permission.user_add',
      parent_id: '1-1'
    }, {
      id: '1-1-3',
      menu_name: '角色管理列表',
      menu_url: 'setting.permission.role_list',
      parent_id: '1-1'
    }, {
      id: '1-2',
      menu_name: '菜单设置',
      menu_url: 'setting.menu',
      parent_id: '1'
    }, {
      id: '1-2-1',
      menu_name: '菜单列表',
      menu_url: 'setting.menu.menu_list',
      parent_id: '1-2'
    }, {
      id: '1-2-2',
      menu_name: '菜单添加',
      menu_url: 'setting.menu.menu_add',
      parent_id: '1-2'
    }, {
      id: '2',
      menu_name: '订单',
      menu_url: 'order',
      parent_id: 0
    }, {
      id: '2-1',
      menu_name: '报单审核',
      menu_url: 'order.orderreview',
      parent_id: '2'
    }, {
      id: '2-2',
      menu_name: '退款管理',
      menu_url: 'order.refundmanagement',
      parent_id: '2'
    }
]

const result = buildTree(menu_list);
console.log(result)
//

```


### 扁平化：数组扁平化、对象扁平化
- 数组扁平化flat
```js
function flattern(arr) {
  var result = []
  for(let i = 0; i < arr.length; i++) {
    if(Array.isArray(arr[i])) {
      result = result.concat(flattern(arr[i]))
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

// test
var a = [[1,2,[3,[4]]], [6,[7]]]
console.log(flattern(a))

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

### 拷贝：深拷贝浅拷贝！！
- 总结写前面：
  - 赋值运算符 = 实现的是浅拷贝，只拷贝对象的引用值
  - javascript 中数组和对象自带的拷贝方法都是‘首层浅拷贝’，如concat,slice
  - JSON.stringify 实现的是深拷贝，但不适用于循环引用和函数
  - 递归实现所有条件下的深拷贝；但是递归性能会不如浅拷贝，酌情使用
- 浅拷贝：浅拷贝只是复制对象的引用，如果拷贝后的对象发生变化，原对象也会发生变化
- 深拷贝
  - 不考虑循环引用和函数时使用JSON.parse(JSON.stringify(obj))，适用于大部分应用场景
 ```js
 function deepCopy(obj) {
   if(typeof obj !== 'object') return;
   var newObj = obj instanceof Array ? [] : {};
   for(var key in obj) {
     if(obj.hasOwnProperty(key)) {
       newObj[key] = typeof obj[key] === 'object' ? deepCopy(obj[key]) : obj[key];
     }
   }
   return newObj;
 }

 // test
 var a = {
   name: 'fengna',
   age: 18,
   right: {
     name: 'nana',
     age: 23,
     left: 'l'
   }
 }
 // 深拷贝
var b = deepCopy(a)
var b = JSON.parse(JSON.stringify(a))
// 浅拷贝
var b = a
 b.name = '123'
 console.log(a, b)

 var d = [1, 2, {a: 'a'}, [4, [5, 6,[7]]]]
 var c = d.slice(0)
 c[2].a = 'b'
 console.log(c, d)
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

// test
var foo = {
    value: 1
};
function bar() {
    console.log(this.value);
}
bar.call(foo); // 1

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

  Function.prototype.bind = function(context) {

    if(typeof this !== 'function') {
      throw new Error('Function.prototype.bind - what is trying to be bound is not callable')
    }

    // 获取bind函数从第二个参数到最后一个参数
    var args = Array.prototype.slice.call(arguments, 1),
        thatFunc = this;
    
    var fNOP = function() {}

    var fBound = function() { // 新函数调用时执行该函数
      // 这个时候的arguments是指bind返回的函数传入的参数
      var bindArgs = Array.prototype.slice.call(arguments)
      // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
      // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
      return thatFunc.apply(this instanceof fNOP ? this : context, args.concat(bindArgs))
    }

    // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
    // fBound.prototype = this.prototype; // 初版
    // 为了避免直接修改 fBound.prototype 的时候，也会直接修改绑定函数的 prototype，加个fNOP函数进行空转
    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound
  }
}

const module = {
  x: 42,
  getX: function() {
    return this.x;
  }
};

const unboundGetX = module.getX;
console.log(unboundGetX()); // The function gets invoked at the global scope
// expected output: undefined

const boundGetX = unboundGetX.bind(module);
console.log(boundGetX());
// expected output: 42

var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}

bar.prototype.friend = 'kevin';

var bindFoo = bar.bind2(foo, 'daisy');

var obj = new bindFoo('18');
// this指向了obj，new bindFoo() -> new fBound()  因此 this instanceof fBound 来判断是不是构造函数，构造函数的话则之前绑定的foo会失效，this会绑定到obj上；普通函数的话则this继续绑定在foo上
// undefined
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// kevin
```


### new的模拟实现
```js
// 需要判断构造函数返回值是不是一个对象，若是一个对象，则返回该对象；若不是则该返回什么就返回什么

function objectFactory() {
  var obj = new Object()
  Constructor = [].shift.call(arguments)

  obj._proto_ = Constructor.prototype

  var ret = Constructor.apply(obj, arguments)

  return typeof ret == 'object' ? ret : obj
  // 判断返回的值是不是一个对象，如果是一个对象，我们就返回这个对象，如果没有，我们该返回什么就返回什么
}

function Otaku(name, age) {
  this.name = name
  this.age = age
  // return 'hi'
}
var person = objectFactory(Otaku, 'kevin', '18')
console.log(person)
```

### 防抖节流
- 防抖：电梯运行--电梯等10秒开始运行，如果中途有人进入电梯，由此刻开始重新计时，即再等10秒开始运行。这就是debounce 防抖的原理
```js
export function debounce(fn, delay) {
    let timer = null;
    //闭包变量
    return function () {
    //保存作用域和参数
        let ctx = this;
        let args = arguments;
        //定时器存在的话，清除定时器，再新增一个定时器
        if (timer) {
            clearTimeout(timer);
            timer = setTimeout(function () {
            //事件执行前，将timer 置为null
                timer = null;
                //执行fn操作
                fn.apply(ctx, args);
            }, delay);
            // return false退出
            return false;
        }
        //第一次进来时 timer为null
        timer = setTimeout(function () {
            timer = null;
            fn.apply(ctx, args);
        }, delay);
    };
}

```
- 节流 事件被执行后才能重新计时 所以某段时间一定有一个事件是被执行的
```js
export function throttle(fn, delay) {
//闭包变量
    let timer     = null,
        firstTime = true;
    return function () {
    //保存作用域和参数
        let ctx  = this,
            args = arguments;
        if (firstTime) {
        //第一次立马执行 不delay
            fn.apply(ctx, args);
            return (firstTime = false);
        }
        //定时器存在的话，触发事件，事件不执行，直到当前定时的事件被执行，定时器置为null
        if (timer) {
            return false;
        }
        //执行事件时，timer置为null，等待执行时，timer不为null，进入上面的if里return false，什么都不做
        timer = setTimeout(function () {
        //这里清除的是当前定义的timer
            clearTimeout(timer);
            timer = null;
            fn.apply(ctx, args);
        }, delay);
    };
}

```


### 实现map 
```js
Array.prorotype.map = function(fn, context) {
  const arr = this, results = []
  context = Object(context) || global

  arr.forEach((item, index) => {
    let result = fn.call(context, item, index, arr)
    results.push(result)
  })

  return results
}

let arr = ["x", "y", "z"];
let obj = { a: 1 };

arr.map(function() {
  console.log(this.a);
}, obj);
// 1  1  1
```
```js
// reduce
Array.prototype.map = function (fn, context) {
    const array = this;
    context = Object(context) || global

    return array.reduce((acc, cur, index) => {
        acc.push(fn.call(context, cur, index, array))
        return acc
    }, [])
}
var m = [1,2,3,4,5].map(function (v, i, arr) {
    return v + v
});
console.log(m)

```
