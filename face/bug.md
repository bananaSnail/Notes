- 倒计时：切换路由时倒计时没有清零
- 样式问题
- 构建问题
- md问题：
  - 转义字符的处理：escape-html 如单双引号 大于小于号
  - 换行问题：录入时使用Excel录入数据库，空格变成了\n，marked转不了改字符，替换成<br>标签
  - 代码块里空格被压缩：渲染时使用pre标签替代div
value.replace(/\n/g, '<br>').replace(/\s+/g, '&nbsp;')
- 图片响应式
- div垂直margin被吃问题
- 雷达图：
  - 自己画，对称问题，通过旋转调整对称，不同边形的雷达图标题的位置不同，找共同点，抽象
  - 用ant g2插件 自定义的tooltip  他不跟随鼠标移动  inPlot: true, // 将 tooltip 展示在指定区域内  置为false就行了


- html2canvas截图跨域问题【转载】  添加useCORS:true属性
- 画圆画不圆   放大再用transform缩小