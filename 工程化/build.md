### 脚手架开启多服务器多项目多环境同时编译部署
- 

### 脚手架本地开发启用vite，加速编译
- 目标
    - 提高编译速度
    - 提高热更新速度
- 方案
    - 开发环境下集成 vite 或 snowpack
- 注意点
    - 新工具可能出现兼容性问题，不兼容require导入，可能要改成import
- 方案对比
    - snowpack
        - 开发模式启动仅需 50ms 甚至更少。
        - 热更新速度非常快。
        - 构建时可以结合任何 bundler，比如 webpack。
        - 内置支持 TS、JSX、CSS Modules 等。
        - 支持自定义构建脚本以及三方插件
    - vite
        - 快速的冷启动；
        - 即时的模块热更新；
        - 真正的按需编译。
    - 相比之下vite列出相比snowpack的优势是：
        - 1.ESM HMR热更新的支持更早；
        - 2.在需要bundle打包的时候，vite使用rollup，snowpack是以Parcel/webpack插件，vite更快更容易；
        - 3.Vue支持程度更好。
- 实际碰到的问题
    - vite不支持cjs，只支持mjs
    ```js
        Uncaught SyntaxError: The requested module '/@fs/Users/project/front-end/packages/react-material-ui/built/Button/index.js' does not provide an export named 'Button'
    ```
    - .mjs指的就是esmodule文件，ES6 模块采用.mjs后缀文件名。也就是说，只要脚本文件里面使用import或者export命令，那么就必须采用.mjs后缀名。Node.js 遇到.mjs文件，就认为它是 ES6 模块，默认启用严格模式，不必在每个模块文件顶部指定"use strict"。使用"use strict"的是commonjs文件
    - 图片导入require问题
        - vite 不支持业务代码里面的require，但是支持第三方依赖包的require
        - vite不支持commonjs语法，只支持esm的形式
        - 因此vite-plugin-commonjs里转换require为import时需要区分是否为业务代码
        - 业务内图片的导入用的是require形式，所以必须要将require转换为import格式
    
### esbuild加速脚手架编译
- 速度对比：
    - tsc 6s 
    - esbuild build 2s 
    - esbuild transform 1s 
- 拓展
    - babel/core.js 集成了主流一些Map方法的polyfill兼容低版本浏览器的代码
    - ??= [proposal-logical-assignment](https://github.com/tc39/proposal-logical-assignment)
- 问题
    - build接口默认会去编译node_modules，解决方案：build exclude node_modules esbuild-node-externals
- transform vs build
    - transform参数是 代码字符串，不基于文件系统；
    - build 参数是可以传入一个 app.js的文件入口，然后esbuild会对app.js进行依赖分析，去编译别的文件。
    - build接口会比transform接口支持的功能多，耗时久一些

### webpack、babel、rollup适用场景
- webpack适用于打包编译整个项目
- babel和rollup适用于打包编译包
- rollup之前用于打包图片路径为base64格式、如athena-mobile-view项目用的next.js，采用服务端渲染，无法识别图片路径，因此需要rollup打包成base64的格式的图片。
- 另外base64打包转换的图片一般会比原图片大10%~20%的体积；用base64打包的图片页面会跟图片一起渲染出来，而不用base64机制的图片页面会先渲染，然后图片由0到1有个变化的过程；一般后台管理系统的图片不用base64；一般来说小图标会用base64，大图片的话不会考虑用base64
- webpack打包也是通过调用webpack api打包
- babel通过调用babel.transformFileAsync api打包，打包出来是esm格式，若需要cjs格式需要通过esbuild.transform接口进行转化，将打包好的内容写入文件系统中output
[rollup.js](https://zhuanlan.zhihu.com/p/75717476)

### esbuild加速wdt编译代码
```js

```
