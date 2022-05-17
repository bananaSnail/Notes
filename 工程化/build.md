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
### [vite原理](https://jishuin.proginn.com/p/763bfbd29d7f)
- vite 利用 ES module，把 “构建 vue 应用” 这个本来需要通过 webpack 打包后才能执行的代码直接放在浏览器里执行，这么做是为了：
    - 去掉打包步骤
    - 实现按需加载
- connect允许接入一些插件作为中间件，vite 通过对请求路径的劫持获取资源的内容返回给浏览器，不过 vite 对于模块导入做了特殊处理。
- 为什么需要 @modules？
    - 浏览器中的 esm 是不可能获取到导入的模块内容，浏览器中 ESM 无法直接访问项目下的 node_modules，所以 vite 对所有 import 都做了处理，用带有 @modules 的前缀重写它们
    - 加入了 optimize 命令，这个命令专门为解决模块引用的坑而开发，例如我们要在 vite 中使用 lodash，只需要在 vite.config.js （vite 配置文件）中，配置 optimizeDeps 对象，在 include 数组中添加 lodash
    - 这样 vite 在执行 runOptimize 的时候中会使用 roolup 对 lodash 包重新编译，将编译成符合 esm 模块规范的新的包放入 node_modules 下的 .vite_opt_cache 中，然后配合 resolver 对 lodash 的导入进行处理：使用编译后的包内容代替原来 lodash 的包的内容，这样就解决了 vite 中不能使用 cjs 包的问题，这部分代码在 depOptimizer.ts 里。
- css
    - 安装对应的 css 预处理器即可。在 cssPlugin 中，通过正则：/(.+).(less|sass|scss|styl|stylus)$/ 判断路径是否需要 css 预编译，如果命中正则，就借助 cssUtils 里的方法借助 postcss 对要导入的 css 文件编译。
    
### esbuild加速脚手架编译
- 速度对比：
    - tsc 6s 
    - esbuild build 2s 
    - esbuild transform 1s 
- esbuild比tsc快的原因是esbuild只支持转化成es6
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

### node cluster、child_process、worker_threads区别
- cluster它可以通过一个父进程管理一堆子进程的方式来实现集群的功能。cluster 底层就是 child_process，master 进程做总控，启动 1 个 agent 和 n 个 worker，agent 来做任务调度，获取任务，并分配给某个空闲的 worker 来做。这样的方式仅仅实现了多进程。
- child_process通过它可以开启多个子进程，在多个子进程之间可以共享内存空间，可以通过子进程的互相通信来实现信息的交换。
- worker_threads 比使用 child_process 或 cluster可以获得的并行性更轻量级。 此外，worker_threads 可以有效地共享内存。开启多线程

### tsc比esbuild慢原因
    - 因为tsc可以编译成es5，es6。但是esbuild只能编译成es6。
### node  一个文件就是一个单例模块，因此不需要额外写代码去维护单例模式
### esbuild加速wdt编译代码
```js

```
### [babel的polyfill和runtime的区别](https://segmentfault.com/q/1010000005596587?from=singlemessage&isappinstalled=1)
- 一些 ES6 的新语法，如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign、Array.from）都需要用 babel-polyfill 进行转译，为当前 [应用开发] 环境铺好路，而不是库或是工具。
- **Runtime transform 更适用于 库/脚本 开发，它提供编译模块复用工具函数，将不同文件中用到的辅助函数如 _extend 统一打包在一块，避免重复出现导致的体积变大。另外一个目的是，在 库/脚本工具开发中不需要直接引用 polyfill，因为会造成实际应用开发中，一些全局变量名污染，比如 Promise、Map 等。Runtime 会自动应用 Polyfill，而不需要再单独引用。**