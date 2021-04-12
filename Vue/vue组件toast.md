1. diff算法、key作用，不要key会怎样【描述】
    - 树diff、组件diff、元素diff；key可以原地复用，没有key无脑会更新（此时你可以发现，index做key其实会形同虚设）
- diff算法的作用: 计算出Virtual DOM中真正变化的部分，并只针对该部分进行原生DOM操作，而非重新渲染整个页面。
- 树diff（tree diff）：
Web UI中DOM节点跨层级的移动操作特别少，可以忽略不计。

- 组件diff（component diff）：
拥有相同类的两个组件 生成相似的树形结构，
拥有不同类的两个组件 生成不同的树形结构。

- 元素diff（element diff）：
对于同一层级的一组子节点，通过唯一id区分。

#### 从组件封装（enrolment-mobile）到Vue源码理解

### Toast组件源码理解
https://juejin.im/post/58e65fb944d904006d3543de
https://juejin.im/post/5ca20e426fb9a05e42555d1d
### Toast 采用命令行调用方式的原因

1. 方便在具体业务逻辑中使用，特别是各类 ajax 请求的处理逻辑
2. 对模板带入的入侵小，不需要在业务组件中去注册
3. 以现有的 Vue 对 TS 的支持程度，TS 对传入函数参数的约束更好，提示也更佳
4. 更方便控制实例的销毁，以及通过 extend 的方式能够更好控制“组件”挂载的容器

### 实现组件命令行调用

1. 原理：`通过暴露一个 toast 函数来使得外部调用该组件，toast 函数通过手动挂载的方式挂载到 body 下面`
2. 实现步骤：

- `调用 Vue.extend()构造器，创建 子类构造函数 toast`
- `实例化 toast，挂载到 DOM 上`
- `处理数据`
- `将 toast DOM 结构挂载到 body 上`
- `控制 toast 的显示与消失`
- `toast 消失动画结束后，移除 DOM 节点`

### 组件实例管理的优点

1. 多条业务线可以并行开发，提高开发效率
2. 解耦，便于项目管理

### 可以采用命令行形式调用的同类型组件

1. Loading 组件
2. Spin 加载中组件
3. Message 全局提示组件
4. notification 通知提示框

### 高效定义这类型组件

- `使用 Vue.extend()去创建子类构造函数`
- `通过调用一个函数来完成组件的创建，挂载以及销毁`
- `将组件 DOM 节点挂载到 body 上，可以不受 #app 和 <router-view /> 管控`
- `避免重复创建 toast 实例，创建 toast 实例时先判断是否已经存在 toast 实例`

- Vue.extend(options)：构造器，接收一个对象，返回子类的构造函数
```js
let ToastConstructor = Vue.extend(require('./Toast.vue'));

// 调用基本的vue构造器，创建toast子类构造函数。

```
- 构造函数的使用
```js
var instance = new ToastConstructor({
    el: document.createElement('div')
});
// 实例化toast，挂载到dom组件上
====>
var instance = new ToastConstructor().$mount(document.createElement('div'))


```
- 实例操作
```js

document.body.appendChild(instance.$el);

// instance.$el 是一整个toast dom 结构
// 将instance实例的dom结构挂载到body上，不受 #app 和 <router-view /> 管控

```

##### toast组件总结

- 怎么实现组件命令行调用
`通过暴露一个toast函数来使得外部调用该组件，而toast函数通过手动挂载的方式挂载到body下面`
- Toast为什么要采用命令行调用方式
```
如果封装成一般的组件，那么需要在每个页面中引入，每个页面还得用一个变量来控制显示隐藏等，toast这种组件使用频率特别高，那么这样去调用就会很繁琐，用命令行式调用可以节省代码量
一般都是多处使用 --需要解决每个页面重复引用+注册
一般都是跟js交互的 --无需 在<template>里面写 <toast :show="true" text="弹窗消息"></toast>

Toast 每次使用都要导入
父组件的 state 要包含一个 visible 的属性 (假设我们没有使用mixin)
设置 父组件的 visible 为 true 之后，用一个定时器再把 visible 设置为 false

类似这样使用频率较高的组件，最便捷的方式是通过调用一个函数来完成组件的创建，挂载以及销毁。

```
- 组件实例管理的好处

