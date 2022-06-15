### 深入理解line-height
- div高度是被行高撑开的，文字也有行高
- 行高的垂直居中性：文字内容与其占据的空间共用中垂线
- 实现单行文字垂直居中时大都会把line-height 设置跟height 一样的值，但其实去掉height，只设置line-height文字还是垂直居中的
- line-height 实现多行文字垂直居中 * {line-height: 150%}
- 在某些情形下，line-height 和 height 可以互换，使用行高代替高度避免haslayout

### grid和flex区别是什么？适用什么场景？
- Flexbox 是一维布局系统，适合做局部布局，比如导航栏组件。
- Grid 是二维布局系统，通常用于整个页面的规划。
- 二者从应用场景来说并不冲突。虽然 Flexbox 也可以用于大的页面布局，但是没有 Grid 强大和灵活。二者结合使用更加轻松。
- Grid：grid-gap指定行间距/列间距；grid-template-areas：网格布局允许指定"区域"（area），一个区域由单个或多个单元格组成。grid-template-areas属性用于定义区域。
```css
grid-template-areas: 'a b c'
                     'd e f'
                     'g h i';
```

### 浮动

### 定位
### 语义化
- 正确的标签做正确的事情
- 页面内容结构化
- 无CSS样子时也容易阅读，便于阅读维护和理解
- 便于浏览器、搜索引擎解析。 利于爬虫标记、利于SEO

### 选择器
- CSS 优先规则3：`优先级关系：内联样式 > ID 选择器 > 类选择器 = 属性选择器 = 伪类选择器 > 标签选择器 = 伪元素选择器`
- 多元素组合选择器
  - `div,p : 同时匹配到 div 元素或 p 元素`，元素间用逗号分隔
  - `div p ：后代选择器，匹配div元素下的所有p标签`，元素间用空格分隔
  - `div > p : 子代选择器，匹配div元素下第一代p元素，用于选定指定标签元素的第一代元素`
  - `div + p ：毗邻元素选择器，匹配所有紧随div元素后面的同级p元素`

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

### position
#### position: sticky！
- 粘性定位可以被认为相对定位和固定定位的混合，元素在跨越特定阈值前为相对定位，之后为固定定位；
- `可以实现页面滚动一段距离后固定导航栏 #one {position: sticky; top: 0} //结果页导航条fixed`

### flex布局
- `flex-grow: 扩展的是flex子项所占据的宽度，`扩展所侵占的空间就是除去元素外的剩余的空白间隙；`空间足够时如何利用空间`
- `flex-shrink: 主要处理当flex容器空间不足时候，单个元素的收缩比例；空间不足时收缩的尺寸`
- `flex-basic: 定义了在分配剩余空间之前元素的默认大小。`
- `flex: none(0 0 auto) | auto(0 1 auto) | [<'flex-grow'> <'flex-shrink'>? || <'flex-basic'>] 默认值 0 1 auto`
- align-self: 指控制单独某一个flex子项的垂直对齐方式
- 在flex布局中flex子元素设置的float、clear以及vertical-align属性都没用
- flex布局适合应用程序的组件和小规模布局
- `flex兼容性：IE10+、Edge、Firefox 2+、Chrome 4+、Safari 3.1+`
- 如果需要ie低版本兼容flex时
```css
div{
  display: flex;
  div {
    float: left;
  }
} 
```

### [flex:0 flex:1 flex:none flex:auto 区别及应用场景](https://www.zhangxinxu.com/wordpress/2020/10/css-flex-0-1-none/)
```js
flex: initial // 0 1 auto; 有剩余空间时不增长，空间不足时自动收缩，尺寸自适应内容
flex: 0 // 0 1 0%; 应用了flex:0的元素全部高高耸起，一柱擎天，表现为最小内容宽度
flex: none // 0 0 auto; 应用了flex:none的元素则无视容器的尺寸限制，直接溢出容器，没有换行，表现为最大内容宽度。应用为左边为文字，右边为按钮，为按钮设置none，则按钮不会被左边的文字挤得换行
flex: 1 // 1 1 0%; 等比例分配宽度，当内容多的时候尺寸表现更为内敛（优先牺牲自己的尺寸）
flex: auto // 1 1 auto; 当内容多的时候尺寸表现则更为霸道（优先扩展自己的尺寸）。
```

