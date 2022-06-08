### js中参数传递
- javascript 总是按值传递，但是当变量引用对象时，"值"是对对象的引用；
- 更改变量的值不会改变底层原始数据或对象，只是改变了数据或指向了另一个对象；
- 改变变量引用对象的一个属性会改变底层对象

### 判断监听器
- element.addEventListener(type, listener[, useCapture]);　　//IE6~8不支持
- element.attachEvent('on' + type, listener) //支持IE6~10，IE11不支持
- element['on' + type] = function(){} //支持所有浏览器
```js
let eventHandler = (element, type, handler) => {
  if (window.addEventListener) {
    eventHandler = (element, type, handler) => {
      element.addEventListener(type, handler)
    }
  }
  if (window.attachEvent) {
    eventHandler = (element, type, handler) => {
      element.attachEvent(`on${type}`, handler)
    }
  } else {
    eventHandler = (element, type, handler) => {
      element[`on${type}`] = handler
    }
  }
}
```
### Object.defineProperty的用法
```js
var user = { age: 10 };
let temp = user.age;

Object.defineProperty(user,'age',{ get() {temp++; return temp}});

console.log(user.age); // 11
console.log(user.age); // 12
```
- 注意：get方法里面不能获取被劫持的属性，否则会无限循环调用get方法
### for in，for of,foreach
- for...in：是为遍历对象属性而构建的，不建议与数组一起使用
  -  for...in 循环是以字符串作为键名
  - `for...in 会遍历对象的整个原型链，性能非常差不推荐使用，而for...of只遍历当前对象不会遍历他的原型链`
  - 某些情况下，for...in 循环会以任意顺序遍历键名
  - for...in 它可以与 break、continue 和 return 配合使用
  ```js
  for..in循环只能遍历可枚举的属性,即在遍历对象时可用的属性,如构造函数属性就不会显示,可以使用propertyIsEnumerable()方法检查哪些属性是可枚举属性
  可以使用hasOwnProperty验证对象属性是不是来自原型链
  ```
- for...of 遍历数组
  - `for...of只能用在可迭代对象上，获取的是迭代器返回的value值`，`for...in可以获取所有对象的键名`
  - 不同于 forEach 方法，它可以与 break、continue 和 return 配合使用
- forEach 遍历数组
  - 缺点：无法中途跳出 forEach 循环，break 命令或 return 命令都不能生效
- 迭代器：一个具有next()方法的对象，调用next()方法会返回value和done属性。自带迭代器的数据结构Array、Map、Set、String、函数的arguments对象、TypedArray(类型化数组对象)

### js数据类型
- 基本数据类型
  - Null
  - undefined类型
  - 字符串类型
  - 数字类型
  - 布尔类型
  - 符号类型（Symbols）：符号（Symbols）类型是唯一且不可修改的原始值，并且可以用来作为对象的键 (key)
  - BigInt类型
- 对象
   - 有序集：数组和类型数组
   - 普通对象
   - 函数：函数是一个附带可被调用功能的常规对象。
   
### 数据的存储
- `基本数据类型变量值存储在栈中`
- `对象类型是存放在堆空间的，在栈空间中只是保留了对象的引用地址，当 JavaScript 需要访问该数据的时候，是通过栈中的引用地址来访问的`，相当于多了一道转手流程
- 栈内存：`栈内存由编译器自动分配与释放。我们可以直接操作栈内存中的值。js中的基本数据类都有固定的大小，被分配到栈内存中。`这些基本类型的值都是按值引用。将一个基本类型的值赋值给另外一个基本类型时，会为这个新的值重新创建一个值并保存在栈内存中。
- 堆内存：`堆内存是链表结构的类型，可以动态分配大小，js引用类型占用内存空间的大小不固定，存储在堆内存中。`由于JS不允许直接访问堆内存中的位置，因此我们不能直接操作js的引用类型。而是生成一个指针，并将它放到栈内存中，通过这个指针来操作引用类型。

### 类数组对象arguments
- Arguments对象：Arguments 对象只定义在函数体中，包括了函数的参数和其他属性。在函数体中，arguments 指代该函数的 Arguments 对象。
- callee属性：可以调用函数自身，arguments.callee.xxx
- arguments和对应参数绑定
  - 非严格模式下，传入的参数，实参和arguments的值会共享，当没有传入时，实参是不会跟arguments值共享的；
  - 严格模式下，实参和arguments是不会共享的
