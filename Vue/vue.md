### 为什么data是一个函数而不是一个对象

### 讲讲diff


### js 和 ts的区别，ts优点



###  MVVM MVC
- MVC: Model(数据保存) View Control(业务逻辑) 单向数据走向
- MVVM：Model(操作数据) View(UI组件) ViewModel(同步view和model的对象) 双向绑定，view 和 model没有直接关系，它们是通过viewModle来进行交互。
- 区别：`MVVM是数据驱动的，MVC是dom驱动的，mvvm的优点在于不用操作大量的dom，不需要关注model和view之间的关系，而MVC需要在model发生改变时，需要手动的去更新view。`大量操作dom使页面渲染性能降低，使加载速度变慢，影响用户体验。
- `MVVM核心是数据劫持、数据代理、数据编译和“发布订阅模式”`
  - `数据劫持：利用Object.defineProperty()给对象属性添加get,set钩子函数`
  - `数据代理：将data,methods,compted上的数据挂载到vm实例上。`让我们不用每次获取数据时，都通过mvvm._data.a.b这种方式，而可以直接通过mvvm.a.b来获取
  - `数据编译：把{{}},v-model,v-html,v-on,里面的对应的变量用data里面的数据进行替换`
  - `发布订阅：消息的发布和订阅是在观察者的数据绑定中进行的——在get钩子函数被调用时进行数据的订阅(在数据编译时通过 new Watcher()来对数据进行订阅)，在set钩子函数被调用时进行数据的发布`

### `Vue数据绑定：实现双向绑定mvvm`
- `vue是通过数据劫持的方式来做数据绑定的，其中最核心的方法便是通过Object.defineProperty()来实现对属性的劫持，达到监听数据变动的目的`
- 要点：
  - 实现一个`数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者`
  - 实现一个`指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数`
  - 实现一个`Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图`
  - mvvm入口函数

### 观察者模式和发布订阅模式的区别
- 目标和观察者之间是互相依赖的。Vue中响应式数据变化是观察者模式 每个响应式属性都有dep，dep存放了依赖这个属性的watcher，watcher是观测数据变化的函数，如果数据发生变化，dep就会通知所有的观察者watcher去调用更新方法。因此， 观察者需要被目标对象收集，目的是通知依赖它的所有观察者。
- 发布订阅模式是由统一的调度中心调用，发布者和订阅者不知道对方的存在。vue中的事件总线就是使用的发布订阅模式



### Vue依赖收集
- Observe 类主要给响应式对象的属性添加 getter/settter，用于依赖的收集和派发更新
- Dep 类 用于收集当前响应式对象的依赖关系
- Watcher 类是观察者，实例分为渲染watcher、计算属性watcher、侦听器watcher
- `每个组件实例都有响应的watcher实例对象，它在组件渲染过程中把属性记录为依赖，之后当依赖项的setter被调用时，会通知watcher重新计算，从而使他关联的组件得以更新`
- 收集依赖的目的是为了当这些响应式发生变化，触发它们的setter时，能知道应该通知哪些订阅者去做相应的逻辑处理，这个过程称为派发更新
### 派发更新
- `当数据发生变化时，触发setter逻辑，把在依赖过程中订阅的所有观察者，也就是watcher，触发它们的update过程，这个过程中用了队列做进一步优化，在nextTick后执行所有的run, 最后执行它们的回调函数`
`补充：run 函数实际上就是执行 this.getAndInvoke 方法，并传入 watcher 的回调函数。getAndInvoke 函数逻辑也很简单，先通过 this.get() 得到它当前的值，然后做判断，如果满足新旧值不等、新值是对象类型、deep 模式任何一个条件，则执行 watcher 的回调，注意回调函数执行的时候会把第一个和第二个参数传入新值 value 和旧值 oldValue，这就是当我们添加自定义 watcher 的时候能在回调函数的参数中拿到新旧值的原因。`

### nextTick原理
- 为什么用Vue.nextTick()
  - Event Loop 分为宏任务和微任务，无论是执行宏任务还是微任务，完成后都会进入到一下tick，并在两个tick之间进行UI渲染。
  - `由于Vue DOM更新是异步执行的，即修改数据时，视图不会立即更新，而是会监听数据变化，并缓存在同一事件循环中，等同一数据循环中的所有数据变化完成之后，再统一进行视图更新。为了确保得到更新后的DOM，所以设置了 Vue.nextTick()方法。`
- 什么是Vue.nextTick()
  - `在下次DOM更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的DOM。`
- 同一事件循环中的代码执行完毕 -> DOM 更新 -> nextTick callback触发
- 应用场景：
  - `在Vue生命周期的created()钩子函数进行的DOM操作一定要放在Vue.nextTick()的回调函数中。`
  - `原因：是created()钩子函数执行时DOM其实并未进行渲染。`
  - `在数据变化后要执行的某个操作，而这个操作需要使用随数据改变而改变的DOM结构的时候，这个操作应该放在Vue.nextTick()的回调函数中。`
  - 原因：`Vue异步执行DOM更新，只要观察到数据变化，Vue将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变，如果同一个watcher被多次触发，只会被推入到队列中一次。`
