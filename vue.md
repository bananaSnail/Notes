###  MVVM MVC
- MVC: Model(数据保存) View Control(业务逻辑) 单向数据走向
- MVVM：Model(操作数据) View(UI组件) ViewModel(同步view和model的对象) 双向绑定，view 和 model没有直接关系，它们是通过viewModle来进行交互。
- 区别：MVVM是数据驱动的，MVC是dom驱动的，mvvm的优点在于不用操作大量的dom，不需要关注model和view之间的关系，而MVC需要在model发生改变时，需要手动的去更新view。大量操作dom使页面渲染性能降低，使加载速度变慢，影响用户体验。
- MVVM核心是数据劫持、数据代理、数据编译和“发布订阅模式”
  - 数据劫持：利用Object.defineProperty()给对象属性添加get,set钩子函数
  - 数据代理：将data,methods,compted上的数据挂载到vm实例上。让我们不用每次获取数据时，都通过mvvm._data.a.b这种方式，而可以直接通过mvvm.a.b来获取
  - 数据编译：把{{}},v-model,v-html,v-on,里面的对应的变量用data里面的数据进行替换
  - 发布订阅：消息的发布和订阅是在观察者的数据绑定中进行的——在get钩子函数被调用时进行数据的订阅(在数据编译时通过 new Watcher()来对数据进行订阅)，在set钩子函数被调用时进行数据的发布

### Vue数据绑定：实现双向绑定mvvm
- vue是通过数据劫持的方式来做数据绑定的，其中最核心的方法便是通过Object.defineProperty()来实现对属性的劫持，达到监听数据变动的目的
- 要点：
  - 实现一个数据监听器Observer，能够对数据对象的所有属性进行监听，如有变动可拿到最新值并通知订阅者
  - 实现一个指令解析器Compile，对每个元素节点的指令进行扫描和解析，根据指令模板替换数据，以及绑定相应的更新函数
  - 实现一个Watcher，作为连接Observer和Compile的桥梁，能够订阅并收到每个属性变动的通知，执行指令绑定的相应回调函数，从而更新视图
  - mvvm入口函数

### Vue依赖收集
- Observe 类主要给响应式对象的属性添加 getter/settter，用于依赖的收集和派发更新
- Dep 类 用于收集当前响应式对象的依赖关系
- Watcher 类是观察者，实例分为渲染watcher、计算属性watcher、侦听器watcher
- `每个组件实例都有响应的watcher实例对象，它在组件渲染过程中把属性记录为依赖，之后当依赖项的setter被调用时，会通知watcher重新计算，从而使他关联的组件得以更新`
- 收集依赖的目的是为了当这些响应式发生变化，触发它们的setter时，能知道应该通知哪些订阅者去做相应的逻辑处理，这个过程称为派发更新
### 派发更新
- 当数据发生变化时，触发setter逻辑，把在依赖过程中订阅的所有观察者，也就是watcher，触发它们的update过程，这个过程中用了队列做进一步优化，在nextTick后执行所有的run, 最后执行它们的回调函数
`补充：run 函数实际上就是执行 this.getAndInvoke 方法，并传入 watcher 的回调函数。getAndInvoke 函数逻辑也很简单，先通过 this.get() 得到它当前的值，然后做判断，如果满足新旧值不等、新值是对象类型、deep 模式任何一个条件，则执行 watcher 的回调，注意回调函数执行的时候会把第一个和第二个参数传入新值 value 和旧值 oldValue，这就是当我们添加自定义 watcher 的时候能在回调函数的参数中拿到新旧值的原因。`

### nextTick原理
- 在浏览器环境中，常见的 macro task 有 setTimeout、MessageChannel、postMessage、setImmediate；常见的 micro task 有 MutationObsever 和 Promise.then
- 把传入的回调函数 cb 压入 callbacks 数组，最后一次性地根据 useMacroTask 条件执行 macroTimerFunc 或者是 microTimerFunc，而它们都会在下一个 tick 执行 flushCallbacks，flushCallbacks 的逻辑非常简单，对 callbacks 遍历，然后执行相应的回调函数。
- 使用 callbacks 而不是直接在 nextTick 中执行回调函数的原因是保证在同一个 tick 内多次执行 nextTick，不会开启多个异步任务，而把这些异步任务都压成一个同步任务，在下一个 tick 执行完毕。