- 应用：参数不定长、函数柯里化、递归调用、函数重载

### 调用栈

### JS代码运行
- 分`编译阶段、真正代码执行阶段，变量声明、函数声明等都是在编译阶段`
- `编译阶段：创建当前上下文，var声明的变量放在变量环境里，let声明的变量放在词法环境里`
- `变量声明：创建-> 初始化 -> 赋值` 三阶段
  - `var创建的时候就初始化了，有值则赋值，没有则是undefined；`
  - `let/const 会被先创建，但是不会被初始化，直到声明语句被执行的时候，let初始化的时候如果没有赋值，则执行时就是undefined，创建阶段到初始化阶段的代码片段就形成了暂时性死区`，在这期间访问的话会报错；
  - const初始化的时候必须赋值，否则报错，如果声明的是一个`引用类型`，则不能改变它的内存地址

### 作用域
- 释义：`作用域指程序中定义变量的区域，该位置决定了变量的生命周期；作用域就是变量与函数的可访问范围，即作用域控制变量和函数的可见性和生命周期`
- 分类：
  - `全局作用域`中的对象在代码中的任何地方都能访问，其生命周期伴随着页面的生命周期。
  - `函数作用域就是在函数内部定义的变量或者函数，并且定义的变量或者函数只能在函数内部被访问`。函数执行结束之后，函数内部定义的变量会被销毁。
  - `块级作用域(ES6新增)：作用块内声明的变量不影响块外面的变量`


### 作用域链
- `通过作用域查找变量的链条称为作用域链，作用域链是通过词法作用域来确定的，而词法作用域反映了代码的结构`。

### 词法作用域
- `词法作用域就是指作用域是由代码中函数声明的位置来决定的，所以词法作用域是静态的作用域`，通过它就能够预测代码在执行过程中如何查找标识符

### 执行上下文
- `执行上下文是评估和执行javaScript代码的环境的抽象概念`
- `调用栈（执行上下文栈）：在执行上下文创建好后，JavaScript引擎会将执行上下文压入栈中，通常把这种用来管理执行上下文的栈称为执行上下文栈，又称调用栈`
- 执行上下文类型：
  - `全局上下文： 任何不在函数内部的代码都在全局上下文中。它会执行两件事：创建一个全局的window对象(浏览器的情况下)，并且设置this的值等于这个全局对象`
  - `函数上下文： 每一个函数被调用时都会创建一个新的上下文。`
  - eval上下文
- 执行上下文中三个重要属性
  - this绑定
  - 作用域链：当查找变量时会先从当前上下文的变量对象中查找，如果没找到，就会从父级执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就是作用域链
  - 变量对象
- 词法作用域：一个函数可以访问在它的调用上下文中定义的变量

```js
function setFirstName(firstName) {
  return function(lastName) {
    return firstName + lastName
  }
}

var setLastName = setFirstName('feng')
setLastName('na')
// 解析：调用setFirstName时返回一个匿名函数，该匿名函数会持有setFirstName函数作用域的变量对象(arguments,firstName)
// setFirstName函数执行完之后其执行环境被销毁，但是他的变量对象会一直保存在内存中不被销毁。
// 同样垃圾回收机制因为变量对象会被一直hold而不做回收处理。内存泄露需要手动释放内存处理。
setLastName = null
```

### 闭包
- `闭包是指那些能访问自由变量的函数 --- 详解：在javascript中，根据词法作用域的规则，内部函数总是可以访问其外部函数中的声明的变量，当通过调用一个外部函数返回一个内部函数后，即该外部函数已经执行结束了，但是内部函数引用外部函数的变量依然保存在内存中，这些变量的集合就是闭包`
- 自由变量指既不是参数也不是函数局部变量的变量
- `函数对象被调用时 会创建一个新的执行上下文，函数对象里面的[[scope]]属性保存着该函数定义的时候能直接访问的作用域对象`
- 闭包什么时候被创建？所有js对象都是闭包，因此当定义一个函数时就定义了一个 闭包
- 闭包什么时候被销毁？当它不被任何其他对象引用的时候

