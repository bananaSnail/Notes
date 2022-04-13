### hooks *函数式组件才有hooks，类组件对应着生命周期*
- hooks是一种函数，该函数允许您从函数式组件“勾住”React状态和生命周期功能，hooks在类内部不起作用
### hooks使用规则
- 只在顶层调用hook，*不要在循环中，条件或嵌套函数中调用hook。相反在函数顶层使用hooks*。因为*memoizedState 数组（单链表结构实现的数组）是按 hook定义的顺序来放置数据的，如果 hook 顺序变化，memoizedState 并不会感知到。*
- 只在react functions中调用hooks，在react函数式组件中调用hooks，在自定义hooks中调用hooks。

### useEffect
### useLayoutEffect
### useState
### useReducer
### useMemo
### useCallback
### useRef


- todo
https://zhuanlan.zhihu.com/p/53077376?utm_source=wechat_session&utm_medium=social&utm_oi=757231104158617600

https://github.com/brickspert/blog/issues/26
