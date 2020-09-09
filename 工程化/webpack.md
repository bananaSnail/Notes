
- 概念：分析项目的结构，找到js模块以及其他的一些浏览器不能直接运行的拓展语言(scss,ts)，并将其打包成合适的格式以供浏览器使用
- 优点：在webpack看来一切都是模块。包括你的JavaScript代码，也包括CSS和fonts以及图片等等，只有通过合适的loaders，它们都可以被当做模块被处理。

### entry：入口文件指是webpack应用哪个模块作为依赖图的开始，默认值是./src/index.js
### output：告诉webpack在哪里输出它所创建的bundle，以及如何命名这些文件。默认值是 ./dist/main.js

### loader
- 作用：处理任意类型文件，并且将它们转换成一个让webpack可以处理的有效模块。
- 加载单文件组件格式撰写的Vue组件 --> vue-loader 
- 加载css
  - 遍历匹配文件中的@import 和 url() --> css-loader
  - 通过 style 元素注入样式、实现了`热模块替换接口` --> style-loader
- webpack只能理解js和json，loader让webpack能去处理其他类型文件，并将他们转换成有效模块
- 各个loader配置在 module.rules ; 使用test正则匹配文件，不要添加引号，/\.txt$/ 跟 '/\.txt$/'不一样,前者是指任何以.txt结尾的文件，后者 匹配具有绝对路径 '.txt' 的单个文件
- 一个use包含了多个loader时，处理顺序应该是最后一个到第一个；
```js
{
    // 匹配.css文件
    test: /\.scss$/,
    use: ['style-loader', 'scss-loader']
},
/* SCSS 源代码会先交给 sass-loader 把 SCSS 转换成 CSS；
把 sass-loader 输出的 CSS 交给 css-loader 处理，找出 CSS 中依赖的资源、压缩 CSS 等；
把 css-loader 输出的 CSS 交给 style-loader 处理，转换成通过脚本加载的 JavaScript 代码；
*/

{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: 'url-loader',
    options: {
      limit: 10000,
      name: utils.assetsPath('img/[name].[hash:7].[ext]')
    }
},
/*
url-loader会接收一个limit参数，单位字节byte
当文件体积小于limit时，url-loader会把文件转为Data URL 的格式内联到引用的地方；
当文件体积大于limit时，url-loader会调用file-loader，把文件存储到输出目录，并把引用的文件路径写成输出后的路径
*/
```
- file-loader：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件 (处理图片和字体)
- url-loader：与 file-loader 类似，区别是用户可以设置一个阈值，大于阈值会交给 file-loader 处理，小于阈值时返回文件 base64 形式编码 (处理图片和字体)
- babel-loader：把 ES6 转换成 ES5
- ts-loader: 将 TypeScript 转换成 JavaScript
- sass-loader：将SCSS/SASS代码转换成CSS
- css-loader：加载 CSS，支持模块化、压缩、文件导入等特性
- style-loader：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS
- vue-loader：加载 Vue.js 单文件组件

### Loader和Plugin的区别
- Loader 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。
- Plugin 就是插件，基于事件流框架 Tapable，插件可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。



### plugin
- 热更新插件:webpack-hot-middleware

### Webpack构建流程
- 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数
- 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译
- 确定入口：根据配置中的 entry 找出所有的入口文件
- 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
- 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
- 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
- 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统
- 总结：
  - `初始化：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler`
  - `编译：从 Entry 出发，针对每个 Module 串行调用对应的 Loader 去翻译文件的内容，再找到该 Module 依赖的 Module，递归地进行编译处理`
  - `输出：将编译后的 Module 组合成 Chunk，将 Chunk 转换成文件，输出到文件系统中`