- 投射到其他组件上是否也可以采用这种形式: 对于使用频率特别高的组件，通用性特别强的，如Loading组件也可以采用命令行式

- 高效的定义此类型组件


###### 代码解析
- [ ] ~duration理解：按位非，不太能明白此处对duration字段取反的意义
```js
// 位运算符 ~
1. 可以当做取整运算符
var a = 1.2;
console.log(~~a); // 1  且不改变a本身的数值

2. js中所有的位运算符
Javascript完全套用了Java的位运算符，包括按位与&、按位或|、按位异或^、按位非~、左移<<、带符号的右移>>和用0补足的右移>>>。

这套运算符针对的是整数，所以对Javascript完全无用，因为Javascript内部，所有数字都保存为双精度浮点数。如果使用它们的话，Javascript不得不将运算数先转为整数，然后再进行运算，这样就降低了速度。而且"按位与运算符"&同"逻辑与运算符"&&，很容易混淆。

```


- [ ] closed 加锁字段
```js
if (instance.closed) return;
    instance.close();
// 调用instance.close()函数加锁，避免重复调用 

```

- [ ] 动画处理
```js
this.$el.addEventListener('transitionend', removeDom); // 在动画结束的时候删除该toast组件，close函数内

instance.$el.removeEventListener('transitionend', removeDom); // 调用close函数前清理监听事件

```
- [ ] Vue.nextTick() 在修改数据后立即使用nextTick方法获取更新后的DOM


- toast 精简代码
```js
import Vue from 'vue';
let ToastConstructor = Vue.extend(require('./Toast.vue'));
let toastPool: Array<any> = [];
let getAnInstance = () => {
    if (toastPool.length) {
        let instance = toastPool[0];
        toastPool.splice(0,1);
        return instance;
    }
    return new ToastConstructor({
        el: document.createElement('div')
    });
};
let returnAnInstance = (instance: any) => {
    if (instance) {
        toastPool.push(instance);
    }
};
let removeDom = (event: any) => {
    if (event.target.parentNode) {
        event.target.parentNode.removeChild(event.target);
    }
};
ToastConstructor.prototype.close = function() {
    this.toastShow = false;
    this.$el.addEventListener('transitionend', removeDom);
    this.closed = true;
    returnAnInstance(this);
};
const Toast = (options: IOptions) => {
    let duration = options.duration || 1500;
    let instance = getAnInstance();
    instance.closed = false;
    instance.unset();
    clearTimeout(instance.timer);
    document.body.appendChild(instance.$el);
    Vue.nextTick(function() {
        instance.toastShow = true;
        instance.$el.removeEventListener('transitionend', removeDom);
        ~duration && (instance.timer = setTimeout(function() {
            if (instance.closed) return;
            instance.close();
        }, duration));
    });
    return instance;
};
export default Toast;
```

##### 修改代码
```js
let typeMap: any = {
  error_network: require('./assets/network-err.svg'),
  failure: require('./assets/failure.svg'),
  success: require('./assets/success.svg'),
  warn: require('./assets/warning.svg'),
};    

export default NetWork = 'error_network'

let typeMap: any = {
  NetWork: require('./assets/network-err.svg'),
  failure: require('./assets/failure.svg'),
  success: require('./assets/success.svg'),
  warn: require('./assets/warning.svg'),
};    

// 使用时
toast({type: NetWork, value: '网络出错'})
```