### 深入响应式原理
- 检测变化的注意事项
  - 当你把一个普通的javascript对象传入Vue实例作为data选项，Vue将遍历此对象所有的property，并使用Object.defineProperty把这些property全部转为getter/setter
  - 每个组件实例都对应一个watcher实例，他会在组件渲染的过程中把"接触"过的数据property记录为依赖。之后当依赖项的setter触发时会通知watcher，从而使他关联的组件重新渲染
  - 对于对象：Vue无法检测property的添加或移除
    - 因为Vue会在初始化实例的时对property执行getter/setter转化，所以property必须在data对象上存在才能让Vue将它转换为响应式的
    - 对于已经创建好的实例可以Vue不允许动态添加根级别的响应式property。但是可以使用下面两种方法
    ```js
    Vue.set(object, propertyName, value)
    this.$set(this.someObject,'b',2)
    // 若是为已有对象赋值多个新property
    // 代替 `Object.assign(this.someObject, { a: 1, b: 2 })`
    this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
    ```
  - 对于数组，Vue不能检测以下数组的变动
    - 当你利用索引直接设置一个数组项时，例如： `vm.items[indexOfItem] = newValue`
    - 当你修改数组的长度时，例如：`vm.items.length = newLength`
  ```js
  // 第一类问题
  Vue.set(vm.items, indexOfItem, newValue)
  vm.items.splice(indexOfItem, 1, newValue)
  vm.$set(vm.items, indexOfItem, newValue)

  // 第二类问题
  vm.items.splice(newLength)
  ``` 
- 异步更新队列
  - Vue在更新DOM时是异步执行的。只要侦听到数据的变化，Vue将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个watcher被多次触发，只会被推入到队列中一次。
  - 这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。
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

### Vue状态管理 --> Vuex


### Vue组件通信
- props $emit
- $parent $child
- provide inject：父组件中通过provide来提供变量, 然后再子组件中通过inject来注入变量。注意: 这里不论子组件嵌套有多深, 只要调用了inject 那么就可以注入provide中的数据，而不局限于只能从当前父组件的props属性中回去数据
- ref refs：如果在普通的 DOM 元素上使用ref，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例，可以通过实例直接调用组件的方法或访问数据
- eventBus
- Vuex

### Proxy defineProperty实现数据监听
- Object.defineProperty 并非不能监控数组下标的变化，Vue2.x 中无法通过数组索引来实现响应式数据的自动更新是 Vue 本身的设计导致的，不是 defineProperty 的锅。
- Object.defineProperty 和 Proxy 本质差别是，defineProperty 只能对属性进行劫持，所以出现了需要递归遍历，新增属性需要手动 Observe 的问题。
- Proxy 作为新标准，浏览器厂商势必会对其进行持续优化，但它的兼容性也是块硬伤，并且目前还没有完整的 polyfill 方案。

### Virtual DOM
- 概念：虚拟DOM就是用JS去按照DOM结构来实现的树形结构对象；一个普通的 JavaScript 对象，包含了 tag、props、children 三个属性
- 优势：
  - 减少 JavaScript 操作真实 DOM 的带来的性能消耗
  - 抽象了原本的渲染过程，实现了跨平台的能力，而不仅仅局限于浏览器的 DOM，可以是安卓和 IOS 的原生组件，可以是近期很火热的小程序，也可以是各种GUI
- 过程：
  - 用JS对象模拟DOM（虚拟DOM）
  - 把此虚拟DOM转成真实DOM并插入页面中（render）createElement，根据 type 生成对应的 DOM，把 data 里定义的 各种属性设置到 DOM 上
  - 如果有事件发生修改了虚拟DOM，比较两棵虚拟DOM树的差异，得到差异对象（diff）；找到新旧虚拟 DOM 对应的位置，然后进行移动，那么就能够尽量减少 DOM 的操作
  - 把差异对象应用到真正的DOM树上（patch）

### Diff
- 头头对比: 对比两个数组的头部，如果找到，把新节点patch到旧节点，头指针后移
- 尾尾对比: 对比两个数组的尾部，如果找到，把新节点patch到旧节点，尾指针前移
- 旧尾新头对比: 交叉对比，旧尾新头，如果找到，把新节点patch到旧节点，旧尾指针前移，新头指针后移
- 旧头新尾对比: 交叉对比，旧头新尾，如果找到，把新节点patch到旧节点，新尾指针前移，旧头指针后移
- 利用key对比: 用新指针对应节点的key去旧数组寻找对应的节点,这里分三种情况,当没有对应的key，那么创建新的节点,如果有key并且是相同的节点，把新节点patch到旧节点,如果有key但是不是相同的节点，则创建新节点


### 计算属性 vs 侦听属性
- 计算属性本质上是computed watcher，监听的是最终计算的值的变化，最终的值发生变化才会触发渲染watcher重新渲染
- 侦听属性本质是侦听某个属性 user watcher
- 适用场景：计算属性适合用在模板渲染中，某个值是依赖他的响应式对象甚至是计算属性计算而来；而侦听属性适用于观测到某个值变化后完成一段复杂的业务逻辑


