###  MVVM MVC
- MVC: Model(数据保存) View Control(业务逻辑) 单向数据走向
- MVVM：Model(操作数据) View(UI组件) ViewModel(同步view和model的对象) 双向绑定，view 和 model没有直接关系，它们是通过viewModle来进行交互。
- 区别：MVVM是数据驱动的，MVC是dom驱动的，mvvm的优点在于不用操作大量的dom，不需要关注model和view之间的关系，而MVC需要在model发生改变时，需要手动的去更新view。大量操作dom使页面渲染性能降低，使加载速度变慢，影响用户体验。
- MVVM核心是数据劫持、数据代理、数据编译和“发布订阅模式”
  - 数据劫持：就是给对象属性添加get,set钩子函数
  - 数据代理：将data,methods,compted上的数据挂载到vm实例上。让我们不用每次获取数据时，都通过mvvm._data.a.b这种方式，而可以直接通过mvvm.b.a来获取
  - 数据编译：把{{}},v-model,v-html,v-on,里面的对应的变量用data里面的数据进行替换
  - 发布订阅：消息的发布和订阅是在观察者的数据绑定中进行的——在get钩子函数被调用时进行数据的订阅(在数据编译时通过 new Watcher()来对数据进行订阅)，在set钩子函数被调用时进行数据的发布


### vue响应式原理
- 派发更新：收集依赖的目的是为了当这些响应式数据发生变化，触发它们的setter时，能知道应该通知哪些订阅者去做响应的逻辑处理

### Vue中发布订阅模式


### Proxy defineProperty实现数据监听
- Object.defineProperty 并非不能监控数组下标的变化，Vue2.x 中无法通过数组索引来实现响应式数据的自动更新是 Vue 本身的设计导致的，不是 defineProperty 的锅。
- Object.defineProperty 和 Proxy 本质差别是，defineProperty 只能对属性进行劫持，所以出现了需要递归遍历，新增属性需要手动 Observe 的问题。
- Proxy 作为新标准，浏览器厂商势必会对其进行持续优化，但它的兼容性也是块硬伤，并且目前还没有完整的 polyfill 方案。