### 层叠上下文z-index
- 层叠上下文是HTML中一个三维概念，一些元素发生堆叠，发现某个元素可能覆盖了另一个元素或者被覆盖时，就产生了层叠上下文
- 层叠水平：决定了同一个层叠上下文中元素在z轴上的显示顺序
- 普通元素的层叠水平优先由其所在的层叠上下文决定，层叠水平的比较只有在当前层叠上下文中才有意义
- 层叠等级的比较只有在当前层叠上下文中比较才有意义。不同层叠上下文中比较层叠等级没有意义
- 层叠顺序：*装饰*层叠上下文bg/border（层叠上下文背景和边框） <  负z-index  <  *布局*block块级水平盒子  <  *布局*float浮动盒子  <  *内容*inline/inline-block 水平盒子  <  z-index:auto/z-index:0  <  正z-index
- 层叠准则：
  - 谁大谁上：当具有明显的层叠水平标示的时候，如识别的z-index值，在同一个层叠上下文领域，层叠水平值大的那一个覆盖小的那一个。通俗讲就是官大的压死官小的
  - 后来居上：当元素的层叠水平一致、层叠顺序相同的时候，在DOM流中处于后面的元素会覆盖前面的元素
- z-index: 0 会创建一个层级上下文， z-index: auto所在的div元素是一个普通元素
- [参考链接](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)

### 回流重绘
- `浏览器使用流式布局模型`
- `浏览器会把HTML解析成DOM，把CSS解析成CSSOM，DOM和CSSOM合并成了renderTree`
- 回流重绘：触发回流一定会引起重绘，重绘不一定会引起回流；回流比重绘的代价更高
- 回流：确定对象位置，浏览器根据各种样式计算出结果，将对象放在他应该出现的位置
- 重绘：确定位置后绘制出该对象
- 导致回流因素：
  - `调整窗口大小`
  - `改变字体`
  - `增加或者移除类`
  - `改变style`
  - `操作class`
  - `操作dom`
  - `计算offsetWidth 和 offsetHeight`
  - `内容变化，如input框输入文字`
- 优化：
  - `设置元素样式时用类名设置`
  - `避免设置多项内联样式`
  - `避免使用table布局`
  - `应用元素动画，使用position时使用fixed 和 absolute(这两个属性只影响自身的回流，不影响其他元素)`

- 注：
 - 回流(reflow), 就是布局引擎为 frame 计算图形, 确定节点位置的一个动作。其中触发回流的原因主要是 DOM 节点大小或位置的改变才会触发回流。
 - 重绘则是表面的视觉效果的改变从而引发重绘。

### display: none 和 visibility: hidden 区别
- display不会存在渲染树中，visibility会
- display会触发回流，进行渲染；visibility会进行重绘
- display不是可继承属性；visibility是可继承的
- display读屏器不会读取元素内容；visibility会读取

### 盒模型
- `标准盒模型：content、padding、border、margin`
- `IE怪异盒模型：content（包含了padding和border）、margin` 
```css
box-sizing: border-box; //怪异模型
box-sizing: content-box; //标准模型 
```
### 水平垂直居中!

### 布局的几种方式
- 静态布局
  - 页上的所有元素的尺寸一律使用px作为单位
- 流式布局
  - 流式布局的特点 是页面元素的宽度按照屏幕分辨率进行适配调整，但整体布局不变。代表作栅栏系统 网格系统
  - 屏幕分辨率变化时，页面里元素的大小会变化而但布局不变
- 响应式布局
  - 每个屏幕分辨率下面会有一个布局样式，即元素位置和大小都会变
  - 媒体查询+流式布局
- 自适应布局
  - 自适应布局的特点是分别为不同的屏幕分辨率定义布局，即创建多个静态布局，每个静态布局对应一个屏幕分辨率范围。
  - 使用 @media 媒体查询给不同尺寸和介质的设备切换不同的样式。
  - 屏幕分辨率变化时，页面里面元素的位置会变化而大小不会变化。
