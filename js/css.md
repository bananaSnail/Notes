### 深入理解line-height
- div高度是被行高撑开的，文字也有行高
- 行高的垂直居中性：文字内容与其占据的空间共用中垂线
- 实现单行文字垂直居中时大都会把line-height 设置跟height 一样的值，但其实去掉height，只设置line-height文字还是垂直居中的
- line-height 实现多行文字垂直居中 * {line-height: 150%}
- 在某些情形下，line-height 和 height 可以互换，使用行高代替高度避免haslayout

### position: sticky
- 粘性定位可以被认为相对定位和固定定位的混合，元素在跨越特定阈值前为相对定位，之后为固定定位；
- 可以实现页面滚动一段距离后固定导航栏 #one {position: sticky; top: 10px}

### flex布局
- flex-grow: 扩展的是flex子项所占据的宽度，扩展所侵占的空间就是除去元素外的剩余的空白间隙；空间足够时如何利用空间
- flex-shrink: 主要处理当flex容器空间不足时候，单个元素的收缩比例；空间不足时收缩的尺寸
- flex-basic: 定义了在分配剩余空间之前元素的默认大小。
- flex: none(0 0 auto) | auto(0 1 auto) | [<'flex-grow'> <'flex-shrink'>? || <'flex-basic'>] 默认值 0 1 auto
- align-self: 指控制单独某一个flex子项的垂直对齐方式
- 在flex布局中flex子元素设置的float、clear以及vertical-align属性都没用
- flex布局适合应用程序的组件和小规模布局

### 回流重绘
- 浏览器使用流式布局模型
- 浏览器会把HTML解析成DOM，把CSS解析成CSSOM，DOM和CSSOM合并成了renderTree
- 回流会引起重绘，重绘不一定引起回流；回流比重绘的代价更高
- 回流重绘：触发回流一定会引起重绘，重绘不一定会引起回流
- 回流：确定对象位置，浏览器根据各种样式计算出结果，将对象放在他应该出现的位置
- 重绘：确定位置后绘制出该对象
- 导致回流因素：
  - 调整窗口大小
  - 改变字体
  - 增加或者移除类
  - 改变style
  - 操作class
  - 操作dom
  - 计算offsetWidth 和 offsetHeight
  - 内容变化，如input框输入文字
- 优化：
  - 设置元素样式时用类名设置
  - 避免设置多项内联样式
  - 避免使用table布局
  - 应用元素动画，使用position时使用fixed 和 absolute(这两个属性只影响自身的回流，不影响其他元素)

- 注：
 - 回流(reflow), 就是布局引擎为 frame 计算图形, 确定节点位置的一个动作。其中触发回流的原因主要是 DOM 节点大小或位置的改变才会触发回流。
 - 重绘则是表面的视觉效果的改变从而引发重绘。

### H5 移动端 （待补充）
- rem
```js
// deviceWidth = 320，font-size = 320 / 6.4 = 50px
// deviceWidth = 375，font-size = 375 / 6.4 = 58.59375px
// deviceWidth = 414，font-size = 414 / 6.4 = 64.6875px
// deviceWidth = 500，font-size = 500 / 6.4 = 78.125px
document.documentElement.style.fontSize = document.documentElement.clientWidth / 6.4 + 'px';

// css 
// width: 6.4rem

// 如果设计稿基于iphone6，横向分辨率为750，body的width为750 / 100 = 7.5rem
// 如果设计稿基于iphone4/5，横向分辨率为640，body的width为640 / 100 = 6.4rem
```
- viewport
```js
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```
- device-width = 设备物理分辨率 / (devicePixelRatio * scale) ; devicePixelRatio 称为设备像素比，每款设备该值是不变的，已知的