### 实例、原型指针_proto_、原型 prototype
- 原型：`每一个javascript对象(null除外)在创建时就会与之关联另一个对象，这个对象就是我们说的原型，每个对象都会从原型 "继承" 属性`。原型是函数特有的
- 原型指针：每一个JavaScript对象(除了 null )都具有的一个属性，叫__proto__，这个属性会指向上一层的原型对象
- 原型链： 由多个_proto_构成的相互关联的原型组成的链状结构的就是原型链
- 并非是真的继承，继承意味着复制操作，这里并没有复制对象的属性，只是在两个对象之间创建一个关联，这样一个对象能通过委托访问到另一个对象的属性和函数，与其叫继承，委托的叫法更准确些
- 总结：`函数的原型对象constructor默认指向函数本身，原型对象除了有原型属性外，为了实现“继承”，还有一个原型指针__proto__，该指针指向上一层的原型对象，而上一层的原型对象的结构类似，这样利用__proto__一直指向Object的原型对象上，而Object的原型对象用Object.prototype.__proto__ = null表示原型链的最顶端，如此形成了js的原型链继承，也解释了为什么所有的js对象都具有Object的基本方法。`

```js
Function.prototype = {
    constructor: Function,
    __proto__:parent.prototype,
    some prototype prototies...
}
```


### 继承
- 当谈到继承时，JavaScript 只有一种结构：对象。每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（prototype ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

### this 指向
- `this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。`
- 优先级
  1. `函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象。`
      var bar = new foo()
  2. `函数是否通过call、apply(显式绑定)或者硬绑定调用?如果是的话，this绑定的是 指定的对象。`
      var bar = foo.call(obj2)
  3. `函数是否在某个上下文对象中调用(隐式绑定)?如果是的话，this 绑定的是那个上下文对象。`
      var bar = obj1.foo()
  4. `如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到 全局对象。`
      var bar = foo()
- this缺陷
  - `嵌套函数中的this不会从外层函数中继承`
    - 第一种是把 this 保存为一个 self 变量，再利用变量的作用域机制传递给嵌套函数。
    - 第二种是继续使用 this，但是要把嵌套函数改为箭头函数，因为箭头函数没有自己的执行上下文，所以它会继承调用函数中的 this
  - 普通函数的this指向window -- 解决方案：严格模式下，this值是undefined
```js
var myObj = {
  name : " 极客时间 ", 
  showThis: function(){
    console.log(this)
    function bar(){console.log(this)}
    bar()
  }
}
myObj.showThis()

// 解决方案： 1 利用另一个变量 self
var myObj = {
  name : " 极客时间 ", 
  showThis: function(){
    console.log(this)
    var self = this
    function bar(){
      self.name = " 极客邦 "
    }
    bar()
  }
}
myObj.showThis()

// 2.箭头函数：因为ES6中的箭头函数并不会创建自身的执行上下文；箭头函数里面的this取决于它的外部函数，通过查找作用域链来确定
var myObj = {
  name : " 极客时间 ", 
  showThis: function(){
    console.log(this)
    var bar = ()=>{
      this.name = " 极客邦 "
      console.log(this)
    }
    bar()
  }
}
```

### 常见运算符/操作符
- instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上
- typeof 操作符返回一个字符串，表示未经计算的操作数的类型

### js数据类型的判断
- `typeof 只能判断出number、string、boolean、undefined、function；对于 null、对象、数组不能判断出来，结果都是object`
- `instanceof 判断某个实例是不是属于原型；无法判断基本数据类型（需要new 声明基本数据类型才能判断出来）；区分不了null和undefined；`但是对于new 声明的实例对象，instanceof可以检测出多重继承关系
- `Object.prototype.toString() 返回一个表示该对象的字符串；Object.prototype.toString.call(1) // "[object Number]"`


### JS 异步编程六种方案
- 同步与异步
- 回调函数
- 事件监听：异步任务的执行不取决于代码的顺序，而取决于某个事件是否发生。
- 发布订阅
- Promise/A+
- 生成器Generators/ yield
  - 语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。
  - Generator 函数除了状态机，还是一个遍历器对象生成函数。
  - 可暂停函数, yield可暂停，next方法可启动，每次返回的是yield后的表达式结果。
  - yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。