- 弹性布局（rem/em布局）
  - CSS编写者常常将页面跟节点字体设为62.5%，比如选择用rem控制字体时
  - 先需要设置根节点html的字体大小，因为浏览器默认字体大小16px*62.5%=10px。这样1rem便是10px，方便了计算。

### H5 移动端 （待补充）
- h5适配：淘宝lib-flexible
  - flexible实际上就是能过JS来动态改写meta标签
  - 动态改写<meta>标签
  - 给<html>元素添加data-dpr属性，并且动态改写data-dpr的值
  - 给<html>元素添加font-size属性，并且动态改写font-size的值
- vw来做移动端的适配
- 使用 postcss-px-to-viewport，自动将px转成视口单位vw
#### CSS单位px,rem,em,vw,vh的区别
- `px：像素缩写，相对长度单位，相对于显示屏分辨率而言`
- `em：相对长度单位，相对于当前对象内文本字体尺寸，参考父元素的font-size；若父元素没有设置字体大小则相对于浏览器的默认字体尺寸`
- `rem: CSS3新增的属性，相对于根元素的字体大小来计算长度的单位；若没有设置根元素的字体大小则使用浏览器默认的字体大小16px`
- `vw：相对于视口的宽度而定的，长度等于视口宽度的1/100，若视口宽度是200px,则1vw为2px`
- vh: 相对于视口高度而定的，高度等于视口高度的1/100；假如浏览器的高度为500px，那么1vh就等于5px
- `rem和em的区别：rem是相对于根元素的字体大小来计算；em是相对于父元素的字体大小来计算`

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

### h5新特性
- `新增语义化标签（header,footer,nav,section）`
- `新增video audio标签`
- `增加localStorage、sessionStorage`
- `增加svg画图、canvas画图`

### css3新特性!
- `CSS3实现圆角（border-radius），阴影（box-shadow）`
- `对文字加特效（text-shadow、），线性渐变（gradient），旋转（transform）`
- `border-image`

### css布局实现
- 文档流布局display：按照文档的顺序一个个显示出来，块元素独占一行，行内元素共享一行
- 浮动布局float: 使元素脱离文档流，浮动起来
- 定位布局position
- flex布局
- 三列布局：圣杯布局和双飞翼布局解决问题的方案在前一半是相同的，也就是三栏全部float浮动，但左右两栏加上负margin让其跟中间栏div并排，以形成三栏布局；不同在于解决”中间栏div内容不被遮挡“问题的思路不一样

```css
// 圣杯布局
// 为了中间div内容不被遮挡，将中间div设置了左右padding-left和padding-right后，将左右两个div用相对布局position: relative并分别配合right和left属性，以便左右两栏div移动后不遮挡中间div

<div class="container">
    <div class="middle">测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试</div>
    <div class="left">left</div>
    <div class="right">right</div>
</div>

// css
.container{
  padding: 0 200px;
}
.middle{
  width: 100%;
  background: paleturquoise;
  height: 200px;
  float: left;
}
.left{
  background: palevioletred;
  width: 200px;
  height: 200px;
  float: left;
  font-size: 40px;
  color: #fff;
  margin-left:-100%;
  position: relative; 
  left: -200px;

}
.right{
  width: 200px;
  height: 200px;
  background: purple;
  font-size: 40px;
  float: left;
  color: #fff;
  margin-left:-200px;
  position: relative; 
  right: -200px;
}
```

```css
// 双飞翼
// 中间栏需要放在前面， 双飞翼布局，为了中间div内容不被遮挡，直接在中间div内部创建子div用于放置内容，在该子div里用margin-left和margin-right为左右两栏div留出位置

<div class="container">
    <div class="middle-container">
        <div class="middle">测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试测试</div>
    </div>
    <div class="left">left</div>
    <div class="right">right</div>
</div>


.middle-container{
  width: 100%;
  background: paleturquoise;
  height: 200px;
  float: left;

}

.middle{
  margin-left: 200px;
  margin-right: 200px;
}
.left{
  background: palevioletred;
  width: 200px;
  height: 200px;
  float: left;
  font-size: 40px;
  color: #fff;
  margin-left:-100%;


}
.right{
  width: 200px;
  height: 200px;
  background: purple;
  font-size: 40px;
  float: left;
  color: #fff;
  margin-left:-200px;

}
```