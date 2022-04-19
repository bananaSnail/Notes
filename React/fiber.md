### fiber
- react16推出为了解决项目性能问题而推出的架构
- fiber涉及到批量更新和时间切片的批量更新
- 其能够将任务分片，划分优先级，同时能够实现类似于操作系统中对线程的抢占式调度(requestIdleCallback)，非常强大。

### Fiber优先级？
### react
- 虚拟DOM层负责描述结构与逻辑
- 内部组件层负责组件更新如React.Render、setState、forceUpdate；多次setState只执行一次真实的渲染，在合适的时机调用组件生命周期钩子
- 底层渲染层
#react将内部组件层改成fiber这种数据结构#，fiber节点拥有return、child、sibling属性，分别对应父节点、第一个子节点、右边兄弟节点，通过这些属性可以形成链表结构，实现深度优化遍历。

### 渲染过程
- 第一次渲染后，react得到了一个fiber tree即current tree，用于反映渲染UI的应用程序的状态。
- 当react开始处理更新时，react会构建一个workInProgress tree，反映了要刷新到屏幕的未来状态。所有work都会在workInProgress中fiber执行。当React遍历current tree时，对于每个现有fiber节点，它会使用render方法返回的React元素中的数据创建一个备用(alternate)fiber节点，这些节点用于构成workInProgress tree(备用tree)，处理完更新并完成所有相关工作，react将备用tree刷新到屏幕上，一旦备用tree呈现在屏幕上后就变成了current tree

### 如何更新？如何DFS？
- react把渲染分为两个阶段，render和commit。
- render阶段是生成一个部分节点标记了side effects的fiber节点树。render阶段可以异步执行，render可以根据时间来处理一个或多个fiber节点，然后停止已完成的工作，并让出调度权去处理某些事件。然后它从它停止的地方继续。但有时候，它可能需要丢弃完成的工作并再次从头。
- commit阶段是同步执行的。

### render阶段详解。
- performUnitOfWork、beginWork、completeUnitOfWork、completeWork 
- 对fiber tree进行深度优先搜索(DFS)

### commit 阶段，在提交阶段运行的主要功能是commitRoot。它会执行以下操作:

- 在标记了Snapshot effect的节点上使用getSnapshotBeforeUpdate生命周期方法
- 在标记了Deletion effect的节点上调用componentWillUnmount生命周期方法
- 执行所有DOM插入，更新和删除
- 将finishedWork树设置为current树
- 在标记了Placement effect的节点上调用componentDidMount生命周期方法
- 在标记了Update effect的节点上调用componentDidUpdate生命周期方法

### DOM updates
- commitAllHostEffects是React执行DOM更新的函数。该函数基本上定义了需要为节点完成并执行它的操作类型.

### Post-mutation lifecycle methods
- commitAllLifecycles是React调用所有剩余生命周期方法componentDidUpdate和componentDidMount的函数.

### window.requestIdleCallback()方法
- 该插入一个函数，这个函数将在浏览器空闲时期被调用。这使开发者能够在主事件循环上执行后台和低优先级工作，而不会影响延迟关键事件，如动画和输入响应。函数一般会按先进先调用的顺序执行，然而，如果回调函数指定了执行超时时间timeout，则有可能为了在超时前执行函数而打乱执行顺序。

参考文献：https://zhuanlan.zhihu.com/p/37095662 https://zhuanlan.zhihu.com/p/57346388

### fiber
- Fiber 是对react reconciler（调和） 核心算法的重构。关键特性：
    - 增量渲染（把渲染任务拆分成块，匀到多帧）
    - 更新时能够暂停，终止，复用渲染任务
    - 给不同类型的更新赋予优先级
    - 并发方面新的基础能力
- reconcile过程分为2个阶段
    - （可中断）render/reconciliation 通过构造workInProgress tree得出change
    - （不可中断）commit 应用这些DOM change（更新DOM树、调用组件生命周期函数以及更新ref等内部状态）
构建workInProgress tree的过程就是diff的过程，通过requestIdleCallback来调度执行一组任务，每完成一个任务后回来看看有没有插队的（更紧急的），每完成一组任务，把时间控制权交还给主线程，直到下一次requestIdleCallback回调再继续构建workInProgress tree
- 生命周期也被分成了两个阶段：
```js
// 第1阶段 render/reconciliation
componentWillMount
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate

// 第2阶段 commit
componentDidMount
componentDidUpdate
componentWillUnmount
```
    - 第1阶段的生命周期函数可能会被多次调用，默认以low优先级执行，被高优先级任务打断的话，稍后重新执行。
- fiber tree与workInProgress tree
    - 双缓冲技术：指的是workInProgress tree构造完毕，得到的就是新的fiber tree，然后把current指针指向workInProgress tree，由于fiber与workInProgress互相持有引用，旧fiber就作为新fiber更新的预留空间，达到复用fiber实例的目的。
    - 每个fiber上都有个alternate属性，也指向一个fiber，创建workInProgress节点时优先取alternate，没有的话就创建一个
- fiber 中断 恢复
    - 中断：检查当前正在处理的工作单元，保存当前成果（firstEffect, lastEffect），修改tag标记一下，迅速收尾并再开一个requestIdleCallback，下次有机会再做
    - 断点恢复：下次再处理到该工作单元时，看tag是被打断的任务，接着做未完成的部分或者重做
- React 页面渲染
    - 首次渲染时在 prerender 阶段会初始化 current 树
    - 在非首次渲染的 render 阶段 React 依赖 current 树通过工作循环（workLoop）构建 workInProgress 树；
    - 在 workInProgress 树进行一些更新计算，得到副作用列表（Effect List）；
    - 在 commit 阶段将副作用列表渲染到页面后，将 current 树替换为 workInProgress 树（执行current = workInProgress）。
    - 补充：
        - current 树是未更新前应用程序对应的 Fiber 树，workInProgress 树是需要更新屏幕的 Fiber 树。
        - FiberRootNode构造函数只有一个实例就是 fiberRoot 对象。而每个 Fiber 节点都是 FiberNode 构造函数的实例，它们通过return，child和sibling三个属性连接起来，形成了一个巨大链表。React 对每个节点的更新计算都是在这个链表上完成的