### ['1', '2', '3'].map(parseInt) 输出结果
```js
['1', '2', '3'].map(parseInt) // [1,NaN,NaN,NaN]
parseInt('1', 0) => 1 // radix为0时，且string参数不以'0x'和'0'开头时，按照10为基数处理。这个时候返回1
parseInt('2', 1) => NaN // 基数为1(一进制)表示的数中，这里的参数没有在2~36范围之间，所以为NaN。
parseInt('3', 2) => NaN // 基数为2 (二进制)表示的数中，二进制能解析的最大值为1，最大值小于'3'，'3'在二进制中是一个非法的值，所以无法解析，返回NaN。
parseInt('4', 3) => NaN // 基数为3(三进制)表示的数中，三进制能解析的最大值为2，最大值小于'4'， '4'在三进制中是一个非法的值，所以无法解析，返回NaN。

```
- parseInt则是用来解析字符串，使字符串成为指定基数的整数；parseInt(string, radix) 接收两个参数，第一个表示被处理的值(字符串)，第二个表示为解析时的基数

### 立即执行函数表达式
- `当圆括号出现在匿名函数的末尾想要调用函数时，它会默认将函数当成是函数声明。`
- `当圆括号包裹函数时，它会默认将函数作为表达式去解析，而不是函数声明。`
- 保存闭包的状态：`就像当函数通过他们的名字被调用时，参数会被传递，而当函数表达式被立即调用时，参数也会被传递。一个立即调用的函数表达式可以用来锁定值并且有效的保存此时的状态，因为任何定义在一个函数内的函数都可以使用外面函数传递进来的参数和变量(这种关系被叫做闭包)。`
```js
var counter = (function(){
    var i = 0;
    return {
        get: function(){
            return i;
        },
        set: function(val){
            i = val;
        },
        increment: function(){
            return ++i;
        }
    }
    }());
    counter.get();//0
    counter.set(3);
    counter.increment();//4
    counter.increment();//5

    conuter.i;//undefined (`i` is not a property of the returned object)
    i;//ReferenceError: i is not defined (it only exists inside the closure)
```
- 模块模式方法不仅相当的厉害而且简单。非常少的代码，你可以有效的利用与方法和属性相关的命名，在一个对象里，`组织全部的模块代码即最小化了全局变量的污染也创造了使用变量。`

### instanceof 和 typeof 的实现原理
- typeof: 判断基本数据类型，（numbel string boolean undefined object symbols function）无法判断null
- Object.prototype.toString 上述几种都可以判断，可以通过 toString() 来获取每个对象的类型
- 我们使用 typeof 来判断基本数据类型是 ok 的，不过需要注意当用 typeof 来判断 null 类型时的问题，如果想要判断一个对象的具体类型可以考虑用 instanceof，但是 `instanceof 也可能判断不准确，比如一个数组，他可以被 instanceof 判断为 Object。`所以我们要想比较准确的判断对象实例的类型时，可以采取 `Object.prototype.toString.call `方法。
```js
console.log(typeof null); // "object"
console.log(null instanceof Object); // false
console.log(Object.prototype.toString.call(null)); // "[object Null]"

```
### mouseenter和mouseover区别
- mouseenter：当鼠标移入某元素时触发。
- mouseleave：当鼠标移出某元素时触发。
- mouseover：当鼠标移入某元素时触发，移入和移出其子元素时也会触发。
- mouseout：当鼠标移出某元素时触发，移入和移出其子元素时也会触发。
- mousemove：鼠标在某元素上移动时触发，即使在其子元素上也会触发。

### let面试题
```js
var total = 0;
var a = 3;
var result = []

function foo(a) {
  var i = 0;
  for(;i<3;i++) {
    result[i] = function() {
      total += i*a;
      console.log(total)
    }
  }
}

foo(1);
result[0]();
result[1]();
result[2]();

3
6
9

// 将var i 改为 let i
function foo(a) {
  for(let i = 0;i<3;i++) {
    result[i] = function() {
      total += i*a;
      console.log(total)
    }
  }
}

0
1
3
// let 声明的变量具有块级作用域，每轮循环i都是一个新值，因此数组中存储了不同的数字
```
```js
let a = 10;

function test() {
  console.log(a);
  let a = 20;
  a++
}

test()
// Uncaught ReferenceError
```
### Promise all 和allSettled区别
- Promise.allSettled永远不会被reject
```js
const promises = [
  delay(100).then(() => 1),
  delay(200).then(() => 2),
  Promise.reject(3)
  ]

Promise.all(promises).then(values=>console.log(values))
// 最终输出： Uncaught (in promise) 3

Promise.allSettled(promises).then(values=>console.log(values))
// 最终输出： 
//    [
//      {status: "fulfilled", value: 1},
//      {status: "fulfilled", value: 2},
//      {status: "rejected", value: 3},
//    ]
```
### js proposal-logical-assignment
- ??=