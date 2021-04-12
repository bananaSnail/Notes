### 普通插槽
- 插槽将<slot></slot>作为子组件承载分发的载体
- 基本用法

```js
var child = {
  template: `<div class="child"><slot></slot></div>`
}
var vm = new Vue({
  el: '#app',
  components: {
    child
  },
  template: `<div id="app"><child>test</child></div>`
})
// 最终渲染结果
<div class="child">test</div>


```

-  组件挂载原理
    - 父组件会优先于子组件进行实例的挂载，模板的解析和render函数的生成阶段在处理上没有特殊的差异。接下来是render函数生成Vnode的过程，在这个阶段会遇到子的占位符节点(即：child),因此会为子组件创建子的Vnode。createComponent执行了创建子占位节点Vnode的过程

### 具有后备内容的插槽
- 在父组件没有提供内容时，slot的后备内容被渲染。

```js
var child = {
  template: `<div class="child"><slot>后备内容</slot></div>`
}
var vm = new Vue({
  el: '#app',
  components: {
    child
  },
  template: `<div id="app"><child></child></div>`
})
// 父没有插槽内容，子的slot会渲染后备内容
<div class="child">后备内容</div>

```

### 具名插槽
```js
var child = {
  template: `<div class="child"><slot name="header"></slot><slot name="footer"></slot></div>`,
}
var vm = new Vue({
  el: '#app',
  components: {
    child
  },
  template: `<div id="app"><child><template v-slot:header><span>头部</span></template><template v-slot:footer><span>底部</span></template></child></div>`,
})

```

### 作用域插槽
- 可以利用作用域插槽让父组件的插槽内容访问到子组件的数据，具体的用法是在子组件中以属性的方式记录在子组件中，父组件通过v-slot:[name]=[props]的形式拿到子组件传递的值。
```js

var child = {
  template: `<div><slot :user="user"></div>`,
  data() {
    return {
      user: {
        firstname: 'test'
      }
    }
  }
}
var vm = new Vue({
  el: '#app',
  components: {
    child
  },
  template: `<div id="app"><child><template v-slot:default="slotProps">{{slotProps.user.firstname}}</template></child></div>`
})

```