### Webpack 的热更新原理
- `Webpack 的热更新又称热替换（Hot Module Replacement），缩写为 HMR。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。`
- HMR的核心就是`客户端从服务端拉去更新后的文件`，准确的说是 chunk diff (chunk 需要更新的部分)，实际上 `WDS(Webpack-dev-server) 与浏览器之间维护了一个 Websocket，当本地资源发生变化时，WDS 会向浏览器推送更新，并带上构建时的 hash，让客户端与上一次资源进行对比。客户端对比出差异后会向 WDS 发起 Ajax 请求来获取更改内容(文件列表、hash)，这样客户端就可以再借助这些信息继续向 WDS 发起 jsonp 请求获取该chunk的增量更新。`
- 后续的部分(拿到增量更新之后如何处理？哪些状态该保留？哪些又需要更新？)由 HotModulePlugin 来完成，提供了相关 API 以供开发者针对自身场景进行处理，像react-hot-loader 和 vue-loader 都是借助这些 API 实现 HMR。



### webpack性能优化
- resolve字段的配置
  - 避免重重查找；
  - 设置尽量少的值可以减少入口文件的搜索步骤；
  - 庞大的第三方模块设置resolve.alias, 使webpack直接使用库的min文件，避免库内解析
- 配置loader时，通过test、exclude、include缩小搜索范围
- 使用HappyPack开启多进程Loader转换
- 开启热更新：原理也是向每一个chunk中注入代理客户端来连接DevServer和网页
- 使用Tree Shaking剔除JS死代码


### [Tree Shaking](https://webpack.docschina.org/guides/tree-shaking/) 
- 概念：Tree shaking 是一种通过清除多余代码方式来优化项目打包体积的技术
- 原理：`tree shaking 只能在静态modules下工作，es6模块加载是静态的，因此整个依赖树可以被静态地推导出解析语法树。所以在es6中使用tree shaking是非常容易的。`
- 背景：在es6以前使用CommonJS引入模块：require()，这种引入是动态的，意味着tree shaking 不适用。因为不确定哪些模块实际运行之前是需要的，或者是不需要的。在es6中，进入了完全静态的导入语法：import.

### side effects
- 概念：`side effects是指哪些当import的时候会执行一些动作，但是不一定会有任何export。`
- tree shaking不能自动识别哪些代码属于side effects，因此手动指定这些代码显得十分重要。
- 总结：
  - tree shaking 不支持动态导入(CommonJS的require()语法)，只支持纯静态的导入(es6的import())。
  - webpack中可以在项目的package.json文件中添加一个'sideEffects'属性，手动指定由副作用的脚本

### CommonJS
- 在 Commonjs 中，一个文件就是一个模块。定义一个模块导出通过 exports 或者 module.exports 挂载即可。
- 导入一个模块也很简单，通过 require 对应模块拿到 exports 对象。



### webpack工作流程
- 从一个脚本命令说起 Webpack -config ./webpack.config.js ![start](assets/start.png)
- webpack-cli
  - 调用webpack函数
  - 整合shell和config配置，生成options对象
  - 根据options配置实例化compiler
  - 给webpack内置插件以及自定义插件挂载compiler的一些hook，例如 NodeJsInputFileSystem、NodeOutputFileSystem、EntryOptionPlugin
![process](assets/process.png)
- compiler对象（负责编译流程）
  - beforeRun  主要处理缓存的模块，加速编译速度
  - Run  正式开启整个编译流程
  - beforeCompile 编译前触发的钩子，在这里实例化模块工厂
  - compile 开始编译，生成compilation对象
  - compilation  实例化compilation后触发的钩子
  - Make  compilation.addEntry()根据入口文件类型确定模块工厂类型，转入compilation主战场
  - afterCompile 编译完成之后触发的钩子
  - shouldEmit 表示编译是否成功的钩子返回boolean
  - emit  编译成功，生成资源到output目录之后触发
  - done compilation成功后触发的钩子
  - failed compilation失败后触发的钩子
![compilation](assets/compilation.png)

### Babel原理
- 解析：将代码转换成 AST
  - 词法分析：将代码(字符串)分割为token流，即语法单元成的数组
  - 语法分析：分析token流(上面生成的数组)并生成 AST
- 转换：访问 AST 的节点进行变换操作生产新的 AST
  - Taro就是利用 babel 完成的小程序语法转换
- 生成：以新的 AST 为基础生成代码