#### z-index的使用
- 事实上，大多数的一切都比z-index为0的层叠等级低
- z-index: auto 和z-index: 0 从层叠顺序上来说两者等价；从层叠上下文来看，z-index: auto不会产生层级上下文，z-index: 0会产生层级上下文
- z-index的层叠顺序比较止步于父级层叠顺序上下文
- 不犯二原则：对于非浮层元素，避免设置z-index，z-index没有任何道理需要超过2
- 对于浮层元素，可以获取body下子元素的最大z-index值，然后在此基础上加1作为浮动元素的z-index的值
- 层叠顺序大小比较：background/border > 负z-index > block块状水平盒子 > float浮动盒子 > inline/inline-block 水平盒子 > z-index: auto / z-index: 0 / 不依赖于z-index的层叠上下文 > 正z-index 
- 如何形成堆叠上下文
1. 根元素
2. 定位元素
3. opacity小于1时
4. transform不为none时
5. z-index不为auto或者flex-item时


#### 深入剖析组件封装代码
1、Vue.extend(require('./Toast.vue')) 是什么
```js
console.log(require('./Toast.vue'))
import toast from './Toast.vue';
console.log( toast);
console.log(Vue.extend(require('./Toast.vue')));

// 输出结果
ƒ VueComponent (options) {
      this._init(options);
    }
    
// 解析
创建一个VueComponent 构造函数

  Vue.extend = function (extendOptions) {
  // extendOptions配置信息
    extendOptions = extendOptions || {};
    var Super = this;
    var SuperId = Super.cid;
    // cachedCtors 缓存该构造函数
    var cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {});
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }
    // 组件名称
    var name = extendOptions.name || Super.options.name;
    
    if (process.env.NODE_ENV !== 'production' && name) {
      validateComponentName(name);
    }


    var Sub = function VueComponent (options) {
      this._init(options);
    };
    // 构造函数原型继承Vue.propotype
    Sub.prototype = Object.create(Super.prototype);
    // 继承一个类。Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。Sub.prototype.__proto__ ==> Super.prototype
    Sub.prototype.constructor = Sub;
    // 重新指定constructor
    Sub.cid = cid++;
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    );
    // super静态属性指向Vue函数
    Sub['super'] = Super;


    // 对于props、computed 使用代理函数进行处理,因为在扩展Vue实例时这样可以避免为每个创建的实例调用Object.defineProperty 
    // Object.defineProperty 静态方法，直接在对象上定义新属性，或者修改现有属性，返回对象
    if (Sub.options.props) {
      initProps$1(Sub);
    }

    if (Sub.options.computed) {
      initComputed$1(Sub);
    }

    // allow further extension/mixin/plugin usage
    Sub.extend = Super.extend;
    Sub.mixin = Super.mixin;
    Sub.use = Super.use;

    // create asset registers, so extended classes
    // can have their private assets too.
    ASSET_TYPES.forEach(function (type) {
      Sub[type] = Super[type];
    });
    // enable recursive self-lookup
    if (name) {
      Sub.options.components[name] = Sub;
    }

    // keep a reference to the super options at extension time.
    // later at instantiation we can check if Super's options have
    // been updated.
    Sub.superOptions = Super.options;
    Sub.extendOptions = extendOptions;
    Sub.sealedOptions = extend({}, Sub.options);

    // cache constructor
    cachedCtors[SuperId] = Sub;
    return Sub
  };
}


function initProps$1 (Comp) {
  var props = Comp.options.props;
  for (var key in props) {
    proxy(Comp.prototype, "_props", key);
  }
}

function initComputed$1 (Comp) {
  var computed = Comp.options.computed;
  for (var key in computed) {
    defineComputed(Comp.prototype, key, computed[key]);
  }
}


function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    var vm = this;
    // a uid
    vm._uid = uid$3++;

    var startTag, endTag;
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }

    // a flag to avoid this being observed
    vm._isVue = true;
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options);
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      );
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm);
    } else {
      vm._renderProxy = vm;
    }
    // expose real self
    vm._self = vm;
    initLifecycle(vm);
    initEvents(vm);
    initRender(vm);
    callHook(vm, 'beforeCreate');
    initInjections(vm); // resolve injections before data/props
    initState(vm);
    initProvide(vm); // resolve provide after data/props
    callHook(vm, 'created');

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false);
      mark(endTag);
      measure(("vue " + (vm._name) + " init"), startTag, endTag);
    }

    if (vm.$options.el) {
      vm.$mount(vm.$options.el);
    }
  };
}
```
总结：
Vue.extend的关键点在于：创建一个构造函数 function VueComponent(options) {this._init(options)}， 通过原型链继承Vue原型上的属性或者方法，再将Vue的静态方法赋值给该构造函数