- 源码解析
  - 在浏览器环境中，常见的 macro task 有 setTimeout、MessageChannel、postMessage、setImmediate；常见的 micro task 有 MutationObsever 和 Promise.then
  - 把传入的回调函数 cb 压入 callbacks 数组，最后一次性地根据 useMacroTask 条件执行 macroTimerFunc 或者是 microTimerFunc，而它们都会在下一个 tick 执行 flushCallbacks，flushCallbacks 的逻辑非常简单，对 callbacks 遍历，然后执行相应的回调函数。
  - 使用 callbacks 而不是直接在 nextTick 中执行回调函数的原因是保证在同一个 tick 内多次执行 nextTick，不会开启多个异步任务，而把这些异步任务都压成一个同步任务，在下一个 tick 执行完毕。

### 深入响应式原理
- 检测变化的注意事项
  - `当你把一个普通的javascript对象传入Vue实例作为data选项，Vue将遍历此对象所有的property，并使用Object.defineProperty把这些property全部转为getter/setter`
  - `每个组件实例都对应一个watcher实例，他会在组件渲染的过程中把"接触"过的数据property记录为依赖。之后当依赖项的setter触发时会通知watcher，从而使他关联的组件重新渲染`
  - 对于对象：`Vue无法检测property的添加或移除`
    - 因为Vue会在初始化实例的时对property执行getter/setter转化，所以property必须在data对象上存在才能让Vue将它转换为响应式的
    - 对于已经创建好的实例可以Vue不允许动态添加根级别的响应式property。但是可以使用下面两种方法
    ```js
    Vue.set(object, propertyName, value)
    this.$set(this.someObject,'b',2)
    // 若是为已有对象赋值多个新property
    // 代替 `Object.assign(this.someObject, { a: 1, b: 2 })`
    this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
    ```
  - 对于数组，Vue不能检测以下数组的变动 vue2.0
    - `当你利用索引直接设置一个数组项时`，例如： `vm.items[indexOfItem] = newValue`
    - `当你修改数组的长度`时，例如：`vm.items.length = newLength`
  ```js
  // 第一类问题
  Vue.set(vm.items, indexOfItem, newValue)
  vm.items.splice(indexOfItem, 1, newValue)
  vm.$set(vm.items, indexOfItem, newValue)

  // 第二类问题
  vm.items.splice(newLength)
  ``` 
- 异步更新队列
  - `Vue在更新DOM时是异步执行的。只要侦听到数据的变化，Vue将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个watcher被多次触发，只会被推入到队列中一次。`
  - `这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。`
  - 解决办法： Vue.nextTick(callback) vm.$nextTick()
  ```js
  methods: {
    updateMessage: async function () {
      this.message = '已更新'
      console.log(this.$el.textContent) // => '未更新'
      await this.$nextTick()
      console.log(this.$el.textContent) // => '已更新'
    }
  }
  ```

### Vue组件通信
- props $emit
- $parent $child
- provide inject：父组件中通过provide来提供变量, 然后再子组件中通过inject来注入变量。注意: 这里不论子组件嵌套有多深, 只要调用了inject 那么就可以注入provide中的数据，而不局限于只能从当前父组件的props属性中回去数据
- ref refs：如果在普通的 DOM 元素上使用ref，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例，可以通过实例直接调用组件的方法或访问数据
- eventBus
- Vuex

### Proxy defineProperty实现数据监听
- Object.defineProperty() 的问题主要有三个：
  - 不能监听数组的变化
  - 必须遍历对象的每个属性
  - 必须深层遍历嵌套的对象
- Proxy优势（Vue3.0）
  - 针对整个对象：而不是对象的某个属性，也就不需要对keys进行遍历了
  - 支持数组：Proxy不需要对数组重载，省去了众多代码

- Object.defineProperty 并非不能监控数组下标的变化，Vue2.x 中无法通过数组索引来实现响应式数据的自动更新是 Vue 本身的设计导致的，不是 defineProperty 的锅。
- Object.defineProperty 和 Proxy 本质差别是，defineProperty 只能对属性进行劫持，所以出现了需要递归遍历，新增属性需要手动 Observe 的问题。
- Proxy 作为新标准，浏览器厂商势必会对其进行持续优化，但它的兼容性也是块硬伤，并且目前还没有完整的 polyfill 方案。


### 计算属性 vs 侦听属性
- `计算属性本质上是computed watcher，监听的是最终计算的值的变化，最终的值发生变化才会触发渲染watcher重新渲染；主要作用是把数据存储到内存中，减少不必要的请求`
- 侦听属性本质是侦听某个属性 user watcher
- 适用场景：计算属性适合用在模板渲染中，某个值是依赖他的响应式对象甚至是计算属性计算而来；而侦听属性适用于观测到某个值变化后完成一段复杂的业务逻辑

