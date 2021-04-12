```js
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

- 当在这些组件之间切换的时候，你有时会想保持这些组件的状态，以避免反复重渲染导致的性能问题
- 原理