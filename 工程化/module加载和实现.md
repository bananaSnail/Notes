https://es6.ruanyifeng.com/#docs/module-loader
### 浏览器加载
- 默认情况下，浏览器是同步加载 JavaScript 脚本，即渲染引擎遇到<script>标签就会停下来，等到执行完脚本，再继续向下渲染。如果是外部脚本，还必须加入脚本下载的时间。
```js
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```
- <script>标签打开defer或async属性，脚本就会异步加载。渲染引擎遇到这一行命令，就会开始下载外部脚本，但不会等它下载和执行，而是直接执行后面的命令。
- defer要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行
- async一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染
- 浏览器加载 ES6 模块，也使用<script>标签，但是要加入type="module"属性；浏览器对于带有type="module"的<script>，都是异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了<script>标签的defer属性。

### ES6 模块与 CommonJS 模块的差异
- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
- CommonJS 模块的require()是同步加载模块，ES6 模块的import命令是异步加载，有一个独立的模块依赖的解析阶段。
- 第二个差异是因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成
- CommonJS 的require()命令不能加载 ES6 模块，会报错，只能使用import()这个方法加载。
```js
(async () => {
  await import('./my-app.mjs');
})();
```
- CommonJS 模块加载 ES6 模块：
    - require()不支持 ES6 模块的一个原因是，它是同步加载，而 ES6 模块内部可以使用顶层await命令，导致无法被同步加载
- ES6 模块加载 CommonJS 模块：
    - ES6 模块的import命令可以加载 CommonJS 模块，但是只能整体加载，不能只加载单一的输出项。因为 ES6 模块需要支持静态代码分析，而 CommonJS 模块的输出接口是module.exports，是一个对象，无法被静态分析，所以只能整体加载。
- Node.js 的内置模块：
    - Node.js 的内置模块可以整体加载，也可以加载指定的输出项。
- 内部变量：Node.js 规定 ES6 模块之中不能使用 CommonJS 模块的特有的一些内部变量。
    - this关键字。ES6 模块之中，顶层的this指向undefined；CommonJS 模块的顶层this指向当前模块，这是两者的一个重大差异。
    - 以下这些顶层变量在 ES6 模块之中都是不存在的。arguments、require、module、exports、__filename、__dirname
### 循环加载：“循环加载”（circular dependency）指的是，a脚本的执行依赖b脚本，而b脚本的执行又依赖a脚本
- CommonJS 模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。CommonJS 输入的是被输出值的拷贝，不是引用
- ES6 处理“循环加载”与 CommonJS 有本质的不同。ES6 模块是动态引用，如果使用import从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。