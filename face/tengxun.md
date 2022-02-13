vue   es  ts  js

### 实现Promise
- 优点，若没有Promise程序会怎么样
- 手写Promise，如何实现链式调用，.then的实现   若当前promise没有被resolve或者reject掉，使用.then会发生什么
- Promise.then为什么是微任务

### 如何实现兼容性
- autoprefixer 自动给css添加浏览器引擎前缀。如谷歌：-webkit- 火狐：-moz- Opera浏览器：-o- IE浏览器：-ms-

### 闭包是es几，闭包一般应用在哪
#### 应用：
- `为响应事件而执行的函数`
```js
function makeSizer(size) {
  return function() {
    document.body.style.fontSize = size + 'px';
  };
}

var size12 = makeSizer(12);
var size14 = makeSizer(14);
document.getElementById('size-12').onclick = size12();
document.getElementById('size-14').onclick = size14();

```
- `用闭包模拟私有方法: 使用闭包来定义公共函数，并令其可以访问私有函数和变量`; 每次调用其中一个计数器时，通过改变这个变量的值，会改变这个闭包的词法环境。然而在一个闭包内对变量的修改，不会影响到另外一个闭包中的变量。
```js
var makeCounter = function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }  
};

var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value()); /* logs 0 */
Counter1.increment();
Counter1.increment();
console.log(Counter1.value()); /* logs 2 */
Counter1.decrement();
console.log(Counter1.value()); /* logs 1 */
console.log(Counter2.value()); /* logs 0 */
注：两个计数器 Counter1 和 Counter2 是如何维护它们各自的独立性的。每个闭包都是引用自己词法作用域内的变量 privateCounter
```

### vuex 原理，踩过的坑
#### 业务添加state属性
- 背景：在组件中调用一个异步请求，不改变已有的state中的属性，而是在添加一个新的属性（该属性未在state中声明过）
- 现象：通过Chrome插件Vue Devtools可以看到state中确实添加了name属性，并且有值，但是`页面中不显示`
- 解决方案：如果要添加一个当前state中没有的属性，vue要将某一个属性作为响应式的处理，必须要通过 `Object.defineProperty` 的操作
- 代码修改：Vue.set(state, "name", name); 或者在state声明时声明name属性
#### 模块化后的模块层级
- getter 回调函数参数：1. state，模块中的 state 仅为模块自身中的 state；2. getters，等同于 store.getters；3. rootState，全局 state
- action 回调函数接收一个 context 上下文参数，context 包含：1. state、2. rootState、3. getters、4. mutations、5. actions 五个属性，为了简便可以在参数中解构。
- 背景：将 store 中的 state 绑定到 Vue 组件中的 computed 计算属性后，对 state 进行更改需要通过 mutation 或者 action，在 Vue 组件中直接进行赋值 (this.myState = ‘ABC’) 是不会生效的。
- 待实验：
  - 在 action 中可以通过 context.commit 跨模块调用 mutation，同时一个模块的 action 也可以调用其他模块的 action。同样的，当不同模块中有同名 action 时，通过 store.dispatch 调用，会依次触发所有同名 actions。
  - 由于 getters 不区分模块，所以不同模块中的 getters 如果重名，Vuex 会报出 ‘duplicate getter key: [重复的getter名]‘ 错误

### 用过最复杂的ts API，用过哪些ts API
- 泛型  类型断言  
- 题目：第一个参数是不定类型，如 type 或 enum这种  第二个参数在第一个参数确定下来后跟第一个参数类型保持一致
- 解析：
  1. `函数参数间的约束考虑用泛型`，`注意：是无法创建泛型枚举和泛型空间的`，因此考虑type类型。
  2. `infer类型推断：`由于第二个参数依赖于第一个参数的类型
```js
type Type = Number | String // 注意要大写，小写的number | string 是ts 里面定义值类型的，Number和String 是其中的包装对象；定义值的话就必须得一样了
type ParamType<T> = T extends (param: infer P) ? P : T 

testFunc<T extends Type>(a: T, b: ParamType<T>) {
  console.log(a, b)
}

testFunc('aaa', 111) // 报错，Argument of type 'number' is not assignable to parameter of type 'string'.
testFunc('aaa', 'bbbb')
```

### 函数科里化一般怎么结合进业务里

### Vue3.0新特性
- Object.defineProperty -> Proxy
- 虚拟 DOM 重写，mounting和patching的速度提高100％

### Vue.nextTick 原理  内部封装
#### 原理：
- 要`拿到更新后有DOM，就必须把nextTick中的cb放到下一个任务队列中（宏任务微任务都行）`
- `微任务总是在宏任务执行前执行。所以，微任务更适合nextTick的场景。把nextTick回调中的脚本放到一个promise.then()中，就能保证是DOM更新后执行`。
- `VUE降级策略（因为API的兼容性问题），(promise -> setTimeout)`

#### dom更新
- Watch对象并不是立即更新视图，而是被push进了一个队列queue，此时状态处于waiting的状态，这时候会继续会有Watch对象被push进这个队列queue，等待下一个tick时，这些Watch对象才会被遍历取出，更新视图

### 箭头函数  除了method不可使用箭头函数  Vue中还有哪里不能使用箭头函数
1. 不应该使用箭头函数来定义一个生命周期方法
2. 不应该使用箭头函数来定义 method 函数
3. 不应该使用箭头函数来定义计算属性函数
4. 不应该对 data 属性使用箭头函数
5. 不应该使用箭头函数来定义 watcher 函数



### MVC MVVM区别   Vue jQuery区别


### MVVM原理


### loading加载时优化


### TCP为什么是三次握手


### url输入到加载出来


### 跨域 


### 安全


### 内网里域名到ip地址是怎么映射上的

### 页面渲染时优化

### node

### PV UV