定义组件：组件构造函数VueComponent，继承于Vue，这样VueComponent就能调用到Vue的诸多方法，比如_init等等。
组件拥有自己的options（通过构造函数参数传进来的），Vue也有自己的options（主要是一些directive的声明），将两者合并重新覆盖组件的options。why？为何要合并覆盖？VueComponent和Vue都有自己的options，如果不合并的话，根据js原型链查找方式，VueComponent的options会覆盖Vue的options，导致无法访问Vue的options。(Vue的options有什么用处？为何要访问？因为在解析组件dom结构时也需要解析里面的各种指令，因此需要Vue的options。
注册组件：完成组件与标签名的映射
渲染组件：
- 识别组件：在初始化app这个Vue实例过程中，当DOM遍历解析到的时候，因为已经进行了组件注册，因此能识别出标签对应组件
- 组件指令化：把组件当成一个组件指令，那么剩下的就是编写指令的bind方法了，在bind方法中完成组件的渲染与挂载
- 渲染、挂载组件：先进行模板解析、props解析、然后编写组件指令的bind方法，真正初始化组件实例。


全局组件与局部组件：
- 全局组件
- [ ] Vue.extend：通过扩展Vue实例的方法创建组件。创建一个构造函数function VueComponent(options) {this._init(options)},通过原型链继承Vue原型上的属性和方法，再将Vue的静态函数赋值给该构造函数
- [ ] Vue.component: 注册组件。Vue.component的关键点在于将组件函数注入到Vue静态属性中，这样可以根据组件名称找到对应的构造函数，从而创建组件实例。


- 局部组件：在创建Vue实例过程中，经过guardComponents()函数处理之后，能够保证该Vue实例中的components属性，都是由{组件名:组件函数}构成的，这样在后续使用时，可以直接利用实例内部的组件构建函数创建组件实例。

###### new Vue
- 合并配置
- 初始化生命周期
- 初始化事件中心
- 初始化渲染
- 调用 beforeCreate 钩子函数
- init injections and reactivity（这个阶段属性都已注入绑定，而且被 $watch 变成reactivity，但是 $el 还是没有生成，也就是DOM没有生成）
- 初始化state状态（初始化了data、props、computed、watcher）
- 调用created钩子函数。

在初始化的最后，检测到如果有 el 属性，则调用 vm.$mount 方法挂载vm，挂载的目标就是把模板渲染成最终的 DOM。


2、new ToastConstructor({
        el: document.createElement('div')
    });
为何要这样挂载  为何不用mount挂载 
let instance = new ToastConstructor().$mount();
跟普通组件挂载有什么区别   
- [ ] 两种写法等价，区别在于一个在执行构造函数时传参，由构造函数去执行$mount方法，一个等构造函数执行结束后手动调用$mount方法进行挂载

```js
let ToastConstructor = Vue.extend(require('./Toast.vue'));
// Vue.extend返回一个构造函数

let instance = new ToastConstructor({
    el: document.createElement('div'),
  });
  // 使用toast构造函数创建一个实例instance，执行构造函数，调用构造函数内部的init方法。init方法会
  //简写Vue源码init方法
function initMixin (Vue) {
Vue.prototype._init = function (options) {
    var vm = this;
     // a uid
    vm._uid = uid$3++;

    var startTag, endTag;
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }

    // a flag to avoid this being observed
    vm._isVue = true;
    // merge options
// 当符合第一个条件是，即当前这个Vue实例是组件。则执行initInternalComponent方法。(该方法主要就是为vm.$options添加一些属性)
// 当符合第二个条件时，即当前Vue实例不是组件。而是实例化Vue对象时，调用mergeOptions方法。

    if (options && options._isComponent) {
      initInternalComponent(vm, options);
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      );
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm);
    } else {
      vm._renderProxy = vm;
    }
    // expose real self
    vm._self = vm;
    initLifecycle(vm); // 初始化生命周期
    initEvents(vm); // 初始化事件中心
    initRender(vm); // 初始化渲染
    callHook(vm, 'beforeCreate'); // 调用beforeCreate钩子
    initInjections(vm); // resolve injections before data/props 这个阶段属性都已注入绑定，而且被 $watch 变成reactivity，但是 $el 还是没有生成，也就是DOM没有生成
    initState(vm); // 初始化了data、props、computed、watcher
    initProvide(vm); // resolve provide after data/props
    callHook(vm, 'created');

 /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false);
      mark(endTag);
      measure(("vue " + (vm._name) + " init"), startTag, endTag);
    }

    // 重点：这就是为什么也可以通过options来挂载
    if (vm.$options.el) {
      vm.$mount(vm.$options.el);
    }
    // 在初始化的最后，检测到如果有 el 属性，则调用 vm.$mount方法挂载vm，挂载的目标就是把模板渲染成最终的 DOM。
    // $mount是挂在Vue.prototype上的方法，public mount method

 };
}

```


3、Vue.nextTick解析   什么时候执行   下次 DOM 更新循环结束之后执行延迟回调  是怎么实现下次dom更新循环结束之后执行延迟回调的 用的是setTimeout 实现还是 什么实现的

```js
// 源码
var isUsingMicroTask = false;

// 回调函数队列
var callbacks = [];

// 异步锁
var pending = false;

// 执行回调函数
function flushCallbacks() {
  pending = false; // 重置异步锁
  var copies = callbacks.slice(0);
  // 防止nextTick中包含nextTick时出现问题，在执行回调函数队列前，提前复制备份，清空回调函数队列
  callbacks.length = 0;
  // 执行回调函数队列
  for (var i = 0; i < copies.length; i++) {
    copies[i]();
  }
}

// 在这里，我们使用微任务异步延迟包装器(microtasks)。
// 在2.5中，我们使用了（宏）任务（结合了微任务）。
// 但是，在重新粉刷之前更改状态时，它存在一些细微的问题
// (e.g. #6813, out-in transitions).（例如＃6813，外向内过渡）。

// 另外，在事件处理程序中使用（宏）任务会导致一些奇怪的行为
// 无法规避的内容（例如＃7109，＃7153，＃7546，＃7834，＃8109）。
// 因此，我们现在再次在各处使用微任务。
// 这种权衡的主要缺点是存在一些方案
// 微任务的优先级过高，并且据称介于两者之间
// 顺序事件 (e.g. #4521, #6690, which have workarounds)
// 甚至在同一事件冒泡之间
var timerFunc;

// nextTick行为利用了微任务队列，
// 可以通过本机Promise.then或MutationObserver对其进行访问。
// MutationObserver具有更广泛的支持，但是当在触摸事件处理程序中触发时，。
// 它在iOS> = 9.3.3的UIWebView中严重错误
// 它触发几次后完全停止工作...
// 因此，如果是原生的Promise可用
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  var p = Promise.resolve();
  timerFunc = function() {
    p.then(flushCallbacks);
    // 在有问题的UIWebViews中，Promise.then不会完全中断
    // 但是它可能会陷入怪异的状态，在这种状态下，
    // 回调被推入微任务队列，但是直到浏览器需要做其他一些工作才刷新队列
    // 例如处理一个计时器。
    // 因此，我们可以通过添加一个空计时器来“强制”刷新微任务队列。

    if (isIOS) {
      setTimeout(noop);
    }
  };
  isUsingMicroTask = true;
} else if (
  !isIE &&
  typeof MutationObserver !== 'undefined' &&
  (isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]')
) {
  // 如果本地Promise不可用，请使用MutationObserver
  // e.g. PhantomJS, iOS7, Android 4.4
  // (#6466 MutationObserver is unreliable in IE11)
  var counter = 1;
  var observer = new MutationObserver(flushCallbacks);
  var textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true,
  });
  timerFunc = function() {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
  isUsingMicroTask = true;
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // Fallback to setImmediate.
  // Techinically it leverages the (macro) task queue,
  // but it is still a better choice than setTimeout.
  timerFunc = function() {
    setImmediate(flushCallbacks);
  };
} else {
  // Fallback to setTimeout.
  timerFunc = function() {
    setTimeout(flushCallbacks, 0);
  };
}

// 调用的nextTick函数 参数：cb是回调函数即dom更新之后需要做的操作，ctx是回调函数中this的指向，回调的 this 自动绑定到调用它的实例上
function nextTick(cb, ctx) {
  var _resolve;
  // 将回调函数推入回调队列
  callbacks.push(function() {
    if (cb) {
      try {
        cb.call(ctx);
        // 指定了cb的this指向
      } catch (e) {
        handleError(e, ctx, 'nextTick');
      }
    } else if (_resolve) {
      _resolve(ctx);
    }
  });
  // 如果异步锁未锁上，锁上异步锁，调用异步函数，准备等同步函数执行完后，就开始执行回调函数队列
  if (!pending) {
    pending = true;
    timerFunc();
  }
  // $flow-disable-line
  // 如果没有提供回调，支持Promise，返回一个Promise
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(function(resolve) {
      _resolve = resolve;
    });
  }
}

```
总结：接收回调函数，将回调函数推入回调函数队列中。同时，在接收第一个回调函数时，执行能力检测中对应的异步方法（异步方法中调用了回调函数队列）。

####### 思考
- 如何保证只在接收第一个回调函数时执行异步方法？nextTick源码中使用了一个异步锁的概念，即接收第一个回调函数时，先关上锁，执行异步方法。此时，浏览器处于等待执行完同步代码就执行异步代码的情况。
- 怎样在下次dom更新循环结束之后执行延迟回调？每一次事件循环都包含一个microtask（微任务）队列，在循环结束后会依次执行队列中的microtask并移除，然后再开始下一次事件循环。vue进行DOM更新内部也是调用nextTick来做异步队列控制。而当我们自己调用nextTick的时候，它就在更新DOM的那个microtask后追加了我们自己的回调函数，从而确保我们的代码在DOM更新后执行，同时也避免了setTimeout可能存在的多次执行问题。
- Vue.nextTick触发时机：同一事件循环中的代码执行完毕 -> DOM 更新(数据发生改变会触发dom更新渲染) -> nextTick callback触发
- 执行过程：在Vue中的nextTick的源码中，使用了3种情况来做延迟操作，首先会判断我们的设备是否支持Promsie对象，如果支持Promise对象，就使用Promise.then()异步函数来延迟，如果不支持，我们会继续判断我们的设备是否支持 MutationObserver, 如果支持，我们就使用 MutationObserver 来监听。最后如果上面两种都不支持的话，我们会使用 setTimeout 来处理
- Vue.nextTick的使用时机
- [ ] 在Vue生命周期的created()钩子函数中进行的DOM操作一定要放在Vue.nextTick()的回调函数中。在created()钩子函数执行的时候，DOM 其实还未进行任何渲染.
- [ ] 在Vue生命周期的mounted()钩子函数中进行的DOM操作不需要使用Vue.nextTick()。因为该钩子函数执行时所有的DOM挂载和渲染都已完成，此时在该钩子函数中进行任何DOM操作都不会有问题 。

4、从vue-loader源码分析CSS Scoped的实现


5、了解vue-loader,url-loader
- vue-loader: Vue.extend 和 require为什么输入结果一样
```js
console.log(require('./Toast.vue'))
import toast from './Toast.vue';
console.log( toast);
console.log(Vue.extend(require('./Toast.vue')));

// 输出结果
ƒ VueComponent (options) {
      this._init(options);
    }

```
- [ ] 概念： vue-loader是一个webpack的loader，他允许你以一种名为单文件组件SFC的格式撰写Vue组件。vue-loader 会解析文件，提取每个语言块，如有必要会通过其它 loader 处理，最后将他们组装成一个 ES Module，它的默认导出是一个Vue.js组件选项的对象
- [ ] 既然打印出来的结果一样，那么去掉Vue.extend可以吗？本质上不是一个东西，所以不能替换。Vue.extend方便控制实例的销毁，以及通过 extend 的方式能够更好控制“组件”挂载的容器

- 转换资源URL
- [ ] file-loader 可以指定要复制和放置资源文件的位置，以及如何使用版本哈希命名以获得更好的缓存。此外，这意味着 你可以就近管理图片文件，可以使用相对路径而不用担心部署时 URL 的问题。使用正确的配置，webpack 将会在打包输出中自动重写文件路径为正确的 URL。

- [ ] url-loader 允许你有条件地将文件转换为内联的 base-64 URL (当文件小于给定的阈值)，这会减少小文件的 HTTP 请求数。如果文件大于该阈值，会自动的交给 file-loader 处理。
- url-loader: 打印引入的一个图片会输出什么----输出结果是一大串编码
```js
console.log(require('./assets/network-err.svg'));

```
- style-loader: 工作原理大概是把 CSS 内容用 JavaScript 里的字符串存储起来， 在网页执行 JavaScript 时通过 DOM 操作动态地往 HTML head 标签里插入 HTML style 标签。
 url-loader 会将引入的文件进行编码，生成DataURL.相当于把文件翻译成了一串字符串，再把这个字符串打包到javascript

6、js  vue 时  this的指向 
- [ ] 为什么method里面的this可以指向data  两个独立模块this指向可以指向另一个模块

```js
// 构造函数 调用initState(vm) 去初始化data
// const vm: Component = this. 此处 this 即 vue实例，然后赋值给了 vm ， vm将做为参数向下传递，也就是一会以下的 vm 都是指向 vue 实例
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm) // 初始化生命周期
    initEvents(vm) // 初始化事件及时间处理
    initRender(vm) // 初始化render,主要作用是挂载可以将render函数转为vnode的方法。此时是虚拟dom
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm) // 初始化data
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
 

    if (vm.$options.el) {
      vm.$mount(vm.$options.el) // 挂载
    }
  }
}

// initState函数
// 这个初始化解决了实例化是通过 this 获取 data props methods 内数据
// 调用initData(vm)初始化data
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) { // 如果实例定义了 data
    initData(vm) // 初始化 data
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  ………

// initData函数
// data是个函数就调用getData(data, vm)
function initData (vm: Component) {
  let data = vm.$options.data // vm.$options 是访问自定属性，此处就是vue实例中的 this.$data
  data = vm._data = typeof data === 'function' // 先执行三元表达式，然后赋值给 vm._data 和 data，这样子这两个值都是同时变化的，就是 this.$data.xxx 和 this._data 同时变化
    ? getData(data, vm)
    : data || {}
  ………
  
   observe(data, true)
}
// 注意：observe(data, true)作用是递归的让data内的每一项数据都变成响应式的


// getData函数
// 重点：return data.call(vm, vm)
// data函数执行的时候 用 call 方法，让vm继承了data的属性和方法，也就是this继承了this.$option.data的属性和方法。所以可以使用this.XXX
// call 是Function自带的一个属性，可以改变函数体内部的this指向，第一个参数就是this要指向的对象，也就是想指的上下文。后面的参数按顺序传进去，它的函数会被立即调用。
export function getData (data: Function, vm: Component): any {

  pushTarget()
  try {
    return data.call(vm, vm)
  } catch (e) {
    handleError(e, vm, `data()`)
    return {}
  } finally {
    popTarget()
  }

```
- [ ] 为什么data得是个函数而不是个对象
- 因为每个实例可以维护一份被返回对象的独立的拷贝
- 如果 data 是个对象，那么整个vue实例将共享一份数据，也就是各个组件实例间可以随意修改其他组件的任意值


7、event-bus 跟 toast 组件原理的对比性共通性

```js
// event-bus.js
import Vue from 'vue';
export const EventBus = new Vue();

// 调用bus组件
import bus from 'event-bus.js'

    bus.$on('something', ($event) => {
        // ...
       });
   
    bus.$emit('event');

```
总结：toast模板实例化后挂载到DOM上相当于组件在使用，event-bus实例化后不需要挂载到DOM上，调用时使用实例事件（方法）。



#### beforeRouteEnter里的next执行时机，mounted执行时机，dom更新时机，this.$nextTick回调函数执行时机，setTimeout回调函数执行时机
```js
// dom 
    <section
      ref="card"
      v-if="info.list && info.list.length"
    ></section>

//  
async beforeRouteEnter(to, from, next) {
// 请求
let { data } = await window.apis.getPlancard({})

// next回调里赋值
let params = next((vm: any) => {
        vm.info = data
        window.Al.done()
      })
}

mounted() {
this.$nextTick(()=>{})
setTimeout(() => {})
}
```
- [ ] 解析：mounted里面nextTick setTimeout
- nextTick微任务 beforeRouteEnter next微任务 beforeRouteEnter next微任务入微任务队列 => 先执行mounted  nextTick回调函数入微任务队列 =>  清空微任务队列(next回调函数执行  nextTick回调函数被执行，因为next函数中有赋值操作所以将dom更新这个微任务压入微任务队列)  
- setTimeout宏任务  beforeRouteEnter next微任务入微任务队列  先执行mounted  setTimeout入宏任务队列  清空(执行)微任务队列也就是next回调函数被执行 dom更新 执行宏任务(setTimeout回调函数被执行)
事件循环机制每次都是从宏任务开始执行 执行宏任务队列的一个宏任务 执行微任务队列的所有微任务  更新dom
- [ ] 总结：
因为修改数据默认会将更新DOM的回调添加到微任务（microtask）队列中，如果我们将获取DOM的操作放到宏任务（macrotask）中，那么注册的位置就变得不那么重要了，无论在哪里注册都是先更新DOM然后再获取DOM。因为微任务（microtask）中的任务比宏任务（macrotask）中的任务先执行。
- 扩展
- macrotasks(宏任务): setTimeout、setInterval、setImmediate、I/O、UI rendering 等。
- microtasks(微任务): Promise、process.nextTick、MutationObserver ，渲染更新(Vue的更新操作默认会将执行渲染操作的函数添加到微任务队列中)等。


### 虚拟dom生成，虚拟dom转换到真实dom过程
- 虚拟dom生成在初始化_init里面有调用initRender()方法，这个方法是挂载可以将render函数转为vnode的方法
- 虚拟dom到真是dom的转换：实例化后调用$mount，mount调用mountComponent
```js
export function mountComponent(vm, el) {
  vm.$el = el
  ...
  callHook(vm, 'beforeMount')
  ...
  const updateComponent = function () {
    vm._update(vm._render())
  }
  ...
}

```
我们已经执行完了vm._render方法拿到了VNode，现在将它作为参数传给vm._update方法并执行。vm._update这个方法的作用就是就是将VNode转为真实的Dom，不过它有两个执行的时机：
- 首次渲染
当执行new Vue到此时就是首次渲染了，会将传入的VNode对象映射为真实的Dom。
- 更新页面
数据变化会驱动页面发生变化，这也是vue最独特的特性之一，数据改变之前和之后会生成两份VNode进行比较，而怎么样在旧的VNode上做最小的改动去渲染页面，这样一个diff算法还是挺复杂的。