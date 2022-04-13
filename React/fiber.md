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