### Vue路由模式
- history：需要后台配置支持
- hash：vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载

### Vue3.0！
- `压缩包体积更小：当前最小化并被压缩的 Vue 运行时大小约为 20kB，Vue 3.0捆绑包的大小大约会减少一半`，即只有10kB！
- `Object.defineProperty -> Proxy`
  - Vue3使用Proxy实现数据劫持，Object.defineProperty只能监听属性，而Proxy能监听整个对象，通过调用new Proxy()，可以创建一个代理用来替代另一个对象被称为目标，这个代理对目标对象进行了虚拟，因此该代理与该目标对象表面上可以被当作同一个对象来对待。
  - 代理允许拦截在目标对象上的底层操作，而这原本是Js引擎的内部能力，拦截行为使用了一个能够响应特定操作的函数，即通过Proxy去对一个对象进行代理之后，我们将得到一个和被代理对象几乎完全一样的对象，并且可以从底层实现对这个对象进行完全的监控。
- diff算法的提升：
  - 在DOM树级别，`在没有动态改变节点结构的模板指令(例如v-if和v-for)的情况下，节点结构保持完全静态，如果我们将一个模板分成由这些结构指令分隔的嵌套块，则每个块中的节点结构将再次完全静态，当我们更新块中的节点时，我们不再需要递归遍历DOM树，该块内的动态绑定可以在一个平面数组中跟踪，这种优化通过将需要执行的树遍历量减少一个数量级来规避虚拟DOM的大部分开销。`
  - `编译器积极地检测模板中的静态节点、子树甚至数据对象，并在生成的代码中将它们提升到渲染函数之外，这样可以避免在每次渲染时重新创建这些对象，从而大大提高内存使用率并减少垃圾回收的频率。`
  - `在元素级别，编译器还根据需要执行的更新类型，为每个具有动态绑定的元素生成一个优化标志`；例如具有动态类绑定和许多静态属性的元素将收到一个标志，提示只需要进行类检查，运行时将获取这些提示并采用专用的快速路径。

- 生命周期的改变
  - beforeCreate -> setup、created -> setup、beforeMount -> onBeforeMount、mounted -> onMounted；
  - 增加了setup这个生命周期
- 新增组合式 API setup
  - 如果能够将同一个逻辑关注点相关代码收集在一起会更好，而这正是组合式 API 使我们能够做到的
- 带 ref 的响应式变量：ref 为我们的值创建了一个响应式引用。在整个组合式 API 中会经常使用引用的概念。

### defineProperty和Proxy区别
- 对于Object.defineProperty来说，处理对象和数组一样，**只是在初始化时去改写get和set达到监测数组或对象的变化。对于新增的属性，需要手动再初始化。**
- 对于数组来说，只不过特别了点，某些方法例如push、unshift等也会新增索引。对于新增的索引亦可以添加observe从而达到监听的效果。而pop和shift则会删除更新索引，也会出发Object.defineProperty的get和set。对于重新赋值length的数组，不会新增索引，因为不清楚新增的索引数量。
- Proxy可以直接劫持整个对象并返回一个新对象。Proxy返回的是一个新对象，故我们可以只操作新对象达到目的，而不是像Object.defineProperty一样笨拙的遍历对象属性并修改。Proxy劣势是兼容性问题(IE不支持)。Proxy可以监听到十几种方法

### Vue 与 React 框架对比
- 相同点：
  - 都引入了虚拟dom
- 不同点：
  - 虚拟DOM：
    - `Vue可以更快地计算出Virtual DOM的差异，这是由于它在渲染过程中，会跟踪每一个组件的依赖关系，不需要重新渲染整个组件树`。
    - `对于React而言，每当应用的状态被改变时，全部子组件都会重新渲染`。当然，这可以通shouldComponentUpdate这个生命周期方法来进行控制，但Vue将此视为默认的优化
  - 模板 vs JSX
    - `Vue中可以使用单文件组件,Vue鼓励你去写近似常规HTML的模板`。写起来很接近标准HTML元素，只是多了一些属性
    - `React中使用JSX模板，赋予了开发者许多编程能力`。样式文件单独引入，设置img src时要import进来  不能直接写src="xxx.png"
  - 状态管理 vs 对象属性
    - `在React中你需要使用setState()方法去更新状态`
    - `在Vue中，state对象并不是必须的，数据由data属性在Vue对象中进行管理。`

### Vue生命周期
![vue生命周期](assets/vue生命周期.png)

### 如何实现axios拦截器


### 如何实现Vue拦截器


### Vue实例和js实例
- Vue实例：每个Vue应用都是Vue函数创建一个Vue实例开始
```js
new Vue({
  el: '#app',
  data: obj
})
```

### 路由自定义属性 meta


### 轮播图原理（无缝切换原理）



