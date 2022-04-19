### hooks *函数式组件才有hooks，类组件对应着生命周期*
- hooks是一种函数，该函数允许您从函数式组件“勾住”React状态和生命周期功能，hooks在类内部不起作用
### hooks使用规则
- 只在顶层调用hook，*不要在循环中，条件或嵌套函数中调用hook。相反在函数顶层使用hooks*。因为*memoizedState 数组（单链表结构实现的数组）是按 hook定义的顺序来放置数据的，如果 hook 顺序变化，memoizedState 并不会感知到。*
- 只在react functions中调用hooks，在react函数式组件中调用hooks，在自定义hooks中调用hooks。

### useEffect
https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/
- 如何用useEffect模拟componentDidMount生命周期？
  - 虽然可以使用**useEffect(fn, [])，但它们并不完全相等。和componentDidMount不一样，useEffect会捕获 props和state。所以即便在回调函数里，你拿到的还是初始的props和state。如果你想得到“最新”的值，你可以使用ref。**effects的心智模型和componentDidMount以及其他生命周期是不同的，试图找到它们之间完全一致的表达反而更容易使你混淆。useEffect的心智模型更接近于实现状态同步，而不是响应生命周期事件。
- 如何正确地在useEffect里请求数据？[]又是什么？
- 我应该把函数当做effect的依赖吗？
- 为什么有时候会出现无限重复请求的问题？
- 为什么有时候在effect里拿到的是旧的state或prop？

### useReducer
### useRef
### useEffect与useLayoutEffect
https://zhuanlan.zhihu.com/p/53077376
- useEffect 是异步执行的，而useLayoutEffect是同步执行的。
- useEffect 的执行时机是浏览器完成渲染之后，而 useLayoutEffect 的执行时机是浏览器把内容真正渲染到界面之前，和 componentDidMount 等价。
- 执行时机：
  - useEffect
    - 触发changeState
    - react 渲染组件
    - 更新视图
    - 执行useEffect
  - useLayoutEffect
    - 触发changeState
    - react 渲染组件
    - 执行useLayout，react等待useLayout执行完
    - 更新视图

- 总结：
  - 优先使用 useEffect，因为它是异步执行的，不会阻塞渲染
  - 会影响到渲染的操作尽量放到 useLayoutEffect中去，避免出现闪烁问题
  - useLayoutEffect和componentDidMount是等价的，会同步调用，阻塞渲染
  - 在服务端渲染的时候使用会有一个 warning，因为它可能导致首屏实际内容和服务端渲染出来的内容不一致。

### useCallback 返回一个 memoized 回调函数。
- 把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。
- useCallback(fn, deps) 相当于 useMemo(() => fn, deps)

### useMemo
- 把“创建”函数和依赖项数组作为参数传入 useMemo，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。
- 记住，传入 useMemo 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 useEffect 的适用范畴，而不是 useMemo。
- 如果没有提供依赖项数组，useMemo 在每次渲染时都会计算新的值。

### React useCallback useMemo 区别
- 这两个api，其实概念上还是很好理解的，一个是「缓存函数」， 一个是缓存「函数的返回值」。
- 在组件内部，那些会成为其他useEffect依赖项的方法，建议用 useCallback 包裹，或者直接编写在引用它的useEffect中。
- 如果你的function会作为props传递给子组件，请一定要使用 useCallback 包裹，对于子组件来说，如果每次render都会导致你传递的函数发生变化，可能会对它造成非常大的困扰。同时也不利于react做渲染优化。
- 对于使用高阶函数的场景，建议一律使用 useMemo

### hooks原理
- 手动实现useState、useEffect
```js
let memoizedState = []; // hooks 存放在这个数组
let cursor = 0; // 当前 memoizedState 下标

function useState(initialValue) {
  memoizedState[cursor] = memoizedState[cursor] || initialValue;
  const currentCursor = cursor;
  function setState(newState) {
    memoizedState[currentCursor] = newState;
    render();
  }
  return [memoizedState[cursor++], setState]; // 返回当前 state，并把 cursor 加 1
}

function useEffect(callback, depArray) {
  const hasNoDeps = !depArray;
  const deps = memoizedState[cursor];
  const hasChangedDeps = deps
    ? !depArray.every((el, i) => el === deps[i])
    : true;
  if (hasNoDeps || hasChangedDeps) {
    callback();
    memoizedState[cursor] = depArray;
  }
  cursor++;
}
```
- QA
    - Q：为什么只能在函数最外层调用 Hook？为什么不要在循环、条件判断或者子函数中调用。
    - A：memoizedState 数组是按 hook定义的顺序来放置数据的，如果 hook 顺序变化，memoizedState 并不会感知到。
    - Q：自定义的 Hook 是如何影响使用它的函数组件的？
    - A：共享同一个 memoizedState，共享同一个顺序。
    - Q：“Capture Value” 特性是如何产生的？
    - A：每一次 ReRender 的时候，都是重新去执行函数组件了，对于之前已经执行过的函数组件，并不会做任何操作。
- 真正的 React 实现
    - React 中是通过类似单链表的形式来代替数组的。通过 next 按顺序串联所有的 hook。