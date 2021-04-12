BFC block formatting context 块级上下文
形成独立的渲染区域，内部元素的渲染不会受到外部的影响

形成BFC的条件
- 浮动元素  float不为none
- position属性 为 absolute 或者 fixed
- 块级元素overflow不为visible
- flex元素
- inline-block 元素


应用场景
- 清除浮动

### BFC！
- 块格式化上下文，是`用于布局块级盒子的一块渲染区域`，是块盒子布局过程中发生的区域，也就是浮动元素和其他元素交互的区域
- 触发条件：
  - 根元素html
  - overflow不为visible
  - float不为none
  - position为absolute或fixed
  - display 的值为 inline-block、table-cell、table-caption
- 布局规则
  1. `内部的Box会在垂直方向，一个接一个地放置。`
  2. `Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠`
  3. 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
  4. BFC的区域不会与float box重叠。
  5. `BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。`
  6. 计算BFC的高度时，浮动元素也参与计算
- 作用：BFC是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素
  - `分属于不同的BFC时可以阻止margin重叠(2)`
  - `可以包含浮动元素——清除内部浮动(3)`
  - 自适应两栏布局(4)
  - 可以阻止元素被浮动元素覆盖(4)
- 清除浮动：（触发相关元素的BFC）
  - 父元素使用overflow: auto; 会影响滚动条
  - `父元素使用display:flow-root; 可以创建无副作用的BFC，实际上是在创建一个行为类似于根元素的东西时，就能发现这个名字的意义了——即创建一个上下文，里面将进行 flow layout。`